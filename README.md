<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">

<title>JARVIS TARGET VISION</title>

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
    font-size:22px;
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

#fps{
    position:absolute;
    top:15px;
    right:75px;
    font-size:9px;
}

#panel{
    position:absolute;
    left:10px;
    bottom:10px;
    width:330px;
    max-height:45vh;
    overflow:auto;
    padding:11px;
    background:rgba(0,10,25,.9);
    border-left:3px solid #00d9ff;
    box-shadow:0 0 20px rgba(0,200,255,.25);
    pointer-events:auto;
}

#jarvisName{
    font-weight:bold;
    letter-spacing:2px;
}

#message{
    min-height:38px;
    margin-top:6px;
    font-size:10px;
    line-height:1.5;
}

#specs{
    margin-top:8px;
    padding-top:8px;
    border-top:1px solid rgba(0,217,255,.4);
    font-size:9px;
    line-height:1.7;
}

.specTitle{
    font-size:11px;
    letter-spacing:2px;
    margin-bottom:4px;
}

.specRow{
    display:flex;
    gap:5px;
}

.specLabel{
    min-width:95px;
    opacity:.65;
}

button{
    color:#00d9ff;
    background:rgba(0,40,70,.9);
    border:1px solid #00d9ff;
    border-radius:5px;
    padding:9px 12px;
    font-family:monospace;
}

#voice{
    margin-top:8px;
}

#analysis{
    position:absolute;
    top:65px;
    right:12px;
    width:205px;
    height:155px;
    display:none;
    background:rgba(0,10,25,.9);
    border:1px solid #00d9ff;
    box-shadow:0 0 20px rgba(0,210,255,.5);
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
    padding:5px 7px;
    background:rgba(0,10,25,.8);
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

#status{
    position:absolute;
    right:10px;
    bottom:10px;
    text-align:right;
    font-size:9px;
    line-height:1.8;
    text-shadow:0 0 7px #00d9ff;
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
<small>TARGET VISION SYSTEM</small>
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
Toca un objeto para analizarlo.
</div>

<div id="specs">

<div class="specTitle">
ESPECIFICACIONES
</div>

<div class="specRow">
<span class="specLabel">OBJETO:</span>
<span id="sName">---</span>
</div>

<div class="specRow">
<span class="specLabel">CONFIANZA:</span>
<span id="sConfidence">---</span>
</div>

<div class="specRow">
<span class="specLabel">DISTANCIA:</span>
<span id="sDistance">---</span>
</div>

<div class="specRow">
<span class="specLabel">ALTURA:</span>
<span id="sHeight">---</span>
</div>

<div class="specRow">
<span class="specLabel">MATERIAL:</span>
<span id="sMaterial">---</span>
</div>

<div class="specRow">
<span class="specLabel">FABRICACIÓN:</span>
<span id="sManufacturing">---</span>
</div>

<div class="specRow">
<span class="specLabel">PROCEDENCIA:</span>
<span id="sOrigin">---</span>
</div>

<div class="specRow">
<span class="specLabel">USO:</span>
<span id="sUse">---</span>
</div>

</div>

<button id="voice">
🎙 HABLAR CON JARVIS
</button>

</div>

<div id="status">

SISTEMA: <span id="systemStatus">ONLINE</span><br>
CÁMARA: <span id="cameraStatus">OFF</span><br>
IA: <span id="aiStatus">OFF</span><br>
TRACKING: <span id="tracking">OFF</span><br>
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

/* =====================================================
   ELEMENTOS
===================================================== */

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
const targetStatus=document.getElementById("targetStatus");
const micStatus=document.getElementById("micStatus");
const fpsText=document.getElementById("fps");

const sName=document.getElementById("sName");
const sConfidence=document.getElementById("sConfidence");
const sDistance=document.getElementById("sDistance");
const sHeight=document.getElementById("sHeight");
const sMaterial=document.getElementById("sMaterial");
const sManufacturing=document.getElementById("sManufacturing");
const sOrigin=document.getElementById("sOrigin");
const sUse=document.getElementById("sUse");


/* =====================================================
   CONFIGURACIÓN
===================================================== */

const SMOOTH=.105;
const PREDICTION=.04;
const MAX_MISSED=70;
const MIN_SCORE=.22;

let model=null;
let active=false;
let detecting=false;

let tracks=[];
let nextID=1;
let selectedID=null;

let frames=0;
let fpsLast=performance.now();

let scan={
    active:false,
    id:null,
    progress:0,
    duration:1300,
    start:0
};


/* =====================================================
   INFORMACIÓN DE OBJETOS
===================================================== */

const objectInfo={

person:{
    material:"tejido, piel y materiales textiles",
    manufacturing:"industria textil y manufactura de prendas",
    origin:"variable; depende de la persona y su ubicación",
    use:"persona"
},

bottle:{
    material:"plástico PET, vidrio o aluminio",
    manufacturing:"fabricación industrial mediante moldeado o conformado",
    origin:"variable; depende de la marca",
    use:"almacenamiento de líquidos"
},

cup:{
    material:"cerámica, plástico, vidrio o papel",
    manufacturing:"moldeado, prensado o fabricación industrial",
    origin:"variable según fabricante",
    use:"beber líquidos"
},

chair:{
    material:"madera, metal, plástico o combinaciones",
    manufacturing:"carpintería, moldeado o fabricación industrial",
    origin:"variable según fabricante",
    use:"sentarse"
},

table:{
    material:"madera, metal, vidrio o materiales compuestos",
    manufacturing:"carpintería y fabricación industrial",
    origin:"variable según fabricante",
    use:"superficie para apoyar objetos"
},

laptop:{
    material:"aluminio, plástico, vidrio y componentes electrónicos",
    manufacturing:"ensamblaje electrónico industrial",
    origin:"variable según marca y modelo",
    use:"computación"
},

cell_phone:{
    material:"vidrio, aluminio/plástico y componentes electrónicos",
    manufacturing:"ensamblaje electrónico industrial",
    origin:"variable según marca y modelo",
    use:"comunicación y computación móvil"
},

keyboard:{
    material:"plástico, metal y componentes electrónicos",
    manufacturing:"moldeado plástico y ensamblaje electrónico",
    origin:"variable según fabricante",
    use:"entrada de texto y comandos"
},

mouse:{
    material:"plástico y componentes electrónicos",
    manufacturing:"moldeado plástico y ensamblaje electrónico",
    origin:"variable según fabricante",
    use:"control de computadora"
},

book:{
    material:"papel, tinta y cartón",
    manufacturing:"impresión y encuadernación",
    origin:"depende de la editorial",
    use:"lectura e información"
},

backpack:{
    material:"poliéster, nylon, cuero o materiales sintéticos",
    manufacturing:"confección textil",
    origin:"variable según fabricante",
    use:"transportar objetos"
},

umbrella:{
    material:"tela sintética, metal y plástico",
    manufacturing:"confección y ensamblaje",
    origin:"variable según fabricante",
    use:"protección contra lluvia o sol"
},

bicycle:{
    material:"principalmente aluminio, acero, carbono y caucho",
    manufacturing:"fabricación mecánica y ensamblaje",
    origin:"variable según fabricante",
    use:"transporte"
},

car:{
    material:"acero, aluminio, plástico, vidrio y componentes electrónicos",
    manufacturing:"producción automotriz industrial",
    origin:"variable según marca y modelo",
    use:"transporte"
},

motorcycle:{
    material:"acero, aluminio, plástico y caucho",
    manufacturing:"producción mecánica y ensamblaje industrial",
    origin:"variable según marca y modelo",
    use:"transporte"
},

tv:{
    material:"vidrio, plástico, metal y componentes electrónicos",
    manufacturing:"ensamblaje electrónico industrial",
    origin:"variable según fabricante",
    use:"visualización de contenido"
},

remote:{
    material:"plástico y componentes electrónicos",
    manufacturing:"moldeado plástico y ensamblaje electrónico",
    origin:"variable según fabricante",
    use:"control remoto de dispositivos"
},

clock:{
    material:"plástico, metal, vidrio y componentes mecánicos/electrónicos",
    manufacturing:"fabricación mecánica o electrónica",
    origin:"variable según fabricante",
    use:"medición del tiempo"
},

bowl:{
    material:"cerámica, vidrio, plástico o metal",
    manufacturing:"moldeado, prensado o fabricación industrial",
    origin:"variable según fabricante",
    use:"contener alimentos"
},

knife:{
    material:"acero y materiales para el mango",
    manufacturing:"forjado o estampado y ensamblaje",
    origin:"variable según fabricante",
    use:"cortar alimentos u otros materiales"
},

fork:{
    material:"acero inoxidable o plástico",
    manufacturing:"estampado o moldeado",
    origin:"variable según fabricante",
    use:"comer alimentos"
},

scissors:{
    material:"acero y plástico o metal",
    manufacturing:"estampado, afilado y ensamblaje",
    origin:"variable según fabricante",
    use:"cortar materiales"
},

toothbrush:{
    material:"plástico y filamentos sintéticos",
    manufacturing:"moldeado e inserción de filamentos",
    origin:"variable según fabricante",
    use:"higiene dental"
},

refrigerator:{
    material:"acero, plástico, vidrio y componentes de refrigeración",
    manufacturing:"fabricación de electrodomésticos",
    origin:"variable según fabricante",
    use:"conservar alimentos"
},

microwave:{
    material:"metal, vidrio, plástico y componentes electrónicos",
    manufacturing:"fabricación y ensamblaje de electrodomésticos",
    origin:"variable según fabricante",
    use:"calentar alimentos"
},

potted_plant:{
    material:"planta, tierra y recipiente de plástico, cerámica u otro material",
    manufacturing:"cultivo vegetal y fabricación del recipiente",
    origin:"depende de la especie y cultivo",
    use:"decoración y cultivo"
},

dog:{
    material:"ser vivo",
    manufacturing:"no aplica",
    origin:"animal; ubicación y raza determinan procedencia",
    use:"animal doméstico o de trabajo"
},

cat:{
    material:"ser vivo",
    manufacturing:"no aplica",
    origin:"animal; ubicación y raza determinan procedencia",
    use:"animal doméstico"
}

};


/* =====================================================
   NOMBRES
===================================================== */

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


/* =====================================================
   ALTURAS
===================================================== */

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


/* =====================================================
   CÁMARA
===================================================== */

async function startCamera(){

try{

const stream=await navigator.mediaDevices.getUserMedia({

video:{
facingMode:{ideal:"environment"},
width:{ideal:1280},
height:{ideal:720}
},

audio:false

});

video.srcObject=stream;

await video.play();

resize();

active=true;

start.style.display="none";

cameraStatus.textContent="ON";

message.textContent="Cámara activa. Cargando visión artificial...";

loadAI();

}

catch(error){

console.error(error);

message.textContent=
"No se pudo acceder a la cámara. Revisa los permisos del navegador.";

}

}


/* =====================================================
   RESIZE
===================================================== */

function resize(){

if(!video.videoWidth)return;

canvas.width=video.videoWidth;
canvas.height=video.videoHeight;

}

window.addEventListener("resize",resize);


/* =====================================================
   IA
===================================================== */

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
"IA lista. Toca cualquier objeto detectado.";

detectionLoop();

}

catch(error){

console.error(error);

aiStatus.textContent="ERROR";

message.textContent="Error cargando la IA.";

}

}


/* =====================================================
   IOU
===================================================== */

function IoU(a,b){

const x1=Math.max(a[0],b[0]);
const y1=Math.max(a[1],b[1]);
const x2=Math.min(a[0]+a[2],b[0]+b[2]);
const y2=Math.min(a[1]+a[3],b[1]+b[3]);

const inter=
Math.max(0,x2-x1)*
Math.max(0,y2-y1);

const union=
a[2]*a[3]+b[2]*b[3]-inter;

return union>0?inter/union:0;

}


/* =====================================================
   CENTRO
===================================================== */

function center(box){

return{
x:box[0]+box[2]/2,
y:box[1]+box[3]/2
};

}


/* =====================================================
   DISTANCIA
===================================================== */

function centerDistance(a,b){

const A=center(a);
const B=center(b);

return Math.hypot(
A.x-B.x,
A.y-B.y
);

}


/* =====================================================
   SUAVIZADO
===================================================== */

function smooth(current,target){

return[
current[0]+(target[0]-current[0])*SMOOTH,
current[1]+(target[1]-current[1])*SMOOTH,
current[2]+(target[2]-current[2])*SMOOTH,
current[3]+(target[3]-current[3])*SMOOTH
];

}


/* =====================================================
   TRACKING
===================================================== */

function updateTracking(predictions){

tracks.forEach(t=>t.missed++);

predictions.forEach(pred=>{

let best=null;
let bestScore=0;

tracks.forEach(track=>{

if(track.className!==pred.class)return;

const iou=IoU(track.target,pred.bbox);

const distance=centerDistance(
track.target,
pred.bbox
);

const maxDistance=
Math.max(150,track.target[2]*2);

const movementScore=
Math.max(0,1-distance/maxDistance);

const score=
iou+movementScore*.6;

if(score>bestScore){

bestScore=score;
best=track;

}

});

if(best){

const old=center(best.target);
const now=center(pred.bbox);

best.velocityX=now.x-old.x;
best.velocityY=now.y-old.y;

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


tracks.forEach(track=>{

let predicted=[

track.target[0]+track.velocityX*PREDICTION,
track.target[1]+track.velocityY*PREDICTION,
track.target[2],
track.target[3]

];

if(track.missed>0){

const p=Math.min(track.missed,15);

predicted=[

track.target[0]+track.velocityX*p,
track.target[1]+track.velocityY*p,
track.target[2],
track.target[3]

];

}

track.box=smooth(
track.box,
predicted
);

});


tracks=tracks.filter(
t=>t.missed<=MAX_MISSED
);


if(
selectedID!==null &&
!tracks.some(t=>t.id===selectedID)
){

selectedID=null;

analysis.style.display="none";

targetStatus.textContent="NINGUNO";

message.textContent=
"Objetivo perdido. Toca otro objeto.";

clearSpecs();

}

}


/* =====================================================
   DETECCIÓN
===================================================== */

async function detectionLoop(){

if(!active||!model){

setTimeout(
detectionLoop,
120
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

}

catch(error){

console.error(error);

}

detecting=false;

}

setTimeout(
detectionLoop,
120
);

}


/* =====================================================
   DISTANCIA
===================================================== */

function getDistance(track){

const realHeight=
heights[track.className]||.5;

const focal=
Math.max(
500,
video.videoWidth*.75
);

const distance=
realHeight*focal/
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


/* =====================================================
   ALTURA
===================================================== */

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


/* =====================================================
   ESQUINAS
===================================================== */

function drawCorner(
x,y,size,dx,dy
){

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


/* =====================================================
   ESCANEO
===================================================== */

function drawScan(track){

if(
!scan.active||
scan.id!==track.id
)return;

const elapsed=
performance.now()-scan.start;

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

ctx.save();

ctx.strokeStyle="#00eaff";
ctx.lineWidth=3;
ctx.shadowColor="#00eaff";
ctx.shadowBlur=25;

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

ctx.shadowBlur=0;

ctx.strokeStyle="rgba(0,220,255,.25)";
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

if(scan.progress>=1){

scan.active=false;
scan.id=null;

scanText.style.display="none";

}

}


/* =====================================================
   DIBUJAR OBJETIVO
===================================================== */

function drawTarget(track){

if(!track)return;

const x=track.box[0];
const y=track.box[1];
const w=track.box[2];
const h=track.box[3];

const selected=
track.id===selectedID;

ctx.globalAlpha=
Math.max(
.35,
1-track.missed*.012
);

ctx.strokeStyle=
selected?
"#ffffff":
"#00d9ff";

ctx.lineWidth=
selected?4:2.5;

ctx.shadowColor="#00d9ff";

ctx.shadowBlur=
selected?22:10;

ctx.strokeRect(
x,y,w,h
);

ctx.shadowBlur=0;

const corner=
Math.min(
28,
Math.max(
10,
Math.min(w,h)*.18
)
);

drawCorner(x,y,corner,1,1);
drawCorner(x+w,y,corner,-1,1);
drawCorner(x,y+h,corner,1,-1);
drawCorner(x+w,y+h,corner,-1,-1);

if(selected){

const distance=
getDistance(track);

const height=
getHeight(track,distance);

const name=
names[track.className]||
track.className;

const pw=
Math.max(
205,
Math.min(
260,
w
)
);

const ph=88;

let py=y-ph-7;

if(py<5)py=y+7;

ctx.fillStyle=
"rgba(0,10,25,.92)";

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
ctx.font="bold 14px monospace";

ctx.fillText(
name.toUpperCase(),
x+8,
py+19
);

ctx.font="11px monospace";

ctx.fillText(
"ID: #"+track.id+
"  CONF: "+
(track.score*100).toFixed(0)+"%",
x+8,
py+38
);

ctx.fillText(
"DISTANCIA: "+
distance.toFixed(1)+" m",
x+8,
py+56
);

ctx.fillText(
"ALTURA: "+
height.toFixed(2)+" m",
x+8,
py+74
);

}

drawScan(track);

ctx.globalAlpha=1;

}


/* =====================================================
   OBJETIVO PRINCIPAL
===================================================== */

function getPrimary(){

if(!tracks.length)return null;

const centerX=canvas.width/2;
const centerY=canvas.height/2;

let best=null;
let bestScore=-Infinity;

tracks.forEach(track=>{

if(track.missed>20)return;

const c=center(track.box);

const distance=
Math.hypot(
c.x-centerX,
c.y-centerY
);

const centerScore=
1-Math.min(
1,
distance/
Math.max(
canvas.width,
canvas.height
)
);

const sizeScore=
Math.min(
1,
(track.box[2]*track.box[3])/
(canvas.width*canvas.height*.25)
);

const score=
track.score*.55+
centerScore*.30+
sizeScore*.15;

if(score>bestScore){

bestScore=score;
best=track;

}

});

return best;

}


/* =====================================================
   INFORMACIÓN DEL OBJETO
===================================================== */

function getObjectInfo(track){

return objectInfo[
track.className
] || {

material:"no determinado por el modelo",
manufacturing:"información no disponible",
origin:"no se puede determinar con esta detección",
use:"objeto identificado visualmente"

};

}


/* =====================================================
   ACTUALIZAR ESPECIFICACIONES
===================================================== */

function updateSpecs(track){

if(!track){

clearSpecs();
return;

}

const name=
names[track.className]||
track.className;

const info=
getObjectInfo(track);

const distance=
getDistance(track);

const height=
getHeight(track,distance);

sName.textContent=
name;

sConfidence.textContent=
(track.score*100).toFixed(0)+"%";

sDistance.textContent=
distance.toFixed(1)+" m aprox.";

sHeight.textContent=
height.toFixed(2)+" m aprox.";

sMaterial.textContent=
info.material;

sManufacturing.textContent=
info.manufacturing;

sOrigin.textContent=
info.origin;

sUse.textContent=
info.use;

}


/* =====================================================
   LIMPIAR ESPECIFICACIONES
===================================================== */

function clearSpecs(){

sName.textContent="---";
sConfidence.textContent="---";
sDistance.textContent="---";
sHeight.textContent="---";
sMaterial.textContent="---";
sManufacturing.textContent="---";
sOrigin.textContent="---";
sUse.textContent="---";

}


/* =====================================================
   SELECCIONAR OBJETO
===================================================== */

function selectObject(track){

selectedID=track.id;

targetStatus.textContent=
"#"+track.id;

const name=
names[track.className]||
track.className;

updateSpecs(track);

message.textContent=
"Objetivo seleccionado. Analizando especificaciones...";

scan.active=true;
scan.id=track.id;
scan.progress=0;
scan.start=performance.now();

scanText.style.display="block";

const info=
getObjectInfo(track);

const distance=
getDistance(track);

const height=
getHeight(track,distance);

const voiceText=
"Objetivo identificado: "+
name+
". Confianza "+
Math.round(track.score*100)+
" por ciento. "+
"Distancia aproximada "+
distance.toFixed(1)+
" metros. "+
"Altura estimada "+
height.toFixed(2)+
" metros. "+
"Material: "+
info.material+
". Fabricación: "+
info.manufacturing+
". Procedencia: "+
info.origin+
". Uso: "+
info.use+".";

speak(
voiceText
);

}


/* =====================================================
   TOQUE
===================================================== */

canvas.addEventListener(
"pointerdown",
event=>{

if(!active)return;

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

let found=null;
let smallestArea=Infinity;

tracks.forEach(track=>{

const x=track.box[0];
const y=track.box[1];
const w=track.box[2];
const h=track.box[3];

if(
px>=x &&
px<=x+w &&
py>=y &&
py<=y+h
){

const area=w*h;

if(area<smallestArea){

smallestArea=area;
found=track;

}

}

});

if(found){

selectObject(found);

}

});


/* =====================================================
   VOZ DE JARVIS
===================================================== */

function speak(text){

if(!window.speechSynthesis)return;

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


/* =====================================================
   RECONOCIMIENTO DE VOZ
===================================================== */

function listen(){

const Recognition=
window.SpeechRecognition||
window.webkitSpeechRecognition;

if(!Recognition){

message.textContent=
"Este navegador no permite reconocimiento de voz.";

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

micStatus.textContent="ERROR";

message.textContent=
"No pude escuchar el comando.";

};

recognition.onend=
()=>{

setTimeout(
()=>{
micStatus.textContent="OFF";
},
800
);

};

}


/* =====================================================
   COMANDOS
===================================================== */

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

let target=null;

if(selectedID!==null){

target=
tracks.find(
t=>t.id===selectedID
);

}

if(!target){

target=
getPrimary();

}

if(target){

const name=
names[target.className]||
target.className;

respond(
"Detecto "+
name+
" como objetivo principal."
);

}

else{

respond(
"No detecto ningún objetivo."
);

}

return;

}


if(
command.includes("especificaciones")||
command.includes("información")||
command.includes("informacion")||
command.includes("qué sabes")||
command.includes("que sabes")
){

if(selectedID!==null){

const target=
tracks.find(
t=>t.id===selectedID
);

if(target){

const name=
names[target.className]||
target.className;

const info=
getObjectInfo(target);

respond(
"El objetivo es "+
name+
". Material: "+
info.material+
". Fabricación: "+
info.manufacturing+
". Procedencia: "+
info.origin+
". Uso: "+
info.use+"."
);

}

}

else{

respond(
"Selecciona un objeto primero para obtener sus especificaciones."
);

}

return;

}


if(
command.includes("analiza")||
command.includes("analizar")||
command.includes("escanea")||
command.includes("escanear")
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
"Toca primero un objeto para seleccionarlo."
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
"Comando recibido. No tengo una respuesta específica para esa solicitud."
);

}


/* =====================================================
   RESPONDER
===================================================== */

function respond(text){

message.textContent=text;

speak(text);

}


/* =====================================================
   DIBUJAR
===================================================== */

function draw(){

ctx.clearRect(
0,
0,
canvas.width,
canvas.height
);

let visibleTarget=null;

if(selectedID!==null){

visibleTarget=
tracks.find(
t=>t.id===selectedID
);

}

else{

visibleTarget=
getPrimary();

}

if(visibleTarget){

drawTarget(
visibleTarget
);

}

if(selectedID!==null){

const target=
tracks.find(
t=>t.id===selectedID
);

if(target){

drawZoom(
target
);

updateSpecs(
target
);

}

}

frames++;

const now=
performance.now();

if(now-fpsLast>=1000){

fpsText.textContent=
"FPS: "+frames;

frames=0;

fpsLast=now;

}

requestAnimationFrame(
draw
);

}


/* =====================================================
   ZOOM
===================================================== */

function drawZoom(track){

if(!track)return;

analysis.style.display="block";

const name=
names[track.className]||
track.className;

analysisText.textContent=
name.toUpperCase()+
" • ID #"+
track.id;

zoomCanvas.width=410;
zoomCanvas.height=300;

zctx.clearRect(
0,
0,
410,
300
);

const pad=.30;

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
track.box[2]*(1+pad*2)
);

const sh=
Math.min(
video.videoHeight-sy,
track.box[3]*(1+pad*2)
);

zctx.drawImage(
video,
sx,
sy,
sw,
sh,
0,
0,
410,
300
);

zctx.strokeStyle="#00d9ff";
zctx.lineWidth=2;
zctx.shadowColor="#00d9ff";
zctx.shadowBlur=14;

zctx.strokeRect(
3,
3,
404,
294
);

zctx.shadowBlur=0;

if(
scan.active &&
scan.id===track.id
){

const sy2=
300*scan.progress;

zctx.strokeStyle="#00eaff";
zctx.lineWidth=3;
zctx.shadowColor="#00eaff";
zctx.shadowBlur=18;

zctx.beginPath();

zctx.moveTo(
0,
sy2
);

zctx.lineTo(
410,
sy2
);

zctx.stroke();

zctx.shadowBlur=0;

}

}


/* =====================================================
   INICIAR
===================================================== */

start.onclick=
startCamera;

voice.onclick=
listen;

draw();

</script>

</body>
</html>
