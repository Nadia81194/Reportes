# Reporte Segmentacion de frutas
En esta practica trabajamos la mascara binaria con rango HSV   

#Actividad 1
Seleccionamops el color amarillo y ajustamos el rango HSV para que la mascara detectara solamente el color amarillo 

```python
#     # actividad 1 detectar los colores amarillos
    lower_yellow = np.array([20, 100, 100])
    upper_yellow = np.array([35, 255, 255])
    mask = cv2.inRange(hsv, lower_yellow, upper_yellow)


