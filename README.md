<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport"
      content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">

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
    text-shadow:
        0 0 8px #00d9ff,
        0 0 20px #008cff;
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
    width:270px;
    padding:11px;
    background:rgba(0,10,25,.84);
    border-left:3px solid #00d9ff;
    box-shadow:0 0 20px rgba(0,200,255,.25);
    backdrop-filter:blur(5px);
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

#voice{
    pointer-events:auto;
    padding:9px 12px;
    color:#00d9ff;
    background:rgba(0,40,70,.9);
    border:1px solid #00d9ff;
    border-radius:5px;
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

<video id="camera"
       autoplay
       playsinline
       muted></video>

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

<div id="analysis">
    <canvas id="zoomCanvas"></canvas>
    <div id="analysisText">SIN OBJETIVO</div>
</div>

<div id="scanText">
    ANALIZANDO OBJETIVO...
</div>

<div id="panel">

    <div id="jarvisName">
        JARVIS
    </div>

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

/* =====================================================
   ELEMENTOS
===================================================== */

const video =
    document.getElementById("camera");

const canvas =
    document.getElementById("canvas");

const ctx =
    canvas.getContext("2d");

const zoomCanvas =
    document.getElementById("zoomCanvas");

const zctx =
    zoomCanvas.getContext("2d");

const start =
    document.getElementById("start");

const voice =
    document.getElementById("voice");

const message =
    document.getElementById("message");

const analysis =
    document.getElementById("analysis");

const analysisText =
    document.getElementById("analysisText");

const scanText =
    document.getElementById("scanText");

const cameraStatus =
    document.getElementById("cameraStatus");

const aiStatus =
    document.getElementById("aiStatus");

const trackingStatus =
    document.getElementById("tracking");

const objectCount =
    document.getElementById("objectCount");

const targetStatus =
    document.getElementById("targetStatus");

const micStatus =
    document.getElementById("micStatus");


/* =====================================================
   VARIABLES
===================================================== */

let model = null;

let active = false;

let detecting = false;

let tracks = [];

let nextID = 1;

let selectedID = null;


/*
    NUEVO SISTEMA DE TRACKING

    prediction:
    detección actual de IA

    target:
    posición que queremos alcanzar

    box:
    posición visual actual

    Esto evita que las cajas "salten".
*/

const SMOOTHING = 0.16;

const POSITION_SMOOTHING = 0.12;


/*
    Si la IA pierde un objeto durante unos frames,
    NO eliminamos inmediatamente la caja.
*/

const MAX_MISSED = 35;


/*
    Cuánto puede moverse un objeto entre
    dos detecciones para seguir considerándolo
    el mismo.
*/

const IOU_LIMIT = 0.08;


/* =====================================================
   ESCANEO
===================================================== */

let scan = {

    active:false,

    id:null,

    progress:0,

    duration:1400,

    startTime:0

};


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
   ALTURAS
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
   CÁMARA
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


        video.srcObject = stream;

        await video.play();


        resize();


        active = true;


        start.style.display = "none";


        cameraStatus.textContent = "ON";


        message.textContent =
            "Cámara activada. Iniciando visión artificial...";


        loadAI();

    }

    catch(error){

        console.error(error);

        message.textContent =
            "No se pudo acceder a la cámara.";

        alert(
            "Permite el acceso a la cámara."
        );

    }

}


/* =====================================================
   RESIZE
===================================================== */

function resize(){

    if(!video.videoWidth)
        return;


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
   CARGAR MODELO
===================================================== */

async function loadAI(){

    try{

        aiStatus.textContent =
            "CARGANDO";


        model =
            await cocoSsd.load({
                base:"lite_mobilenet_v2"
            });


        aiStatus.textContent =
            "ONLINE";


        trackingStatus.textContent =
            "ONLINE";


        message.textContent =
            "Visión artificial activada.";


        speak(
            "Visión artificial activada."
        );


        detectionLoop();

    }

    catch(error){

        console.error(error);

        aiStatus.textContent =
            "ERROR";

    }

}


/* =====================================================
   IOU
===================================================== */

function IoU(a,b){

    const x1 =
        Math.max(
            a[0],
            b[0]
        );

    const y1 =
        Math.max(
            a[1],
            b[1]
        );

    const x2 =
        Math.min(
            a[0]+a[2],
            b[0]+b[2]
        );

    const y2 =
        Math.min(
            a[1]+a[3],
            b[1]+b[3]
        );


    const inter =
        Math.max(
            0,
            x2-x1
        )
        *
        Math.max(
            0,
            y2-y1
        );


    const union =
        a[2]*a[3]
        +
        b[2]*b[3]
        -
        inter;


    return union > 0
        ? inter/union
        : 0;

}


/* =====================================================
   DISTANCIA ENTRE CAJAS
===================================================== */

function centerDistance(a,b){

    const ax =
        a[0]+a[2]/2;

    const ay =
        a[1]+a[3]/2;

    const bx =
        b[0]+b[2]/2;

    const by =
        b[1]+b[3]/2;


    return Math.sqrt(
        Math.pow(ax-bx,2)
        +
        Math.pow(ay-by,2)
    );

}


/* =====================================================
   SUAVIZADO
===================================================== */

function smoothBox(current,target){

    /*
       Interpolación progresiva.

       0.16 = movimiento suave.
       No saltamos directamente a la nueva
       posición detectada.
    */

    return [

        current[0] +
        (target[0]-current[0])
        *SMOOTHING,

        current[1] +
        (target[1]-current[1])
        *SMOOTHING,

        current[2] +
        (target[2]-current[2])
        *SMOOTHING,

        current[3] +
        (target[3]-current[3])
        *SMOOTHING

    ];

}


/* =====================================================
   TRACKING MEJORADO
===================================================== */

function updateTracking(predictions){

    /*
       Primero marcamos todos como
       "posiblemente perdidos".
    */

    tracks.forEach(
        t => {

            t.missed++;

        }
    );


    /*
       Asociamos cada detección con
       el objeto existente más parecido.
    */

    predictions.forEach(
        prediction => {

            let best = null;

            let bestScore = 0;


            tracks.forEach(
                track => {

                    if(
                        track.className
                        !==
                        prediction.class
                    )
                        return;


                    const iou =
                        IoU(
                            track.box,
                            prediction.bbox
                        );


                    const distance =
                        centerDistance(
                            track.box,
                            prediction.bbox
                        );


                    /*
                       Combinamos IoU y distancia.

                       Esto ayuda mucho cuando la
                       cámara se mueve.
                    */

                    const maxDistance =
                        Math.max(
                            120,
                            track.box[2]*1.5
                        );


                    if(
                        iou >= IOU_LIMIT
                        ||
                        distance < maxDistance
                    ){

                        const score =
                            iou
                            +
                            Math.max(
                                0,
                                1-distance/
                                maxDistance
                            )*.35;


                        if(
                            score > bestScore
                        ){

                            bestScore =
                                score;

                            best =
                                track;

                        }

                    }

                }
            );


            if(best){

                /*
                   Actualizamos el objetivo,
                   NO la caja directamente.
                */

                best.target =
                    prediction.bbox.slice();


                best.score =
                    prediction.score;


                best.missed = 0;

                best.age++;

            }

            else{

                tracks.push({

                    id:nextID++,

                    className:
                        prediction.class,

                    box:
                        prediction.bbox.slice(),

                    target:
                        prediction.bbox.slice(),

                    score:
                        prediction.score,

                    missed:0,

                    age:1

                });

            }

        }
    );


    /*
       Movimiento suave incluso entre detecciones.
    */

    tracks.forEach(
        track => {

            track.box =
                smoothBox(
                    track.box,
                    track.target
                );

        }
    );


    /*
       IMPORTANTE:

       Ahora esperamos 35 ciclos antes de
       borrar una caja.

       Esto hace que desaparezca mucho menos
       cuando la IA pierde momentáneamente
       el objeto.
    */

    tracks =
        tracks.filter(
            track =>
                track.missed <= MAX_MISSED
        );


    /*
       Si el objeto seleccionado desaparece
       momentáneamente, mantenemos la selección.
    */

    if(selectedID !== null){

        const selected =
            tracks.find(
                t =>
                    t.id === selectedID
            );


        if(!selected){

            selectedID = null;

            analysis.style.display =
                "none";

            targetStatus.textContent =
                "NINGUNO";

        }

    }

}


/* =====================================================
   DETECCIÓN
===================================================== */

async function detectionLoop(){

    if(
        !active ||
        !model
    ){

        requestAnimationFrame(
            detectionLoop
        );

        return;

    }


    if(!detecting){

        detecting = true;


        try{

            const predictions =
                await model.detect(
                    video,
                    25,
                    .15
                );


            updateTracking(
                predictions
            );


            objectCount.textContent =
                tracks.length;


        }

        catch(error){

            console.error(error);

        }


        detecting = false;

    }


    setTimeout(
        detectionLoop,
        100
    );

}


/* =====================================================
   DISTANCIA
===================================================== */

function getDistance(track){

    const realHeight =
        heights[
            track.className
        ]
        ||
        .5;


    const focal =
        Math.max(
            500,
            video.videoWidth*.75
        );


    let distance =
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


/* =====================================================
   ALTURA
===================================================== */

function getHeight(
    track,
    distance
){

    const focal =
        Math.max(
            500,
            video.videoWidth*.75
        );


    const height =
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
   DIBUJAR
===================================================== */

function draw(){

    ctx.clearRect(
        0,
        0,
        canvas.width,
        canvas.height
    );


    tracks.forEach(
        track => {

            const x =
                track.box[0];

            const y =
                track.box[1];

            const w =
                track.box[2];

            const h =
                track.box[3];


            const selected =
                track.id === selectedID;


            const distance =
                getDistance(
                    track
                );


            const height =
                getHeight(
                    track,
                    distance
                );


            const name =
                names[
                    track.className
                ]
                ||
                track.className;


            /*
               Si está perdido,
               reducimos ligeramente la opacidad,
               pero NO quitamos la caja.
            */

            ctx.globalAlpha =
                Math.max(
                    .42,
                    1 -
                    track.missed*.015
                );


            /*
               CAJA
            */

            ctx.strokeStyle =
                selected
                ? "#ffffff"
                : "#00d9ff";


            ctx.lineWidth =
                selected
                ? 4
                : 2.5;


            ctx.shadowColor =
                "#00d9ff";


            ctx.shadowBlur =
                selected
                ? 22
                : 10;


            ctx.strokeRect(
                x,
                y,
                w,
                h
            );


            ctx.shadowBlur = 0;


            /*
               ESQUINAS
            */

            const c =
                Math.min(
                    22,
                    Math.max(
                        9,
                        Math.min(
                            w,
                            h
                        )*.18
                    )
                );


            drawCorner(
                x,
                y,
                c,
                1,
                1
            );

            drawCorner(
                x+w,
                y,
                c,
                -1,
                1
            );

            drawCorner(
                x,
                y+h,
                c,
                1,
                -1
            );

            drawCorner(
                x+w,
                y+h,
                c,
                -1,
                -1
            );


            /*
               INFORMACIÓN
            */

            const pw =
                Math.max(
                    190,
                    Math.min(
                        245,
                        w
                    )
                );


            const ph = 82;


            let py =
                y -
                ph -
                5;


            if(py < 5)
                py = y+5;


            ctx.fillStyle =
                "rgba(0,10,25,.90)";


            ctx.fillRect(
                x,
                py,
                pw,
                ph
            );


            ctx.strokeStyle =
                "#00d9ff";

            ctx.lineWidth = 1;


            ctx.strokeRect(
                x,
                py,
                pw,
                ph
            );


            ctx.fillStyle =
                "#00eaff";


            ctx.font =
                "bold 14px monospace";


            ctx.fillText(
                name.toUpperCase(),
                x+7,
                py+18
            );


            ctx.font =
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


            /*
               ESCANEO
            */

            if(
                scan.active &&
                scan.id === track.id
            ){

                drawObjectScan(
                    track
                );

            }


            ctx.globalAlpha = 1;

        }
    );

}


/* =====================================================
   ESQUINAS
===================================================== */

function drawCorner(
    x,
    y,
    size,
    dx,
    dy
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
   ESCANEO AZUL
===================================================== */

function drawObjectScan(track){

    const now =
        performance.now();


    const elapsed =
        now -
        scan.startTime;


    /*
       0 -> 1

       El recorrido dura exactamente
       una vez.
    */

    scan.progress =
        Math.min(
            1,
            elapsed /
            scan.duration
        );


    const x =
        track.box[0];

    const y =
        track.box[1];

    const w =
        track.box[2];

    const h =
        track.box[3];


    /*
       Posición de la línea:

       arriba -> abajo
    */

    const scanY =
        y +
        h *
        scan.progress;


    ctx.save();


    /*
       Resplandor.
    */

    ctx.shadowColor =
        "#00eaff";

    ctx.shadowBlur =
        25;


    /*
       Línea principal.
    */

    ctx.strokeStyle =
        "#00eaff";

    ctx.lineWidth = 3;


    ctx.beginPath();

    ctx.moveTo(
        x-12,
        scanY
    );

    ctx.lineTo(
        x+w+12,
        scanY
    );

    ctx.stroke();


    /*
       Halo más ancho.
    */

    ctx.strokeStyle =
        "rgba(0,210,255,.28)";

    ctx.lineWidth = 13;


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


    /*
       Pequeñas líneas verticales
       en los extremos para darle
       aspecto de escáner.
    */

    ctx.strokeStyle =
        "rgba(0,235,255,.8)";

    ctx.lineWidth = 1;


    ctx.beginPath();

    ctx.moveTo(
        x,
        y
    );

    ctx.lineTo(
        x,
        scanY
    );

    ctx.moveTo(
        x+w,
        y
    );

    ctx.lineTo(
        x+w,
        scanY
    );

    ctx.stroke();


    ctx.restore();


    /*
       Cuando llega abajo:
       se detiene y desaparece.
    */

    if(
        scan.progress >= 1
    ){

        scan.active = false;

        scan.id = null;

        scanText.style.display =
            "none";

    }

}


/* =====================================================
   SELECCIÓN
===================================================== */

function selectObject(track){

    selectedID =
        track.id;


    targetStatus.textContent =
        "#"+
        track.id;


    const name =
        names[
            track.className
        ]
        ||
        track.className;


    message.textContent =
        "Objetivo seleccionado: "+
        name+
        ".";


    /*
       INICIAR ESCANEO
    */

    scan.active = true;

    scan.id =
        track.id;

    scan.progress = 0;

    scan.startTime =
        performance.now();


    scanText.style.display =
        "block";


    speak(
        "Objetivo seleccionado. Analizando "+
        name+
        "."
    );


    /*
       Solo dura una pasada.
    */

    setTimeout(
        ()=>{

            if(
                scan.id ===
                track.id
            ){

                scan.active =
                    false;

                scan.id =
                    null;

                scanText.style.display =
                    "none";

            }

        },
        scan.duration+100
    );

}


/* =====================================================
   ZOOM
===================================================== */

function drawZoom(track){

    if(!track)
        return;


    analysis.style.display =
        "block";


    const name =
        names[
            track.className
        ]
        ||
        track.className;


    analysisText.textContent =
        name.toUpperCase()+
        " • ID #"+
        track.id;


    zoomCanvas.width = 380;

    zoomCanvas.height = 300;


    zctx.clearRect(
        0,
        0,
        zoomCanvas.width,
        zoomCanvas.height
    );


    const pad = .20;


    const sx =
        Math.max(
            0,
            track.box[0]-
            track.box[2]*pad
        );


    const sy =
        Math.max(
            0,
            track.box[1]-
            track.box[3]*pad
        );


    const sw =
        Math.min(
            video.videoWidth-sx,
            track.box[2]*
            (1+pad*2)
        );


    const sh =
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
        zoomCanvas.width,
        zoomCanvas.height

    );


    /*
       Marco azul.
    */

    zctx.strokeStyle =
        "#00d9ff";

    zctx.lineWidth = 2;

    zctx.shadowColor =
        "#00d9ff";

    zctx.shadowBlur = 10;


    zctx.strokeRect(
        3,
        3,
        zoomCanvas.width-6,
        zoomCanvas.height-6
    );


    zctx.shadowBlur = 0;


    /*
       Solo dibujamos una línea de
       escaneo en el zoom mientras
       se está realizando el análisis.
    */

    if(
        scan.active &&
        scan.id === track.id
    ){

        const scanY =
            zoomCanvas.height*
            scan.progress;


        zctx.strokeStyle =
            "#00eaff";

        zctx.lineWidth = 3;

        zctx.shadowColor =
            "#00eaff";

        zctx.shadowBlur = 15;


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

        zctx.shadowBlur = 0;

    }

}


/* =====================================================
   TOUCH / CLICK
===================================================== */

canvas.addEventListener(
    "pointerdown",
    function(event){

        if(!active)
            return;


        const rect =
            canvas.getBoundingClientRect();


        const px =
            (event.clientX-
            rect.left)*
            canvas.width/
            rect.width;


        const py =
            (event.clientY-
            rect.top)*
            canvas.height/
            rect.height;


        let selected = null;


        for(
            let i =
            tracks.length-1;
            i>=0;
            i--
        ){

            const t =
                tracks[i];


            const x =
                t.box[0];

            const y =
                t.box[1];

            const w =
                t.box[2];

            const h =
                t.box[3];


            if(

                px >= x &&
                px <= x+w &&
                py >= y &&
                py <= y+h

            ){

                selected =
                    t;

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
   VOZ
===================================================== */

function speak(text){

    if(
        !window.speechSynthesis
    )
        return;


    speechSynthesis.cancel();


    const utterance =
        new SpeechSynthesisUtterance(
            text
        );


    utterance.lang =
        "es-ES";

    utterance.rate =
        .9;

    utterance.pitch =
        .72;


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
            "Reconocimiento de voz no disponible.";

        micStatus.textContent =
            "NO DISP.";

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


    micStatus.textContent =
        "ESCUCHANDO";


    message.textContent =
        "Te escucho...";


    try{

        recognition.start();

    }

    catch(error){

        console.log(error);

        return;

    }


    recognition.onresult =
        function(event){

            const command =
                event.results[0][0]
                .transcript
                .toLowerCase();


            micStatus.textContent =
                "ON";


            processCommand(
                command
            );

        };


    recognition.onerror =
        function(event){

            console.log(
                event.error
            );


            micStatus.textContent =
                "ERROR";


            if(
                event.error ===
                "not-allowed"
            ){

                message.textContent =
                    "El micrófono está bloqueado. Permite el micrófono en el navegador.";

            }

            else{

                message.textContent =
                    "No pude escuchar. Inténtalo nuevamente.";

            }

        };


    recognition.onend =
        function(){

            setTimeout(
                ()=>{
                    micStatus.textContent =
                        "OFF";
                },
                1000
            );

        };

}


/* =====================================================
   COMANDOS
===================================================== */

function processCommand(command){

    if(
        command.includes("hola") ||
        command.includes("jarvis")
    ){

        respond(
            "A sus órdenes. Sistemas operativos."
        );

        return;

    }


    if(
        command.includes("qué ves") ||
        command.includes("que ves") ||
        command.includes("qué hay") ||
        command.includes("que hay")
    ){

        describeScene();

        return;

    }


    if(
        command.includes("cuántos") ||
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
        command.includes("selecciona") ||
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
        command.includes("analiza") ||
        command.includes("analizar") ||
        command.includes("escanea")
    ){

        if(selectedID !== null){

            const target =
                tracks.find(
                    t =>
                        t.id ===
                        selectedID
                );


            if(target){

                selectObject(
                    target
                );

            }

        }

        else{

            respond(
                "Selecciona primero un objeto."
            );

        }

        return;

    }


    if(
        command.includes("más cerca") ||
        command.includes("mas cerca")
    ){

        closest();

        return;

    }


    if(
        command.includes("estado") ||
        command.includes("sistemas")
    ){

        respond(
            "Todos los sistemas principales están operativos."
        );

        return;

    }


    respond(
        "Comando recibido. Mi módulo local todavía no conoce una respuesta para esa pregunta."
    );

}


/* =====================================================
   RESPUESTA
===================================================== */

function respond(text){

    message.textContent =
        text;

    speak(text);

}


/* =====================================================
   DESCRIPCIÓN
===================================================== */

function describeScene(){

    if(!tracks.length){

        respond(
            "No detecto objetos."
        );

        return;

    }


    const list = [];


    tracks
    .slice(0,6)
    .forEach(
        track => {

            const name =
                names[
                    track.className
                ]
                ||
                track.className;


            if(
                !list.includes(name)
            ){

                list.push(
                    name
                );

            }

        }
    );


    respond(
        "Detecto "+
        tracks.length+
        " objetos. Veo: "+
        list.join(", ")+"."
    );

}


/* =====================================================
   OBJETO MÁS CERCANO
===================================================== */

function closest(){

    if(!tracks.length){

        respond(
            "No detecto objetos."
        );

        return;

    }


    let nearest =
        tracks[0];


    let distance =
        getDistance(
            nearest
        );


    tracks.forEach(
        track => {

            const d =
                getDistance(
                    track
                );


            if(d < distance){

                nearest =
                    track;

                distance =
                    d;

            }

        }
    );


    const name =
        names[
            nearest.className
        ]
        ||
        nearest.className;


    respond(
        "El objeto más cercano es "+
        name+
        ", aproximadamente a "+
        distance.toFixed(1)+
        " metros."
    );

}


/* =====================================================
   INICIALIZACIÓN
===================================================== */

start.onclick =
    startCamera;

voice.onclick =
    listen;


/* =====================================================
   ANIMACIÓN PRINCIPAL
===================================================== */

function animation(){

    /*
       Dibujamos constantemente.

       Esto es importante:
       la caja se mueve suavemente incluso
       cuando la IA todavía no ha producido
       una nueva detección.
    */

    draw();


    if(
        selectedID !== null
    ){

        const target =
            tracks.find(
                t =>
                    t.id ===
                    selectedID
            );


        if(target){

            drawZoom(
                target
            );

        }

    }


    requestAnimationFrame(
        animation
    );

}


animation();


</script>

</body>
</html>
