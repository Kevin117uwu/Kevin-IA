# Detección y Reconocimiento Facial

## Detección de Rostros

```python
import numpy as np
import cv2 as cv
import math

# Cargar el clasificador Haar Cascade para la detección de rostros
rostro = cv.CascadeClassifier('haarcascade_frontalface_alt2.xml')

# Abrir un objeto de captura de video
cap = cv.VideoCapture(0)
i = 0

while True:
    # Leer un fotograma del video
    ret, frame = cap.read()
    gray = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)

    # Detectar rostros en la imagen en escala de grises
    rostros = rostro.detectMultiScale(gray, 1.3, 5)

    # Procesar cada rostro detectado
    for (x, y, w, h) in rostros:
        frame = cv.rectangle(frame, (x, y), (x + w, y + h), (0, 255, 0), 2)
        frame2 = frame[y - 10:y + h + 10, x - 10:x + w + 10]
        cv.imshow('rostros2', frame2)
        frame2 = cv.resize(frame2, (100, 100), interpolation=cv.INTER_AREA)
        cv.imwrite('C:\\Users\\kevin\\caras\\quique\\quique' + str(i) + '.png', frame2)

    # Mostrar el fotograma con los rostros detectados
    cv.imshow('rostros', frame)
    i = i + 1
    k = cv.waitKey(1)

    # Salir del bucle si se presiona la tecla 'Esc'
    if k == 27:
        break

# Liberar el objeto de captura de video y cerrar todas las ventanas
cap.release()
cv.destroyAllWindows()

import cv2 as cv
import numpy as np
import os

# Definir la ruta al conjunto de datos
dataSet = 'C:\\Users\\kevin\\caras'

# Listar todas las carpetas de rostros en el conjunto de datos
faces = os.listdir(dataSet)
print(faces)

# Inicializar listas para almacenar etiquetas y datos de rostros
labels = []
facesData = []
label = 0

# Iterar a través de cada carpeta de rostros
for face in faces:
    facePath = dataSet + '\\' + face
    print(facePath)

    # Iterar a través de cada imagen en la carpeta de rostros
    for faceName in os.listdir(facePath):
        labels.append(label)
        facesData.append(cv.imread(facePath + '\\' + faceName, 0))

    label = label + 1

# Imprimir el número de imágenes para la primera etiqueta (asumiendo que las etiquetas comienzan desde 0)
print(np.count_nonzero(np.array(labels) == 0))

# Entrenar un reconocedor EigenFace utilizando los datos recopilados
faceRecognizer = cv.face.EigenFaceRecognizer_create()
faceRecognizer.train(facesData, np.array(labels))

# Guardar el modelo entrenado
faceRecognizer.write('Eigenface.xml')


import cv2 as cv
import os

# Cargar el reconocedor EigenFace entrenado
faceRecognizer = cv.face.EigenFaceRecognizer_create()
faceRecognizer.read('Eigenface.xml')

# Abrir un objeto de captura de video
cap = cv.VideoCapture(0)
rostro = cv.CascadeClassifier('haarcascade_frontalface_alt2.xml')

while True:
    # Leer un fotograma del video
    ret, frame = cap.read()

    # Salir del bucle si no se lee ningún fotograma
    if ret == False:
        break

    # Convertir el fotograma a escala de grises
    gray = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)
    cpGray = gray.copy()

    # Detectar rostros en la imagen en escala de grises
    rostros = rostro.detectMultiScale(gray, 1.3, 3)

    # Procesar cada rostro detectado
    for (x, y, w, h) in rostros:
        frame2 = cpGray[y:y + h, x:x + w]
        frame2 = cv.resize(frame2, (100, 100), interpolation=cv.INTER_CUBIC)
        result = faceRecognizer.predict(frame2)

        # Mostrar el rostro reconocido o etiqueta 'Desconocido' para rostros desconocidos
        if result[1] > 2800:
            cv.putText(frame, '{}'.format(faces[result[0]]), (x, y - 25), 2, 1.1, (0, 255, 0), 1, cv.LINE_AA)
            cv.rectangle(frame, (x, y), (x + w, y + h), (0, 255, 0), 2)
        else:
            cv.putText(frame, 'Desconocido', (x, y - 20), 2, 0.8, (0, 0, 255), 1, cv.LINE_AA)
            cv.rectangle(frame, (x, y), (x + w, y + h), (0, 0, 255), 2)

    # Mostrar el fotograma con los resultados del reconocimiento facial
    cv.imshow('frame', frame)
    k = cv.waitKey(1)

    # Salir del bucle si se presiona la tecla 'Esc'
    if k == 27:
        break

# Liberar el objeto de captura de video y cerrar todas las ventanas
cap.release()
cv.destroyAllWindows()

