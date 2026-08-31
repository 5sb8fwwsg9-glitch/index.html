<!DOCTYPE html>
<html lang="es">
<head>

<meta charset="UTF-8">

<meta name="viewport"
content="width=device-width,
initial-scale=1,
maximum-scale=1,
user-scalable=no">

<title>JARVIS VISION</title>

<style>

*{
    box-sizing:border-box;
    -webkit-tap-highlight-color:transparent;
}

html,body{
    margin:0;
    width:100%;
    height:100%;
    overflow:hidden;
    background:#000;
    color:#00d9ff;
    font-family:monospace;
}

#camera{
    position:fixed;
    inset:0;
    width:100%;
    height:100%;
    object-fit:cover;
}

#canvas{
    position:fixed;
    inset:0;
    width:100%;
    height:100%;
    z-index:2;
    touch-action:none;
}

#hud{
    position:fixed;
    inset:0;
    z-index:3;
    pointer-events:none;
}

#title{
    position:absolute;
    top:15px;
    left:50%;
    transform:translateX(-50%);
    text-align:center;
    text-shadow:0 0 10px #00d9ff;
}

#title b{
    font-size:21px;
    letter-spacing:5px;
}

#title small{
    display:block;
    font-size:8px;
    margin-top:3px;
    letter-spacing:3px;
}

.corner{
    position:absolute;
    width:45px;
    height:45px;
    border-color:#00d9ff;
    filter:drop-shadow(0 0 7px #00d9ff);
}

.tl{
    top:10px;
    left:10px;
    border-top:2px solid;
    border-left:2px solid;
}

.tr{
    top:10px;
    right:10px;
    border-top:2px solid;
    border-right:2px solid;
}

.bl{
    bottom:10px;
    left:10px;
    border-bottom:2px solid;
    border-left:2px solid;
}

.br{
    bottom:10px;
    right:10px;
    border-bottom:2px solid;
    border-right:2px solid;
}

#panel{
    position:absolute;
    left:10px;
    bottom:10px;

    width:320px;
    max-width:calc(100vw - 20px);
    max-height:42vh;

    overflow:auto;

    padding:11px;

    background:rgba(0,8,20,.9);

    border-left:2px solid #00d9ff;

    box-shadow:
    0 0 20px rgba(0,217,255,.25);

    pointer-events:auto;
}

#jarvisName{
    font-weight:bold;
    letter-spacing:2px;
}

#mode{
    margin-top:3px;
    font-size:8px;
    opacity:.6;
}

#message{
    margin-top:7px;
    font-size:9px;
    line-height:1.5;
}

#specs{
    display:none;
    margin-top:8px;
    padding-top:8px;
    border-top:1px solid rgba(0,217,255,.4);
    font-size:8px;
    line-height:1.6;
}

.specTitle{
    font-size:10px;
    letter-spacing:2px;
    margin-bottom:5px;
}

.row{
    display:flex;
    gap:5px;
}

.label{
    width:85px;
    opacity:.55;
    flex-shrink:0;
}

button{
    margin-top:8px;
    padding:9px 12px;

    background:rgba(0,30,55,.9);

    border:1px solid #00d9ff;
    border-radius:5px;

    color:#00d9ff;

    font-family:monospace;
    font-size:9px;
}

button:active{
    background:#00d9ff;
    color:#00101a;
}

#start{
    position:fixed;
    z-index:20;

    top:50%;
    left:50%;

    transform:translate(-50%,-50%);

    padding:17px 25px;

    color:#00d9ff;

    background:rgba(0,8,20,.97);

    border:1px solid #00d9ff;
    border-radius:6px;

    box-shadow:0 0 25px #008cff;

    font-family:monospace;
    font-size:14px;
}

#scanText{
    position:absolute;

    top:42%;
    left:50%;

    transform:translate(-50%,-50%);

    display:none;

    font-size:11px;
    letter-spacing:3px;

    text-shadow:
    0 0 8px #00d9ff,
    0 0 20px #008cff;
}

#zoom{
    position:absolute;

    top:65px;
    right:10px;

    width:190px;
    height:130px;

    display:none;

    background:#00101d;

    border:1px solid #00d9ff;

    box-shadow:
    0 0 15px rgba(0,217,255,.4);

    overflow:hidden;
}

#zoomCanvas{
    width:100%;
    height:100%;
}

#status{
    position:absolute;

    right:12px;
    bottom:12px;

    text-align:right;

    font-size:8px;
    line-height:1.8;

    text-shadow:0 0 6px #00d9ff;
}

</style>

</head>

<body>


<video
id="camera"
autoplay
muted
playsinline>
</video>


<canvas id="canvas"></canvas>


<div id="hud">


<div class="corner tl"></div>
<div class="corner tr"></div>
<div class="corner bl"></div>
<div class="corner br"></div>


<div id="title">

<b>J.A.R.V.I.S.</b>

<small>
VISION SYSTEM
</small>

</div>


<div id="scanText">
ANALIZANDO...
</div>


<div id="zoom">

<canvas id="zoomCanvas">
</canvas>

</div>


<div id="panel">


<div id="jarvisName">
JARVIS
</div>


<div id="mode">
MODO JARVIS
</div>


<div id="message">

Sistema listo.
Pulsa "Hablar con JARVIS".

</div>


<div id="specs">

<div class="specTitle">
TARGET ANALYSIS
</div>


<div class="row">
<span class="label">OBJETO</span>
<span id="name">---</span>
</div>


<div class="row">
<span class="label">CONFIANZA</span>
<span id="confidence">---</span>
</div>


<div class="row">
<span class="label">DISTANCIA</span>
<span id="distance">---</span>
</div>


<div class="row">
<span class="label">ALTURA</span>
<span id="height">---</span>
</div>


<div class="row">
<span class="label">MATERIAL</span>
<span id="material">---</span>
</div>


<div class="row">
<span class="label">FABRICACIÓN</span>
<span id="manufacturing">---</span>
</div>


<div class="row">
<span class="label">PROCEDENCIA</span>
<span id="origin">---</span>
</div>


<div class="row">
<span class="label">USO</span>
<span id="use">---</span>
</div>

</div>


<button id="voiceButton">
🎙 HABLAR CON JARVIS
</button>


<button id="analysisButton">
🔎 MODO ANÁLISIS
</button>


</div>


<div id="status">

SISTEMA:
<span id="system">ONLINE</span>
<br>

CÁMARA:
<span id="cam">OFF</span>
<br>

IA:
<span id="ai">OFF</span>
<br>

MODO:
<span id="modeStatus">JARVIS</span>

</div>


</div>


<button id="start">
ACTIVAR JARVIS
</button>


<script
src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs">
</script>


<script
src="https://cdn.jsdelivr.net/npm/@tensorflow-models/coco-ssd">
</script>


<script>

/* =========================================
   ELEMENTOS
========================================= */

const video =
document.getElementById("camera");

const canvas =
document.getElementById("canvas");

const ctx =
canvas.getContext("2d");

const start =
document.getElementById("start");

const voiceButton =
document.getElementById("voiceButton");

const analysisButton =
document.getElementById("analysisButton");

const message =
document.getElementById("message");

const specs =
document.getElementById("specs");

const scanText =
document.getElementById("scanText");

const zoom =
document.getElementById("zoom");

const zoomCanvas =
document.getElementById("zoomCanvas");

const zctx =
zoomCanvas.getContext("2d");

const ai =
document.getElementById("ai");

const cam =
document.getElementById("cam");

const modeStatus =
document.getElementById("modeStatus");

const modeText =
document.getElementById("mode");

const targetName =
document.getElementById("name");

const confidence =
document.getElementById("confidence");

const distance =
document.getElementById("distance");

const height =
document.getElementById("height");

const material =
document.getElementById("material");

const manufacturing =
document.getElementById("manufacturing");

const origin =
document.getElementById("origin");

const use =
document.getElementById("use");


/* =========================================
   VARIABLES
========================================= */

let model = null;

let running = false;

let analysisMode = false;

let detections = [];

let selected = null;

let smoothedBox = null;

let lastDetection = 0;

let scanning = false;

let scanStart = 0;


/* =========================================
   NOMBRES
========================================= */

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

bird:"pájaro",
cat:"gato",
dog:"perro",
horse:"caballo",
sheep:"oveja",
cow:"vaca",

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


/* =========================================
   INFORMACIÓN
========================================= */

const info = {

bottle:[
"plástico PET, vidrio o aluminio",
"fabricación industrial",
"depende del fabricante",
"almacenar líquidos"
],

cup:[
"cerámica, vidrio o plástico",
"moldeado y fabricación industrial",
"depende del fabricante",
"beber líquidos"
],

chair:[
"madera, plástico o metal",
"carpintería o fabricación industrial",
"depende del fabricante",
"sentarse"
],

dining_table:[
"madera, vidrio o metal",
"carpintería y fabricación industrial",
"depende del fabricante",
"comer o apoyar objetos"
],

laptop:[
"aluminio, plástico, vidrio y electrónica",
"ensamblaje electrónico industrial",
"depende de la marca",
"computación"
],

cell_phone:[
"vidrio, metal, plástico y electrónica",
"ensamblaje electrónico industrial",
"depende de la marca",
"comunicación"
],

keyboard:[
"plástico, metal y electrónica",
"ensamblaje industrial",
"depende del fabricante",
"entrada de texto"
],

mouse:[
"plástico y componentes electrónicos",
"ensamblaje industrial",
"depende del fabricante",
"control de computadora"
],

book:[
"papel, tinta y cartón",
"impresión y encuadernación",
"depende de la editorial",
"lectura"
],

backpack:[
"nylon o poliéster",
"confección textil",
"depende del fabricante",
"transportar objetos"
],

car:[
"acero, aluminio, plástico, vidrio y electrónica",
"producción automotriz",
"depende de la marca",
"transporte"
],

bicycle:[
"aluminio, acero, carbono y caucho",
"fabricación mecánica",
"depende del fabricante",
"transporte"
],

dog:[
"ser vivo",
"no aplica",
"animal",
"animal doméstico"
],

cat:[
"ser vivo",
"no aplica",
"animal",
"animal doméstico"
],

person:[
"ser vivo",
"no aplica",
"persona",
"persona"
]

};


/* =========================================
   ALTURAS
========================================= */

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

laptop:.025,
cell_phone:.15,

backpack:.45,
suitcase:.70,

dog:.60,
cat:.30,

book:.25,
tv:.70,
refrigerator:1.70

};


/* =========================================
   HABLAR
========================================= */

function speak(text){

    if(!("speechSynthesis" in window)){

        message.textContent =
        "La voz no está disponible.";

        return;

    }

    speechSynthesis.cancel();

    const utterance =
    new SpeechSynthesisUtterance(text);

    utterance.lang="es-ES";

    utterance.rate=.88;

    utterance.pitch=.75;

    utterance.volume=1;

    const voices =
    speechSynthesis.getVoices();

    const spanish =
    voices.find(
        v =>
        v.lang &&
        v.lang.toLowerCase()
        .startsWith("es")
    );

    if(spanish)
        utterance.voice=spanish;

    speechSynthesis.speak(
        utterance
    );

}


/* =========================================
   JARVIS NORMAL
========================================= */

function jarvisAnswer(question){

    const q =
    question
    .toLowerCase()
    .trim();


    if(
        q.includes("hola") ||
        q.includes("buenas")
    ){

        return "Hola. Todos los sistemas están funcionando correctamente.";

    }


    if(
        q.includes("cómo estás") ||
        q.includes("como estas")
    ){

        return "Todos mis sistemas están operativos.";

    }


    if(
        q.includes("quién eres") ||
        q.includes("quien eres")
    ){

        return "Soy JARVIS, tu asistente virtual de visión.";

    }


    if(
        q.includes("qué puedes hacer") ||
        q.includes("que puedes hacer")
    ){

        return "Puedo controlar la cámara, analizar objetos y proporcionarte información sobre ellos.";

    }


    if(
        q.includes("modo análisis") ||
        q.includes("modo analisis")
    ){

        analysisMode=true;

        updateMode();

        return "Modo análisis activado.";

    }


    if(
        q.includes("modo jarvis") ||
        q.includes("modo normal")
    ){

        analysisMode=false;

        selected=null;

        smoothedBox=null;

        specs.style.display="none";

        zoom.style.display="none";

        updateMode();

        return "Modo JARVIS activado.";

    }


    if(
        q.includes("gracias")
    ){

        return "Siempre a tu servicio.";

    }


    if(
        q.includes("hora")
    ){

        const now =
        new Date();

        return "La hora actual es "+
        now.toLocaleTimeString(
            "es-PA",
            {
                hour:"2-digit",
                minute:"2-digit"
            }
        )+".";

    }


    return "He recibido tu pregunta. Para responder preguntas generales con inteligencia artificial necesitaría conectarme a un servicio de IA.";

}


/* =========================================
   RECONOCIMIENTO DE VOZ
========================================= */

function listen(){

    const SpeechRecognition =
    window.SpeechRecognition ||
    window.webkitSpeechRecognition;


    if(!SpeechRecognition){

        message.textContent =
        "El reconocimiento de voz no está disponible en este navegador.";

        speak(
        "El reconocimiento de voz no está disponible en este navegador."
        );

        return;

    }


    const recognition =
    new SpeechRecognition();

    recognition.lang="es-ES";

    recognition.continuous=false;

    recognition.interimResults=false;


    message.textContent =
    "JARVIS escuchando...";


    recognition.onresult =
    function(event){

        const text =
        event.results[0][0].transcript;

        message.textContent =
        "Tú: "+text;


        const answer =
        jarvisAnswer(text);


        setTimeout(
            ()=>{
                message.textContent =
                "JARVIS: "+answer;

                speak(answer);
            },
            200
        );

    };


    recognition.onerror =
    function(){

        message.textContent =
        "No pude entenderte. Inténtalo nuevamente.";

    };


    recognition.start();

}


/* =========================================
   MODO
========================================= */

function updateMode(){

    if(analysisMode){

        modeText.textContent =
        "MODO ANÁLISIS";

        modeStatus.textContent =
        "ANÁLISIS";

        analysisButton.textContent =
        "🤖 MODO JARVIS";

    }
    else{

        modeText.textContent =
        "MODO JARVIS";

        modeStatus.textContent =
        "JARVIS";

        analysisButton.textContent =
        "🔎 MODO ANÁLISIS";

        ctx.clearRect(
            0,
            0,
            canvas.width,
            canvas.height
        );

    }

}


/* =========================================
   CÁMARA
========================================= */

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

            audio:true

        });


        video.srcObject =
        stream;

        await video.play();


        canvas.width =
        video.videoWidth;

        canvas.height =
        video.videoHeight;


        running=true;

        start.style.display=
        "none";

        cam.textContent=
        "ON";


        await startAI();

    }

    catch(error){

        console.error(error);

        message.textContent =
        "No se pudo acceder a la cámara. Revisa los permisos.";

    }

}


/* =========================================
   IA
========================================= */

async function startAI(){

    try{

        ai.textContent=
        "CARGANDO";

        await tf.ready();

        model =
        await cocoSsd.load({
            base:"mobilenet_v2"
        });


        ai.textContent=
        "ONLINE";

        message.textContent =
        "JARVIS listo.";

        detectionLoop();

    }

    catch(error){

        console.error(error);

        ai.textContent=
        "ERROR";

    }

}


/* =========================================
   DETECCIÓN
========================================= */

async function detectionLoop(){

    if(
        !running ||
        !model
    ){

        requestAnimationFrame(
            detectionLoop
        );

        return;

    }


    const now =
    performance.now();


    if(
        now-lastDetection>100
    ){

        lastDetection=
        now;


        try{

            detections =
            await model.detect(
                video,
                20,
                .35
            );

        }

        catch(error){

            console.error(error);

        }

    }


    requestAnimationFrame(
        detectionLoop
    );

}


/* =========================================
   CENTRO
========================================= */

function center(box){

    return {

        x:
        box[0]+box[2]/2,

        y:
        box[1]+box[3]/2

    };

}


/* =========================================
   OBJETO MÁS CERCANO AL CENTRO
========================================= */

function getBestTarget(){

    if(!detections.length)
        return null;


    const cx =
    canvas.width/2;

    const cy =
    canvas.height/2;


    let best=null;

    let bestScore=-Infinity;


    for(
        const d of detections
    ){

        if(d.score<.40)
            continue;


        const c =
        center(d.bbox);


        const dist =
        Math.hypot(
            c.x-cx,
            c.y-cy
        );


        const centerScore =
        1-
        dist/
        Math.max(
            canvas.width,
            canvas.height
        );


        const score =
        d.score*.6+
        centerScore*.4;


        if(
            score>bestScore
        ){

            bestScore=
            score;

            best=d;

        }

    }


    return best;

}


/* =========================================
   SUAVIZADO
========================================= */

function smoothBox(
    current,
    target
){

    if(!current)
        return target.slice();


    const t=.18;


    return [

        current[0]+
        (target[0]-current[0])*t,

        current[1]+
        (target[1]-current[1])*t,

        current[2]+
        (target[2]-current[2])*t,

        current[3]+
        (target[3]-current[3])*t

    ];

}


/* =========================================
   ESQUINAS
========================================= */

function drawCorners(
    x,
    y,
    w,
    h
){

    const size =
    Math.min(
        22,
        Math.max(
            8,
            Math.min(w,h)*.2
        )
    );


    ctx.beginPath();


    ctx.moveTo(x,y+size);
    ctx.lineTo(x,y);
    ctx.lineTo(x+size,y);


    ctx.moveTo(x+w-size,y);
    ctx.lineTo(x+w,y);
    ctx.lineTo(x+w,y+size);


    ctx.moveTo(x,y+h-size);
    ctx.lineTo(x,y+h);
    ctx.lineTo(x+size,y+h);


    ctx.moveTo(x+w-size,y+h);
    ctx.lineTo(x+w,y+h);
    ctx.lineTo(x+w,y+h-size);


    ctx.stroke();

}


/* =========================================
   ESCANEO
========================================= */

function drawScan(box){

    if(!scanning)
        return;


    const progress =
    Math.min(
        1,
        (performance.now()-
        scanStart)/1200
    );


    const y =
    box[1]+
    box[3]*progress;


    ctx.save();


    ctx.strokeStyle=
    "#00eaff";

    ctx.lineWidth=3;

    ctx.shadowColor=
    "#00eaff";

    ctx.shadowBlur=18;


    ctx.beginPath();

    ctx.moveTo(
        box[0]-5,
        y
    );

    ctx.lineTo(
        box[0]+box[2]+5,
        y
    );

    ctx.stroke();


    ctx.restore();


    if(progress>=1){

        scanning=false;

        scanText.style.display=
        "none";

    }

}


/* =========================================
   DIBUJAR OBJETO
========================================= */

function drawTarget(target){

    if(!target){

        smoothedBox=null;

        ctx.clearRect(
            0,
            0,
            canvas.width,
            canvas.height
        );

        return;

    }


    smoothedBox =
    smoothBox(
        smoothedBox,
        target.bbox
    );


    const box =
    smoothedBox;


    ctx.save();


    ctx.strokeStyle=
    "#00d9ff";

    ctx.lineWidth=2;

    ctx.shadowColor=
    "#00d9ff";

    ctx.shadowBlur=10;


    drawCorners(
        box[0],
        box[1],
        box[2],
        box[3]
    );


    ctx.restore();


    drawScan(box);

}


/* =========================================
   DISTANCIA
========================================= */

function estimateDistance(d){

    const realHeight =
    heights[d.class] ||
    .5;


    const focal =
    video.videoWidth*.75;


    return Math.max(
        .3,
        Math.min(
            20,
            realHeight*focal/
            Math.max(
                10,
                d.bbox[3]
            )
        )
    );

}


/* =========================================
   DATOS
========================================= */

function showSpecs(target){

    const objectName =
    names[target.class] ||
    target.class;


    const data =
    info[target.class] ||
    [
        "no determinado",
        "no determinada",
        "no determinada",
        "no determinado"
    ];


    const dist =
    estimateDistance(target);


    const estimatedHeight =
    Math.max(
        .03,
        Math.min(
            5,
            target.bbox[3]*
            dist/
            (video.videoWidth*.75)
        )
    );


    targetName.textContent=
    objectName;

    confidence.textContent=
    Math.round(
        target.score*100
    )+"%";

    distance.textContent=
    dist.toFixed(1)+" m aprox.";

    height.textContent=
    estimatedHeight.toFixed(2)+
    " m aprox.";

    material.textContent=
    data[0];

    manufacturing.textContent=
    data[1];

    origin.textContent=
    data[2];

    use.textContent=
    data[3];


    specs.style.display=
    "block";

}


/* =========================================
   SELECCIONAR
========================================= */

function selectTarget(target){

    if(!target)
        return;


    selected=
    target;


    smoothedBox=
    target.bbox.slice();


    scanning=true;

    scanStart=
    performance.now();


    scanText.style.display=
    "block";


    zoom.style.display=
    "block";


    showSpecs(target);


    const objectName =
    names[target.class] ||
    target.class;


    speak(
        "Objetivo identificado: "+
        objectName+
        ". Iniciando análisis."
    );

}


/* =========================================
   TOQUE
========================================= */

canvas.addEventListener(
"pointerdown",
function(event){

    if(
        !running ||
        !analysisMode
    )
        return;


    const rect =
    canvas.getBoundingClientRect();


    const x =
    (event.clientX-
    rect.left)*
    canvas.width/
    rect.width;


    const y =
    (event.clientY-
    rect.top)*
    canvas.height/
    rect.height;


    let closest=null;

    let smallest=Infinity;


    for(
        const d of detections
    ){

        const b=d.bbox;


        if(
            x>=b[0] &&
            x<=b[0]+b[2] &&
            y>=b[1] &&
            y<=b[1]+b[3]
        ){

            const area=
            b[2]*b[3];


            if(
                area<smallest
            ){

                smallest=area;

                closest=d;

            }

        }

    }


    if(closest){

        selectTarget(
            closest
        );

    }

});


/* =========================================
   SEGUIMIENTO
========================================= */

function updateSelected(){

    if(!selected)
        return null;


    const matches =
    detections.filter(
        d =>
        d.class===
        selected.class
    );


    if(!matches.length){

        /*
          OBJETO PERDIDO
        */

        selected=null;

        smoothedBox=null;

        scanning=false;


        specs.style.display=
        "none";

        zoom.style.display=
        "none";


        scanText.style.display=
        "none";


        message.textContent=
        "JARVIS listo. Puedes hablar conmigo.";


        /*
          JARVIS vuelve al modo normal
        */

        if("speechSynthesis" in window){

            speechSynthesis.cancel();

        }


        return null;

    }


    let best=
    matches[0];


    if(smoothedBox){

        const old=
        center(smoothedBox);


        let bestDistance=
        Infinity;


        for(
            const d of matches
        ){

            const c=
            center(d.bbox);


            const dist=
            Math.hypot(
                c.x-old.x,
                c.y-old.y
            );


            if(
                dist<bestDistance
            ){

                bestDistance=
                dist;

                best=d;

            }

        }

    }


    selected=
    best;


    return best;

}


/* =========================================
   ZOOM
========================================= */

function drawZoom(){

    if(
        !selected ||
        !smoothedBox
    )
        return;


    zoomCanvas.width=
    380;

    zoomCanvas.height=
    260;


    zctx.clearRect(
        0,
        0,
        380,
        260
    );


    const b=
    smoothedBox;


    const padding=.35;


    const sx=
    Math.max(
        0,
        b[0]-b[2]*padding
    );


    const sy=
    Math.max(
        0,
        b[1]-b[3]*padding
    );


    const sw=
    Math.min(
        video.videoWidth-sx,
        b[2]*(1+padding*2)
    );


    const sh=
    Math.min(
        video.videoHeight-sy,
        b[3]*(1+padding*2)
    );


    zctx.drawImage(
        video,
        sx,
        sy,
        sw,
        sh,
        0,
        0,
        380,
        260
    );


    zctx.strokeStyle=
    "#00d9ff";

    zctx.lineWidth=2;


    zctx.strokeRect(
        2,
        2,
        376,
        256
    );

}


/* =========================================
   RENDER 60 FPS
========================================= */

function render(){

    ctx.clearRect(
        0,
        0,
        canvas.width,
        canvas.height
    );


    if(analysisMode){

        let target;


        if(selected){

            target=
            updateSelected();

        }
        else{

            target=
            null;

        }


        drawTarget(target);


        if(selected){

            drawZoom();

        }

    }


    requestAnimationFrame(
        render
    );

}


/* =========================================
   BOTONES
========================================= */

start.addEventListener(
"click",
function(){

    startCamera();

});


voiceButton.addEventListener(
"click",
function(){

    listen();

});


analysisButton.addEventListener(
"click",
function(){

    analysisMode=
    !analysisMode;


    if(!analysisMode){

        selected=null;

        smoothedBox=null;

        scanning=false;


        specs.style.display=
        "none";

        zoom.style.display=
        "none";

    }


    updateMode();

});


/* =========================================
   INICIO
========================================= */

updateMode();

render();

</script>

</body>
</html>
