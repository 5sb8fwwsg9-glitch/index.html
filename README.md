<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">

<title>JARVIS VISION</title>

<style>
*{box-sizing:border-box}

html,body{
    margin:0;
    width:100%;
    height:100%;
    overflow:hidden;
    background:#000;
    font-family:monospace;
    color:#00d9ff;
}

#camera{
    position:fixed;
    inset:0;
    width:100%;
    height:100%;
    object-fit:cover;
    filter:
        contrast(1.25)
        brightness(.75)
        saturate(.7)
        hue-rotate(150deg);
}

#hud{
    position:fixed;
    inset:0;
    pointer-events:none;
}

#title{
    position:absolute;
    top:18px;
    left:50%;
    transform:translateX(-50%);
    text-align:center;
    text-shadow:0 0 12px #00bfff;
}

#title b{
    font-size:25px;
    letter-spacing:5px;
}

#title small{
    display:block;
    margin-top:4px;
    opacity:.7;
    letter-spacing:2px;
}

.corner{
    position:absolute;
    width:60px;
    height:60px;
    border-color:#00cfff;
    filter:drop-shadow(0 0 8px #00cfff);
}

.tl{
    top:15px;
    left:15px;
    border-top:2px solid;
    border-left:2px solid;
}

.tr{
    top:15px;
    right:15px;
    border-top:2px solid;
    border-right:2px solid;
}

.bl{
    bottom:15px;
    left:15px;
    border-bottom:2px solid;
    border-left:2px solid;
}

.br{
    bottom:15px;
    right:15px;
    border-bottom:2px solid;
    border-right:2px solid;
}

#scan{
    position:absolute;
    width:100%;
    height:2px;
    background:#00d9ff;
    opacity:.3;
    box-shadow:0 0 15px #00d9ff;
    animation:scan 4s linear infinite;
}

@keyframes scan{
    from{top:0}
    to{top:100%}
}

#objects{
    position:absolute;
    top:85px;
    right:12px;
    width:190px;
    max-height:50%;
    overflow:auto;
}

.object{
    margin-bottom:7px;
    padding:8px;
    background:rgba(0,20,40,.75);
    border:1px solid rgba(0,210,255,.7);
    box-shadow:0 0 10px rgba(0,180,255,.25);
    font-size:11px;
    text-shadow:0 0 5px #00bfff;
}

#bottom{
    position:absolute;
    bottom:20px;
    left:20px;
    right:20px;
    display:flex;
    justify-content:space-between;
    align-items:end;
}

#jarvis{
    padding:10px 14px;
    background:rgba(0,20,35,.7);
    border-left:3px solid #00d9ff;
    max-width:270px;
}

#jarvis b{
    font-size:13px;
}

#message{
    margin-top:5px;
    font-size:11px;
}

#status{
    text-align:right;
    font-size:11px;
    line-height:1.7;
    text-shadow:0 0 8px #00cfff;
}

button{
    pointer-events:auto;
    margin-top:8px;
    padding:10px 14px;
    background:rgba(0,40,70,.8);
    color:#00d9ff;
    border:1px solid #00d9ff;
    font-family:monospace;
    border-radius:5px;
}

#start{
    position:fixed;
    z-index:20;
    left:50%;
    top:50%;
    transform:translate(-50%,-50%);
    padding:18px 25px;
    font-size:16px;
    background:rgba(0,15,30,.95);
    box-shadow:0 0 25px #008cff;
}

@media(max-width:600px){
    #title b{font-size:18px}
    #objects{width:165px}
    #bottom{left:12px;right:12px}
}
</style>
</head>

<body>

<video id="camera" autoplay playsinline muted></video>

<div id="hud">

<div id="scan"></div>

<div class="corner tl"></div>
<div class="corner tr"></div>
<div class="corner bl"></div>
<div class="corner br"></div>

<div id="title">
    <b>J.A.R.V.I.S.</b>
    <small>VISION SYSTEM</small>
</div>

<div id="objects"></div>

<div id="bottom">

<div id="jarvis">
    <b>JARVIS</b>
    <div id="message">
        Sistema preparado.
    </div>

    <button onclick="voice()">
        🎙 HABLAR CON JARVIS
    </button>
</div>

<div id="status">
    SISTEMA: ONLINE<br>
    CÁMARA: <span id="camStatus">OFF</span><br>
    OBJETOS: <span id="count">0</span><br>
    VOZ: <span id="voiceStatus">OFF</span>
</div>

</div>
</div>

<button id="start">
    ACTIVAR CÁMARA
</button>


<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>

<script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/coco-ssd"></script>


<script>

let video =
    document.getElementById("camera");

let model;

let active=false;

let lastObjects=[];


// ======================================
// ACTIVAR CÁMARA
// ======================================

async function startCamera(){

    try{

        const stream =
            await navigator.mediaDevices
            .getUserMedia({

                video:{
                    facingMode:{
                        ideal:"environment"
                    },

                    width:{
                        ideal:1280
                    },

                    height:{
                        ideal:720
                    }
                },

                audio:false
            });

        video.srcObject=stream;

        await video.play();

        document.getElementById("start")
        .style.display="none";

        document.getElementById("camStatus")
        .textContent="ON";

        active=true;

        jarvisSay(
            "Cámara activada. "
            +"Sistema de visión operativo."
        );

        model =
            await cocoSsd.load();

        detect();

    }

    catch(error){

        alert(
            "No se pudo acceder a la cámara. "
            +"Debes permitir el acceso a la cámara."
        );

        console.log(error);
    }
}


// ======================================
// DETECCIÓN
// ======================================

async function detect(){

    if(!active || !model){

        requestAnimationFrame(detect);

        return;
    }

    try{

        const predictions =
            await model.detect(video);

        lastObjects =
            predictions;

        showObjects(
            predictions
        );

    }

    catch(error){

        console.log(error);
    }

    setTimeout(
        detect,
        300
    );
}


// ======================================
// MOSTRAR OBJETOS
// ======================================

function showObjects(objects){

    const container =
        document.getElementById(
            "objects"
        );

    container.innerHTML="";

    document.getElementById(
        "count"
    ).textContent=
        objects.length;


    objects.forEach(
        function(obj){

            let name =
                translate(obj.class);

            let width =
                obj.bbox[2];

            let height =
                obj.bbox[3];


            /*
             * ESTIMACIÓN DE DISTANCIA
             *
             * No es una medición real.
             * Depende del tamaño esperado
             * del objeto.
             */

            let distance =
                estimateDistance(
                    obj.class,
                    height
                );


            let estimatedHeight =
                estimateHeight(
                    obj.class,
                    height,
                    distance
                );


            let div =
                document.createElement(
                    "div"
                );


            div.className="object";


            div.innerHTML=

                "<b>"
                +name.toUpperCase()
                +"</b><br>"

                +"CONFIANZA: "
                +(obj.score*100)
                .toFixed(0)
                +"%<br>"

                +"DISTANCIA: "
                +distance
                +" m<br>"

                +"ALTURA EST.: "
                +estimatedHeight
                +" m";


            container.appendChild(
                div
            );

        }
    );
}


// ======================================
// ALTURAS DE REFERENCIA
// ======================================

const heights={

    person:1.70,

    car:1.50,

    bicycle:1.05,

    motorcycle:1.10,

    bus:3.20,

    truck:2.50,

    chair:.90,

    table:.75,

    bottle:.25,

    cup:.12,

    laptop:.25,

    tv:.70,

    backpack:.45,

    suitcase:.70,

    dog:.60,

    cat:.30

};


// ======================================
// DISTANCIA APROXIMADA
// ======================================

function estimateDistance(
    object,
    pixelHeight
){

    let realHeight =
        heights[object] || .5;

    if(pixelHeight<=0)
        return 0;

    /*
     * Focal aproximada.
     */

    let focal=700;

    let distance =
        realHeight*focal/
        pixelHeight;


    distance =
        Math.max(
            .2,
            Math.min(
                20,
                distance
            )
        );


    return distance.toFixed(1);
}


// ======================================
// ALTURA APROXIMADA
// ======================================

function estimateHeight(
    object,
    pixelHeight,
    distance
){

    let focal=700;

    let h =
        pixelHeight*
        parseFloat(distance)/
        focal;


    let reference =
        heights[object] || .5;


    /*
     * Limitamos el resultado
     * para evitar valores absurdos.
     */

    h =
        Math.max(
            .03,
            Math.min(
                4,
                h
            )
        );


    return h.toFixed(2);
}


// ======================================
// TRADUCIR NOMBRES
// ======================================

function translate(name){

    const names={

        person:"persona",

        bicycle:"bicicleta",

        car:"auto",

        motorcycle:"motocicleta",

        airplane:"avión",

        bus:"autobús",

        train:"tren",

        truck:"camión",

        boat:"barco",

        traffic_light:
            "semáforo",

        fire_hydrant:
            "hidrante",

        stop_sign:
            "señal de stop",

        parking_meter:
            "parquímetro",

        bench:"banco",

        bird:"pájaro",

        cat:"gato",

        dog:"perro",

        horse:"caballo",

        sheep:"oveja",

        cow:"vaca",

        elephant:"elefante",

        bear:"oso",

        zebra:"cebra",

        giraffe:"jirafa",

        backpack:"mochila",

        umbrella:"paraguas",

        handbag:"bolso",

        suitcase:"maleta",

        frisbee:"frisbee",

        skis:"esquís",

        snowboard:
            "snowboard",

        sports_ball:
            "pelota",

        kite:"cometa",

        baseball_bat:
            "bate",

        baseball_glove:
            "guante",

        skateboard:
            "skateboard",

        surfboard:
            "tabla de surf",

        bottle:"botella",

        wine_glass:
            "copa",

        cup:"taza",

        fork:"tenedor",

        knife:"cuchillo",

        spoon:"cuchara",

        bowl:"tazón",

        banana:"banana",

        apple:"manzana",

        sandwich:"sándwich",

        orange:"naranja",

        broccoli:"brócoli",

        carrot:"zanahoria",

        pizza:"pizza",

        donut:"donut",

        cake:"pastel",

        chair:"silla",

        couch:"sofá",

        potted_plant:
            "planta",

        bed:"cama",

        dining_table:
            "mesa",

        toilet:"inodoro",

        tv:"televisor",

        laptop:"computadora",

        mouse:"ratón",

        remote:"control",

        keyboard:"teclado",

        cell_phone:
            "teléfono",

        microwave:
            "microondas",

        oven:"horno",

        toaster:"tostadora",

        sink:"fregadero",

        refrigerator:
            "refrigerador",

        book:"libro",

        clock:"reloj",

        vase:"florero",

        scissors:"tijeras",

        teddy_bear:
            "oso de peluche",

        hair_drier:
            "secador",

        toothbrush:
            "cepillo de dientes"
    };


    return names[name] || name;
}


// ======================================
// VOZ
// ======================================

function voice(){

    const Recognition =
        window.SpeechRecognition ||
        window.webkitSpeechRecognition;


    if(!Recognition){

        jarvisSay(
            "El reconocimiento de voz "
            +"no está disponible "
            +"en este navegador."
        );

        return;
    }


    const recognition =
        new Recognition();


    recognition.lang="es-ES";

    recognition.continuous=false;

    recognition.interimResults=false;


    document.getElementById(
        "voiceStatus"
    ).textContent=
        "ESCUCHANDO";


    jarvisSay(
        "Te escucho."
    );


    recognition.onresult =
        function(event){

            let command =
                event.results[0][0]
                .transcript
                .toLowerCase();


            document.getElementById(
                "voiceStatus"
            ).textContent=
                "ON";


            processCommand(
                command
            );
        };


    recognition.onerror =
        function(){

            document.getElementById(
                "voiceStatus"
            ).textContent=
                "OFF";
        };


    recognition.start();
}


// ======================================
// COMANDOS
// ======================================

function processCommand(command){

    if(
        command.includes("qué ves") ||
        command.includes("que ves") ||
        command.includes("objetos")
    ){

        describeObjects();

        return;
    }


    if(
        command.includes("cuántos") ||
        command.includes("cuantos")
    ){

        jarvisSay(
            "Actualmente detecto "
            +lastObjects.length
            +" objetos."
        );

        return;
    }


    if(
        command.includes("hola") ||
        command.includes("jarvis")
    ){

        jarvisSay(
            "Hola. Todos los sistemas "
            +"funcionan correctamente."
        );

        return;
    }


    jarvisSay(
        "He recibido tu comando."
    );
}


// ======================================
// DESCRIBIR OBJETOS
// ======================================

function describeObjects(){

    if(lastObjects.length===0){

        jarvisSay(
            "No detecto objetos."
        );

        return;
    }


    let text =
        "Detecto "
        +lastObjects.length
        +" objetos. ";


    lastObjects
    .slice(0,5)
    .forEach(
        function(obj){

            text +=
                translate(obj.class)
                +". ";

        }
    );


    jarvisSay(text);
}


// ======================================
// VOZ DE JARVIS
// ======================================

function jarvisSay(text){

    document.getElementById(
        "message"
    ).textContent=
        text;


    if(
        !("speechSynthesis"
          in window)
    ){
        return;
    }


    speechSynthesis.cancel();


    const speech =
        new SpeechSynthesisUtterance(
            text
        );


    speech.lang="es-ES";

    speech.rate=.92;

    speech.pitch=.75;

    speech.volume=1;


    speechSynthesis.speak(
        speech
    );
}


// ======================================
// INICIO
// ======================================

document.getElementById(
    "start"
).onclick=
    startCamera;

</script>

</body>
</html># index.html