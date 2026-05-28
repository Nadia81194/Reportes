# Reporte de Práctica: Demo Procedural 64K (OpenGL + OpenCV)

**Instituto Tecnológico de Morelia** **Materia:** Graficación (Grupo B)  
**Estudiante:** Nadia Coria Aragon  
**Fecha:** 28 de Mayo de 2026  

---

## 1. Objetivo de la Práctica
El objetivo principal de esta práctica es diseñar y desarrollar una demostración gráfica ("Demo 64K") completamente procedural utilizando OpenGL y la manipulación de gráficos e imágenes bidimensionales con OpenCV . 


## 2. ¿Qué hace este código?
Este código es un **generador automático de video y animaciones en tiempo real**. Su objetivo principal es crear una presentación audiovisual de 60 segundos de duración dividida en 6 escenas continuas de 10 segundos cada una, guardando el resultado en un archivo de video de alta calidad (`NadiaCoria_Demo_Mejorada.mp4`).

Visualmente, el programa rinde un homenaje a la estética de las computadoras retro y las pantallas de cinescopio (CRT). En lugar de cargar imágenes o videos externos desde el disco duro, **el programa dibuja absolutamente todo desde cero mediante operaciones matemáticas** frame por frame (fotograma a fotograma) a una velocidad fluida de 30 imágenes por segundo.

Las fases o escenas que van apareciendo de manera secuencial son:
1. **Portada Dinámica:** Una presentación oficial interactiva con mis datos personales (Nadia Coria Argón, TecNM, Fecha actual), decorada con estrellas artificiales y un marco cuyos colores cambian velozmente.
2. **Escena 1 (Curvas de Lissajous):** Figuras geométricas entrelazadas basadas en ondas oscilatorias que se estiran y pulsan al ritmo del tiempo.
3. **Escena 2 (Rosa Polar Modificada):** Una flor matemática de múltiples pétalos que muta de tamaño e incorpora círculos saltarines que simulan un ecualizador de audio ("Beats").
4. **Escena 3 (Espirógrafo Loco):** El clásico efecto de los juguetes de dibujo de engranajes repetitivos, pero duplicado en espejo para rellenar la pantalla con trazos psicodélicos.
5. **Escena 4 (Tormenta de Partículas):** Una simulación de 1,500 pequeños bloques retro que vuelan caóticamente por la pantalla como si fueran arrastrados por ráfagas de viento invisible.
6. **Escena 5 (Infierno Procedural):** Una simulación interactiva de fuego místico que se eleva desde la base cambiando entre colores verdes, azules y rojos, terminando con un desvanecimiento suave a negro.

---

## 2. ¿Cómo se logró? (Metodología y Herramientas)

Para construir este sistema gráfico me apoyé en el lenguaje de programación **Python** y utilicé tres herramientas de software clave que facilitan el trabajo con matrices de datos y gráficos:
* **OpenCV (`cv2`):** Es el motor visual encargado de pintar figuras geométricas básicas (líneas, círculos, textos), manejar el color, gestionar la ventana en pantalla y codificar el archivo de video final en formato MP4.
* **NumPy (`np`):** Funciona como el cerebro analítico. Permite manipular millones de pixeles de forma masiva y ultrarrápida sin ralentizar la computadora.
* **Math / Time:** Librerías nativas para procesar las funciones trigonométricas y controlar que el video corra exactamente al tiempo de la vida real.

### El proceso paso a paso:
Para lograr la fluidez y el dinamismo de la animación, implementé los siguientes pilares de desarrollo:

* **El Espacio de Color HSV (Tono, Saturación, Valor):** En lugar de usar los aburridos colores estáticos RGB, programé los fondos y trazos usando el sistema HSV. Esto me permitió hacer algo genial: alterando únicamente el valor del "Tono" en base al reloj del sistema, logré que los degradados del fondo y las figuras giren salvajemente por toda la paleta de colores del arcoíris de forma natural.
* **Modelado Matemático de Curvas:** Para dibujar las figuras complejas (como la Rosa Polar o el Espirógrafo), utilicé ecuaciones que transforman coordenadas angulares en puntos de pantalla. Al multiplicar los ángulos por funciones `sin` (seno) y `cos` (coseno) que cambian con los segundos, causé que las figuras se deformen, roten y cobren vida por sí mismas.
* **Efectos Estroboscópicos y Transiciones Fluidas:** En lugar de hacer cortes bruscos entre escenas, utilicé una función matemática llamada `smoothstep`. Con ella, logré transiciones muy suaves durante los últimos 1.2 segundos de cada escena, mezclando de manera intermitente la pantalla vieja con la nueva, acompañado de un destello blanco rápido para darle mayor impacto visual.
* **Filtro de Pantalla CRT Antigua:** Para darle el toque final "retro", alteré los renglones del lienzo final. Modifiqué el código para que, cada tres líneas horizontales de pixeles, el brillo se reduzca en un 30%. Esto genera de forma artificial las famosas líneas de exploración o "scanlines" características de los monitores de fósforo antiguos.

---

## 3. Conclusión
La realización de este proyecto me demostró que los gráficos por computadora no dependen necesariamente de almacenar archivos multimedia pesados, sino de la creatividad lógica aplicada a las matemáticas. 

Pude comprobar cómo unas simples fórmulas trigonométricas combinadas con el manejo correcto de matrices en NumPy pueden transformarse en fuego procedural, tormentas de viento o figuras geométricas dinámicas que cautivan al espectador. Esta experiencia expandió significativamente mi visión sobre el renderizado procedural, dándome las bases técnicas fundamentales en la materia de Graficación para diseñar interfaces visuales interactivas, dinámicas y eficientes desde código puro.
"""

# Guardar el contenido del reporte en un archivo Markdown (.md)
with open("Reporte_Graficacion_NadiaCoria.md", "w", encoding="utf-8") as file:
    file.write(reporte_contenido)

print("Archivo Markdown generado con éxito.")
