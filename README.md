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
    filter:none;
}

#canvas{
    position:fixed;
    inset:0;
    width:100%;
    height:100%;
    pointer-events:auto;
}

#hud{
    position:fixed;
    inset:0;
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
    font-size:9px;
    letter-spacing:3px;
    margin-top:3px;
}

.corner{
    position:absolute;
    width:55px;
    height:55px;
    border-color:#00d9ff;
    filter:drop-shadow(0 0 7px #00d9ff);
}

.tl{top:12px;left:12px;border-top:2px solid;border-left:2px solid}
.tr{top:12px;right:12px;border-top:2px solid;border-right:2px solid}
.bl{bottom:12px;left:12px;border-bottom:2px solid;border-left:2px solid}
.br{bottom:12px;right:12px;border-bottom:2px solid;border-right:2px solid}

#scanLine{
    position:absolute;
    left:0;
    width:100%;
    height:2px;
    background:#00d9ff;
    opacity:.20;
    box-shadow:0 0 15px #00d9ff;
    animation:scan 4s linear infinite;
}

@keyframes scan{
    from{top:0}
    to{top:100%}
}

#panel{
    position:absolute;
    left:10px;
    bottom:10px;
    width:270px;
    padding:11px;
    background:rgba(0,10,25,.82);
    border-left:3px solid #00d9ff;
    box-shadow:0 0 20px rgba(0,200,255,.25);
    backdrop-filter:blur(5px);
}

#jarvisName{
    font-weight:bold;
    letter-spacing:2px;
}

#message{
    margin-top:6px;
    min-height:35px;
    font-size:10px;
    line-height:1.45;
}

button{
    font-family:monospace;
}

#voice{
    pointer-events:auto;
    padding:9px 12px;
    color:#00d9ff;
    background:rgba(0,40,70,.9);
    border:1px solid #00d9ff;
    border-radius:5px;
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
    font-size:15px;
}

/* =====================================================
   VENTANA DE ANÁLISIS
===================================================== */

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

/* =====================================================
   ESCANEO DEL OBJETO
===================================================== */

#scanText{
    position:absolute;
    top:38%;
    left:50%;
    transform:translate(-50%,-50%);
    display:none;
    font-size:13px;
    letter-spacing:3px;
    text-shadow:0 0 10px #00d9ff;
    animation:blink .5s infinite alternate;
}

@keyframes blink{
    from{opacity:.45}
    to{opacity:1}
}

@media(max-width:600px){

    #title b{
        font-size:18px;
    }

    #panel{
        width:225px;
    }

    #analysis{
        width:165px;
        height:130px;
        top:60px;
        right:8px;
    }

    #status{
        right:8px;
        bottom:8px;
        font-size:8px;
    }
}
</style>
</head>

<body>

<video id="camera" autoplay playsinline muted></video>

<canvas id="canvas"></canvas>

<div id="hud">

<div id="scanLine"></div>

<div class="corner tl"></div>
<div class="corner tr"></div>
<div class="corner bl"></div>
<div class="corner br"></div>

<div id="title">
<b>J.A.R.V.I.S.</b>
<small>ADVANCED VISION SYSTEM</small>
</div>

<div id="analysis">
<canvas id="zoomCanvas"></canvas>
<div id="analysisText">SIN OBJETIVO</div>
</div>

<div id="scanText">
ANALIZANDO OBJETIVO...
</div>

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

SISTEMA:
<span>ONLINE</span>

<br>

CÁMARA:
<span id="cameraStatus">OFF</span>

<br>

IA:
<span id="aiStatus">OFF</span>

<br>

TRACKING:
<span id="tracking">OFF</span>

<br>

OBJETOS:
<span id="objectCount">0</span>

<br>

OBJETIVO:
<span id="targetStatus">NINGUNO</span>

<br>

MIC:
<span id="micStatus">OFF</span>

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
const tracking=document.getElementById("tracking");
const objectCount=document.getElementById("objectCount");
const targetStatus=document.getElementById("targetStatus");
const micStatus=document.getElementById("micStatus");


/* =====================================================
   VARIABLES
===================================================== */

let model=null;
let active=false;
let detecting=false;

let tracks=[];
let nextID=1;

let selectedID=null;

let scanProgress=0;
let scanning=false;


/* =====================================================
   CONFIGURACIÓN
===================================================== */

const SMOOTH=.32;
const MAX_MISSED=10;
const IOU_LIMIT=.15;


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
   ALTURAS DE REFERENCIA
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
            "Cámara activada. Iniciando visión artificial...";

        loadAI();

    }

    catch(error){

        console.error(error);

        message.textContent=
            "No se pudo acceder a la cámara.";

        alert(
            "Permite la cámara en Safari."
        );

    }

}


/* =====================================================
   REDIMENSIONAR
===================================================== */

function resize(){

    if(!video.videoWidth)return;

    canvas.width=video.videoWidth;
    canvas.height=video.videoHeight;

}

window.addEventListener(
    "resize",
    resize
);


/* =====================================================
   CARGAR IA
===================================================== */

async function loadAI(){

    try{

        aiStatus.textContent="CARGANDO";

        model=
        await cocoSsd.load({
            base:"lite_mobilenet_v2"
        });

        aiStatus.textContent="ONLINE";

        tracking.textContent="ONLINE";

        message.textContent=
            "Sistema de visión activado.";

        speak(
            "Sistema de visión activado."
        );

        detectionLoop();

    }

    catch(error){

        console.error(error);

        aiStatus.textContent="ERROR";

        message.textContent=
            "No se pudo cargar la inteligencia artificial.";

    }

}


/* =====================================================
   IOU
===================================================== */

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

    return union>0?
        inter/union:0;

}


/* =====================================================
   SUAVIZAR
===================================================== */

function smoothBox(oldBox,newBox){

    return [

        oldBox[0]+
        (newBox[0]-oldBox[0])*SMOOTH,

        oldBox[1]+
        (newBox[1]-oldBox[1])*SMOOTH,

        oldBox[2]+
        (newBox[2]-oldBox[2])*SMOOTH,

        oldBox[3]+
        (newBox[3]-oldBox[3])*SMOOTH

    ];

}


/* =====================================================
   TRACKING
===================================================== */

function updateTracking(predictions){

    tracks.forEach(
        t=>t.missed++
    );


    predictions.forEach(
        p=>{

            let best=null;
            let bestScore=0;


            tracks.forEach(
                t=>{

                    if(
                        t.className!==p.class
                    )return;


                    const score=
                        IoU(
                            t.box,
                            p.bbox
                        );


                    if(score>bestScore){

                        bestScore=score;
                        best=t;

                    }

                }
            );


            if(
                best &&
                bestScore>=IOU_LIMIT
            ){

                best.box=
                    smoothBox(
                        best.box,
                        p.bbox
                    );

                best.score=p.score;

                best.missed=0;

                best.age++;

            }

            else{

                tracks.push({

                    id:nextID++,

                    className:p.class,

                    box:p.bbox.slice(),

                    score:p.score,

                    missed:0,

                    age:1

                });

            }

        }
    );


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

        targetStatus.textContent="NINGUNO";

    }

}


/* =====================================================
   DETECCIÓN
===================================================== */

async function detectionLoop(){

    if(
        !active ||
        !model ||
        detecting
    ){

        requestAnimationFrame(
            detectionLoop
        );

        return;

    }


    detecting=true;


    try{

        const predictions=
        await model.detect(
            video,
            20,
            .18
        );


        updateTracking(
            predictions
        );

        draw();

    }

    catch(error){

        console.error(error);

    }


    detecting=false;


    setTimeout(
        detectionLoop,
        120
    );

}


/* =====================================================
   DISTANCIA
===================================================== */

function getDistance(track){

    const real=
        heights[track.className]||
        .5;

    const focal=
        Math.max(
            500,
            video.videoWidth*.75
        );

    let d=
        real*focal/
        track.box[3];

    return Math.max(
        .2,
        Math.min(
            30,
            d
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

    let h=
        track.box[3]*
        distance/
        focal;

    return Math.max(
        .03,
        Math.min(
            5,
            h
        )
    );

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


    objectCount.textContent=
        tracks.length;


    tracks.forEach(
        track=>{

            const x=track.box[0];
            const y=track.box[1];
            const w=track.box[2];
            const h=track.box[3];


            const selected=
                track.id===selectedID;


            const distance=
                getDistance(track);


            const height=
                getHeight(
                    track,
                    distance
                );


            const name=
                names[track.className]||
                track.className;


            /* -----------------------------------------
               TRANSPARENCIA
            ----------------------------------------- */

            ctx.globalAlpha=
                Math.max(
                    .35,
                    1-track.missed*.08
                );


            /* -----------------------------------------
               CAJA
            ----------------------------------------- */

            ctx.strokeStyle=
                selected?
                "#ffffff":
                "#00d9ff";

            ctx.lineWidth=
                selected?4:2.5;

            ctx.shadowColor=
                "#00d9ff";

            ctx.shadowBlur=
                selected?22:12;


            ctx.strokeRect(
                x,y,w,h
            );


            ctx.shadowBlur=0;


            /* -----------------------------------------
               ESQUINAS
            ----------------------------------------- */

            const c=
                Math.min(
                    20,
                    Math.max(
                        8,
                        Math.min(w,h)*.18
                    )
                );


            ctx.strokeStyle=
                "#00eaff";

            ctx.lineWidth=2;


            drawCorner(
                x,y,c,1,1
            );

            drawCorner(
                x+w,y,c,-1,1
            );

            drawCorner(
                x,y+h,c,1,-1
            );

            drawCorner(
                x+w,y+h,c,-1,-1
            );


            /* -----------------------------------------
               PANEL
            ----------------------------------------- */

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


            if(py<5){
                py=y+5;
            }


            ctx.fillStyle=
                "rgba(0,10,25,.90)";

            ctx.fillRect(
                x,
                py,
                pw,
                ph
            );


            ctx.strokeStyle=
                "#00d9ff";

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
                (track.score*100)
                .toFixed(0)+
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


            /* -----------------------------------------
               SILUETA / CONTORNO ESTILIZADO
            ----------------------------------------- */

            if(selected){

                drawScanContour(
                    x,y,w,h
                );

            }


            ctx.globalAlpha=1;

        }
    );


    if(selectedID!==null){

        const target=
            tracks.find(
                t=>t.id===selectedID
            );

        if(target){

            drawZoom(target);

        }

    }

}


/* =====================================================
   ESQUINAS DE CAJA
===================================================== */

function drawCorner(
    x,y,c,dx,dy
){

    ctx.beginPath();

    ctx.moveTo(
        x,
        y+dy*c
    );

    ctx.lineTo(
        x,
        y
    );

    ctx.lineTo(
        x+dx*c,
        y
    );

    ctx.stroke();

}


/* =====================================================
   CONTORNO DE ESCANEO
===================================================== */

function drawScanContour(
    x,y,w,h
){

    const time=
        performance.now()/500;


    const wave=
        (Math.sin(time)+1)/2;


    /*
       Este efecto simula una silueta
       tecnológica alrededor del objeto.
    */

    ctx.save();

    ctx.strokeStyle=
        "rgba(0,230,255,.85)";

    ctx.lineWidth=1.5;

    ctx.shadowColor="#00d9ff";

    ctx.shadowBlur=10;


    ctx.setLineDash([
        6,
        5
    ]);


    ctx.lineDashOffset=
        -performance.now()/40;


    ctx.strokeRect(
        x-5-wave*4,
        y-5-wave*4,
        w+10+wave*8,
        h+10+wave*8
    );


    ctx.restore();

}


/* =====================================================
   ZOOM
===================================================== */

function drawZoom(track){

    analysis.style.display="block";

    analysisText.textContent=
        (
            names[track.className]||
            track.className
        ).toUpperCase()+
        "  •  ID #"+
        track.id;


    const bw=
        track.box[2];

    const bh=
        track.box[3];


    zoomCanvas.width=380;
    zoomCanvas.height=300;


    zctx.clearRect(
        0,
        0,
        zoomCanvas.width,
        zoomCanvas.height
    );


    /*
       Ampliamos un poco alrededor
       del objeto.
    */

    const pad=.20;

    const sx=
        Math.max(
            0,
            track.box[0]-
            bw*pad
        );

    const sy=
        Math.max(
            0,
            track.box[1]-
            bh*pad
        );

    const sw=
        Math.min(
            video.videoWidth-sx,
            bw*(1+pad*2)
        );

    const sh=
        Math.min(
            video.videoHeight-sy,
            bh*(1+pad*2)
        );


    zctx.drawImage(

        video,

        sx,
        sy,
        sw,
        sh,

        0,
        0,
        zoomCanvas.width,
        zoomCanvas.height

    );


    /*
       Capa de escaneo.
    */

    const scanY=
        (performance.now()/4)
        %
        zoomCanvas.height;


    zctx.strokeStyle=
        "#00eaff";

    zctx.lineWidth=2;

    zctx.shadowColor="#00d9ff";

    zctx.shadowBlur=10;


    zctx.beginPath();

    zctx.moveTo(
        0,
        scanY
    );

    zctx.lineTo(
        zoomCanvas.width,
        scanY
    );

    zctx.stroke();


    /*
       Marco de análisis.
    */

    zctx.shadowBlur=0;

    zctx.strokeStyle="#00d9ff";

    zctx.lineWidth=2;

    zctx.strokeRect(
        3,
        3,
        zoomCanvas.width-6,
        zoomCanvas.height-6
    );


    if(selectedID!==null){

        requestAnimationFrame(
            ()=>{
                if(selectedID!==null){
                    drawZoom(track);
                }
            }
        );

    }

}


/* =====================================================
   CLICK / TOUCH EN OBJETO
===================================================== */

canvas.addEventListener(
    "pointerdown",
    function(event){

        if(!active)return;


        const rect=
            canvas.getBoundingClientRect();


        /*
           Convertimos el toque de la pantalla
           a coordenadas reales del vídeo.
        */

        const px=
            (event.clientX-rect.left)*
            canvas.width/
            rect.width;


        const py=
            (event.clientY-rect.top)*
            canvas.height/
            rect.height;


        let selected=null;


        /*
           Recorremos de atrás hacia delante
           para seleccionar el objeto superior.
        */

        for(
            let i=tracks.length-1;
            i>=0;
            i--
        ){

            const t=tracks[i];

            const x=t.box[0];
            const y=t.box[1];
            const w=t.box[2];
            const h=t.box[3];


            if(
                px>=x &&
                px<=x+w &&
                py>=y &&
                py<=y+h
            ){

                selected=t;

                break;

            }

        }


        if(selected){

            selectObject(
                selected
            );

        }

    }
);


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


    message.textContent=
        "Objetivo seleccionado: "+
        name+
        ". Iniciando escaneo.";


    scanning=true;

    scanProgress=0;

    scanText.style.display="block";


    speak(
        "Objetivo seleccionado. Analizando "+
        name+
        "."
    );


    setTimeout(
        ()=>{
            scanning=false;
            scanText.style.display="none";
        },
        2200
    );

}


/* =====================================================
   VOZ
===================================================== */

function speak(text){

    if(!window.speechSynthesis)return;

    speechSynthesis.cancel();


    const u=
        new SpeechSynthesisUtterance(
            text
        );


    u.lang="es-ES";

    u.rate=.9;

    u.pitch=.72;

    u.volume=1;


    speechSynthesis.speak(u);

}


/* =====================================================
   RECONOCIMIENTO DE VOZ
===================================================== */

function listen(){

    const Recognition=
        window.SpeechRecognition||
        window.webkitSpeechRecognition;


    if(!Recognition){

        micStatus.textContent=
            "NO DISP.";

        message.textContent=
            "Este navegador no permite reconocimiento de voz.";

        speak(
            "El reconocimiento de voz no está disponible."
        );

        return;

    }


    const recognition=
        new Recognition();


    recognition.lang="es-ES";

    recognition.continuous=false;

    recognition.interimResults=false;

    recognition.maxAlternatives=1;


    micStatus.textContent=
        "ESCUCHANDO";

    message.textContent=
        "Te escucho...";


    try{

        recognition.start();

    }

    catch(error){

        console.log(error);

        message.textContent=
            "Pulsa nuevamente el botón del micrófono.";

        return;

    }


    recognition.onresult=
        function(event){

            const text=
                event
                .results[0][0]
                .transcript
                .toLowerCase();


            micStatus.textContent=
                "ON";


            processCommand(
                text
            );

        };


    recognition.onerror=
        function(event){

            console.log(
                "VOICE:",
                event.error
            );


            micStatus.textContent=
                "ERROR";


            if(
                event.error===
                "not-allowed"
            ){

                message.textContent=
                    "Permiso de micrófono bloqueado. Revisa Ajustes > Safari > Micrófono.";

            }

            else{

                message.textContent=
                    "No pude escuchar correctamente. Inténtalo otra vez.";

            }

        };


    recognition.onend=
        function(){

            if(
                micStatus.textContent===
                "ON"
            ){

                setTimeout(
                    ()=>{
                        micStatus.textContent="OFF";
                    },
                    1000
                );

            }

        };

}


/* =====================================================
   CEREBRO LOCAL DE JARVIS
===================================================== */

function processCommand(command){

    console.log(
        "Comando:",
        command
    );


    if(
        command.includes("hola")||
        command.includes("jarvis")
    ){

        respond(
            "A sus órdenes. Todos los sistemas funcionan correctamente."
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
            "Tengo "+
            tracks.length+
            " objetos actualmente bajo seguimiento."
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
                "No encuentro un objeto para seleccionar."
            );

        }

        return;

    }


    if(
        command.includes("más cerca")||
        command.includes("mas cerca")
    ){

        closest();

        return;

    }


    if(
        command.includes("estado")||
        command.includes("sistemas")
    ){

        respond(
            "Cámara, inteligencia visual y tracking están operativos."
        );

        return;

    }


    if(
        command.includes("analiza")||
        command.includes("analizar")||
        command.includes("escanea")||
        command.includes("escanea")
    ){

        if(selectedID!==null){

            const target=
                tracks.find(
                    t=>t.id===selectedID
                );

            if(target){

                selectObject(
                    target
                );

            }

        }
        else{

            respond(
                "Seleccione primero un objeto."
            );

        }

        return;

    }


    if(
        command.includes("ayuda")||
        command.includes("comandos")
    ){

        respond(
            "Puede preguntarme qué veo, cuántos objetos detecto, cuál está más cerca, o seleccionar y analizar un objeto."
        );

        return;

    }


    respond(
        "Comando recibido. Mi módulo local todavía no tiene una respuesta específica para esa pregunta."
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
   DESCRIBIR ESCENA
===================================================== */

function describeScene(){

    if(
        tracks.length===0
    ){

        respond(
            "No detecto objetos en este momento."
        );

        return;

    }


    const list=[];


    tracks
    .slice(0,6)
    .forEach(
        t=>{

            const name=
                names[t.className]||
                t.className;


            if(
                !list.includes(name)
            ){

                list.push(name);

            }

        }
    );


    respond(
        "Detecto "+
        tracks.length+
        " objetos. Entre ellos: "+
        list.join(", ")+"."
    );

}


/* =====================================================
   MÁS CERCANO
===================================================== */

function closest(){

    if(!tracks.length){

        respond(
            "No detecto objetos."
        );

        return;

    }


    let nearest=
        tracks[0];


    let d=
        getDistance(nearest);


    tracks.forEach(
        t=>{

            const td=
                getDistance(t);


            if(td<d){

                nearest=t;
                d=td;

            }

        }
    );


    const name=
        names[nearest.className]||
        nearest.className;


    respond(
        "El objeto más cercano parece ser "+
        name+
        ", aproximadamente a "+
        d.toFixed(1)+
        " metros."
    );

}


/* =====================================================
   INICIAR
===================================================== */

start.onclick=
    startCamera;


voice.onclick=
    listen;


/* =====================================================
   ANIMACIÓN CONTINUA
===================================================== */

function animation(){

    if(
        selectedID!==null
    ){

        const target=
            tracks.find(
                t=>t.id===selectedID
            );


        if(target){

            drawZoom(target);

        }

    }


    requestAnimationFrame(
        animation
    );

}


animation();


console.log(
    "JARVIS VISION ONLINE"
);

</script>

</body>
</html>
