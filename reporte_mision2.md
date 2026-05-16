# Reporte de Misión: Graficación Táctica II
**Agente Especial:** [Nadia Coria Aragon /24120411]

---
## Evidencias
### Misión 1

- Código:
```python
import cv2
import numpy as np

ruta_imagen = r'C:\Users\tigre\Downloads\m1_oscura 1.png'
foto = cv2.imread(ruta_imagen, 0)

if foto is not None:
    alto, ancho = foto.shape
    
    capa1 = np.zeros((alto, ancho), dtype=np.int32)
    for i in range(0, alto):
        for j in range(0, ancho):
            pixel = foto[i, j]
            nuevo_p = pixel * 50
            if nuevo_p > 255:
                nuevo_p = 255
            capa1[i, j] = nuevo_p

    capa2 = np.zeros((alto, ancho), dtype=np.int32)
    for i in range(alto):
        for j in range(ancho):
            val = foto[i, j]
            res = (val * 50) + 20
            if res > 255:
                res = 255
            if res < 0:
                res = 0
            capa2[i, j] = res

    final1 = capa1.astype(np.uint8)
    final2 = capa2.astype(np.uint8)

    cv2.imshow('Resultado 1', final1)
    cv2.imshow('Resultado 2', final2)
    
    print("Mostrando imagenes...")
    cv2.waitKey(0)
    cv2.destroyAllWindows()
else:
    print("no se encontro el archivo")


```
# Resultado
![mascara](imagenes/mision1e2.jpg)

### Misión 2

- Código:
```python
import cv2
import numpy as np

m1 = cv2.imread(r'C:\Users\tigre\Downloads\m2_mitad1.png')
m2 = cv2.imread(r'C:\Users\tigre\Downloads\m2_mitad2.png')
lienzo = np.full((600, 600, 3), 255, dtype=np.uint8)

if m1 is not None and m2 is not None:
    h1, w1 = m1.shape[:2]
    h2, w2 = m2.shape[:2]

    # Mitad 1
    m_t = np.float32([[1, 0, 0], [0, 1, 0]])
    p1 = cv2.warpAffine(m1, m_t, (w1, h1))

    # Mitad 2
    c = (w2 // 2, h2 // 2)
    m_r = cv2.getRotationMatrix2D(c, 180, 1.0)
    p2 = cv2.warpAffine(m2, m_r, (w2, h2))

    y = 50
    x = 50
   
    lienzo[y : y + h1, x : x + w1] = p1
    lienzo[y + h1 : y + h1 + h2, x : x + w2] = p2

    cv2.imwrite(" qr reconstruido.png", lienzo)
    cv2.imshow("resultado", lienzo)
    cv2.waitKey(0)
    cv2.destroyAllWindows()
else:
    print("error con la ruta de las imagenes")


```
# Resultado
![mascara](imagenes/mision2e2.jpg)

### Misión 3

- Código:
```python
import cv2
import numpy as np
import math

sello = np.zeros((600, 600, 3), dtype=np.uint8)
sello[:] = (40, 20, 20)

cv2.circle(sello, (300, 300), 170, (0, 255, 255), 3)
cv2.circle(sello, (300, 300), 110, (0, 255, 255), 2)

cv2.rectangle(sello, (250, 260), (350, 340), (0, 0, 255), -1)

cv2.line(sello, (0, 0), (600, 600), (255, 255, 255), 2)
cv2.line(sello, (600, 0), (0, 600), (255, 255, 255), 2)

cx = 300
cy = 300
d = 140

for i in range(8):
    a = i * (360 / 8)
    r = a * (3.1416 / 180)
    x = int(cx + d * math.cos(r))
    y = int(cy + d * math.sin(r))
    cv2.circle(sello, (x, y), 8, (0, 255, 0), -1)

f = cv2.FONT_HERSHEY_SIMPLEX
cv2.putText(sello, "SECTOR-9", (210, 560), f, 1.5, (255, 255, 255), 3)

cv2.imwrite("m3_sello", sello)
cv2.imshow("sello", sello)
cv2.waitKey(0)
cv2.destroyAllWindows()

```
# Resultado
![mascara](imagenes/mision3e2.jpg)
### Misión 4
- Código:
```python

import cv2
import numpy as np

img = cv2.imread(r'C:\Users\tigre\Downloads\m4_ruido.png')

kernel = np.array([[1, 1, 1], 
                   [1, 1, 1], 
                   [1, 1, 1]], dtype=np.float32) / 9

img_suave = cv2.filter2D(img, -1, kernel)

hsv = cv2.cvtColor(img_suave, cv2.COLOR_BGR2HSV)

bajo = np.array([80, 50, 50])
alto = np.array([100, 255, 255])

mascara = cv2.inRange(hsv, bajo, alto)
cv2.imshow("Imagen Suavizada", img_suave)
cv2.imshow("Mascara Cyan", mascara)
cv2.waitKey(0)
cv2.destroyAllWindows()

```
# Resultado
![mascara](imagenes/mision4e2.png)
### Misión 5
- Código:
```python
import cv2
import numpy as np
img = np.random.randint(0, 255, (300, 700, 3), dtype=np.uint8)
font = cv2.FONT_HERSHEY_SIMPLEX
cv2.putText(img, "CODIGO_SECRETO_5", (50, 150), font, 2, (30, 240, 120), 5)

b, g, r = cv2.split(img)
diff_gb = cv2.absdiff(g, b)
resta_rg = cv2.subtract(r, g)
_, mensaje_final = cv2.threshold(diff_gb, 50, 255, cv2.THRESH_BINARY)

cv2.imshow("B (Azul)", b)
cv2.imshow("G (Verde)", g)
cv2.imshow("R (Rojo)", r)
cv2.imshow("Prueba(G-B)", diff_gb)
cv2.imshow("Prueba R-G", resta_rg)
cv2.imshow("Mensaje ", mensaje_final)

cv2.waitKey(0)
cv2.destroyAllWindows()

```
# Resultado
![mascara](imagenes/mision5e2-1.jpg)
![mascara](imagenes/mision5e2-2.jpg)

---
## Análisis del Analista (Reflexiones Finales)

1. **Operadores puntuales (M1):** ¿Qué diferencia visual hay entre recuperar con multiplicación (x50) y recuperar con suma (+50)? ¿Cuál preserva mejor el contraste del texto?
> [Respuesta]

2. **Transformaciones geométricas (M2):** ¿Por qué es importante escoger el centro correcto al rotar una imagen con `getRotationMatrix2D`?
> [Respuesta]

3. **Convolución (M4):** ¿Por qué un filtro promedio puede ayudar a reducir falsos positivos antes de segmentar por HSV, y qué desventaja tiene sobre los bordes del texto?
> [Respuesta]

4. **Canales (M5):** ¿Por qué separar canales puede revelar información que en la imagen a color “no se ve” a simple vista?
> [Respuesta]
