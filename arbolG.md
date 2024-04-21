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
```

## Inicialización del Juego
La inicialización del juego se realiza creando una instancia de Phaser con las dimensiones definidas. Aquí se define el tipo de renderizado, los eventos, y se asocian las funciones `preload`, `create`, `update`, y `render`.

```javascript
var juego = new Phaser.Game(w, h, Phaser.CANVAS, '', { preload: preload, create: create, update: update, render: render });
```

## Función `preload`
En esta función, se cargan los recursos necesarios para el juego, como imágenes y sprites. Se define qué archivos se utilizarán para el fondo, personajes y elementos del juego.

```javascript
function preload() {
    juego.load.image('fondo', 'assets/game/fondo2.jpg');
    juego.load.spritesheet('mono', 'assets/sprites/goku.png', 38, 63);
    juego.load.image('nave', 'assets/game/ufo.png');
    juego.load.image('bala', 'assets/sprites/purple_ball.png');
    juego.load.image('menu', 'assets/game/menu.png');
}
```

## Función `create`
En la función `create`, se inicializa el sistema de física y se establecen las propiedades iniciales del juego, como la gravedad y los objetos presentes en el mismo.

```javascript
function create() {
    # Configuración del sistema de física
    juego.physics.startSystem(Phaser.Physics.ARCADE);
    juego.physics.arcade.gravity.y = 800;
    juego.time.desiredFps = 30;

    # Agregar elementos al juego
    fondo = juego.add.tileSprite(0, 0, w, h, 'fondo');
    nave = juego.add.sprite(w - 100, h - 70, 'nave');
    bala = juego.add.sprite(w - 100, h, 'bala');
    jugador = juego.add.sprite(50, h, 'mono');

    # Configurar la física para el jugador
    juego.physics.enable(jugador);
    jugador.body.collideWorldBounds = true;

    # Agregar animaciones al personaje
    jugador.animations.add('corre', [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]);
    jugador.animations.play('corre', 10, true);

    # Configurar la física para la bala
    juego.physics.enable(bala);
    bala.body.collideWorldBounds = true;

    # Configurar la opción de pausa
    pausaL = juego.add.text(w - 100, 20, 'Pausa', { font: '20px Arial', fill: '#fff' });
    pausaL.inputEnabled = true;
    pausaL.events.onInputUp.add(pausa, self);
    juego.input.onDown.add(mPausa, self);

    # Configurar el salto del personaje
    salto = juego.input.keyboard.addKey(Phaser.Keyboard.SPACEBAR);

    # Configuración de la red neuronal
    nnNetwork = new synaptic.Architect.Perceptron(2, 6, 6, 2);
    nnEntrenamiento = new synaptic.Trainer(nnNetwork);
}
```

## Función `update`
La función `update` se ejecuta en cada ciclo del juego, manejando la lógica del juego y las interacciones entre personajes y objetos.

```javascript
function update() {
    # Mover el fondo para simular movimiento
    fondo.tilePosition.x -= 1;

    # Detectar colisiones entre la bala y el jugador
    juego.physics.arcade.collide(bala, jugador, colisionH, null, this);

    # Determinar si el jugador está en el suelo o en el aire
    estatuSuelo = 1;
    estatusAire = 0;

    if (!jugador.body.onFloor()) {
        estatuSuelo = 0;
        estatusAire = 1;
    }

    # Calcular el desplazamiento entre el jugador y la bala
    despBala = Math.floor(jugador.position.x - bala.position.x);

    # Control del salto del personaje
    if (!modoAuto && salto.isDown && jugador.body.onFloor()) {
        saltar();
    }
    
    # Modo automático para el salto usando la red neuronal
    if (modoAuto && bala.position.x > 0 && jugador.body.onFloor()) {
        if (datosDeEntrenamiento([despBala, velocidadBala])) {
            saltar();
        }
    }

    # Disparar la bala si no ha sido disparada antes
    if (!balaD) {
        disparo();
    }

    # Reiniciar el juego si la bala alcanza el extremo izquierdo
    if (bala.position.x <= 0) {
        resetVariables();
    }

    # Recopilar datos de entrenamiento para la red neuronal
    if (!modoAuto && bala.position.x > 0) {
        datosEntrenamiento.push({
            'input': [despBala, velocidadBala],
            'output': [estatusAire, estatuSuelo]
        });

        console.log("Desplazamiento Bala, Velocidad Bala, Estatus, Estatus: ",
            despBala + " " + velocidadBala + " " + estatusAire + " " + estatuSuelo);
   }
}
```

## Función `disparo`
La función `disparo` establece la velocidad y dirección de la bala, disparándola hacia el jugador.

```javascript
function disparo() {
    velocidadBala = -1 * velocidadRandom(300, 800);  # Velocidad aleatoria de la bala
    bala.body.velocity.y = 0;
    bala.body.velocity.x = velocidadBala;  # Dirección de la bala hacia la izquierda
    balaD = true;  # Indicador de que la bala ha sido disparada
}
```

## Función `pausa` y `mPausa`
Estas funciones manejan la pausa del juego y el menú asociado. Se define la lógica para detener el juego y reiniciar el modo automático y el entrenamiento de la red neuronal.

```javascript
function pausa() {
    juego.paused = true;  # Detener el juego
    menu = juego.add.sprite(w / 2, h / 2, 'menu');  # Crear el menú de pausa
    menu.anchor.setTo(0.5, 0.5);  # Centrar el menú
}

function mPausa(event) {
    if (juego.paused) {
        # Coordenadas del menú
        var menu_x1 = w / 2 - 270 / 2, menu_x2 = w / 2 + 270 / 2,
            menu_y1 = h / 2 - 180 / 2, menu_y2 = h / 2 + 180 / 2;

        # Coordenadas del ratón
        var mouse_x = event.x,
            mouse_y = event.y;

        # Verificar si el ratón está dentro del menú para seleccionar opciones
        if (mouse_x > menu_x1 && mouse_x < menu_x2 && mouse_y > menu_y1 && mouse_y < menu_y2) {
            if (mouse_x >= menu_x1 && mouse_x <= menu_x2 && mouse_y >= menu_x1 && mouse_y <= menu_x1 + 90) {
                eCompleto = false;  # Restablecer el estado de entrenamiento
                datosEntrenamiento = [];  # Limpiar datos de entrenamiento
                modoAuto = false;  # Desactivar modo automático
            } else if (mouse_x >= menu_x1 y mouse_x <= menu_x2 y mouse_y >= menu_x1 + 90 y mouse_y <= menu_y2) {
                if (!eCompleto) {
                    enRedNeural();  # Entrenar la red neuronal
                    eCompleto = true;  # Marcar el entrenamiento como completo
                }
                modoAuto = true;  # Activar modo automático
            }

            menu.destroy();  # Eliminar el menú de pausa
            resetVariables();  # Restablecer variables
            juego.paused = false;  # Reanudar el juego
        }
    }
}
```

## Función `resetVariables`
Esta función reinicia las variables para volver a la posición y velocidad iniciales. 

```javascript
function resetVariables() {
    jugador.body.velocity.x = 0;
    jugador.body.velocity.y = 0;


    balaD = false;
    bala.position.setTo(w - 100, h);
}
```

## Función `velocidadRandom`
Genera una velocidad aleatoria dentro de un rango determinado.

```javascript
function velocidadRandom(min, max) {
    return Math.random() * (max - min) + min;
}
```
```

