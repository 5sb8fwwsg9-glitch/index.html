<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport"
      content="width=device-width, initial-scale=1.0,
               maximum-scale=1.0, user-scalable=no">

<title>JARVIS VISION</title>

<style>

*{
    box-sizing:border-box;
}

html,body{
    margin:0;
    width:100%;
    height:100%;
    overflow:hidden;
    background:#000;
    font-family:monospace;
    color:#00d9ff;
}


/* =====================================================
   CÁMARA
===================================================== */

#camera{
    position:fixed;
    inset:0;

    width:100%;
    height:100%;

    object-fit:cover;

    /*
       Azul futurista SIN invertir los colores.
    */

    filter:
        grayscale(100%)
        contrast(125%)
        brightness(85%)
        sepia(100%)
        hue-rotate(165deg)
        saturate(500%);
}


/* =====================================================
   CANVAS DE DETECCIÓN
===================================================== */

#detectionCanvas{
    position:fixed;
    inset:0;

    width:100%;
    height:100%;

    pointer-events:none;
}


/* =====================================================
   HUD
===================================================== */

#hud{
    position:fixed;
    inset:0;

    pointer-events:none;
}


/* =====================================================
   TÍTULO
===================================================== */

#title{
    position:absolute;

    top:18px;
    left:50%;

    transform:translateX(-50%);

    text-align:center;

    text-shadow:
        0 0 8px #00c8ff,
        0 0 20px #008cff;
}

#title b{
    font-size:24px;
    letter-spacing:5px;
}

#title small{
    display:block;

    margin-top:4px;

    font-size:10px;

    letter-spacing:3px;

    opacity:.8;
}


/* =====================================================
   ESQUINAS HUD
===================================================== */

.corner{
    position:absolute;

    width:65px;
    height:65px;

    border-color:#00d9ff;

    filter:
        drop-shadow(
            0 0 8px #00c8ff
        );
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


/* =====================================================
   ESCÁNER
===================================================== */

#scan{
    position:absolute;

    left:0;

    width:100%;
    height:2px;

    background:#00d9ff;

    opacity:.25;

    box-shadow:
        0 0 15px #00d9ff;

    animation:
        scan 4s linear infinite;
}

@keyframes scan{

    from{
        top:0;
    }

    to{
        top:100%;
    }

}


/* =====================================================
   PANEL JARVIS
===================================================== */

#jarvisPanel{

    position:absolute;

    left:18px;
    bottom:18px;

    width:280px;

    padding:12px;

    background:
        rgba(0,15,30,.78);

    border-left:
        3px solid #00d9ff;

    box-shadow:
        0 0 20px
        rgba(0,180,255,.25);

    backdrop-filter:
        blur(5px);
}

#jarvisTitle{

    font-size:14px;

    font-weight:bold;

    letter-spacing:2px;
}

#message{

    margin-top:6px;

    font-size:11px;

    line-height:1.4;

    min-height:30px;
}


/* =====================================================
   ESTADO
===================================================== */

#status{

    position:absolute;

    right:18px;
    bottom:18px;

    text-align:right;

    font-size:11px;

    line-height:1.7;

    text-shadow:
        0 0 8px #00c8ff;
}


/* =====================================================
   BOTÓN
===================================================== */

button{

    pointer-events:auto;

    margin-top:8px;

    padding:9px 13px;

    background:
        rgba(0,40,70,.85);

    border:
        1px solid #00d9ff;

    border-radius:5px;

    color:#00d9ff;

    font-family:monospace;

    font-size:11px;

    box-shadow:
        0 0 10px
        rgba(0,180,255,.3);
}

button:active{

    background:
        rgba(0,120,180,.7);
}


/* =====================================================
   BOTÓN ACTIVAR
===================================================== */

#start{

    position:fixed;

    z-index:50;

    left:50%;
    top:50%;

    transform:
        translate(-50%,-50%);

    padding:18px 26px;

    font-size:16px;

    background:
        rgba(0,10,25,.95);

    box-shadow:
        0 0 25px #008cff,
        0 0 60px
        rgba(0,150,255,.25);
}


/* =====================================================
   RESPONSIVE
===================================================== */

@media(max-width:600px){

    #title b{
        font-size:18px;
    }

    #jarvisPanel{

        left:10px;
        bottom:10px;

        width:230px;
    }

    #status{

        right:10px;
        bottom:10px;

        font-size:9px;
    }

}

</style>
</head>


<body>


<!-- ===================================================
     CÁMARA
=================================================== -->

<video
    id="camera"
    autoplay
    playsinline
    muted>
</video>


<!-- ===================================================
     CANVAS
=================================================== -->

<canvas
    id="detectionCanvas">
</canvas>


<!-- ===================================================
     HUD
=================================================== -->

<div id="hud">

    <div id="scan"></div>

    <div class="corner tl"></div>
    <div class="corner tr"></div>
    <div class="corner bl"></div>
    <div class="corner br"></div>


    <div id="title">

        <b>J.A.R.V.I.S.</b>

        <small>
            ADVANCED VISION SYSTEM
        </small>

    </div>


    <!-- JARVIS -->

    <div id="jarvisPanel">

        <div id="jarvisTitle">
            JARVIS
        </div>

        <div id="message">
            Sistema preparado.
        </div>

        <button
            id="voiceButton">
            🎙 HABLAR CON JARVIS
        </button>

    </div>


    <!-- ESTADO -->

    <div id="status">

        SISTEMA:
        <span id="system">
            ONLINE
        </span>

        <br>

        CÁMARA:
        <span id="cameraStatus">
            OFF
        </span>

        <br>

        IA:
        <span id="aiStatus">
            CARGANDO
        </span>

        <br>

        OBJETOS:
        <span id="objectCount">
            0
        </span>

        <br>

        VOZ:
        <span id="voiceStatus">
            OFF
        </span>

    </div>

</div>


<!-- ===================================================
     BOTÓN
=================================================== -->

<button id="start">
    ACTIVAR JARVIS
</button>


<!-- ===================================================
     TENSORFLOW
=================================================== -->

<script src=
"https://cdn.jsdelivr.net/npm/@tensorflow/tfjs">
</script>

<script src=
"https://cdn.jsdelivr.net/npm/@tensorflow-models/coco-ssd">
</script>


<script>

/* =====================================================
   VARIABLES
===================================================== */

const video =
    document.getElementById(
        "camera"
    );

const canvas =
    document.getElementById(
        "detectionCanvas"
    );

const ctx =
    canvas.getContext("2d");

const startButton =
    document.getElementById(
        "start"
    );

const message =
    document.getElementById(
        "message"
    );

const objectCount =
    document.getElementById(
        "objectCount"
    );

const aiStatus =
    document.getElementById(
        "aiStatus"
    );

const cameraStatus =
    document.getElementById(
        "cameraStatus"
    );

const voiceStatus =
    document.getElementById(
        "voiceStatus"
    );


let model = null;

let active = false;

let detections = [];

let detecting = false;


/* =====================================================
   ALTURAS DE REFERENCIA
===================================================== */

const objectHeights = {

    person:1.70,

    bicycle:1.05,

    car:1.50,

    motorcycle:1.10,

    bus:3.20,

    truck:2.50,

    chair:.90,

    couch:.80,

    table:.75,

    bottle:.25,

    cup:.12,

    laptop:.25,

    tv:.70,

    backpack:.45,

    suitcase:.70,

    dog:.60,

    cat:.30,

    horse:1.50,

    cow:1.40,

    sheep:.80,

    bird:.25,

    book:.25,

    keyboard:.04,

    cell_phone:.15,

    microwave:.35,

    refrigerator:1.70

};


/* =====================================================
   NOMBRES EN ESPAÑOL
===================================================== */

const spanish = {

    person:"persona",

    bicycle:"bicicleta",

    car:"auto",

    motorcycle:"motocicleta",

    airplane:"avión",

    bus:"autobús",

    train:"tren",

    truck:"camión",

    boat:"barco",

    traffic_light:"semáforo",

    stop_sign:"señal de stop",

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

    snowboard:"snowboard",

    sports_ball:"pelota",

    kite:"cometa",

    skateboard:"skateboard",

    bottle:"botella",

    wine_glass:"copa",

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

    potted_plant:"planta",

    bed:"cama",

    dining_table:"mesa",

    toilet:"inodoro",

    tv:"televisor",

    laptop:"computadora",

    mouse:"ratón",

    remote:"control",

    keyboard:"teclado",

    cell_phone:"teléfono",

    microwave:"microondas",

    oven:"horno",

    toaster:"tostadora",

    sink:"fregadero",

    refrigerator:"refrigerador",

    book:"libro",

    clock:"reloj",

    vase:"florero",

    scissors:"tijeras",

    teddy_bear:"oso de peluche",

    toothbrush:"cepillo de dientes"

};


/* =====================================================
   REDIMENSIONAR CANVAS
===================================================== */

function resizeCanvas(){

    canvas.width =
        video.videoWidth ||
        window.innerWidth;

    canvas.height =
        video.videoHeight ||
        window.innerHeight;
}

window.addEventListener(
    "resize",
    resizeCanvas
);


/* =====================================================
   ACTIVAR CÁMARA
===================================================== */

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


        video.srcObject =
            stream;


        await video.play();


        resizeCanvas();


        active = true;


        startButton.style.display =
            "none";


        cameraStatus.textContent =
            "ON";


        message.textContent =
            "Cámara activada. Iniciando inteligencia visual...";


        loadAI();

    }

    catch(error){

        console.error(error);

        message.textContent =
            "No se pudo acceder a la cámara.";

        alert(
            "Permite el acceso a la cámara para utilizar JARVIS."
        );
    }
}


/* =====================================================
   CARGAR IA
===================================================== */

async function loadAI(){

    try{

        aiStatus.textContent =
            "CARGANDO";


        model =
            await cocoSsd.load();


        aiStatus.textContent =
            "ONLINE";


        message.textContent =
            "Inteligencia visual activada.";


        speak(
            "Sistema de visión activado."
        );


        detectLoop();

    }

    catch(error){

        console.error(error);

        aiStatus.textContent =
            "ERROR";

        message.textContent =
            "No se pudo cargar el sistema de inteligencia artificial.";
    }
}


/* =====================================================
   DETECCIÓN
===================================================== */

async function detectLoop(){

    if(
        !active ||
        !model ||
        detecting
    ){

        requestAnimationFrame(
            detectLoop
        );

        return;
    }


    detecting = true;


    try{

        detections =
            await model.detect(
                video
            );


        drawDetections(
            detections
        );

    }

    catch(error){

        console.error(error);
    }


    detecting = false;


    setTimeout(
        detectLoop,
        150
    );
}


/* =====================================================
   DISTANCIA
===================================================== */

function estimateDistance(
    object,
    pixelHeight
){

    const realHeight =
        objectHeights[object]
        || .50;


    if(
        pixelHeight <= 0
    ){

        return 0;
    }


    /*
       Estimación visual.

       NO es LiDAR ni medición real.
    */

    const focal =
        Math.max(
            500,
            video.videoWidth * .75
        );


    let distance =
        realHeight *
        focal /
        pixelHeight;


    distance =
        Math.max(
            .2,
            Math.min(
                30,
                distance
            )
        );


    return distance;
}


/* =====================================================
   ALTURA
===================================================== */

function estimateHeight(
    object,
    pixelHeight,
    distance
){

    const focal =
        Math.max(
            500,
            video.videoWidth * .75
        );


    let height =
        pixelHeight *
        distance /
        focal;


    const reference =
        objectHeights[object]
        || .50;


    /*
       Para evitar resultados
       completamente absurdos.
    */

    if(
        object === "person"
    ){

        height =
            Math.max(
                1.0,
                Math.min(
                    2.3,
                    height
                )
            );
    }

    else{

        height =
            Math.max(
                .03,
                Math.min(
                    5,
                    height
                )
            );
    }


    return height;
}


/* =====================================================
   DIBUJAR CAJAS
===================================================== */

function drawDetections(
    objects
){

    ctx.clearRect(
        0,
        0,
        canvas.width,
        canvas.height
    );


    objectCount.textContent =
        objects.length;


    objects.forEach(
        function(obj){

            const [
                x,
                y,
                w,
                h
            ] = obj.bbox;


            const distance =
                estimateDistance(
                    obj.class,
                    h
                );


            const height =
                estimateHeight(
                    obj.class,
                    h,
                    distance
                );


            const name =
                spanish[obj.class]
                || obj.class;


            /* -------------------------------------
               CAJA
            ------------------------------------- */

            ctx.strokeStyle =
                "#00d9ff";

            ctx.lineWidth = 3;

            ctx.shadowBlur = 15;

            ctx.shadowColor =
                "#00bfff";


            ctx.strokeRect(
                x,
                y,
                w,
                h
            );


            ctx.shadowBlur = 0;


            /* -------------------------------------
               ESQUINAS DE LA CAJA
            ------------------------------------- */

            const corner = 15;


            ctx.strokeStyle =
                "#ffffff";

            ctx.lineWidth = 2;


            // esquina superior izquierda

            ctx.beginPath();

            ctx.moveTo(
                x,
                y + corner
            );

            ctx.lineTo(
                x,
                y
            );

            ctx.lineTo(
                x + corner,
                y
            );

            ctx.stroke();


            // esquina superior derecha

            ctx.beginPath();

            ctx.moveTo(
                x+w-corner,
                y
            );

            ctx.lineTo(
                x+w,
                y
            );

            ctx.lineTo(
                x+w,
                y+corner
            );

            ctx.stroke();


            // esquina inferior izquierda

            ctx.beginPath();

            ctx.moveTo(
                x,
                y+h-corner
            );

            ctx.lineTo(
                x,
                y+h
            );

            ctx.lineTo(
                x+corner,
                y+h
            );

            ctx.stroke();


            // esquina inferior derecha

            ctx.beginPath();

            ctx.moveTo(
                x+w-corner,
                y+h
            );

            ctx.lineTo(
                x+w,
                y+h
            );

            ctx.lineTo(
                x+w,
                y+h-corner
            );

            ctx.stroke();


            /* -------------------------------------
               PANEL DE INFORMACIÓN
            ------------------------------------- */

            const panelHeight =
                82;


            const panelWidth =
                Math.max(
                    180,
                    Math.min(
                        260,
                        w
                    )
                );


            let panelY =
                y - panelHeight - 5;


            if(
                panelY < 5
            ){

                panelY =
                    y + 5;
            }


            /*
               Fondo
            */

            ctx.fillStyle =
                "rgba(0,15,30,.88)";


            ctx.fillRect(
                x,
                panelY,
                panelWidth,
                panelHeight
            );


            /*
               Borde
            */

            ctx.strokeStyle =
                "#00d9ff";

            ctx.lineWidth = 1;


            ctx.strokeRect(
                x,
                panelY,
                panelWidth,
                panelHeight
            );


            /* -------------------------------------
               TEXTO
            ------------------------------------- */

            ctx.fillStyle =
                "#00eaff";


            ctx.font =
                "bold 14px monospace";


            ctx.fillText(
                name.toUpperCase(),
                x + 7,
                panelY + 17
            );


            ctx.font =
                "11px monospace";


            ctx.fillText(
                "CONFIANZA: " +
                (obj.score*100)
                .toFixed(0) +
                "%",
                x + 7,
                panelY + 34
            );


            ctx.fillText(
                "DISTANCIA: " +
                distance.toFixed(1) +
                " m",
                x + 7,
                panelY + 50
            );


            ctx.fillText(
                "ALTURA EST.: " +
                height.toFixed(2) +
                " m",
                x + 7,
                panelY + 66
            );

        }
    );
}


/* =====================================================
   VOZ
===================================================== */

function speak(text){

    if(
        !("speechSynthesis"
        in window)
    ){

        return;
    }


    speechSynthesis.cancel();


    const utterance =
        new SpeechSynthesisUtterance(
            text
        );


    utterance.lang =
        "es-ES";


    utterance.rate =
        .90;


    /*
       Voz más grave para darle
       sensación de asistente.
    */

    utterance.pitch =
        .72;


    utterance.volume =
        1;


    speechSynthesis.speak(
        utterance
    );
}


/* =====================================================
   RECONOCIMIENTO DE VOZ
===================================================== */

function listen(){

    const Recognition =
        window.SpeechRecognition ||
        window.webkitSpeechRecognition;


    if(!Recognition){

        message.textContent =
            "El navegador no permite reconocimiento de voz.";

        return;
    }


    const recognition =
        new Recognition();


    recognition.lang =
        "es-ES";


    recognition.continuous =
        false;


    recognition.interimResults =
        false;


    voiceStatus.textContent =
        "ESCUCHANDO";


    message.textContent =
        "Te escucho.";


    speak(
        "Te escucho."
    );


    recognition.onresult =
        function(event){

            const command =
                event.results[0][0]
                .transcript
                .toLowerCase();


            voiceStatus.textContent =
                "ON";


            processCommand(
                command
            );
        };


    recognition.onerror =
        function(){

            voiceStatus.textContent =
                "ERROR";

            message.textContent =
                "No pude entenderte.";
        };


    recognition.onend =
        function(){

            voiceStatus.textContent =
                "OFF";
        };


    recognition.start();
}


/* =====================================================
   COMANDOS JARVIS
===================================================== */

function processCommand(
    command
){

    console.log(
        "Comando:",
        command
    );


    /* -----------------------------------------------
       SALUDO
    ----------------------------------------------- */

    if(
        command.includes("hola")
        ||
        command.includes("jarvis")
    ){

        const response =
            "Hola. Todos los sistemas funcionan correctamente.";

        message.textContent =
            response;

        speak(response);

        return;
    }


    /* -----------------------------------------------
       QUÉ VES
    ----------------------------------------------- */

    if(
        command.includes("qué ves")
        ||
        command.includes("que ves")
        ||
        command.includes("qué hay")
        ||
        command.includes("que hay")
        ||
        command.includes("describe")
    ){

        describeScene();

        return;
    }


    /* -----------------------------------------------
       CUÁNTOS OBJETOS
    ----------------------------------------------- */

    if(
        command.includes("cuántos")
        ||
        command.includes("cuantos")
    ){

        const response =
            "Detecto " +
            detections.length +
            " objetos.";

        message.textContent =
            response;

        speak(response);

        return;
    }


    /* -----------------------------------------------
       ESTADO
    ----------------------------------------------- */

    if(
        command.includes("estado")
        ||
        command.includes("sistemas")
    ){

        const response =
            "Sistemas operativos. " +
            "Cámara activa. " +
            "Inteligencia visual funcionando.";

        message.textContent =
            response;

        speak(response);

        return;
    }


    /* -----------------------------------------------
       OBJETO MÁS CERCANO
    ----------------------------------------------- */

    if(
        command.includes("más cerca")
        ||
        command.includes("mas cerca")
        ||
        command.includes("cerca")
    ){

        closestObject();

        return;
    }


    /* -----------------------------------------------
       AYUDA
    ----------------------------------------------- */

    if(
        command.includes("ayuda")
        ||
        command.includes("comandos")
    ){

        const response =
            "Puedes preguntarme qué veo, " +
            "cuántos objetos hay, " +
            "cuál está más cerca o decir estado.";

        message.textContent =
            response;

        speak(response);

        return;
    }


    /* -----------------------------------------------
       NO RECONOCIDO
    ----------------------------------------------- */

    const response =
        "No tengo una respuesta configurada " +
        "para ese comando.";

    message.textContent =
        response;

    speak(response);
}


/* =====================================================
   DESCRIBIR ESCENA
===================================================== */

function describeScene(){

    if(
        detections.length === 0
    ){

        const response =
            "No detecto objetos en este momento.";

        message.textContent =
            response;

        speak(response);

        return;
    }


    let response =
        "Detecto " +
        detections.length +
        " objetos. ";


    const used = {};


    detections
    .slice(0,6)
    .forEach(
        function(obj){

            const name =
                spanish[obj.class]
                || obj.class;


            if(
                !used[name]
            ){

                response +=
                    name +
                    ". ";

                used[name] = true;
            }

        }
    );


    message.textContent =
        response;

    speak(response);
}


/* =====================================================
   OBJETO MÁS CERCANO
===================================================== */

function closestObject(){

    if(
        detections.length === 0
    ){

        const response =
            "No detecto ningún objeto.";

        message.textContent =
            response;

        speak(response);

        return;
    }


    let closest =
        detections[0];


    let closestDistance =
        estimateDistance(
            closest.class,
            closest.bbox[3]
        );


    detections
    .forEach(
        function(obj){

            const distance =
                estimateDistance(
                    obj.class,
                    obj.bbox[3]
                );


            if(
                distance <
                closestDistance
            ){

                closest =
                    obj;

                closestDistance =
                    distance;
            }

        }
    );


    const name =
        spanish[closest.class]
        || closest.class;


    const response =
        "El objeto que parece estar " +
        "más cerca es " +
        name +
        ", aproximadamente a " +
        closestDistance.toFixed(1) +
        " metros.";


    message.textContent =
        response;

    speak(response);
}


/* =====================================================
   BOTONES
===================================================== */

startButton.addEventListener(
    "click",
    startCamera
);


document.getElementById(
    "voiceButton"
).addEventListener(
    "click",
    listen
);


/* =====================================================
   INICIO
===================================================== */

console.log(
    "JARVIS Vision System cargado."
);

</script>

</body>
</html>
