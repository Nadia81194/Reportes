# Reporte de Misión: Graficación Táctica II
**Agente Especial:** [Nadia Coria Aragon /Matrícula]

---
## Evidencias
### Misión 1
- Imagen recuperada x50: (inserta)
- Imagen recuperada x50 + 20: (inserta)
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
- QR reconstruido: (inserta)
- Código:

### Misión 3
- Sello forjado: (inserta)
- Código:

### Misión 4
- Máscara Cyan: (inserta)
- Código:

### Misión 5
- Evidencia tricolor: (inserta)
- Mensaje recuperado: (inserta)
- Código:

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
