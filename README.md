<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">

<meta name="viewport"
content="width=device-width,
initial-scale=1.0,
maximum-scale=1.0,
user-scalable=no">

<title>JARVIS VISION</title>

<style>

/* =====================================================
   BASE
===================================================== */

*{
    box-sizing:border-box;
}

html,body{
    margin:0;
    padding:0;
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
       IMPORTANTE:
       Sin filtro.
       La cámara conserva sus colores normales.
    */

    filter:none;
}


/* =====================================================
   CANVAS
===================================================== */

#canvas{

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

    transform:
        translateX(-50%);

    text-align:center;

    text-shadow:
        0 0 8px #00d9ff,
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
   ESQUINAS
===================================================== */

.corner{

    position:absolute;

    width:60px;

    height:60px;

    border-color:#00d9ff;

    filter:
        drop-shadow(
            0 0 8px #00cfff
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
   LÍNEA DE ESCANEO
===================================================== */

#scan{

    position:absolute;

    width:100%;

    height:2px;

    background:#00d9ff;

    opacity:.22;

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

#jarvis{

    position:absolute;

    left:15px;

    bottom:15px;

    width:285px;

    padding:12px;

    background:
        rgba(0,15,30,.78);

    border-left:
        3px solid #00d9ff;

    box-shadow:
        0 0 20px
        rgba(0,190,255,.25);

    backdrop-filter:
        blur(5px);
}

#jarvisName{

    font-size:14px;

    font-weight:bold;

    letter-spacing:2px;
}

#message{

    margin-top:6px;

    min-height:32px;

    font-size:11px;

    line-height:1.5;
}


/* =====================================================
   BOTÓN MICRO
===================================================== */

#voiceButton{

    pointer-events:auto;

    margin-top:8px;

    padding:10px 14px;

    color:#00d9ff;

    background:
        rgba(0,40,70,.9);

    border:
        1px solid #00d9ff;

    border-radius:5px;

    font-family:monospace;

    font-size:11px;
}

#voiceButton:active{

    background:
        rgba(0,120,190,.8);

}


/* =====================================================
   ESTADO
===================================================== */

#status{

    position:absolute;

    right:15px;

    bottom:15px;

    text-align:right;

    font-size:10px;

    line-height:1.8;

    text-shadow:
        0 0 8px #00d9ff;
}


/* =====================================================
   BOTÓN INICIAL
===================================================== */

#start{

    position:fixed;

    z-index:50;

    top:50%;
    left:50%;

    transform:
        translate(-50%,-50%);

    padding:18px 25px;

    color:#00d9ff;

    background:
        rgba(0,10,25,.95);

    border:
        1px solid #00d9ff;

    border-radius:6px;

    font-family:monospace;

    font-size:16px;

    box-shadow:
        0 0 25px #008cff;
}


/* =====================================================
   MÓVIL
===================================================== */

@media(max-width:600px){

    #title b{

        font-size:18px;
    }

    #jarvis{

        width:235px;

        left:10px;

        bottom:10px;
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
     CANVAS PARA LAS CAJAS
=================================================== -->

<canvas
id="canvas">
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


    <!-- PANEL JARVIS -->

    <div id="jarvis">

        <div id="jarvisName">
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
            OFF
        </span>

        <br>

        OBJETOS:
        <span id="objectCount">
            0
        </span>

        <br>

        MICRÓFONO:
        <span id="micStatus">
            OFF
        </span>

    </div>

</div>


<!-- ===================================================
     ACTIVAR
=================================================== -->

<button id="start">

    ACTIVAR JARVIS

</button>


<!-- ===================================================
     TENSORFLOW
=================================================== -->

<script
src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs">
</script>


<script
src="https://cdn.jsdelivr.net/npm/@tensorflow-models/coco-ssd">
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
    "canvas"
);

const ctx =
canvas.getContext(
    "2d"
);

const startButton =
document.getElementById(
    "start"
);

const voiceButton =
document.getElementById(
    "voiceButton"
);

const message =
document.getElementById(
    "message"
);

const cameraStatus =
document.getElementById(
    "cameraStatus"
);

const aiStatus =
document.getElementById(
    "aiStatus"
);

const objectCount =
document.getElementById(
    "objectCount"
);

const micStatus =
document.getElementById(
    "micStatus"
);


let model = null;

let active = false;

let objects = [];

let detecting = false;


/* =====================================================
   NOMBRES
===================================================== */

const names = {

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

    bottle:"botella",

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
   ALTURAS DE REFERENCIA
===================================================== */

const heights = {

    person:1.70,

    bicycle:1.05,

    car:1.50,

    motorcycle:1.10,

    bus:3.20,

    truck:2.50,

    chair:.90,

    couch:.80,

    dining_table:.75,

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

    cell_phone:.15,

    refrigerator:1.70

};


/* =====================================================
   AJUSTAR CANVAS
===================================================== */

function resize(){

    canvas.width =
        video.videoWidth;

    canvas.height =
        video.videoHeight;
}

window.addEventListener(
    "resize",
    resize
);


/* =====================================================
   CÁMARA
===================================================== */

async function startCamera(){

    try{

        const stream =
        await navigator
        .mediaDevices
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


        resize();


        active = true;


        startButton.style.display =
            "none";


        cameraStatus.textContent =
            "ON";


        message.textContent =
            "Cámara activada. Cargando IA...";


        loadAI();

    }

    catch(error){

        console.error(error);

        message.textContent =
            "No se pudo activar la cámara.";

        alert(
            "Debes permitir el acceso a la cámara."
        );

    }

}


/* =====================================================
   IA
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
            "Visión artificial activada.";


        speak(
            "Sistema de visión activado."
        );


        detect();

    }

    catch(error){

        console.error(error);

        aiStatus.textContent =
            "ERROR";

        message.textContent =
            "Error cargando la inteligencia artificial.";

    }

}


/* =====================================================
   DETECTAR
===================================================== */

async function detect(){

    if(
        !active ||
        !model ||
        detecting
    ){

        requestAnimationFrame(
            detect
        );

        return;
    }


    detecting = true;


    try{

        objects =
            await model.detect(
                video
            );


        draw(objects);

    }

    catch(error){

        console.error(error);

    }


    detecting = false;


    setTimeout(
        detect,
        200
    );

}


/* =====================================================
   DISTANCIA
===================================================== */

function distance(
    object,
    pixelHeight
){

    const realHeight =
        heights[object]
        || .5;


    const focal =
        Math.max(
            500,
            video.videoWidth * .75
        );


    let d =
        realHeight *
        focal /
        pixelHeight;


    d =
        Math.max(
            .2,
            Math.min(
                30,
                d
            )
        );


    return d;
}


/* =====================================================
   ALTURA
===================================================== */

function objectHeight(
    object,
    pixelHeight,
    d
){

    const focal =
        Math.max(
            500,
            video.videoWidth * .75
        );


    let h =
        pixelHeight *
        d /
        focal;


    h =
        Math.max(
            .03,
            Math.min(
                5,
                h
            )
        );


    return h;
}


/* =====================================================
   DIBUJAR
===================================================== */

function draw(
    predictions
){

    ctx.clearRect(
        0,
        0,
        canvas.width,
        canvas.height
    );


    objectCount.textContent =
        predictions.length;


    predictions.forEach(
        function(obj){

            const x =
                obj.bbox[0];

            const y =
                obj.bbox[1];

            const w =
                obj.bbox[2];

            const h =
                obj.bbox[3];


            const d =
                distance(
                    obj.class,
                    h
                );


            const realH =
                objectHeight(
                    obj.class,
                    h,
                    d
                );


            const name =
                names[obj.class]
                ||
                obj.class;


            /* -----------------------------------------
               CAJA PRINCIPAL
            ----------------------------------------- */

            ctx.strokeStyle =
                "#00d9ff";

            ctx.lineWidth =
                3;

            ctx.shadowColor =
                "#00d9ff";

            ctx.shadowBlur =
                15;


            ctx.strokeRect(
                x,
                y,
                w,
                h
            );


            ctx.shadowBlur =
                0;


            /* -----------------------------------------
               PANEL
            ----------------------------------------- */

            const panelWidth =
                Math.max(
                    180,
                    Math.min(
                        250,
                        w
                    )
                );


            const panelHeight =
                78;


            let py =
                y -
                panelHeight -
                5;


            if(
                py < 5
            ){

                py =
                    y + 5;

            }


            ctx.fillStyle =
                "rgba(0,15,30,.90)";


            ctx.fillRect(
                x,
                py,
                panelWidth,
                panelHeight
            );


            ctx.strokeStyle =
                "#00d9ff";

            ctx.lineWidth =
                1;


            ctx.strokeRect(
                x,
                py,
                panelWidth,
                panelHeight
            );


            /* -----------------------------------------
               TEXTO
            ----------------------------------------- */

            ctx.fillStyle =
                "#00eaff";


            ctx.font =
                "bold 14px monospace";


            ctx.fillText(
                name.toUpperCase(),
                x+7,
                py+17
            );


            ctx.font =
                "11px monospace";


            ctx.fillText(
                "CONFIANZA: " +
                (obj.score*100)
                .toFixed(0) +
                "%",
                x+7,
                py+34
            );


            ctx.fillText(
                "DISTANCIA: " +
                d.toFixed(1) +
                " m",
                x+7,
                py+50
            );


            ctx.fillText(
                "ALTURA EST.: " +
                realH.toFixed(2) +
                " m",
                x+7,
                py+66
            );

        }
    );

}


/* =====================================================
   VOZ DE JARVIS
===================================================== */

function speak(text){

    if(
        !window.speechSynthesis
    ){

        return;
    }


    speechSynthesis.cancel();


    const voice =
        new SpeechSynthesisUtterance(
            text
        );


    voice.lang =
        "es-ES";


    voice.rate =
        .9;


    voice.pitch =
        .75;


    voice.volume =
        1;


    speechSynthesis.speak(
        voice
    );

}


/* =====================================================
   RECONOCIMIENTO DE VOZ
===================================================== */

function listen(){

    /*
       Safari/iOS utiliza normalmente
       webkitSpeechRecognition cuando
       está disponible.
    */

    const Recognition =
        window.SpeechRecognition ||
        window.webkitSpeechRecognition;


    if(
        !Recognition
    ){

        micStatus.textContent =
            "NO DISP.";


        message.textContent =
            "El reconocimiento de voz no está disponible en este navegador.";


        speak(
            "El reconocimiento de voz no está disponible."
        );


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


    recognition.maxAlternatives =
        1;


    micStatus.textContent =
        "ESCUCHANDO";


    message.textContent =
        "Te escucho...";


    /*
       Esperamos un pequeño momento
       antes de escuchar.
    */

    setTimeout(
        function(){

            try{

                recognition.start();

            }

            catch(error){

                console.log(error);

            }

        },
        150
    );


    recognition.onresult =
        function(event){

            const text =
                event
                .results[0][0]
                .transcript
                .toLowerCase();


            console.log(
                "Usuario:",
                text
            );


            micStatus.textContent =
                "ON";


            processCommand(
                text
            );

        };


    recognition.onspeechstart =
        function(){

            micStatus.textContent =
                "ESCUCHANDO";

        };


    recognition.onspeechend =
        function(){

            micStatus.textContent =
                "PROCESANDO";

        };


    recognition.onerror =
        function(event){

            console.log(
                "Micrófono:",
                event.error
            );


            micStatus.textContent =
                "ERROR";


            if(
                event.error ===
                "not-allowed"
            ){

                message.textContent =
                    "Permiso de micrófono bloqueado. Permite el micrófono en Safari.";

            }

            else if(
                event.error ===
                "network"
            ){

                message.textContent =
                    "Safari no pudo conectar con el servicio de reconocimiento de voz.";

            }

            else{

                message.textContent =
                    "No pude escuchar correctamente.";

            }

        };


    recognition.onend =
        function(){

            if(
                micStatus.textContent !==
                "ERROR"
            ){

                micStatus.textContent =
                    "OFF";
            }

        };

}


/* =====================================================
   PROCESAR COMANDO
===================================================== */

function processCommand(
    command
){

    console.log(
        command
    );


    /* SALUDO */

    if(
        command.includes("hola")
        ||
        command.includes("jarvis")
    ){

        respond(
            "Hola. Todos los sistemas funcionan correctamente."
        );

        return;
    }


    /* QUÉ VES */

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

        describe();

        return;
    }


    /* CUÁNTOS */

    if(
        command.includes("cuántos")
        ||
        command.includes("cuantos")
    ){

        respond(
            "Actualmente detecto " +
            objects.length +
            " objetos."
        );

        return;
    }


    /* ESTADO */

    if(
        command.includes("estado")
        ||
        command.includes("sistemas")
    ){

        respond(
            "Todos los sistemas están operativos. Cámara e inteligencia visual activadas."
        );

        return;
    }


    /* MÁS CERCANO */

    if(
        command.includes("más cerca")
        ||
        command.includes("mas cerca")
    ){

        closest();

        return;
    }


    /* AYUDA */

    if(
        command.includes("ayuda")
        ||
        command.includes("comandos")
    ){

        respond(
            "Puedes preguntarme qué veo, cuántos objetos detecto, cuál está más cerca o cuál es el estado del sistema."
        );

        return;
    }


    /* NO ENTENDIDO */

    respond(
        "He escuchado tu comando, pero todavía no tengo una respuesta configurada para esa pregunta."
    );

}


/* =====================================================
   RESPONDER
===================================================== */

function respond(text){

    message.textContent =
        text;

    speak(
        text
    );

}


/* =====================================================
   DESCRIBIR ESCENA
===================================================== */

function describe(){

    if(
        objects.length === 0
    ){

        respond(
            "No detecto ningún objeto en este momento."
        );

        return;
    }


    let text =
        "Detecto " +
        objects.length +
        " objetos. ";


    const found = {};


    objects
    .slice(0,6)
    .forEach(
        function(obj){

            const name =
                names[obj.class]
                ||
                obj.class;


            if(
                !found[name]
            ){

                text +=
                    name +
                    ". ";

                found[name] =
                    true;
            }

        }
    );


    respond(
        text
    );

}


/* =====================================================
   MÁS CERCANO
===================================================== */

function closest(){

    if(
        objects.length === 0
    ){

        respond(
            "No detecto objetos."
        );

        return;
    }


    let nearest =
        objects[0];


    let nearestDistance =
        distance(
            nearest.class,
            nearest.bbox[3]
        );


    objects.forEach(
        function(obj){

            const d =
                distance(
                    obj.class,
                    obj.bbox[3]
                );


            if(
                d <
                nearestDistance
            ){

                nearest =
                    obj;

                nearestDistance =
                    d;
            }

        }
    );


    const name =
        names[nearest.class]
        ||
        nearest.class;


    respond(
        "El objeto que parece estar más cerca es " +
        name +
        ", aproximadamente a " +
        nearestDistance.toFixed(1) +
        " metros."
    );

}


/* =====================================================
   BOTONES
===================================================== */

startButton.onclick =
    startCamera;


voiceButton.onclick =
    listen;


/* =====================================================
   INICIO
===================================================== */

console.log(
    "JARVIS VISION ONLINE"
);

</script>

</body>
</html>
