# Reporte : Simulador de Acuario en Realidad Aumentada Estilo Minecraft
**Asignatura:** Graficación  
**Agente:** Nadia Coria Aragon  
**Institución:** Instituto Tecnológico de Morelia  

---

## 1. Introducción
El proyecto consiste en el desarrollo de una aplicación interactiva de Realidad Aumentada (RA) basada en marcadores visuales. El sistema integra captura de video en tiempo real, visión artificial para el rastreo posicional y renderizado de gráficos tridimensionales con estética voxelada (estilo Minecraft) utilizando librerías avanzadas de Python.

## 2. Tecnologías Utilizadas
La arquitectura del software se compone de las siguientes herramientas de desarrollo:
* **Python:** Lenguaje base de programación.
* **OpenCV (cv2):** Encargado de la captura de video por hardware, conversión de espacios de color y el procesamiento de la librería **ArUco** para detectar marcadores fiduciales e interpolar matrices espaciales mediante algoritmos PnP (*Perspective-n-Point*).
* **GLFW:** API multiplataforma encargada de la gestión nativa de ventanas del sistema operativo, el contexto OpenGL y el manejo de callbacks del teclado.
* **PyOpenGL (GL, GLU):** Envoltura de funciones gráficas que implementa el pipeline de renderizado 3D acelerado por hardware (GPU).

## 3. Elementos Desarrollados en el Código
El sistema proyecta sobre un único marcador (`ID 0`) un entorno marino tridimensional dinámico de dimensiones $30 \times 30 \times 30$ , compuesto por:

### A. Estructuras del Entorno (Escenografía)
* **Suelo de Arena:** Un prisma base rectangular plano sólido que simula el fondo marino.
* **Bloques de Coral:** Bloques tridimensionales rígidos en colores rojo (Fuego), azul (Tubo) y rosa (Cerebro) distribuidos asimétricamente para dar profundidad espacial.
* **Bosque de Algas Animadas (Kelp):** Seis estructuras de vegetación generadas mediante **transformaciones jerárquicas encadenadas**. Cada segmento del alga rota ligeramente respecto a su predecesor utilizando una función armónica sinusoidal ($\sin(t)$), emulando de manera orgánica la fricción y el vaivén provocados por corrientes de agua.

### B. Entidades Biológicas Interactivas
* **Cardumen de Peces Tropicales:** Seis entidades geométricas distribuidas en dos grupos con rotaciones inversas (manecillas del reloj y antihorario). Utilizan oscilaciones rápidas en la aleta trasera para simular nado de alta frecuencia.
* **Especies de Ajolotes (Tamaño Grande):** Cinco modelos construidos a partir de primitivas de cubos puros con las paletas de color oficiales del videojuego: *Lucy (Rosa), Cyan (Celeste), Wild (Marrón), Gold (Dorado) y Rare (Morado)*. Cada ajolote posee animación coordinada en su cola articulada y extremidades inferiores.

## 4. Conceptos Gráficos Clave Implementados
1. **Pipeline de Realidad Aumentada:** OpenCV mapea los vértices del marcador 2D en la imagen plana y calcula sus coordenadas de proyección del mundo real usando un archivo de calibración (`camera_ar.npz`). Esa información se inyecta directamente en las matrices `GL_PROJECTION` y `GL_MODELVIEW` de OpenGL para alinear perfectamente los gráficos virtuales con la inclinación física de la cámara.
2. **Uso del Canal Alfa (Transparencia):** Para simular el agua dentro del cubo de cristal de $30 \times 30$, se habilitó `GL_BLEND` junto a funciones de mezcla de origen y destino (`glBlendFunc`). El búfer de profundidad (`glDepthMask`) se manipula dinámicamente para prevenir errores de oclusión en los polígonos traslúcidos.
3. **Modelado Jerárquico:** El movimiento del ajolote se rige por herencia matricial utilizando `glPushMatrix()` y `glPopMatrix()`. Las aletas y las patas se desplazan y rotan respecto a la matriz de transformación del cuerpo principal, evitando que las piezas del modelo se separen durante el desplazamiento circular.

## 5. Conclusiones
El proyecto cumple exitosamente con los requisitos de la materia de Graficación al aplicar de forma práctica conceptos complejos como transformaciones lineales espaciales, texturizado en tiempo real, iluminación matemática difusa y control de profundidad de renderizado, logrando un demo procedural fluido y visualmente atractivo.
