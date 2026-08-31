<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">

<meta name="viewport"
content="width=device-width,
initial-scale=1.0,
maximum-scale=1.0,
user-scalable=no">

<title>JARVIS VISION AI</title>

<style>

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
    top:17px;
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
    margin-top:4px;
    font-size:9px;
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

/* =====================================================
   ESCÁNER
===================================================== */

#scan{
    position:absolute;
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

/* =====================================================
   PANEL JARVIS
===================================================== */

#jarvis{
    position:absolute;
    left:12px;
    bottom:12px;
    width:270px;
    padding:11px;
    background:rgba(0,15,30,.82);
    border-left:3px solid #00d9ff;
    box-shadow:0 0 20px rgba(0,190,255,.25);
    backdrop-filter:blur(5px);
}

#jarvisName{
    font-size:13px;
    font-weight:bold;
    letter-spacing:2px;
}

#message{
    margin-top:6px;
    min-height:32px;
    font-size:10px;
    line-height:1.45;
}

#voiceButton{
    pointer-events:auto;
    margin-top:7px;
    padding:9px 12px;
    color:#00d9ff;
    background:rgba(0,40,70,.9);
    border:1px solid #00d9ff;
    border-radius:5px;
    font-family:monospace;
    font-size:10px;
}

/* =====================================================
   ESTADO
===================================================== */

#status{
    position:absolute;
    right:12px;
    bottom:12px;
    text-align:right;
    font-size:9px;
    line-height:1.8;
    text-shadow:0 0 8px #00d9ff;
}

/* =====================================================
   BOTÓN INICIAL
===================================================== */

#start{
    position:fixed;
    z-index:50;
    top:50%;
    left:50%;
    transform:translate(-50%,-50%);
    padding:18px 25px;
    color:#00d9ff;
    background:rgba(0,10,25,.96);
    border:1px solid #00d9ff;
    border-radius:6px;
    font-family:monospace;
    font-size:15px;
    box-shadow:0 0 25px #008cff;
}

/* =====================================================
   MÓVIL
===================================================== */

@media(max-width:600px){

    #title b{
        font-size:18px;
    }

    #jarvis{
        width:225px;
        left:8px;
        bottom:8px;
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

<video
id="camera"
autoplay
playsinline
muted>
</video>

<canvas id="canvas"></canvas>

<div id="hud">

<div id="scan"></div>

<div class="corner tl"></div>
<div class="corner tr"></div>
<div class="corner bl"></div>
<div class="corner br"></div>

<div id="title">
<b>J.A.R.V.I.S.</b>
<small>AI VISION • OBJECT TRACKING</small>
</div>

<div id="jarvis">

<div id="jarvisName">
JARVIS
</div>

<div id="message">
Sistema preparado.
</div>

<button id="voiceButton">
🎙 HABLAR CON JARVIS
</button>

</div>

<div id="status">

SISTEMA:
<span id="system">ONLINE</span>

<br>

CÁMARA:
<span id="cameraStatus">OFF</span>

<br>

IA:
<span id="aiStatus">OFF</span>

<br>

TRACKING:
<span id="trackingStatus">OFF</span>

<br>

OBJETOS:
<span id="objectCount">0</span>

<br>

FPS:
<span id="fps">0</span>

</div>

</div>

<button id="start">
ACTIVAR JARVIS
</button>


<!-- ===================================================
     IA
=================================================== -->

<script src=
"https://cdn.jsdelivr.net/npm/@tensorflow/tfjs">
</script>

<script src=
"https://cdn.jsdelivr.net/npm/@tensorflow-models/coco-ssd">
</script>


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

const startButton =
document.getElementById("start");

const voiceButton =
document.getElementById("voiceButton");

const message =
document.getElementById("message");

const cameraStatus =
document.getElementById("cameraStatus");

const aiStatus =
document.getElementById("aiStatus");

const trackingStatus =
document.getElementById("trackingStatus");

const objectCount =
document.getElementById("objectCount");

const fpsElement =
document.getElementById("fps");


/* =====================================================
   VARIABLES
===================================================== */

let model=null;

let active=false;

let detecting=false;

let tracks=[];

let nextTrackID=1;

let lastDetectionTime=0;

let frameCounter=0;

let fpsTime=performance.now();


/*
   Cuánto tiempo sobrevive una caja
   cuando una detección desaparece.
*/

const MAX_MISSED=8;


/*
   Umbral de coincidencia.
*/

const IOU_THRESHOLD=.18;


/*
   Factor de suavizado.

   Menor = más estable.
   Mayor = sigue más rápido.
*/

const SMOOTHING=.35;


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
   CANVAS
===================================================== */

function resizeCanvas(){

    if(
        video.videoWidth &&
        video.videoHeight
    ){

        canvas.width=
            video.videoWidth;

        canvas.height=
            video.videoHeight;
    }

}


window.addEventListener(
    "resize",
    resizeCanvas
);


/* =====================================================
   CÁMARA
===================================================== */

async function startCamera(){

    try{

        const stream=
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

        resizeCanvas();

        active=true;

        startButton.style.display=
            "none";

        cameraStatus.textContent=
            "ON";

        message.textContent=
            "Cámara activa. Cargando sistema de visión...";

        loadAI();

    }

    catch(error){

        console.error(error);

        message.textContent=
            "No se pudo activar la cámara.";

        alert(
            "Permite el acceso a la cámara."
        );

    }

}


/* =====================================================
   CARGAR IA
===================================================== */

async function loadAI(){

    try{

        aiStatus.textContent=
            "CARGANDO";


        model=
            await cocoSsd.load({

                base:"lite_mobilenet_v2"

            });


        aiStatus.textContent=
            "ONLINE";


        trackingStatus.textContent=
            "ONLINE";


        message.textContent=
            "IA y sistema de tracking activados.";


        speak(
            "Sistema de visión y seguimiento activado."
        );


        detectLoop();

    }

    catch(error){

        console.error(error);

        aiStatus.textContent=
            "ERROR";

        message.textContent=
            "Error cargando la inteligencia artificial.";

    }

}


/* =====================================================
   IOU
===================================================== */

function IoU(a,b){

    const ax=a[0];
    const ay=a[1];
    const aw=a[2];
    const ah=a[3];

    const bx=b[0];
    const by=b[1];
    const bw=b[2];
    const bh=b[3];


    const x1=
        Math.max(
            ax,
            bx
        );

    const y1=
        Math.max(
            ay,
            by
        );


    const x2=
        Math.min(
            ax+aw,
            bx+bw
        );

    const y2=
        Math.min(
            ay+ah,
            by+bh
        );


    const intersection=
        Math.max(
            0,
            x2-x1
        )
        *
        Math.max(
            0,
            y2-y1
        );


    const areaA=
        aw*ah;

    const areaB=
        bw*bh;


    const union=
        areaA+
        areaB-
        intersection;


    if(union<=0)
        return 0;


    return intersection/union;

}


/* =====================================================
   DISTANCIA
===================================================== */

function estimateDistance(
    object,
    pixelHeight
){

    const realHeight=
        heights[object]||.50;


    if(pixelHeight<=0)
        return 0;


    const focal=
        Math.max(
            500,
            video.videoWidth*.75
        );


    let d=
        realHeight*
        focal/
        pixelHeight;


    d=
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

function estimateHeight(
    object,
    pixelHeight,
    distance
){

    const focal=
        Math.max(
            500,
            video.videoWidth*.75
        );


    let h=
        pixelHeight*
        distance/
        focal;


    h=
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
   SUAVIZAR CAJA
===================================================== */

function smoothBox(
    oldBox,
    newBox
){

    return [

        oldBox[0]+
        (newBox[0]-oldBox[0])
        *SMOOTHING,

        oldBox[1]+
        (newBox[1]-oldBox[1])
        *SMOOTHING,

        oldBox[2]+
        (newBox[2]-oldBox[2])
        *SMOOTHING,

        oldBox[3]+
        (newBox[3]-oldBox[3])
        *SMOOTHING

    ];

}


/* =====================================================
   ACTUALIZAR TRACKS
===================================================== */

function updateTracks(
    predictions
){

    /*
       Primero aumentamos el contador
       de objetos que no fueron vistos.
    */

    tracks.forEach(
        function(track){

            track.missed++;

        }
    );


    /*
       Intentamos encontrar una caja
       existente para cada detección.
    */

    predictions.forEach(
        function(prediction){

            let bestTrack=null;

            let bestScore=0;


            tracks.forEach(
                function(track){

                    if(
                        track.className !==
                        prediction.class
                    ){

                        return;
                    }


                    const score=
                        IoU(
                            track.box,
                            prediction.bbox
                        );


                    if(
                        score>
                        bestScore
                    ){

                        bestScore=
                            score;

                        bestTrack=
                            track;
                    }

                }
            );


            if(
                bestTrack &&
                bestScore>=
                IOU_THRESHOLD
            ){

                /*
                   La caja no salta.
                   Se acerca progresivamente
                   a la nueva posición.
                */

                bestTrack.box=
                    smoothBox(
                        bestTrack.box,
                        prediction.bbox
                    );


                bestTrack.score=
                    prediction.score;


                bestTrack.missed=0;


                bestTrack.age++;


            }

            else{

                /*
                   Nuevo objeto.
                */

                tracks.push({

                    id:
                        nextTrackID++,

                    className:
                        prediction.class,

                    box:
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
       Eliminamos únicamente objetos
       que llevan demasiado tiempo sin
       ser detectados.
    */

    tracks=
        tracks.filter(
            function(track){

                return (
                    track.missed
                    <=
                    MAX_MISSED
                );

            }
        );

}


/* =====================================================
   DETECCIÓN PRINCIPAL
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


    detecting=true;


    try{

        /*
           Detección normal.
        */

        const predictions=
            await model.detect(
                video,
                12,
                .20
            );


        /*
           Detecciones demasiado pequeñas
           pueden perderse.
           
           Por eso también analizamos
           regiones ampliadas.
        */

        const extra=
            await detectSmallObjects();


        /*
           Unimos resultados.
        */

        const combined=
            mergePredictions(
                predictions,
                extra
            );


        updateTracks(
            combined
        );


        drawTracks();


        updateFPS();

    }

    catch(error){

        console.error(
            "Detection:",
            error
        );

    }


    detecting=false;


    /*
       Aproximadamente 6-8 análisis por
       segundo dependiendo del iPhone.
    */

    setTimeout(
        detectLoop,
        120
    );

}


/* =====================================================
   DETECCIÓN DE OBJETOS PEQUEÑOS
===================================================== */

async function detectSmallObjects(){

    const result=[];


    /*
       Si el vídeo todavía no tiene
       dimensiones válidas, no hacemos
       el análisis.
    */

    if(
        video.videoWidth<300 ||
        video.videoHeight<200
    ){

        return result;
    }


    /*
       Analizamos cuatro zonas de la imagen.

       El modelo recibe cada zona
       ampliada, aumentando el tamaño
       aparente de objetos pequeños.
    */

    const zones=[

        {
            x:0,
            y:0,
            w:.60,
            h:.60
        },

        {
            x:.40,
            y:0,
            w:.60,
            h:.60
        },

        {
            x:0,
            y:.40,
            w:.60,
            h:.60
        },

        {
            x:.40,
            y:.40,
            w:.60,
            h:.60
        }

    ];


    /*
       Para no crear demasiados canvas,
       usamos un canvas temporal.
    */

    const temp=
        document.createElement(
            "canvas"
        );


    const size=640;

    temp.width=size;
    temp.height=size;


    const tctx=
        temp.getContext(
            "2d"
        );


    for(
        const zone
        of zones
    ){

        const sx=
            zone.x*
            video.videoWidth;

        const sy=
            zone.y*
            video.videoHeight;

        const sw=
            zone.w*
            video.videoWidth;

        const sh=
            zone.h*
            video.videoHeight;


        tctx.clearRect(
            0,
            0,
            size,
            size
        );


        tctx.drawImage(

            video,

            sx,
            sy,
            sw,
            sh,

            0,
            0,
            size,
            size

        );


        let detections=[];


        try{

            detections=
                await model.detect(
                    temp,
                    8,
                    .25
                );

        }

        catch(error){

            continue;
        }


        detections.forEach(
            function(obj){

                /*
                   Convertimos las coordenadas
                   del recorte a coordenadas
                   de la cámara completa.
                */

                const bx=
                    obj.bbox[0]
                    /size*
                    sw+
                    sx;


                const by=
                    obj.bbox[1]
                    /size*
                    sh+
                    sy;


                const bw=
                    obj.bbox[2]
                    /size*
                    sw;


                const bh=
                    obj.bbox[3]
                    /size*
                    sh;


                /*
                   Ignoramos detecciones
                   extremadamente pequeñas
                   o poco confiables.
                */

                if(
                    bw<4 ||
                    bh<4 ||
                    obj.score<.25
                ){

                    return;
                }


                result.push({

                    class:
                        obj.class,

                    score:
                        obj.score,

                    bbox:[
                        bx,
                        by,
                        bw,
                        bh
                    ]

                });

            }
        );

    }


    return result;

}


/* =====================================================
   ELIMINAR DUPLICADOS
===================================================== */

function mergePredictions(
    predictions
){

    const final=[];


    predictions.forEach(
        function(pred){

            let duplicate=false;


            final.forEach(
                function(existing){

                    if(
                        existing.class !==
                        pred.class
                    ){

                        return;
                    }


                    const overlap=
                        IoU(
                            existing.bbox,
                            pred.bbox
                        );


                    if(
                        overlap>.45
                    ){

                        duplicate=true;


                        /*
                           Conservamos la detección
                           con mayor confianza.
                        */

                        if(
                            pred.score>
                            existing.score
                        ){

                            existing.bbox=
                                pred.bbox;

                            existing.score=
                                pred.score;
                        }

                    }

                }
            );


            if(!duplicate){

                final.push(
                    pred
                );

            }

        }
    );


    return final;

}


/* =====================================================
   DIBUJAR TRACKS
===================================================== */

function drawTracks(){

    ctx.clearRect(
        0,
        0,
        canvas.width,
        canvas.height
    );


    objectCount.textContent=
        tracks.length;


    tracks.forEach(
        function(track){

            /*
               Si está desapareciendo,
               hacemos la caja ligeramente
               transparente.
            */

            const alpha=
                Math.max(
                    .35,
                    1-
                    track.missed*.08
                );


            ctx.globalAlpha=
                alpha;


            const x=
                track.box[0];

            const y=
                track.box[1];

            const w=
                track.box[2];

            const h=
                track.box[3];


            const d=
                estimateDistance(
                    track.className,
                    h
                );


            const height=
                estimateHeight(
                    track.className,
                    h,
                    d
                );


            const name=
                names[track.className]
                ||
                track.className;


            /* -----------------------------------------
               CAJA
            ----------------------------------------- */

            ctx.strokeStyle=
                "#00d9ff";

            ctx.lineWidth=3;

            ctx.shadowColor=
                "#00d9ff";

            ctx.shadowBlur=14;


            ctx.strokeRect(
                x,
                y,
                w,
                h
            );


            ctx.shadowBlur=0;


            /* -----------------------------------------
               ESQUINAS
            ----------------------------------------- */

            const c=
                Math.min(
                    18,
                    Math.max(
                        8,
                        Math.min(w,h)*.18
                    )
                );


            ctx.strokeStyle=
                "#ffffff";

            ctx.lineWidth=2;


            /* arriba izquierda */

            ctx.beginPath();

            ctx.moveTo(
                x,
                y+c
            );

            ctx.lineTo(
                x,
                y
            );

            ctx.lineTo(
                x+c,
                y
            );

            ctx.stroke();


            /* arriba derecha */

            ctx.beginPath();

            ctx.moveTo(
                x+w-c,
                y
            );

            ctx.lineTo(
                x+w,
                y
            );

            ctx.lineTo(
                x+w,
                y+c
            );

            ctx.stroke();


            /* abajo izquierda */

            ctx.beginPath();

            ctx.moveTo(
                x,
                y+h-c
            );

            ctx.lineTo(
                x,
                y+h
            );

            ctx.lineTo(
                x+c,
                y+h
            );

            ctx.stroke();


            /* abajo derecha */

            ctx.beginPath();

            ctx.moveTo(
                x+w-c,
                y+h
            );

            ctx.lineTo(
                x+w,
                y+h
            );

            ctx.lineTo(
                x+w,
                y+h-c
            );

            ctx.stroke();


            /* -----------------------------------------
               PANEL DE INFORMACIÓN
            ----------------------------------------- */

            const panelWidth=
                Math.max(
                    185,
                    Math.min(
                        250,
                        w
                    )
                );


            const panelHeight=
                78;


            let py=
                y-
                panelHeight-
                5;


            if(
                py<5
            ){

                py=
                    y+5;

            }


            ctx.fillStyle=
                "rgba(0,12,25,.92)";


            ctx.fillRect(
                x,
                py,
                panelWidth,
                panelHeight
            );


            ctx.strokeStyle=
                "#00d9ff";

            ctx.lineWidth=1;


            ctx.strokeRect(
                x,
                py,
                panelWidth,
                panelHeight
            );


            /* -----------------------------------------
               TEXTO
            ----------------------------------------- */

            ctx.fillStyle=
                "#00eaff";


            ctx.font=
                "bold 14px monospace";


            ctx.fillText(
                name.toUpperCase(),
                x+7,
                py+17
            );


            ctx.font=
                "11px monospace";


            ctx.fillText(
                "ID: #" +
                track.id +
                "   " +
                "CONF: " +
                (track.score*100)
                .toFixed(0) +
                "%",
                x+7,
                py+34
            );


            ctx.fillText(
                "DIST: " +
                d.toFixed(1) +
                " m",
                x+7,
                py+50
            );


            ctx.fillText(
                "ALTURA: " +
                height.toFixed(2) +
                " m",
                x+7,
                py+66
            );


            /*
               Línea de tracking.
            */

            ctx.strokeStyle=
                "rgba(0,220,255,.55)";

            ctx.lineWidth=1;


            ctx.beginPath();

            ctx.moveTo(
                x+w/2,
                y+h
            );

            ctx.lineTo(
                x+w/2,
                y+h+12
            );

            ctx.stroke();


            ctx.globalAlpha=1;

        }
    );

}


/* =====================================================
   FPS
===================================================== */

function updateFPS(){

    frameCounter++;


    const now=
        performance.now();


    if(
        now-fpsTime>=1000
    ){

        fpsElement.textContent=
            frameCounter;


        frameCounter=0;

        fpsTime=now;
    }

}


/* =====================================================
   VOZ
===================================================== */

function speak(text){

    if(
        !window.speechSynthesis
    ){

        return;
    }


    speechSynthesis.cancel();


    const utterance=
        new SpeechSynthesisUtterance(
            text
        );


    utterance.lang=
        "es-ES";


    utterance.rate=
        .9;


    utterance.pitch=
        .72;


    utterance.volume=
        1;


    speechSynthesis.speak(
        utterance
    );

}


/* =====================================================
   MICROFONO
===================================================== */

function listen(){

    const Recognition=
        window.SpeechRecognition ||
        window.webkitSpeechRecognition;


    if(!Recognition){

        message.textContent=
            "El reconocimiento de voz no está disponible en este navegador.";

        speak(
            "El reconocimiento de voz no está disponible."
        );

        return;
    }


    const recognition=
        new Recognition();


    recognition.lang=
        "es-ES";


    recognition.continuous=
        false;


    recognition.interimResults=
        false;


    recognition.maxAlternatives=
        1;


    message.textContent=
        "Te escucho...";


    try{

        recognition.start();

    }

    catch(error){

        console.log(error);

        message.textContent=
            "No se pudo iniciar el micrófono.";

        return;
    }


    recognition.onresult=
        function(event){

            const command=
                event
                .results[0][0]
                .transcript
                .toLowerCase();


            processCommand(
                command
            );

        };


    recognition.onerror=
        function(event){

            console.log(
                "VOICE ERROR:",
                event.error
            );


            if(
                event.error===
                "not-allowed"
            ){

                message.textContent=
                    "Permiso de micrófono bloqueado.";

            }

            else{

                message.textContent=
                    "No pude entenderte.";

            }

        };

}


/* =====================================================
   COMANDOS JARVIS
===================================================== */

function processCommand(
    command
){

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


    if(
        command.includes("cuántos")
        ||
        command.includes("cuantos")
    ){

        respond(
            "Actualmente mantengo " +
            tracks.length +
            " objetos en seguimiento."
        );

        return;
    }


    if(
        command.includes("más cerca")
        ||
        command.includes("mas cerca")
    ){

        closestObject();

        return;
    }


    if(
        command.includes("estado")
        ||
        command.includes("sistemas")
    ){

        respond(
            "Cámara, inteligencia visual y tracking están operativos."
        );

        return;
    }


    if(
        command.includes("ayuda")
        ||
        command.includes("comandos")
    ){

        respond(
            "Puedes preguntarme qué veo, cuántos objetos detecto, cuál está más cerca o pedir el estado del sistema."
        );

        return;
    }


    respond(
        "Comando recibido. Todavía no tengo una respuesta configurada para esa pregunta."
    );

}


/* =====================================================
   RESPONDER
===================================================== */

function respond(text){

    message.textContent=
        text;

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


    let text=
        "Detecto " +
        tracks.length +
        " objetos. ";


    const used={};


    tracks
    .slice(0,6)
    .forEach(
        function(track){

            const name=
                names[track.className]
                ||
                track.className;


            if(
                !used[name]
            ){

                text+=
                    name+
                    ". ";

                used[name]=true;

            }

        }
    );


    respond(
        text
    );

}


/* =====================================================
   OBJETO MÁS CERCANO
===================================================== */

function closestObject(){

    if(
        tracks.length===0
    ){

        respond(
            "No detecto objetos."
        );

        return;
    }


    let nearest=
        tracks[0];


    let nearestDistance=
        distanceForTrack(
            nearest
        );


    tracks.forEach(
        function(track){

            const d=
                distanceForTrack(
                    track
                );


            if(
                d<
                nearestDistance
            ){

                nearest=
                    track;

                nearestDistance=
                    d;
            }

        }
    );


    const name=
        names[nearest.className]
        ||
        nearest.className;


    respond(
        "El objeto más cercano parece ser " +
        name +
        ", aproximadamente a " +
        nearestDistance.toFixed(1) +
        " metros."
    );

}


/* =====================================================
   DISTANCIA DE TRACK
===================================================== */

function distanceForTrack(
    track
){

    return estimateDistance(
        track.className,
        track.box[3]
    );

}


/* =====================================================
   BOTONES
===================================================== */

startButton.onclick=
    startCamera;


voiceButton.onclick=
    listen;


/* =====================================================
   INICIO
===================================================== */

console.log(
    "JARVIS AI VISION + TRACKING ONLINE"
);

</script>

</body>
</html>
