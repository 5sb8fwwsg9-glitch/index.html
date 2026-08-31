<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">

<title>JARVIS VISION 60FPS</title>

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
    text-shadow:0 0 8px #00d9ff,0 0 20px #008cff;
}

#title b{
    font-size:23px;
    letter-spacing:5px;
}

#title small{
    display:block;
    margin-top:3px;
    font-size:9px;
    letter-spacing:3px;
}

.corner{
    position:absolute;
    width:55px;
    height:55px;
    border-color:#00d9ff;
    filter:drop-shadow(0 0 7px #00d9ff);
}

.tl{
    top:12px;
    left:12px;
    border-top:2px solid;
    border-left:2px solid;
}

.tr{
    top:12px;
    right:12px;
    border-top:2px solid;
    border-right:2px solid;
}

.bl{
    bottom:12px;
    left:12px;
    border-bottom:2px solid;
    border-left:2px solid;
}

.br{
    bottom:12px;
    right:12px;
    border-bottom:2px solid;
    border-right:2px solid;
}

#panel{
    position:absolute;
    left:10px;
    bottom:10px;
    width:280px;
    padding:11px;
    background:rgba(0,10,25,.84);
    border-left:3px solid #00d9ff;
    box-shadow:0 0 20px rgba(0,200,255,.25);
    backdrop-filter:blur(5px);
    pointer-events:auto;
}

#jarvisName{
    font-weight:bold;
    letter-spacing:2px;
}

#message{
    min-height:35px;
    margin-top:6px;
    font-size:10px;
    line-height:1.45;
}

button{
    color:#00d9ff;
    background:rgba(0,40,70,.9);
    border:1px solid #00d9ff;
    border-radius:5px;
    padding:9px 12px;
    font-family:monospace;
}

#status{
    position:absolute;
    right:10px;
    bottom:10px;
    text-align:right;
    font-size:9px;
    line-height:1.8;
    text-shadow:0 0 7px #00d9ff;
}

#analysis{
    position:absolute;
    top:65px;
    right:12px;
    width:190px;
    height:150px;
    display:none;
    background:rgba(0,10,25,.88);
    border:1px solid #00d9ff;
    box-shadow:0 0 18px rgba(0,210,255,.45);
    overflow:hidden;
}

#zoomCanvas{
    width:100%;
    height:100%;
}

#analysisText{
    position:absolute;
    left:5px;
    bottom:5px;
    padding:4px 6px;
    background:rgba(0,10,25,.75);
    font-size:9px;
}

#scanText{
    position:absolute;
    top:38%;
    left:50%;
    transform:translate(-50%,-50%);
    display:none;
    font-size:13px;
    letter-spacing:3px;
    text-shadow:0 0 10px #00d9ff;
}

#fps{
    position:absolute;
    top:12px;
    right:75px;
    font-size:9px;
}

#start{
    position:fixed;
    z-index:20;
    top:50%;
    left:50%;
    transform:translate(-50%,-50%);
    padding:18px 25px;
    color:#00d9ff;
    background:rgba(0,10,25,.97);
    border:1px solid #00d9ff;
    border-radius:6px;
    box-shadow:0 0 25px #008cff;
    font-family:monospace;
    font-size:15px;
}
</style>
</head>

<body>

<video id="camera" autoplay playsinline muted></video>

<canvas id="canvas"></canvas>

<div id="hud">

<div class="corner tl"></div>
<div class="corner tr"></div>
<div class="corner bl"></div>
<div class="corner br"></div>

<div id="title">
<b>J.A.R.V.I.S.</b>
<small>ADVANCED VISION SYSTEM</small>
</div>

<div id="fps">FPS: --</div>

<div id="analysis">
<canvas id="zoomCanvas"></canvas>
<div id="analysisText">SIN OBJETIVO</div>
</div>

<div id="scanText">ANALIZANDO OBJETIVO...</div>

<div id="panel">

<div id="jarvisName">JARVIS</div>

<div id="message">
Sistema preparado.
</div>

<button id="voice">
🎙 HABLAR CON JARVIS
</button>

</div>

<div id="status">

SISTEMA: ONLINE<br>
CÁMARA: <span id="cameraStatus">OFF</span><br>
IA: <span id="aiStatus">OFF</span><br>
TRACKING: <span id="tracking">OFF</span><br>
OBJETOS: <span id="objectCount">0</span><br>
OBJETIVO: <span id="targetStatus">NINGUNO</span><br>
MIC: <span id="micStatus">OFF</span>

</div>

</div>

<button id="start">
ACTIVAR JARVIS
</button>

<script src="https://cdn.jsdelivr.net/npm/@tensorflow/tfjs"></script>
<script src="https://cdn.jsdelivr.net/npm/@tensorflow-models/coco-ssd"></script>

<script>

/* =========================================================
   ELEMENTOS
========================================================= */

const video=document.getElementById("camera");
const canvas=document.getElementById("canvas");
const ctx=canvas.getContext("2d");

const zoomCanvas=document.getElementById("zoomCanvas");
const zctx=zoomCanvas.getContext("2d");

const start=document.getElementById("start");
const voice=document.getElementById("voice");

const message=document.getElementById("message");
const analysis=document.getElementById("analysis");
const analysisText=document.getElementById("analysisText");
const scanText=document.getElementById("scanText");

const cameraStatus=document.getElementById("cameraStatus");
const aiStatus=document.getElementById("aiStatus");
const trackingStatus=document.getElementById("tracking");

const objectCount=document.getElementById("objectCount");
const targetStatus=document.getElementById("targetStatus");
const micStatus=document.getElementById("micStatus");
const fpsText=document.getElementById("fps");


/* =========================================================
   CONFIGURACIÓN
========================================================= */

/*
   La IA detecta aproximadamente cada 100-130 ms.

   El HUD NO espera a la IA.

   El HUD se dibuja continuamente con requestAnimationFrame,
   normalmente hasta 60 FPS.
*/

const DETECTION_INTERVAL=110;


/*
   Suavizado visual.

   Cuanto menor:
   más suave.

   Cuanto mayor:
   más rápido responde.

   0.10 - 0.18 funciona bien para cámara móvil.
*/

const SMOOTH=.115;


/*
   Predicción del movimiento.
*/

const PREDICTION=.035;


/*
   Persistencia.

   Aumentado respecto a la versión anterior.
*/

const MAX_MISSED=55;


/*
   Confianza mínima.

   0.25 permite detectar objetos pequeños.
*/

const MIN_SCORE=.25;


/* =========================================================
   VARIABLES
========================================================= */

let model=null;
let active=false;
let detecting=false;

let tracks=[];
let nextID=1;

let selectedID=null;


/* =========================================================
   ESCANEO
========================================================= */

let scan={
    active:false,
    id:null,
    progress:0,
    duration:1400,
    start:0
};


/* =========================================================
   FPS
========================================================= */

let frames=0;
let fpsLast=performance.now();


/* =========================================================
   NOMBRES
========================================================= */

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


/* =========================================================
   ALTURAS APROXIMADAS
========================================================= */

const heights={

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


/* =========================================================
   CÁMARA
========================================================= */

async function startCamera(){

try{

const stream=
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

video.srcObject=stream;

await video.play();

resize();

active=true;

start.style.display="none";

cameraStatus.textContent="ON";

message.textContent=
"Cámara activada. Preparando visión artificial...";

loadAI();

}

catch(error){

console.error(error);

message.textContent=
"No se pudo acceder a la cámara.";

}

}


/* =========================================================
   RESIZE
========================================================= */

function resize(){

if(!video.videoWidth)
return;

canvas.width=video.videoWidth;
canvas.height=video.videoHeight;

}

window.addEventListener("resize",resize);


/* =========================================================
   IA
========================================================= */

async function loadAI(){

try{

aiStatus.textContent="CARGANDO";

await tf.ready();

model=await cocoSsd.load({
base:"mobilenet_v2"
});

aiStatus.textContent="ONLINE";
trackingStatus.textContent="60FPS";

message.textContent=
"Visión artificial activada.";

detectionLoop();

}

catch(error){

console.error(error);

aiStatus.textContent="ERROR";

message.textContent=
"Error cargando la IA.";

}

}


/* =========================================================
   IOU
========================================================= */

function IoU(a,b){

const x1=Math.max(a[0],b[0]);
const y1=Math.max(a[1],b[1]);

const x2=Math.min(
a[0]+a[2],
b[0]+b[2]
);

const y2=Math.min(
a[1]+a[3],
b[1]+b[3]
);

const inter=
Math.max(0,x2-x1)*
Math.max(0,y2-y1);

const union=
a[2]*a[3]+
b[2]*b[3]-
inter;

return union>0?inter/union:0;

}


/* =========================================================
   CENTRO
========================================================= */

function center(box){

return{

x:box[0]+box[2]/2,
y:box[1]+box[3]/2

};

}


/* =========================================================
   DISTANCIA ENTRE CENTROS
========================================================= */

function centerDistance(a,b){

const A=center(a);
const B=center(b);

return Math.hypot(
A.x-B.x,
A.y-B.y
);

}


/* =========================================================
   SUAVIZADO
========================================================= */

function smooth(current,target){

return[

current[0]+
(target[0]-current[0])*SMOOTH,

current[1]+
(target[1]-current[1])*SMOOTH,

current[2]+
(target[2]-current[2])*SMOOTH,

current[3]+
(target[3]-current[3])*SMOOTH

];

}


/* =========================================================
   TRACKING
========================================================= */

function updateTracking(predictions){

tracks.forEach(t=>{
t.missed++;
});


predictions.forEach(pred=>{

let best=null;
let bestScore=0;

tracks.forEach(track=>{

if(track.className!==pred.class)
return;


/*
   Comparación por IoU.
*/

const iou=
IoU(
track.target,
pred.bbox
);


/*
   Comparación por distancia.
*/

const distance=
centerDistance(
track.target,
pred.bbox
);


const maxDistance=
Math.max(
140,
track.target[2]*1.8
);


/*
   Score combinado.
*/

const movementScore=
Math.max(
0,
1-distance/maxDistance
);


const score=
iou+
movementScore*.55;


if(score>bestScore){

bestScore=score;
best=track;

}

});


if(best){

/*
   Calculamos velocidad aproximada.

   Esto permite predecir ligeramente
   hacia dónde se está moviendo el objeto.
*/

const old=center(best.target);
const now=center(pred.bbox);

best.velocityX=
(now.x-old.x);

best.velocityY=
(now.y-old.y);


/*
   Guardamos la nueva posición objetivo.
*/

best.target=pred.bbox.slice();

best.score=pred.score;

best.missed=0;

best.age++;

}

else{

tracks.push({

id:nextID++,

className:pred.class,

box:pred.bbox.slice(),

target:pred.bbox.slice(),

score:pred.score,

velocityX:0,
velocityY:0,

missed:0,

age:1

});

}

});


/*
   Predicción + suavizado.

   La caja visual nunca salta directamente
   a la detección.
*/

tracks.forEach(track=>{

let predicted=[
track.target[0]+
track.velocityX*
PREDICTION,

track.target[1]+
track.velocityY*
PREDICTION,

track.target[2],

track.target[3]
];


/*
   Si el objeto se perdió,
   mantenemos el movimiento anterior
   durante un pequeño período.
*/

if(track.missed>0){

predicted=[
track.target[0]+
track.velocityX*
Math.min(
track.missed,
12
),

track.target[1]+
track.velocityY*
Math.min(
track.missed,
12
),

track.target[2],

track.target[3]

];

}


track.box=
smooth(
track.box,
predicted
);

});


/*
   Eliminamos solamente objetos
   que llevan bastante tiempo perdidos.
*/

tracks=
tracks.filter(
t=>t.missed<=MAX_MISSED
);


if(
selectedID!==null &&
!tracks.some(
t=>t.id===selectedID
)
){

selectedID=null;

analysis.style.display="none";

targetStatus.textContent=
"NINGUNO";

}

}


/* =========================================================
   DETECCIÓN
========================================================= */

async function detectionLoop(){

if(
!active||
!model
){

setTimeout(
detectionLoop,
DETECTION_INTERVAL
);

return;

}


if(!detecting){

detecting=true;

try{

const predictions=
await model.detect(
video,
30,
MIN_SCORE
);

updateTracking(predictions);

objectCount.textContent=
tracks.length;

}

catch(error){

console.error(error);

}

detecting=false;

}


setTimeout(
detectionLoop,
DETECTION_INTERVAL
);

}


/* =========================================================
   DISTANCIA
========================================================= */

function getDistance(track){

const realHeight=
heights[
track.className
]||.5;


/*
   Esta distancia es una ESTIMACIÓN.

   La cámara no puede medir metros reales
   de forma fiable solamente con una imagen.
*/

const focal=
Math.max(
500,
video.videoWidth*.75
);


const distance=
realHeight*
focal/
Math.max(
10,
track.box[3]
);


return Math.max(
.2,
Math.min(
30,
distance
)
);

}


/* =========================================================
   ALTURA
========================================================= */

function getHeight(track,distance){

const focal=
Math.max(
500,
video.videoWidth*.75
);


const height=
track.box[3]*
distance/
focal;


return Math.max(
.03,
Math.min(
5,
height
)
);

}


/* =========================================================
   ESQUINAS
========================================================= */

function drawCorner(x,y,size,dx,dy){

ctx.beginPath();

ctx.moveTo(
x,
y+dy*size
);

ctx.lineTo(
x,
y
);

ctx.lineTo(
x+dx*size,
y
);

ctx.stroke();

}


/* =========================================================
   ESCANEO
========================================================= */

function drawScan(track){

if(
!scan.active||
scan.id!==track.id
)
return;


const elapsed=
performance.now()-
scan.start;

scan.progress=
Math.min(
1,
elapsed/scan.duration
);


const x=track.box[0];
const y=track.box[1];

const w=track.box[2];
const h=track.box[3];


const scanY=
y+h*scan.progress;


/*
   Línea principal.
*/

ctx.save();

ctx.shadowColor="#00eaff";
ctx.shadowBlur=25;

ctx.strokeStyle="#00eaff";
ctx.lineWidth=3;

ctx.beginPath();

ctx.moveTo(
x-15,
scanY
);

ctx.lineTo(
x+w+15,
scanY
);

ctx.stroke();


/*
   Halo.
*/

ctx.shadowBlur=0;

ctx.strokeStyle=
"rgba(0,220,255,.25)";

ctx.lineWidth=14;

ctx.beginPath();

ctx.moveTo(
x-10,
scanY
);

ctx.lineTo(
x+w+10,
scanY
);

ctx.stroke();

ctx.restore();


/*
   Final de escaneo.
*/

if(scan.progress>=1){

scan.active=false;

scan.id=null;

scanText.style.display="none";

}

}


/* =========================================================
   DIBUJAR OBJETOS
========================================================= */

function draw(){

ctx.clearRect(
0,
0,
canvas.width,
canvas.height
);


tracks.forEach(track=>{

const x=track.box[0];
const y=track.box[1];

const w=track.box[2];
const h=track.box[3];

const selected=
track.id===selectedID;


/*
   Objetos perdidos permanecen,
   pero se vuelven ligeramente transparentes.
*/

ctx.globalAlpha=
Math.max(
.38,
1-track.missed*.012
);


/*
   Caja.
*/

ctx.strokeStyle=
selected
?"#ffffff"
:"#00d9ff";

ctx.lineWidth=
selected?4:2.5;

ctx.shadowColor="#00d9ff";

ctx.shadowBlur=
selected?20:9;

ctx.strokeRect(
x,
y,
w,
h
);

ctx.shadowBlur=0;


/*
   Esquinas HUD.
*/

const corner=
Math.min(
24,
Math.max(
10,
Math.min(w,h)*.18
)
);

drawCorner(
x,
y,
corner,
1,
1
);

drawCorner(
x+w,
y,
corner,
-1,
1
);

drawCorner(
x,
y+h,
corner,
1,
-1
);

drawCorner(
x+w,
y+h,
corner,
-1,
-1
);


/*
   Información.
*/

const distance=
getDistance(track);

const height=
getHeight(
track,
distance
);

const name=
names[
track.className
]||
track.className;


const pw=
Math.max(
190,
Math.min(
245,
w
)
);

const ph=82;


let py=
y-ph-5;

if(py<5)
py=y+5;


ctx.fillStyle=
"rgba(0,10,25,.90)";

ctx.fillRect(
x,
py,
pw,
ph
);

ctx.strokeStyle="#00d9ff";
ctx.lineWidth=1;

ctx.strokeRect(
x,
py,
pw,
ph
);


ctx.fillStyle="#00eaff";

ctx.font=
"bold 14px monospace";

ctx.fillText(
name.toUpperCase(),
x+7,
py+18
);

ctx.font=
"11px monospace";

ctx.fillText(
"ID: #"+
track.id+
"  CONF: "+
(track.score*100).toFixed(0)+
"%",
x+7,
py+36
);

ctx.fillText(
"DIST: "+
distance.toFixed(1)+
" m",
x+7,
py+53
);

ctx.fillText(
"ALTURA: "+
height.toFixed(2)+
" m",
x+7,
py+70
);


/*
   Escaneo.
*/

drawScan(track);

ctx.globalAlpha=1;

});

}


/* =========================================================
   SELECCIÓN
========================================================= */

function selectObject(track){

selectedID=track.id;

targetStatus.textContent=
"#"+track.id;


const name=
names[
track.className
]||
track.className;


message.textContent=
"Objetivo seleccionado: "+
name;


/*
   Inicia exactamente una pasada.
*/

scan.active=true;

scan.id=track.id;

scan.progress=0;

scan.start=
performance.now();

scanText.style.display=
"block";


speak(
"Objetivo seleccionado. Analizando "+
name+"."
);

}


/* =========================================================
   TOUCH
========================================================= */

canvas.addEventListener(
"pointerdown",
event=>{

if(!active)
return;


const rect=
canvas.getBoundingClientRect();


const px=
(event.clientX-rect.left)*
canvas.width/
rect.width;


const py=
(event.clientY-rect.top)*
canvas.height/
rect.height;


for(
let i=tracks.length-1;
i>=0;
i--
){

const t=tracks[i];

if(

px>=t.box[0]&&
px<=t.box[0]+t.box[2]&&
py>=t.box[1]&&
py<=t.box[1]+t.box[3]

){

selectObject(t);

break;

}

}

}
);


/* =========================================================
   ZOOM
========================================================= */

function drawZoom(track){

if(!track)
return;


analysis.style.display=
"block";


const name=
names[
track.className
]||
track.className;


analysisText.textContent=
name.toUpperCase()+
" • ID #"+
track.id;


zoomCanvas.width=380;
zoomCanvas.height=300;


zctx.clearRect(
0,
0,
380,
300
);


const pad=.25;


const sx=
Math.max(
0,
track.box[0]-
track.box[2]*pad
);

const sy=
Math.max(
0,
track.box[1]-
track.box[3]*pad
);


const sw=
Math.min(
video.videoWidth-sx,
track.box[2]*
(1+pad*2)
);

const sh=
Math.min(
video.videoHeight-sy,
track.box[3]*
(1+pad*2)
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
300
);


/*
   Marco.
*/

zctx.strokeStyle="#00d9ff";
zctx.lineWidth=2;

zctx.shadowColor="#00d9ff";
zctx.shadowBlur=12;

zctx.strokeRect(
3,
3,
374,
294
);

zctx.shadowBlur=0;


/*
   Línea de escaneo en el zoom.
*/

if(
scan.active &&
scan.id===track.id
){

const sy2=
300*scan.progress;

zctx.strokeStyle="#00eaff";

zctx.lineWidth=3;

zctx.shadowColor="#00eaff";

zctx.shadowBlur=15;

zctx.beginPath();

zctx.moveTo(
0,
sy2
);

zctx.lineTo(
380,
sy2
);

zctx.stroke();

zctx.shadowBlur=0;

}

}


/* =========================================================
   VOZ
========================================================= */

function speak(text){

if(!window.speechSynthesis)
return;

speechSynthesis.cancel();

const utterance=
new SpeechSynthesisUtterance(text);

utterance.lang="es-ES";
utterance.rate=.9;
utterance.pitch=.72;

speechSynthesis.speak(
utterance
);

}


function listen(){

const Recognition=
window.SpeechRecognition||
window.webkitSpeechRecognition;


if(!Recognition){

message.textContent=
"Reconocimiento de voz no disponible.";

micStatus.textContent=
"NO DISP.";

return;

}


const recognition=
new Recognition();

recognition.lang="es-ES";

recognition.continuous=false;

recognition.interimResults=false;


micStatus.textContent=
"ESCUCHANDO";

message.textContent=
"Te escucho...";


try{

recognition.start();

}

catch(error){

console.log(error);

return;

}


recognition.onresult=
event=>{

const command=
event.results[0][0]
.transcript
.toLowerCase();

micStatus.textContent="ON";

processCommand(command);

};


recognition.onerror=
event=>{

console.log(event.error);

micStatus.textContent=
"ERROR";

message.textContent=
"Error de micrófono. Revisa los permisos del navegador.";

};


recognition.onend=
()=>{

setTimeout(
()=>{
micStatus.textContent="OFF";
},
1000
);

};

}


/* =========================================================
   COMANDOS
========================================================= */

function processCommand(command){

if(
command.includes("hola")||
command.includes("jarvis")
){

respond(
"A sus órdenes. Sistemas operativos."
);

return;

}


if(
command.includes("qué ves")||
command.includes("que ves")||
command.includes("qué hay")||
command.includes("que hay")
){

describeScene();

return;

}


if(
command.includes("cuántos")||
command.includes("cuantos")
){

respond(
"Detecto "+
tracks.length+
" objetos."
);

return;

}


if(
command.includes("selecciona")||
command.includes("seleccionar")
){

if(tracks.length){

selectObject(
tracks[0]
);

}
else{

respond(
"No encuentro ningún objeto."
);

}

return;

}


if(
command.includes("analiza")||
command.includes("analizar")||
command.includes("escanea")
){

if(selectedID!==null){

const target=
tracks.find(
t=>t.id===selectedID
);

if(target)
selectObject(target);

}
else{

respond(
"Selecciona primero un objeto."
);

}

return;

}


if(
command.includes("estado")||
command.includes("sistemas")
){

respond(
"Todos los sistemas principales están operativos."
);

return;

}


respond(
"Comando recibido."
);

}


/* =========================================================
   RESPUESTA
========================================================= */

function respond(text){

message.textContent=text;

speak(text);

}


/* =========================================================
   DESCRIBIR ESCENA
========================================================= */

function describeScene(){

if(!tracks.length){

respond(
"No detecto objetos."
);

return;

}


const list=[];


tracks.slice(0,7).forEach(
track=>{

const name=
names[
track.className
]||
track.className;


if(!list.includes(name))
list.push(name);

}
);


respond(
"Detecto "+
tracks.length+
" objetos. Veo: "+
list.join(", ")+"."
);

}


/* =========================================================
   ANIMACIÓN 60 FPS
========================================================= */

function animation(){

/*
   Esta función NO espera a la IA.

   requestAnimationFrame intenta ejecutarse
   sincronizado con la pantalla.

   En una pantalla de 60 Hz:
   aproximadamente 60 actualizaciones por segundo.
*/

draw();


if(selectedID!==null){

const target=
tracks.find(
t=>t.id===selectedID
);

if(target)
drawZoom(target);

}


/*
   FPS visual.
*/

frames++;

const now=
performance.now();

if(now-fpsLast>=1000){

fpsText.textContent=
"FPS: "+
frames;

frames=0;

fpsLast=now;

}


requestAnimationFrame(
animation
);

}


/* =========================================================
   BOTONES
========================================================= */

start.onclick=
startCamera;

voice.onclick=
listen;


/* =========================================================
   INICIAR HUD
========================================================= */

animation();

</script>

</body>
</html>
