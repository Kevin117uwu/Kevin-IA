## Variables globales del juego

```javascript
var w = 800;  // Ancho del juego
var h = 400;  // Alto del juego
var jugador;  // Sprite del jugador
var fondo;    // Fondo del juego
var bala;     // Sprite de la bala
var VOLVIENDOV = false;  // Indicador de retorno vertical
var VOLVIENDOH = false;  // Indicador de retorno horizontal
var cursors;  // Teclas de dirección
var menu;     // Menú de pausa
var estatusIzquierda, estatusDerecha, estatusArriba, estatusAbajo, estatusMovimiento;  // Estados de movimiento
var nnNetwork, nnEntrenamiento, nnSalida, datosEntrenamiento = [];  // Red neuronal y datos de entrenamiento
var modoAuto = false, eCompleto = false;  // Modo automático y estado de entrenamiento completado
var JX = 200;  // Posición X inicial del jugador
var JY = 200;  // Posición Y inicial del jugador
var autoMode = false;  // Modo automático
var juego = new Phaser.Game(w, h, Phaser.CANVAS, '', { preload: preload, create: create, update: update, render: render });  // Inicialización del juego
```

## Función de carga de recursos
```javascript
function preload() {
    juego.load.image('fondo', 'assets/game/fondo3.jpg');
    juego.load.spritesheet('mono', 'assets/sprites/goku.png', 38, 63);
    juego.load.image('menu', 'assets/game/menu.png');
    juego.load.image('bala', 'assets/sprites/purple_ball.png');
}
```

## Función de creación del juego
```javascript
function create() {
    juego.physics.startSystem(Phaser.Physics.ARCADE); 
    juego.time.desiredFps = 30;
    juego.physics.arcade.gravity.y = 0; // Sin gravedad

    fondo = juego.add.tileSprite(0, 0, w, h, 'fondo');
    jugador = juego.add.sprite(w / 2, h / 2, 'mono');

    juego.physics.enable(jugador);
    jugador.body.collideWorldBounds = true; // El jugador no sale de los límites
    var corre = jugador.animations.add('corre', [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]); 
    jugador.animations.play('corre', 10, true); 

    // Bala en la esquina superior izquierda
    bala = juego.add.sprite(0, 0, 'bala');
    juego.physics.enable(bala);
    bala.body.collideWorldBounds = true; // La bala no sale de los límites
    bala.body.bounce.set(1); // Bouncing de la bala
    setRandomBalaVelocity(); // Velocidad de la bala aleatoria

    // Pausa
    pausaL = juego.add.text(w - 100, 20, 'Pausa', { font: '20px Arial', fill: '#fff' });
    pausaL.inputEnabled = true;
    pausaL.events.onInputUp.add(pausa, self);
    juego.input.onDown.add(mPausa, self);

    // Cursores
    cursors = juego.input.keyboard.createCursorKeys();
    
    nnNetwork = new synaptic.Architect.Perceptron(3, 6, 6, 6, 5);
    nnEntrenamiento = new synaptic.Trainer(nnNetwork);
}
```

## Función de entrenamiento de la red neuronal

```javascript
function enRedNeural() {
    nnEntrenamiento.train(datosEntrenamiento, { rate: 0.0003, iterations: 10000, shuffle: true });
}
```

## Funciones de procesamiento de datos

```javascript
function datosVertical(param_entrada) {
    console.log("Entrada", param_entrada.join(" "));
    nnSalida = nnNetwork.activate(param_entrada);
    var Izq = Math.round(nnSalida[0] * 100);
    var Der = Math.round(nnSalida[1] * 100);
    var Arr = Math.round(nnSalida[2] * 100);
    var Aba = Math.round(nnSalida[3] * 100);
    var xde = Math.round(nnSalida[4] * 100);

    if (param_entrada[2] < 80) {
        if (Arr > 40 && Arr < 60) {
            return false;    
        }
    }
    
    console.log(`\n En la estatusArriba %: ${nnSalida[2] * 100}\n En la estatusAbajo %: ${nnSalida[3] * 100}\n En la estatusIzq %: ${nnSalida[0] * 100}\n En la estatusDer %: ${nnSalida[1] * 100}\n En movimiento %: ${nnSalida[4] * 100}`);
}

function datosHorizontal(param_entrada) {
    console.log("Entrada", param_entrada.join(" "));
    nnSalida = nnNetwork.activate(param_entrada);
    var Izq = Math.round(nnSalida[0] * 100);
    var Der = Math.round(nnSalida[1] * 100);
    var Arr = Math.round(nnSalida[2] * 100);
    var Aba = Math.round(nnSalida[3] * 100);
    var xde = Math.round(nnSalida[4] * 100);

    if (param_entrada[2] < 80) {
        if (Der > 40 && Der < 60) {
            return false;    
        }
    }
    
    console.log(`\n En la estatusArriba %: ${nnSalida[2] * 100}\n En la estatusAbajo %: ${nnSalida[3] * 100}\n En la estatusIzq %: ${nnSalida[0] * 100}\n En la estatusDer %: ${nnSalida[1] * 100}\n En movimiento %: ${nnSalida[4] * 100}`);
    console.log("OUTPUTS: " + (nnSalida[2] >= nnSalida[3]));
    return nnSalida[0] >= nnSalida[1];
}

function datosMovimiento(param_entrada) {
    console.log("Entrada", param_entrada.join(" "));
    nnSalida = nnNetwork.activate(param_entrada);
    var Izq = Math.round(nnSalida[0] * 100);
    var Der = Math.round(nnSalida[1] * 100);
    var Arr = Math.round(nnSalida[2] * 100);
    var Aba = Math.round(nnSalida[3] * 100);
    var xde = Math.round(nnSalida[4] * 100);

    if (param_entrada[2] < 80) {
        if (Der > 40 && Der < 60) {
            return false;    
        }
    }
    
    console.log(`\n En la estatusArriba %: ${nnSalida[2] * 100}\n En la estatusAbajo %: ${nnSalida[3] * 100}\n En la estatusIzq %: ${nnSalida[0] * 100}\n En la estatusDer %: ${nnSalida[1] * 100}\n La disque desta %: ${nnSalida[4] * 100}`);
    console.log("OUTPUTS: " + (nnSalida[2] >= nnSalida[3]));
    return nnSalida[4] * 100 >= 20;
}
```

## Función para pausar el juego
```javascript
function pausa() {
    juego.paused = true;  // Pausar el juego
    menu = juego.add.sprite(w / 2, h / 2, 'menu');  // Añadir menú de pausa
    menu.anchor.setTo(0.5, 0.5);
}
```

## Función para manejar la pausa del juego

```javascript
function mPausa(event) {
    if (juego.paused) {
        var menu_x1 = w / 2 - 270 / 2, menu_x2 = w / 2 + 270 / 2,
            menu_y1 = h / 2 - 180 / 2, menu_y2 = h / 2 + 180 / 2;

        var mouse_x = event.x, mouse_y = event.y;

        if (mouse_x > menu_x1 && mouse_x < menu_x2 && mouse_y > menu_y1 && mouse_y < menu_y2) {
            if (mouse_x >= menu_x1 && mouse_x <= menu_x2 && mouse_y >= menu_y1 && mouse_y <= menu_y1 + 90) {
                eCompleto = false;
                datosEntrenamiento = [];
                modoAuto = false;
            } else if (mouse_x >= menu_x1 && mouse_x <= menu_x2 && mouse_y >= menu_y1 + 90 && mouse_y <= menu_y2) {
                if (!eCompleto) {
                    console.log("", "Entrenamiento " + datosEntrenamiento.length + " valores");
                    enRedNeural();
                    eCompleto = true;
                }
                modoAuto = true;
            }
            menu.destroy();
            resetGame();  // Resetear el juego
            juego.paused = false;
        }
    }
}
```

## Función para resetear el juego

```javascript
function resetGame() {
    // Resetear la posición y velocidad del jugador
    jugador.x = w / 2;
    jugador.y = h / 2;
    jugador.body.velocity.setTo(0, 0);
    
    // Resetear la posición y velocidad de la bala
    bala.x = 0;
    bala.y = 0;
    setRandomBalaVelocity();
}
```

## Función para establecer una velocidad aleatoria para la bala

```javascript
function setRandomBalaVelocity() {
    var min = 200;
    var max = 250;
    var velocityX = (Math.random() * (max - min) + min) * (Math.random() < 0.5 ? 1 : -1);
    var velocityY = (Math.random() * (max - min) + min) * (Math.random() < 0.5 ? 1 : -1);
    bala.body.velocity.setTo(velocityX, velocityY);
}
```

## Función de actualización del juego

```javascript
function update() {
    fondo.tilePosition.x += 0.5;  // Movimiento del fondo

    if (!modoAuto) {
        controlManual();  // Control manual
    } else {
        controlAutomatico();  // Control automático
    }

    if (cursors.left.isDown) {  // Movimiento manual a la izquierda
        jugador.body.velocity.x = -200;
    } else if (cursors.right.isDown) {  // Movimiento manual a la derecha
        jugador.body.velocity.x = 200;
    } else {  // Sin movimiento horizontal
        jugador.body.velocity.x = 0;
    }
    
    if (cursors.up.isDown) {  // Movimiento manual hacia arriba
        jugador.body.velocity.y = -200;
    } else if (cursors.down.isDown) {  // Movimiento manual hacia abajo
        jugador.body.velocity.y = 200;
    } else {  // Sin movimiento vertical
        jugador.body.velocity.y = 0;
    }

    if (bala.body.blocked.up) {
        VOLVIENDOV = true;
    } else if (bala.body.blocked.down) {
        VOLVIENDOV = false;
    }
    if (bala.body.blocked.left) {
        VOLVIENDOH = false;
    } else if (bala.body.blocked.right) {
        VOLVIENDOH = true;
    }

    if (!autoMode) {
        var nuevoMovimiento = {
            input: [
                jugador.y,
                jugador.x,
                Phaser.Math.distance(jugador.x, jugador.y, bala.x, bala.y)
            ],
            output: [0, 0, 0, 0, 0]
        };
        
        if (cursors.left.isDown) {
            nuevoMovimiento.output[0] = 1;
        } else if (cursors.right.isDown) {
            nuevoMovimiento.output[1] = 1;
        }
        
        if (cursors.up.isDown) {
            nuevoMovimiento.output[2] = 1;
        } else if (cursors.down.isDown) {
            nuevoMovimiento.output[3] = 1;
        }
        
        if (jugador.body.velocity.x !== 0 || jugador.body.velocity.y !== 0) {
            nuevoMovimiento.output[4] = 1;
        }
        
        datosEntrenamiento.push(nuevoMovimiento);
    }
}
```

## Control manual del jugador

```javascript
function controlManual() {
    if (cursors.left.isDown) {
        jugador.body.velocity.x = -200;
    } else if (cursors.right.isDown) {
        jugador.body.velocity.x = 200;
    } else {
        jugador.body.velocity.x = 0;
    }

    if (cursors.up.isDown) {
        jugador.body.velocity.y = -200;
    } else if (cursors.down.isDown) {
        jugador.body.velocity.y = 200;
    } else {
        jugador.body.velocity.y = 0;
    }
}
```

## Control automático del jugador

```javascript
function controlAutomatico() {
    if (datosVertical([jugador.x, jugador.y, Phaser.Math.distance(jugador.x, jugador.y, bala.x, bala.y)])) {
        if (datosHorizontal([jugador.x, jugador.y, Phaser.Math.distance(jugador.x, jugador.y, bala.x, bala.y)])) {
            datosMovimiento([jugador.x, jugador.y, Phaser.Math.distance(jugador.x, jugador.y, bala.x, bala.y)]);
        }
    }
}
```

## Función de renderizado del juego
```javascript
function render() {
    juego.debug.text('Estatus: ' + modoAuto, 32, 32);
}
```


