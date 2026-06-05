
```markdown
# Reporte de Práctica: Iluminación y Materiales en un Ojo 3D
**Curso:** Graficación  
**Estudiante:** Nadia Coria Aragon  
**Fecha:** Junio 2026  

---

## 1. Introducción
En esta práctica se realizó la transición de un modelo 3D basado en colores planos (`glColor3f`) a un entorno con iluminación realista utilizando el pipeline fijo de OpenGL. El objeto de prueba consiste en un ojo estructurado mediante cuatro esferas concéntricas y desfasadas que representan la piel/borde, la esclerótica (blanco del ojo), el iris y la pupila. El objetivo principal fue aprender a configurar luces, calcular normales geométricas y aplicar propiedades de materiales (brillo especular y rugosidad) para lograr volumen y realismo.

---

## 2. Desarrollo de las Misiones (Checkpoints)

### Misión 1: "Enciende la luz"
**Objetivo:** Activar el motor de iluminación de OpenGL para dejar de renderizar colores planos y empezar a calcular intensidades basadas en la orientación de las luces.

**Cómo se logró:** Se modificó la función `setup_lighting()` para habilitar los estados globales de iluminación (`GL_LIGHTING`) y la primera fuente de luz (`GL_LIGHT0`). También se activó el buffer de profundidad (`GL_DEPTH_TEST`) para que las esferas se oculten correctamente unas detrás de otras. Se definieron las propiedades de color de la luz (Ambient, Diffuse y Specular) utilizando un tono blanco puro y se cambió el cuarto componente de la posición a `1.0` para que actúe como una luz posicional (foco o lámpara local) en lugar de una luz direccional (como el sol).

**Código implementado:**
```python
def setup_lighting():
    # Activamos la iluminación general y el z-buffer
    glEnable(GL_LIGHTING)
    glEnable(GL_LIGHT0)
    glEnable(GL_DEPTH_TEST) 

    # Colores de la luz (Blanca)
    glLightfv(GL_LIGHT0, GL_AMBIENT, [0.2, 0.2, 0.2, 1.0])
    glLightfv(GL_LIGHT0, GL_DIFFUSE, [0.8, 0.8, 0.8, 1.0])
    glLightfv(GL_LIGHT0, GL_SPECULAR, [1.0, 1.0, 1.0, 1.0])

```

---

### Misión 2: "La esfera necesita normales"

**Objetivo:** Lograr que la luz entienda la curvatura de la esfera para generar un sombreado suave.

**Cómo se logró:** Por defecto, las esferas de GLU no calculan los vectores normales de cada vértice si no se le indica de forma explícita. Sin normales, OpenGL no sabe en qué dirección "rebota" la luz, provocando errores visuales o ausencia de sombras. Se solucionó agregando la instrucción `gluQuadricNormals(quad, GLU_SMOOTH)` justo antes de dibujar la esfera. Esto genera normales promediadas que dan una apariencia completamente redondeada y orgánica.

**Código implementado:**

```python
def draw_sphere(radius, slices=30, stacks=30):
    quad = gluNewQuadric()
    # Misión 2: Generar normales suaves para el sombreado
    gluQuadricNormals(quad, GLU_SMOOTH) 
    gluSphere(quad, radius, slices, stacks)
    gluDeleteQuadric(quad)

```

---

### Misión 3: "Materiales: esclerótica, iris y pupila"

**Objetivo:** Controlar de forma independiente el comportamiento del brillo y reflejo en cada parte del ojo.

**Cómo se logró:** Se creó la función auxiliar `set_material(amb, diff, spec, shine)` que simplifica las llamadas a `glMaterialfv` y `glMaterialf`. Con este método se definieron comportamientos específicos para cada componente del ojo:

* **Esclerótica (Blanco):** Un brillo especular muy alto (`[0.9, 0.9, 0.9]`) y un exponente de brillo (shininess) de `100` para que parezca una superficie húmeda y pulida.
* **Iris (Azul):** Un brillo medio (`30`) que reduce la intensidad del reflejo.
* **Pupila (Negro):** Un brillo casi nulo (`1`) para simular que absorbe la luz por completo.
* **Piel:** Un acabado mayormente mate con baja respuesta especular.

**Código implementado:**

```python
def set_material(amb, diff, spec, shine):
    glMaterialfv(GL_FRONT, GL_AMBIENT, amb)
    glMaterialfv(GL_FRONT, GL_DIFFUSE, diff)
    glMaterialfv(GL_FRONT, GL_SPECULAR, spec)
    glMaterialf(GL_FRONT, GL_SHININESS, shine)

# Ejemplo de aplicación en el dibujo (Esclerótica):
glColor3f(1.0, 1.0, 1.0)
set_material([0.2, 0.2, 0.2, 1.0], [1.0, 1.0, 1.0, 1.0], [0.9, 0.9, 0.9, 1.0], 100)

```

---

### Misión 4: "Que glColor afecte el material"

**Objetivo:** Combinar el uso práctico de `glColor3f()` con el sistema de iluminación, evitando tener que redefinir el color difuso en los arreglos de materiales cada vez.

**Cómo se logró:** Se activó el estado `GL_COLOR_MATERIAL` dentro de la configuración inicial y se le indicó a OpenGL que vincule los valores de `glColor3f` directamente a las propiedades ambientales y difusas del material actual mediante `glColorMaterial(GL_FRONT_AND_BACK, GL_AMBIENT_AND_DIFFUSE)`. Gracias a esto, el color base se define de forma sencilla con `glColor` mientras que las propiedades plásticas/metálicas avanzadas (especularidad y brillo) siguen dictadas por los coeficientes de los materiales.

**Código implementado:**

```python
# Dentro de setup_lighting():
glEnable(GL_COLOR_MATERIAL)
glColorMaterial(GL_FRONT_AND_BACK, GL_AMBIENT_AND_DIFFUSE)

```

---

### Misión 5: Opción A - "Luz fija en el mundo"

**Objetivo:** Lograr que la fuente de luz permanezca estática en el escenario mientras el ojo gira sobre su propio eje.

**Cómo se logró:** Se colocaron las líneas de posicionamiento de la luz (`glLightfv(GL_LIGHT0, GL_POSITION, light_pos)`) dentro del ciclo principal de renderizado, justo después de reiniciar la matriz con `glLoadIdentity()` y aplicar la perspectiva, pero **antes** de realizar las operaciones de traslación y rotación del ojo (`glRotatef`).

**Código implementado:**

```python
# Dentro del loop while de main():
glMatrixMode(GL_MODELVIEW)
glLoadIdentity()

# OPCIÓN A: Posición de la luz fija en las coordenadas del mundo
light_pos = [2.0, 2.0, 2.0, 1.0] 
glLightfv(GL_LIGHT0, GL_POSITION, light_pos)

# Después de la luz se transforma el objeto
glTranslatef(0, 0, -5)
rotation += 0.5
glRotatef(rotation, 0, 1, 0)
draw_eye()

```

---

## 3. Conclusiones y Explicación Teórica (Misión 5)

La razón por la que la luz se comporta de forma estática en la **Opción A** radica en el orden en que OpenGL procesa las matrices de transformación. En el pipeline fijo, la posición definida en `glLightfv` se multiplica de manera inmediata por la matriz **MODELVIEW** que esté activa en ese preciso instante.

Al declarar la posición de la luz inmediatamente después de un `glLoadIdentity()`, la matriz está limpia (es la identidad), lo que provoca que la luz se posicione directamente en las coordenadas globales del mundo (o de la cámara). Cuando posteriormente se aplican los comandos `glTranslatef` y `glRotatef`, estos comandos sólo afectan a los objetos que se dibujen después (las esferas del ojo). Como resultado, al rotar el ojo, podemos observar cómo las sombras y los brillos especulares se desplazan dinámicamente sobre su superficie, validando que el cálculo de la iluminación es correcto y que la luz no se encuentra "pegada" al objeto.

```

```
![mascara](imagenes/ojo.mp4)
