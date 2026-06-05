

# Reporte Técnico: Simulación de Entorno 3D Interactiva con Control por Gestos
Este documento describe la estructura, funcionamiento y componentes del código de simulación en 3D que integra técnicas de renderizado clásico en **OpenGL** con un sistema de control asíncrono por visión artificial mediante **MediaPipe**.


## 1. ¿Qué hace el código?

El script genera un entorno virtual interactivo tridimensional (un pueblo tradicional) que incluye elementos arquitectónicos (una iglesia, una escuela, tiendas estilo *OXXO*, quioscos, casas, áreas verdes) y entidades dinámicas con animaciones procedimentales (personas y burros en movimiento continuo).

El usuario puede explorar y navegar libremente por este mundo a través de una **cámara en primera persona**. La navegación es **híbrida**, permitiendo el control simultáneo mediante dos métodos:

1. **Teclado clásico:** Controles convencionales (W, A, S, D, Q, Z).
2. **Visión artificial (Gestos sin contacto):** Procesamiento en tiempo real de la señal de la webcam para traducir la posición de la mano en comandos de vuelo/navegación.

---

## 2. Componentes Utilizados (¿Qué se utilizó?)

El proyecto se apoya en un conjunto de librerías especializadas para optimizar el rendimiento de la CPU/GPU y gestionar hilos de ejecución paralelos:

* **`os` y `sys`:** Utilizados para interactuar con el sistema operativo. En este script específico, modifican las variables de entorno de bajo nivel (`TF_CPP_MIN_LOG_LEVEL` y `GLOG_minloglevel`) para silenciar alertas y mensajes de diagnóstico irrelevantes de las plataformas de Machine Learning (TensorFlow/Google Log), manteniendo la consola limpia.
* **`math` y `random`:** Manejan los cálculos trigonométricos necesarios para las rotaciones de la cámara, transformaciones espaciales y la generación de valores pseudoaleatorios (con semilla fija `42` y `77`) para distribuir flores y variaciones cromáticas de los adoquines.
* **`time` y `threading`:** Herramientas fundamentales para calcular el tiempo delta de las animaciones y construir una arquitectura multi-hilo (*multithreading*), evitando que el pesado procesamiento de imágenes ralentice el bucle de renderizado 3D.
* **`glfw`:** Biblioteca para la gestión de ventanas nativas de escritorio, configuraciones del contexto gráfico y la captura directa de periféricos de entrada (eventos de teclado).
* **`cv2` (OpenCV):** Se encarga de la comunicación directa con la cámara web, la captura de los fotogramas, su inversión horizontal (efecto espejo) y la superposición de interfaces gráficas analógicas (HUD de texto en el stream de video).
* **`mediapipe`:** Framework de aprendizaje automático de Google. Utiliza el módulo especializado `HandLandmarker` mediante la carga de un archivo de tareas optimizado (`hand_landmarker.task`) para realizar el rastreo tridimensional de los 21 puntos clave de la mano de forma asíncrona.
* **`OpenGL.GL` y `OpenGL.GLU`:** Módulos de enlace (*bindings*) de Python para la API de OpenGL. Se encargan de la comunicación con la tarjeta gráfica para configurar perspectivas, habilitar búferes de profundidad, mezclas alfa de transparencia y dibujar primitivas geométricas espaciales.

---

## 3. ¿Cómo lo hace? (Arquitectura y Funcionamiento)

### A. Paradigma Multi-Hilo y Sincronización

Para mantener una tasa de refresco fluida en la escena 3D, el programa segmenta la carga de trabajo en dos hilos de ejecución concurrentes que se comunican mediante variables globales protegidas por bloqueos de exclusión mutua (`threading.Lock`):

```
 ┌──────────────────────────────────────┐      ┌──────────────────────────────────────┐
 │     HILO PRINCIPAL (Main Thread)     │      │      HILO DE VISIÓN (Vision Thread)  │
 │  - Renderizado OpenGL a ~60 FPS.     │      │  - Captura OpenCV a 30 FPS.          │
 │  - Escucha de Teclado.               │      │  - Inferencia Asíncrona MediaPipe.  │
 │  - Lectura de comandos de cámara.    │      │  - Escritura de Gestos Detectados.   │
 └──────────────────┬───────────────────┘      └──────────────────┬───────────────────┘
                    │                                             │
                    └───────────────> [gesture_lock] <────────────┘
                               (Sincronización Segura)

```

1. **Hilo de Visión (`vision_thread`):** Captura fotogramas a resolución reducida ($320 \times 240$ píxeles a 30 FPS) para minimizar la carga computacional. Utiliza el modo `LIVE_STREAM` de MediaPipe para procesar **un fotograma de cada dos** de manera asíncrona. Cuando detecta coordenadas válidas, invoca una función callback (`on_hand_result`) que resguarda de manera segura el comando actual.
2. **Hilo Principal (Bucle de Gráficos):** Ejecuta la inicialización de GLFW y el ciclo infinito de renderizado. En cada iteración, consulta de forma segura el estado del gesto, procesa las entradas del teclado, actualiza matrices de transformación y redibuja la escena.

### B. Sistema de Clasificación de Gestos

La función `classify_gesture` procesa los puntos clave (*landmarks*) normalizados de la mano. Determina el estado de extensión de los dedos mediante la relación geométrica entre las puntas (*tips*) y las articulaciones intermedias (*pip*).

* **Lógica de Dedos Extendidos:** Si la coordenada $Y$ del *tip* es menor (más arriba en la pantalla) que el *pip*, el dedo se considera extendido.
* **Matriz de Comandos:**
* `FORWARD`: Índice, medio, anular y meñique activos simultáneamente (Palma).
* `BACK`: Todos los dedos cerrados (Puño).
* `TURN_R`: Únicamente el dedo índice extendido.
* `TURN_L`: Dedos índice y medio extendidos en simultáneo (Signo de la paz).
* `UP`: Pulgar extendido verticalmente mientras el resto de los dedos permanecen cerrados.
* `DOWN`: Únicamente el dedo meñique extendido.



### C. Navegación en Espacio Tridimensional

La cámara se modela mediante coordenadas esféricas polares utilizando los ángulos `cam_yaw` (rotación horizontal) y `cam_pitch` (rotación vertical).

Para avanzar en la dirección exacta hacia la que mira el usuario, el script calcula el vector de dirección frontal ($fx, fz$) aplicando trigonometría elemental sobre el ángulo de rotación actual:

$$fx = -\sin(\text{cam\_yaw})$$

$$fz = -\cos(\text{cam\_yaw})$$

Cada comando modifica de manera incremental las variables globales de posición (`cam_x`, `cam_y`, `cam_z`), las cuales se inyectan en la matriz de vista del pipeline gráfico usando la función matemática de utilidad `gluLookAt`.

### D. Renderizado Geométrico Procedimental y Texturizado HUD

La escena no depende de modelos externos pesados; en su lugar, se construye mediante geometría procedimental algorítmica combinando formas primitivas básicas (`GL_QUADS`, `GL_TRIANGLES`, cilindros y esferas de GLU):

* **Estructuras Complejas:** Funciones específicas como `draw_oxxo`, `draw_house` u `draw_iglesia` apilan cajas sólidas de diferentes colores, simulan letras corporativas vectorizando polígonos, y añaden techos a dos aguas calculando coordenadas triangulares en el espacio.
* **Inyección de Video (HUD):** La función `draw_hud_camera` genera un identificador de textura en OpenGL (`hud_tex_id`). Al final de cada ciclo, copia de manera segura la matriz de pixeles de la cámara web, la convierte al espacio de color RGB, invierte su eje vertical y la proyecta en la esquina de la pantalla mediante una matriz de proyección ortogonal bidimensional (`glOrtho`), permitiendo al usuario ver su propia mano reflejada en la simulación sin ventanas secundarias.
#Aquí tienes el desglose técnico en formato Markdown que explica la magia detrás de la construcción geométrica y matemática de cada elemento de tu pueblo.

En este script, todo se logró mediante **Modelado Procedimental Basado en Primitivas** y **Animación Cíclica Trigonométrica**. No se usaron archivos `.obj` ni texturas externas; todo está dibujado punto por punto con funciones de OpenGL.

---

#  Arquitectura del codigo

## 1. El Suelo y los Caminos (El Piso)

El piso combina extensiones masivas de terreno con un sistema de cuadrículas para simular adoquines individuales sin saturar la memoria gráfica.

* **El Terreno Base:** Se creó un plano gigante grisáceo (`glColor3f(.38,.36,.33)`) usando un único polígono de cuatro vértices (`GL_QUADS`) que va desde las coordenadas $-100$ hasta $100$ en los ejes $X$ y $Z$.
* **El Jardín Central:** Sobre el plano base, se encimó otro cuadrilátero verde texturizado algorítmicamente mediante la función `draw_jardin`. Para las flores, se utilizó un generador numérico pseudoaleatorio (`random.Random(77)`) que dispersa micro-esferas (`sph(.09)`) de colores aleatorios sobre el pasto.
* **Los Adoquines (`draw_adoquin`):** Para simular una calle realista, se calcula la longitud y el ángulo del camino usando trigonometría (`math.atan2`). Luego, mediante un ciclo anidado (*for loops*), se divide el espacio en filas y columnas de $0.55 \times 0.55$ unidades. Cada bloque es un cubo estirado (`box`) cuyo color gris varía sutilmente de forma aleatoria (`.44 + rng2.uniform(-.04, .04)`), logrando ese aspecto natural de piedra rústica.

---

## 2. La Iglesia (`draw_iglesia`)

La iglesia es el edificio más alto y complejo. Se construyó apilando cajas y calculando techos inclinados manualmente.

* **Nave Central:** Es una caja masiva (`box`) de proporciones $9.0 \times 5.5 \times 12.0$ pintada de color beige claro.
* **Techo a Dos Aguas:** Para hacer el techo triangular, se utilizaron dos primitivas `GL_TRIANGLES` para las fachadas frontal/trasera, y `GL_QUADS` inclinados para las caídas de agua.
* **Torres Laterales:** Se posicionaron dos cubos verticales esbeltos en las coordenadas de los extremos (`tx = -3.5` y `tx = 3.5`) que suben hasta una altura de $8.5$ unidades. En la punta de cada torre se colocó una cúpula usando esferas de GLU (`sph(1.05)`).
* **Cruces:** Se dibujaron de forma vectorial cruzando dos cajas delgadas (`box`) doradas en el eje $X$ y $Y$ arriba de las cúpulas.
* **Detalles Interiores:** El gran rosetón (vitral circular frontal) se logró orientando un cilindro de GLU muy delgado en el eje Z (`glRotatef(90, 0, 1, 0)`) con un radio de $0.65$. Las escaleras de la entrada son una sucesión de 4 cajas que se van ensanchando y aplanando hacia abajo.

---

## 3. La Escuela y las Banderas (`draw_escuela`)

La escuela destaca por su distribución simétrica y el uso de elementos patrios escalados.

* **Estructura:** Es un bloque horizontal beige de $16$ unidades de ancho. Cuenta con un sótano perimetral verde y una cornisa superior que sobresale para darle profundidad a la fachada.
* **Pórtico con Columnas:** En la entrada central, se usaron cilindros de GLU (`cyl(.22, 4.5)`) rotados $-90^\circ$ en el eje $X$ para que se erigieran verticalmente, simulando columnas clásicas.
* **La Bandera de México:** * **El Asta:** Es una caja gris muy delgada y alta ($7.5$ unidades) colocada a la izquierda.
* **El Lienzo:** Se construyó uniendo tres rectángulos contiguos (`box`) de igual tamaño pero con colores específicos: Verde `(0, .55, .27)`, Blanco `(.95, .95, .95)` y Rojo `(.80, 0, 0)`. Al estar pegados en el eje $X$, simulan perfectamente la bandera tricolor.



---

## 4. Animación de las Personas (`draw_person_at`)

Para lograr que las personas se desplazaran por el pueblo y movieran sus extremidades de forma realista, se combinaron dos tipos de matemáticas:

### A. Traslación Circular (Movimiento en el Pueblo)

La posición de cada persona en el mapa no es fija. Se calcula en cada fotograma mediante la función `animated_pos`:

$$\text{Angulo} = \text{Dirección Inicial} + (\text{Tiempo} \times \text{Velocidad})$$

Usando las funciones coseno y seno, la persona camina en círculos perfectos alrededor de un punto central de radio variable:

* $X = \text{Centro}_X + \text{Radio} \times \cos(\text{Angulo})$
* $Z = \text{Centro}_Z + \text{Radio} \times \sin(\text{Angulo})$

### B. Animación Articular (Caminar)

Para simular el braceo y el paso al caminar, se utilizó una función senoidal basada en el tiempo acumulado (`t = time.time()`):

```python
sw = math.sin(anim_t * 5.0) * 15.0

```

Esta variable `sw` genera una oscilación suave que va de $-15^\circ$ a $+15^\circ$. Al dibujar las piernas y los brazos, se usa `glRotatef(sw, 1, 0, 0)` para un miembro y `glRotatef(-sw, 1, 0, 0)` para el opuesto. Esto hace que cuando el brazo izquierdo va hacia adelante, el derecho vaya hacia atrás de forma cíclica y automática.

---

## 5. Los Burros (`draw_burro_at`)

Los burros utilizan una lógica volumétrica similar a la de las personas, pero adaptada a una anatomía cuadrúpeda.

* **Cuerpo y Cabeza:** El torso principal es una caja gris horizontal grande, el cuello es una caja inclinada hacia arriba, y el hocico es otra caja que proyecta la cabeza hacia el frente.
* **Las Orejas:** Dos cajas delgadas y largas modeladas en la parte superior de la cabeza con una leve rotación exterior de $15^\circ$ (`glRotatef(15, 0, 0, 1)`) para darles expresividad.
* **Animación de las 4 Patas:** Al igual que con las personas, se utiliza la onda senoidal del tiempo multiplicada por la dirección `s` (donde `s = 1` o `s = -1`):
* Las patas **delantera izquierda** y **trasera derecha** se mueven sincronizadas en una dirección.
* Las patas **delantera derecha** y **trasera izquierda** se mueven en la dirección opuesta.
Esto recrea fielmente el patrón de marcha cruzada natural de un cuadrúpedo mientras avanza en sus órbitas alrededor de la plaza.
