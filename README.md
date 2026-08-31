<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport"
      content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">

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
    font-family:monospace;
    color:#00d9ff;
}

#camera{
    position:fixed;
    inset:0;
    width:100%;
    height:100%;
    object-fit:cover;
    z-index:1;
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
    text-shadow:0 0 8px #00d9ff;
}

#title b{
    font-size:21px;
    letter-spacing:5px;
}

#title small{
    display:block;
    margin-top:3px;
    font-size:8px;
    letter-spacing:3px;
}

.corner{
    position:absolute;
    width:48px;
    height:48px;
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

#status{
    position:absolute;
    right:12px;
    bottom:12px;
    text-align:right;
    font-size:8px;
    line-height:1.8;
    text-shadow:0 0 6px #00d9ff;
}

#panel{
    position:absolute;
    left:10px;
    bottom:10px;
    width:310px;
    max-width:calc(100vw - 20px);
    max-height:40vh;
    overflow:auto;
    padding:10px;

    background:rgba(0,8,20,.88);
    border-left:2px solid #00d9ff;

    box-shadow:
        0 0 15px rgba(0,210,255,.25);

    pointer-events:auto;
}

#jarvisName{
    font-weight:bold;
    letter-spacing:2px;
}

#message{
    margin-top:5px;
    font-size:9px;
    line-height:1.5;
}

#specs{
    display:none;
    margin-top:8px;
    padding-top:7px;
    border-top:1px solid rgba(0,217,255,.4);
    font-size:8px;
    line-height:1.65;
}

.specTitle{
    font-size:10px;
    letter-spacing:2px;
    margin-bottom:4px;
}

.row{
    display:flex;
    gap:5px;
}

.label{
    width:80px;
    opacity:.55;
    flex-shrink:0;
}

button{
    margin-top:8px;
    padding:9px 12px;
    background:rgba(0,35,60,.9);
    border:1px solid #00d9ff;
    border-radius:5px;
    color:#00d9ff;
    font-family:monospace;
    font-size:9px;
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

    box-shadow:0 0 15px rgba(0,210,255,.4);

    overflow:hidden;
}

#zoomCanvas{
    width:100%;
    height:100%;
}
</style>
</head>

<body>

<video id="camera" autoplay muted playsinline></video>

<canvas id="canvas"></canvas>

<div id="hud">

    <div class="corner tl"></div>
    <div class="corner tr"></div>
    <div class="corner bl"></div>
    <div class="corner br"></div>

    <div id="title">
        <b>J.A.R.V.I.S.</b>
        <small>VISION SYSTEM</small>
    </div>

    <div id="scanText">
        ANALIZANDO...
    </div>

    <div id="zoom">
        <canvas id="zoomCanvas"></canvas>
    </div>

    <div id="panel">

        <div id="jarvisName">
            JARVIS
        </div>

        <div id="message">
            Apunta la cámara y toca un objeto para analizarlo.
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

    </div>

    <div id="status">

        SISTEMA: <span id="system">ONLINE</span><br>
        CÁMARA: <span id="cam">OFF</span><br>
        IA: <span id="ai">OFF</span><br>
        OBJETIVO: <span id="target">NINGUNO</span>

    </div>

</div>

<button id="start">
    ACTIVAR JARVIS
</button>


<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>
<script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/coco-ssd"></script>

<script>

/* =====================================================
   ELEMENTOS
===================================================== */

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

const system =
document.getElementById("system");

const cam =
document.getElementById("cam");

const ai =
document.getElementById("ai");

const targetStatus =
document.getElementById("target");

const nameEl =
document.getElementById("name");

const confidenceEl =
document.getElementById("confidence");

const distanceEl =
document.getElementById("distance");

const heightEl =
document.getElementById("height");

const materialEl =
document.getElementById("material");

const manufacturingEl =
document.getElementById("manufacturing");

const originEl =
document.getElementById("origin");

const useEl =
document.getElementById("use");


/* =====================================================
   VARIABLES
===================================================== */

let model = null;

let running = false;

let detections = [];

let selected = null;

let smoothedBox = null;

let lastDetection = 0;

let scanning = false;

let scanStart = 0;

let voices = [];


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
   DATOS
===================================================== */

const info = {

bottle:[
"plástico PET, vidrio o aluminio",
"moldeado y fabricación industrial",
"variable según la marca",
"almacenar líquidos"
],

cup:[
"cerámica, vidrio, plástico o papel",
"moldeado o fabricación industrial",
"variable según fabricante",
"beber líquidos"
],

chair:[
"madera, plástico o metal",
"carpintería o fabricación industrial",
"variable según fabricante",
"sentarse"
],

table:[
"madera, vidrio o metal",
"carpintería y fabricación industrial",
"variable según fabricante",
"superficie de apoyo"
],

dining_table:[
"madera, vidrio o metal",
"carpintería y fabricación industrial",
"variable según fabricante",
"comer o apoyar objetos"
],

laptop:[
"aluminio, plástico, vidrio y electrónica",
"ensamblaje electrónico industrial",
"variable según marca y modelo",
"computación"
],

cell_phone:[
"vidrio, metal, plástico y electrónica",
"ensamblaje electrónico industrial",
"variable según marca y modelo",
"comunicación y computación"
],

keyboard:[
"plástico, metal y electrónica",
"moldeado y ensamblaje electrónico",
"variable según fabricante",
"entrada de texto"
],

mouse:[
"plástico y componentes electrónicos",
"moldeado y ensamblaje",
"variable según fabricante",
"control de computadora"
],

book:[
"papel, tinta y cartón",
"impresión y encuadernación",
"depende de la editorial",
"lectura"
],

backpack:[
"nylon, poliéster o materiales sintéticos",
"confección textil",
"variable según fabricante",
"transportar objetos"
],

car:[
"acero, aluminio, plástico, vidrio y electrónica",
"producción automotriz",
"variable según marca y modelo",
"transporte"
],

bicycle:[
"aluminio, acero, carbono y caucho",
"fabricación mecánica y ensamblaje",
"variable según fabricante",
"transporte"
],

dog:[
"ser vivo",
"no aplica",
"animal; ubicación y raza determinan procedencia",
"animal doméstico"
],

cat:[
"ser vivo",
"no aplica",
"animal; ubicación y raza determinan procedencia",
"animal doméstico"
],

person:[
"ser vivo",
"no aplica",
"persona",
"persona"
]

};


/* =====================================================
   ALTURAS PROMEDIO
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


/* =====================================================
   VOCES
===================================================== */

function loadVoices(){

    voices =
    window.speechSynthesis
    ?
    window.speechSynthesis.getVoices()
    :
    [];

}

if("speechSynthesis" in window){

    loadVoices();

    speechSynthesis.onvoiceschanged =
    loadVoices;

}


/* =====================================================
   HABLAR
===================================================== */

function speak(text){

    if(!("speechSynthesis" in window)){

        message.textContent =
        "Tu navegador no permite voz.";

        return;

    }

    speechSynthesis.cancel();

    if(!voices.length){

        loadVoices();

    }

    const utterance =
    new SpeechSynthesisUtterance(text);

    utterance.lang = "es-ES";

    utterance.rate = .88;

    utterance.pitch = .75;

    utterance.volume = 1;

    let spanish =
    voices.find(v =>
        v.lang &&
        v.lang.toLowerCase().startsWith("es")
    );

    if(spanish){

        utterance.voice = spanish;

    }

    utterance.onstart = ()=>{

        message.textContent =
        "JARVIS: transmitiendo información...";

    };

    utterance.onend = ()=>{

        message.textContent =
        "Análisis completado.";

    };

    speechSynthesis.speak(
        utterance
    );

}


/* =====================================================
   CÁMARA
===================================================== */

async function startCamera(){

    try{

        const stream =
        await navigator.mediaDevices.getUserMedia({

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

        running = true;

        start.style.display =
        "none";

        cam.textContent =
        "ON";

        message.textContent =
        "Cámara activa. Toca un objeto.";

        await startAI();

    }

    catch(error){

        console.error(error);

        message.textContent =
        "No se pudo activar la cámara. Revisa los permisos.";

    }

}


/* =====================================================
   RESIZE
===================================================== */

function resizeCanvas(){

    if(!video.videoWidth)return;

    canvas.width =
    video.videoWidth;

    canvas.height =
    video.videoHeight;

}

window.addEventListener(
"resize",
resizeCanvas
);


/* =====================================================
   IA
===================================================== */

async function startAI(){

    try{

        ai.textContent =
        "CARGANDO";

        await tf.ready();

        model =
        await cocoSsd.load({
            base:"mobilenet_v2"
        });

        ai.textContent =
        "ONLINE";

        message.textContent =
        "IA lista. Toca un objeto.";

        detectionLoop();

    }

    catch(error){

        console.error(error);

        ai.textContent =
        "ERROR";

    }

}


/* =====================================================
   DETECCIÓN
===================================================== */

async function detectionLoop(){

    if(!running ||
       !model){

        requestAnimationFrame(
        detectionLoop
        );

        return;

    }

    const now =
    performance.now();

    /*
      La IA analiza periódicamente,
      mientras el dibujo funciona
      a 60 FPS.
    */

    if(
        now-lastDetection > 90
    ){

        lastDetection =
        now;

        try{

            detections =
            await model.detect(
                video,
                25,
                .30
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


/* =====================================================
   CENTRO
===================================================== */

function center(box){

    return {

        x:box[0]+box[2]/2,

        y:box[1]+box[3]/2

    };

}


/* =====================================================
   DISTANCIA
===================================================== */

function estimateDistance(d){

    const realHeight =
    heights[d.class] || .5;

    const focal =
    video.videoWidth * .75;

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


/* =====================================================
   CAJA MÁS CERCANA AL CENTRO
===================================================== */

function getBestTarget(){

    if(!detections.length)
        return null;

    const cx =
    canvas.width/2;

    const cy =
    canvas.height/2;

    let best = null;

    let bestScore = -Infinity;

    for(const d of detections){

        if(d.score < .35)
            continue;

        const c =
        center(d.bbox);

        const dist =
        Math.hypot(
            c.x-cx,
            c.y-cy
        );

        const normalized =
        dist /
        Math.max(
            canvas.width,
            canvas.height
        );

        const centerScore =
        1-normalized;

        const size =
        Math.min(
            1,
            d.bbox[2]*
            d.bbox[3]/
            (canvas.width*
             canvas.height*.30)
        );

        const score =
        d.score*.55+
        centerScore*.35+
        size*.10;

        if(score>bestScore){

            bestScore =
            score;

            best = d;

        }

    }

    return best;

}


/* =====================================================
   INTERPOLACIÓN
===================================================== */

function lerp(a,b,t){

    return a+(b-a)*t;

}


/* =====================================================
   CAJA SUAVE
===================================================== */

function smoothBox(current,target){

    if(!current)
        return target.slice();

    /*
       Suavizado fuerte.
       Evita los movimientos bruscos.
    */

    const t=.16;

    return [

        lerp(
            current[0],
            target[0],
            t
        ),

        lerp(
            current[1],
            target[1],
            t
        ),

        lerp(
            current[2],
            target[2],
            t
        ),

        lerp(
            current[3],
            target[3],
            t
        )

    ];

}


/* =====================================================
   DIBUJAR MARCO PEQUEÑO
===================================================== */

function drawCorners(x,y,w,h){

    const size =
    Math.min(
        22,
        Math.max(
            8,
            Math.min(w,h)*.22
        )
    );

    ctx.beginPath();

    /* esquina superior izquierda */

    ctx.moveTo(
        x,
        y+size
    );

    ctx.lineTo(
        x,
        y
    );

    ctx.lineTo(
        x+size,
        y
    );

    /* superior derecha */

    ctx.moveTo(
        x+w-size,
        y
    );

    ctx.lineTo(
        x+w,
        y
    );

    ctx.lineTo(
        x+w,
        y+size
    );

    /* inferior izquierda */

    ctx.moveTo(
        x,
        y+h-size
    );

    ctx.lineTo(
        x,
        y+h
    );

    ctx.lineTo(
        x+size,
        y+h
    );

    /* inferior derecha */

    ctx.moveTo(
        x+w-size,
        y+h
    );

    ctx.lineTo(
        x+w,
        y+h
    );

    ctx.lineTo(
        x+w,
        y+h-size
    );

    ctx.stroke();

}


/* =====================================================
   ESCANEO
===================================================== */

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

    ctx.strokeStyle =
    "#00eaff";

    ctx.lineWidth =
    3;

    ctx.shadowColor =
    "#00eaff";

    ctx.shadowBlur =
    18;

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

        scanText.style.display =
        "none";

    }

}


/* =====================================================
   DIBUJAR OBJETIVO
===================================================== */

function drawTarget(target){

    if(!target)
        return;

    smoothedBox =
    smoothBox(
        smoothedBox,
        target.bbox
    );

    const box =
    smoothedBox;

    const x=box[0];
    const y=box[1];
    const w=box[2];
    const h=box[3];

    ctx.save();

    ctx.strokeStyle =
    "#00d9ff";

    ctx.lineWidth =
    2;

    ctx.shadowColor =
    "#00d9ff";

    ctx.shadowBlur =
    10;

    drawCorners(
        x,y,w,h
    );

    ctx.restore();

    drawScan(box);

}


/* =====================================================
   MOSTRAR DATOS
===================================================== */

function showSpecs(target){

    if(!target)
        return;

    const objectName =
    names[target.class] ||
    target.class;

    const data =
    info[target.class] ||
    [
        "material no determinado",
        "fabricación no determinada",
        "procedencia no determinada",
        "uso no determinado"
    ];

    const distance =
    estimateDistance(target);

    const estimatedHeight =
    Math.max(
        .03,
        Math.min(
            5,
            target.bbox[3]*
            distance/
            (video.videoWidth*.75)
        )
    );

    nameEl.textContent =
    objectName;

    confidenceEl.textContent =
    Math.round(
        target.score*100
    )+"%";

    distanceEl.textContent =
    distance.toFixed(1)+" m aprox.";

    heightEl.textContent =
    estimatedHeight.toFixed(2)+" m aprox.";

    materialEl.textContent =
    data[0];

    manufacturingEl.textContent =
    data[1];

    originEl.textContent =
    data[2];

    useEl.textContent =
    data[3];

    specs.style.display =
    "block";

    targetStatus.textContent =
    objectName.toUpperCase();

}


/* =====================================================
   SELECCIONAR OBJETO
===================================================== */

function selectTarget(target){

    if(!target)
        return;

    selected =
    target;

    smoothedBox =
    target.bbox.slice();

    scanning = true;

    scanStart =
    performance.now();

    scanText.style.display =
    "block";

    zoom.style.display =
    "block";

    showSpecs(target);

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

    const distance =
    estimateDistance(target);

    const estimatedHeight =
    target.bbox[3]*
    distance/
    (video.videoWidth*.75);

    const speech =

    "Objetivo identificado: "+
    objectName+
    ". Confianza "+
    Math.round(target.score*100)+
    " por ciento. "+
    "Distancia aproximada "+
    distance.toFixed(1)+
    " metros. "+
    "Altura estimada "+
    estimatedHeight.toFixed(2)+
    " metros. "+
    "Material: "+
    data[0]+
    ". Fabricación: "+
    data[1]+
    ". Procedencia: "+
    data[2]+
    ". Uso: "+
    data[3]+".";

    /*
      Pequeño retraso para garantizar
      que Safari haya recibido el toque.
    */

    setTimeout(
        ()=>{
            speak(speech);
        },
        100
    );

}


/* =====================================================
   TOQUE EN PANTALLA
===================================================== */

canvas.addEventListener(
"pointerdown",
function(event){

    if(!running)
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

    let closest =
    null;

    let smallest =
    Infinity;

    for(const d of detections){

        const bx=d.bbox[0];
        const by=d.bbox[1];
        const bw=d.bbox[2];
        const bh=d.bbox[3];

        if(
            x>=bx &&
            x<=bx+bw &&
            y>=by &&
            y<=by+bh
        ){

            const area =
            bw*bh;

            if(area<smallest){

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


/* =====================================================
   BOTÓN DE VOZ
===================================================== */

voiceButton.addEventListener(
"click",
function(){

    /*
      Este botón también desbloquea
      speechSynthesis en algunos móviles.
    */

    if(!selected){

        speak(
        "Selecciona un objeto primero."
        );

        return;

    }

    const objectName =
    names[selected.class] ||
    selected.class;

    const data =
    info[selected.class] ||
    [
        "no determinado",
        "no determinada",
        "no determinada",
        "no determinado"
    ];

    speak(
        "El objetivo seleccionado es "+
        objectName+
        ". "+
        "Material: "+
        data[0]+
        ". "+
        "Fabricación: "+
        data[1]+
        ". "+
        "Procedencia: "+
        data[2]+
        ". "+
        "Uso: "+
        data[3]+"."
    );

});


/* =====================================================
   ZOOM
===================================================== */

function drawZoom(){

    if(!selected ||
       !smoothedBox)
        return;

    zoomCanvas.width =
    380;

    zoomCanvas.height =
    260;

    zctx.clearRect(
        0,
        0,
        380,
        260
    );

    const b =
    smoothedBox;

    const padding =
    .35;

    let sx =
    Math.max(
        0,
        b[0]-b[2]*padding
    );

    let sy =
    Math.max(
        0,
        b[1]-b[3]*padding
    );

    let sw =
    Math.min(
        video.videoWidth-sx,
        b[2]*(1+padding*2)
    );

    let sh =
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

    zctx.strokeStyle =
    "#00d9ff";

    zctx.lineWidth =
    2;

    zctx.strokeRect(
        2,
        2,
        376,
        256
    );

}


/* =====================================================
   RENDER A 60 FPS
===================================================== */

function render(){

    ctx.clearRect(
        0,
        0,
        canvas.width,
        canvas.height
    );

    /*
      SOLO se dibuja una detección:
      el objetivo seleccionado.
      Si no hay selección,
      se muestra el objetivo más cercano
      al centro.
    */

    let target =
    selected ||
    getBestTarget();

    /*
      Si el usuario seleccionó algo,
      buscamos la detección actual de
      la misma clase más cercana a la
      caja anterior para mantenerla.
    */

    if(selected){

        let matching =
        detections
        .filter(
            d =>
            d.class === selected.class
        );

        if(matching.length){

            let best =
            matching[0];

            let bestDist =
            Infinity;

            const old =
            smoothedBox ||
            selected.bbox;

            const oldCenter =
            center(old);

            for(const d of matching){

                const c =
                center(d.bbox);

                const dist =
                Math.hypot(
                    c.x-oldCenter.x,
                    c.y-oldCenter.y
                );

                if(dist<bestDist){

                    bestDist=dist;
                    best=d;

                }

            }

            selected =
            best;

        }

    }

    drawTarget(target);

    if(selected){

        showSpecs(selected);

        drawZoom();

    }

    requestAnimationFrame(
    render
    );

}


/* =====================================================
   INICIAR
===================================================== */

start.addEventListener(
"click",
startCamera
);

render();

</script>

</body>
</html>
