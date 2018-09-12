## 百度地图渐变轨迹渲染
> 最新的项目中，需要做车辆轨迹和轨迹播放，看见百度地图有鹰眼轨迹平台，里面的轨迹这一块做得很好，所以就想办法把里面的代码迁移到我的项目当中
![tab](/img/2018-09-08-1.png)

原理剖析：
1. 其实这个渐变轨迹有三层，如下图：<br><br>
![tab](/img/2018-09-08-2.png)
<br><br>
2. 实现原理其实就是先用百度地图`JavascriptAPI`里面的`pointToPixel`先把地理坐标转换为物理像素坐标点，<br>  
[1]. 先根据得到的物理像素坐标点在`canvas`里面画一个`width`为`8`的线条
<br> 
```javascript (type)
  function updateBack() {
    const nextArray = [];
    const ctx = self.canvasLayerBack.canvas.getContext('2d');
    if (!ctx) {
      return;
    }
    ctx.clearRect(0, 0, ctx.canvas.width, ctx.canvas.height);
    if (totalPoints.length !== 0) {
      for (let i = 0, len = totalPoints.length; i < len - 1; i += 1) {
        const pixel = self.SMap.pointToPixel(totalPoints[i]);
        const nextPixel = self.SMap.pointToPixel(totalPoints[i + 1]);
        ctx.beginPath();
        ctx.moveTo(pixel.x, pixel.y);
        ctx.lineWidth = 8;
        ctx.strokeStyle = '#8b8b89';
        ctx.lineTo(nextPixel.x, nextPixel.y);
        ctx.lineCap = 'round';
        ctx.stroke();
      }
    }
  }
```

[2]. 然后渲染渐变层，根据当前当前点和下一个点的速度获取到相应的颜色值，并通过`canvas`上`ctx.beginPath`和`ctx.createLinearGradient`渲染一个渐变的线条，`width`为`6`。

```javascript (type)
function update() {
  const ctx = self.canvasLayer.canvas.getContext('2d');
  if (!ctx) {
    return;
  }
  ctx.clearRect(0, 0, ctx.canvas.width, ctx.canvas.height);
  if (totalPoints.length !== 0) {
    // 绘制带速度颜色的轨迹
    for (let i = 0, len = totalPoints.length; i < len -2; i+=1) {
      const pixel = self.SMap.pointToPixel(totalPoints[i]);
      const nextPixel = self.SMap.pointToPixel(totalPoints[i + 1]);
      ctx.beginPath();
      ctx.moveTo(pixel.x, pixel.y);
      ctx.lineCap = 'round';
      ctx.lineWidth = 6;
      const grd = ctx.createLinearGradient(pixel.x, pixel.y, nextPixel.x, nextPixel.y);
      const { speed } = totalPoints[i];
      const speedNext = totalPoints[i + 1].speed;
      grd.addColorStop(0, self.getColorBySpeed(speed));
      grd.addColorStop(1, self.getColorBySpeed(speedNext));
      ctx.strokeStyle = grd;
      ctx.lineTo(nextPixel.x, nextPixel.y);
      ctx.stroke();
    }
  }
}
```

[3]. 最后渲染箭头层，在堵车的地方，车速慢，坐标点多，这里就要根据渲染的距离判断这里的左边的需要渲染箭头不，已保证箭头不能太密集了

```javascript (type)
function updatePointer() {
  const ctx = self.CanvasLayerPointer.canvas.getContext('2d');
  if (!ctx) {
    return;
  }
  ctx.clearRect(0, 0, ctx.canvas.width, ctx.canvas.height);
  if (totalPoints.length !== 0) {
    const lineObj = {};
    let pixelPart = 0;
    const pixelPartUnit = 40;
    for (let i = 0, len = totalPoints.length; i < len - 1; i += 1) {
      const pixel = self.SMap.pointToPixel(totalPoints[i]);
      const nextPixel = self.SMap.pointToPixel(totalPoints[i + 1]);
      pixelPart += (((nextPixel.x - pixel.x) ** 2) + ((nextPixel.y - pixel.y) ** 2)) ** 0.5;
      if (pixelPart <= pixelPartUnit) {
        // continue;
      }
      pixelPart = 0;
      ctx.beginPath();
      // 根据渲染像素距离渲染箭头
      if (Math.abs(nextPixel.x - pixel.x) > 10 || Math.abs(nextPixel.y - pixel.y) > 10) {
        // 箭头一共需要5个点：起点、终点、中心点、箭头端点1、箭头端点2

        const midPixel = new self.BMap.Pixel(
          (pixel.x + nextPixel.x) / 2,
          (pixel.y + nextPixel.y) / 2,
        );

        // 起点终点距离
        const distance = (((nextPixel.x - pixel.x) ** 2)
        + ((nextPixel.y - pixel.y) ** 2)) ** 0.5;
        // 箭头长度
        const pointerLong = 4;
        const aPixel: any = {};
        const bPixel: any = {};
        if (nextPixel.x - pixel.x === 0) {
          if (nextPixel.y - pixel.y > 0) {
            aPixel.x = midPixel.x - (pointerLong * (0.5 ** 0.5));
            aPixel.y = midPixel.y - (pointerLong * (0.5 ** 0.5));
            bPixel.x = midPixel.x + (pointerLong * (0.5 ** 0.5));
            bPixel.y = midPixel.y - (pointerLong * (0.5 ** 0.5));
          } else if (nextPixel.y - pixel.y < 0) {
            aPixel.x = midPixel.x - (pointerLong * (0.5 ** 0.5));
            aPixel.y = midPixel.y + (pointerLong * (0.5 ** 0.5));
            bPixel.x = midPixel.x + (pointerLong * (0.5 ** 0.5));
            bPixel.y = midPixel.y + (pointerLong * (0.5 ** 0.5));
          } else {
            // continue;
          }
        } else {
          const k0 = (
            (
              (-(2 ** 0.5) * distance * pointerLong)
              + (2 * (nextPixel.y - pixel.y) * midPixel.y)
            ) / (2 * (nextPixel.x - pixel.x))) + midPixel.x;
          const k1 = -((nextPixel.y - pixel.y) / (nextPixel.x - pixel.x));
          const a = (k1 ** 2) + 1;
          const b = (2 * k1 * (k0 - midPixel.x)) - (2 * midPixel.y);
          const c = (((k0 - midPixel.x) ** 2) + (midPixel.y ** 2)) - (pointerLong ** 2);

          aPixel.y = (-b + (((b * b) - (4 * a * c)) ** 0.5)) / (2 * a);
          bPixel.y = (-b - (((b * b) - (4 * a * c)) ** 0.5)) / (2 * a);
          aPixel.x = (k1 * aPixel.y) + k0;
          bPixel.x = (k1 * bPixel.y) + k0;
        }
        ctx.moveTo(aPixel.x, aPixel.y);
        ctx.lineWidth = 2;
        ctx.strokeStyle = '#eee';
        ctx.lineTo(midPixel.x, midPixel.y);
        ctx.lineTo(bPixel.x, bPixel.y);
        ctx.lineCap = 'round';
        ctx.stroke();
      }
    }
  }
}
```

最后说一下，百度地图`canvas`图层的官方插件`canvasLayer`,github地址：https://github.com/huiyan-fe/CanvasLayer