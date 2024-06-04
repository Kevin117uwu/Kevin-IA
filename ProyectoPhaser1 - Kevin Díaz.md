# Documentación de Código en js
# Documentación del Juego con Phaser y Red Neuronal

## Variables Globales: 
El juego contiene varias variables globales que controlan diferentes aspectos, como el tamaño del juego, personajes, elementos, redes neuronales, entre otros.

```javascript
var w = 800;  // Anchura del juego
var h = 400;  // Altura del juego
var jugador;  // Personaje principal
var fondo;  // Fondo del juego

var bala, balaD = false, nave, nave2, bala2, bala2D = false;  // Balas y naves

var salto;  // Control de salto
var desplazo;  // Control de desplazamiento
var menu;  // Menú

var velocidadBala;  // Velocidad de la bala
var despBala;  // Desplazamiento de la bala
var estatusAire;  // Estado del personaje en el aire

var velocidadBala2;  // Velocidad de la segunda bala
var despBala2;  // Desplazamiento de la segunda bala
var estatusDesp;  // Estado del desplazamiento

var nnNetwork, nnEntrenamiento, nnSalida, datosEntrenamiento = [];  // Red neuronal y datos de entrenamiento
var modoAuto = false, eCompleto = false;  // Modo automático y estado de entrenamiento completo
var data_phaser = "";  // Datos para exportar
```

## Inicialización del Juego: 
Se realiza creando una instancia de Phaser con las dimensiones definidas. Aquí se define el tipo de renderizado, los eventos, y se asocian las funciones `preload`, `create`, `update`, `render`, etc.

```javascript
var juego = new Phaser.Game(w, h, Phaser.CANVAS, '', { preload: preload, create: create, update: update, render: render });
```

## Función `preload` 
En esta función, se cargan los recursos necesarios para el juego, como imágenes y sprites. Se define qué archivos se utilizarán para el fondo, personajes y elementos del juego:

```javascript
function preload() {
    juego.load.image('fondo', 'assets/game/fondo3.jpg');
    juego.load.spritesheet('mono', 'assets/sprites/goku.png', 38, 63);
    juego.load.image('nave', 'assets/game/nave2.png');
    juego.load.image('bala', 'assets/sprites/purple_ball.png');
    juego.load.image('menu', 'assets/game/menu.png');
}
```

## Función `create`
En la función `create`, se inicializa el sistema de física y se establecen las propiedades iniciales del juego, como la gravedad y los objetos presentes en el mismo.

```javascript
function create() {
    // Configuración del sistema de física
    juego.physics.startSystem(Phaser.Physics.ARCADE);
    // juego.physics.arcade.gravity.y = 800;
    juego.time.desiredFps = 30;

    // Agregar elementos al juego
    fondo = juego.add.tileSprite(0, 0, w, h, 'fondo');
    nave = juego.add.sprite(w - 115, h - 65, 'nave');
    bala = juego.add.sprite(w - 100, h, 'bala');
    nave2 = juego.add.sprite(5, 5, 'nave');
    bala2 = juego.add.sprite(50, 0, 'bala');
    jugador = juego.add.sprite(50, h, 'mono');

    // Configurar la física para el jugador
    juego.physics.enable(jugador);
    jugador.body.collideWorldBounds = true;
    jugador.body.gravity.y = 500;
    var corre = jugador.animations.add('corre', [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11]);
    jugador.animations.play('corre', 10, true);

    // Configurar la física para la bala
    juego.physics.enable(bala);
    bala.body.collideWorldBounds = true;

    // Configurar la física para la segunda bala
    juego.physics.enable(bala2);
    bala2.body.collideWorldBounds = true;

    // Configurar la opción de pausa
    pausaL = juego.add.text(w - 100, 20, 'Pausa', { font: '20px Arial', fill: '#fff' });
    pausaL.inputEnabled = true;
    pausaL.events.onInputUp.add(pausa, self);
    juego.input.onDown.add(mPausa, self);

    // Configurar el salto y desplazamiento del personaje
    salto = juego.input.keyboard.addKey(Phaser.Keyboard.UP);
    desplazo = juego.input.keyboard.addKey(Phaser.Keyboard.RIGHT);

    // Configuración de la red neuronal
    nnNetwork = new synaptic.Architect.Perceptron(4, 8, 4, 2);
    nnEntrenamiento = new synaptic.Trainer(nnNetwork);
}
```
## Función `enRedNeuronal`
En la función `enRedNeuronal`, se entrena la red neuronal con los datos de entrenamiento recopilados.

```javascript
function enRedNeural() {
    nnEntrenamiento.train(datosEntrenamiento, { rate: 0.0003, iterations: 30000, shuffle: true });
}
```

## Función `datosDeEntrenamiento`
En esta función, se utiliza la red neuronal para predecir acciones basadas en la entrada de parámetros.

```javascript
function datosDeEntrenamiento(param_entrada) {
    console.log("Entrada", param_entrada[0] + " " + param_entrada[1] + ' ' + param_entrada[2] + " " + param_entrada[3]);
    nnSalida = nnNetwork.activate(param_entrada);
    var aire = Math.round(nnSalida[0] * 100);
    var desp = Math.round(nnSalida[1] * 100);
    console.log("Valor ", "En el Aire %: " + aire + " En desplazamiento %: " + desp);
    var status = [false, false];
    if (aire > 50)
        status[0] = true;
    if (desp > 50)
        status[1] = true;
    return status;
}
```

## Función `pausa` y `mPausa`
Estas funciones manejan la pausa del juego y el menú asociado. Se define la lógica para detener el juego y reiniciar el modo automático y el entrenamiento de la red neuronal.

```javascript
function pausa() {
    juego.paused = true;  // Detener el juego
    menu = juego.add.sprite(w / 2, h / 2, 'menu');  // Crear el menú de pausa
    menu.anchor.setTo(0.5, 0.5);  // Centrar el menú
}

function mPausa(event) {
    if (juego.paused) {
        // Coordenadas del menú
        var menu_x1 = w / 2 - 270 / 2, menu_x2 = w / 2 + 270 / 2,
            menu_y1 = h / 2 - 180 / 2, menu_y2 = h / 2 + 180 / 2;

        // Coordenadas del ratón
        var mouse_x = event.x,
            mouse_y = event.y;

        // Verificar si el ratón está dentro del menú para seleccionar opciones
        if (mouse_x > menu_x1 && mouse_x < menu_x2 && mouse_y > menu_y1 && mouse_y < menu_y2) {
            if (mouse_x >= menu_x1 && mouse_x <= menu_x2 && mouse_y >= menu_x1 && mouse_y <= menu_y1 + 90) {
                if (eCompleto)
                    datosEntrenamiento = [];
                else
                    for (var auxiliar = 0; auxiliar < 20; auxiliar++)
                        datosEntrenamiento.pop();
                modoAuto = false;
                eCompleto = false;
            } else if (mouse_x >= menu_x1 && mouse_x <= menu_x2 && mouse_y >= menu_y1 + 90 && mouse_y <= menu_y2) {
                if (!eCompleto) {
                    enRedNeural();  // Entrenar la red neuronal
                    eCompleto = true;  // Marcar el entrenamiento como completo
                }
                modoAuto = true;  // Activar modo automático
            }

            menu.destroy();  // Eliminar el menú de pausa
            resetVariables();  // Restablecer variables
            resetVariablesB2();  // Restablecer variables de la segunda bala
            jugador.position.x = 50;
            juego.paused = false;  // Reanudar el juego
        }
    }
}
```

## Función `resetVariables` y `resetVariablesB2`
Estas funciones reinician las variables para volver a la posición y velocidad iniciales.

```javascript
function resetVariables() {
    jugador.body.velocity.x = 0;
    jugador.body.velocity.y = 0;
    bala.body.velocity.x = 0;
    bala.position.x = w - 100;
    balaD = false;
}

function resetVariablesB2() {
    jugador.body.velocity.x = 0;
    jugador.body.velocity.y = 0;
    bala2.body.velocity.y = 0;
    bala2.position.y = 0;
    bala2D = false;
}
```

## Función `saltar` y `desplazarDer`
Controlan el salto y el desplazamiento del personaje.

```javascript
function saltar() {
    jugador.body.velocity.y = -270;
}

function desplazarDer() {
    jugador.body.velocity.x = 150;
}
```

## Función `update`
Es el núcleo del ciclo del juego, donde se actualizan todas las variables y se maneja la lógica del juego en cada frame.

```javascript
function update() {
     fondo.tilePosition.x -= 1; 

    juego.physics.arcade.collide(bala, jugador, colisionH, null, this);
    juego.physics.arcade.collide(bala2, jugador, colisionH, null, this);

    estatusAire = 0;
    estatusDesp = 0;

    if(!jugador.body.onFloor())
        estatusAire = 1;
    
    if(jugador.position.x != 50)
        estatusDesp = 1;
	
    despBala = Math.floor( jugador.position.x - bala.position.x );
    despBala2 = Math.floor(jugador.position.y - bala2.position.y);

    if(!modoAuto){
        if( salto.isDown &&  jugador.body.onFloor() )
            saltar();
        
        if( desplazo.isDown )
            desplazarDer();
        
        if( !desplazo.isDown && jugador.position.x != 50 )
            reset();
    }
    

    if( modoAuto  && (bala.position.x>0 || bala2.position.y>0)) {
        var resultConsulta = datosDeEntrenamiento( [despBala , velocidadBala, despBala2, velocidadBala2] );
        if( resultConsulta[0] && jugador.body.onFloor()){
            saltar();
        }

        if(resultConsulta[1])
            desplazarDer();
        else if(jugador.position.x != 50)
            reset();
    }

    if( balaD==false ){
        disparo();
    }

    if( bala2D==false ){
        disparoVert();
    }

    if( bala.position.x <= 0  ){
        resetVariables();
    }
    
    if( bala2.body.onFloor()  ){
        resetVariablesB2();
        
    }

    if( modoAuto ==false  && (bala.position.x>0 || bala2.position.y>0) ){

        datosEntrenamiento.push({
                'input' :  [despBala , velocidadBala, despBala2, velocidadBala2],
                'output':  [estatusAire, estatusDesp]  
        });

        //console.log("Desplazamiento Bala, Velocidad Bala, Estatus, Estatus: ",
        //   despBala + " " +velocidadBala + " "+ estatusAire+" "+  estatuSuelo);
        //dataset.push([despBala, velocidadBala, despBala2, velocidadBala2, estatusAire, estatusDesp]);
        console.log(despBala + ' ' + velocidadBala + ' ' + despBala2 + ' ' + velocidadBala2 + ' ' + estatusAire + ' ' + estatusDesp);
        data_phaser += despBala + ' ' + velocidadBala + ' ' + despBala2 + ' ' + velocidadBala2 + ' ' + estatusAire + ' ' + estatusDesp + "\n";
   }

}


function disparo(){
    velocidadBala =  -1 * velocidadRandom(150,160);
    bala.body.velocity.y = 0 ;
    bala.body.velocity.x = velocidadBala ;
    balaD=true;
}

function disparoVert(){
    velocidadBala2 =  velocidadRandom(50,300);
    bala2.body.velocity.x = 0 ;
    bala2.body.velocity.y = velocidadBala2;
    bala2D=true;
}

function colisionH(){
    pausa();
}

function velocidadRandom(min, max) {
    return Math.floor(Math.random() * (max - min + 1)) + min;
}

```

## Función `render`
Función que permite la representación visual en pantalla de elementos del juego.

```javascript
function render() {}
```

## Exportación de Datos de Phaser
Proporciona un enlace para descargar los datos de entrenamiento utilizados por el juego.

```javascript
document.getElementById("export").onclick = function () {
    this.href = 'data:plain/text,' + JSON.stringify(datosEntrenamiento);
    this.download = "dataset.json";
};
```


