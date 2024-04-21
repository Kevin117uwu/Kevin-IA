# Documentación de Código en js
# Documentación del Juego con Phaser y Red Neuronal

## Variables Globales
El juego tiene varias variables globales que controlan diferentes aspectos, como el tamaño del juego, personajes, elementos, redes neuronales, entre otros.

```javascript
var w = 800;  # Anchura del juego
var h = 400;  # Altura del juego
var jugador;  # Personaje principal
var fondo;  # Fondo del juego
var bala, balaD = false, nave;  # Bala, bandera de disparo, y nave
var salto, menu, izq, der;  # Variables de control
var velocidadBala, despBala;  # Velocidad y desplazamiento de la bala
var estatusAire, estatuSuelo;  # Estado del personaje (aire o suelo)
var nnNetwork, nnEntrenamiento, nnSalida, datosEntrenamiento = [];  # Red neuronal y datos de entrenamiento
var modoAuto = false, eCompleto = false;  # Modo automático y bandera de entrenamiento completo
