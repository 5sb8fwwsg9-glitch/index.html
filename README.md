function drawTarget(target){

    // Si ya no existe el objeto, borrar la caja inmediatamente
    if(!target){

        smoothedBox = null;

        ctx.clearRect(
            0,
            0,
            canvas.width,
            canvas.height
        );

        return;
    }

    // Si existe, seguirlo suavemente
    smoothedBox = smoothBox(
        smoothedBox,
        target.bbox
    );

    const box = smoothedBox;

    const x = box[0];
    const y = box[1];
    const w = box[2];
    const h = box[3];

    ctx.save();

    ctx.strokeStyle = "#00d9ff";
    ctx.lineWidth = 2;

    ctx.shadowColor = "#00d9ff";
    ctx.shadowBlur = 10;

    drawCorners(x, y, w, h);

    ctx.restore();

    drawScan(box);
}
