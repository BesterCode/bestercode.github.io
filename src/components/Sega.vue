<template>
  <div id="home__content" ref="homeContent">
    <canvas id="canvasWhiteLines"></canvas>
    <canvas id="canvasGreenLines"></canvas>    
    <div class="content">
      <div class="header">
        <img src="../assets/Sega_Mega_Drive_Logo.png" alt="Sega Genesis Logo"/>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, onUnmounted, ref } from 'vue';

const homeContent = ref(null);
const resizeListener = ref(null);

onMounted(() => {
  createWhiteGrid();
  createGreenGrid();  

  resizeListener.value = () => {
    createWhiteGrid();
    createGreenGrid();    
  };
  window.addEventListener('resize', resizeListener.value);
});

onUnmounted(() => {
  window.removeEventListener('resize', resizeListener.value);
});

function createGreenGrid() {
  const container = homeContent.value;
  if (!container) return;
  const canvas = container.querySelector('#canvasGreenLines');
  canvas.width = container.offsetWidth;
  canvas.height = container.offsetHeight;
  const ctx = canvas.getContext('2d');  

  // the lines start drawing at this offset from the top
  let verticalOffset = 200;
  
  // Create a vertical gradient for the lines
  // the transition lasts for 300 pixels, i.e. from verticalOffset to verticalOffset+300
  const lineGradient = ctx.createLinearGradient(0, verticalOffset, 0, verticalOffset+300);
  lineGradient.addColorStop(0, 'rgba(49, 255, 109, 0.0)'); // fully transparent at the top
  lineGradient.addColorStop(0.1, 'rgba(49, 255, 109, 0.1)'); // fully transparent at the top
  lineGradient.addColorStop(0.2, 'rgba(49, 255, 109, 0.3)'); // fully transparent at the top
  lineGradient.addColorStop(1, 'rgba(49, 255, 109, 1.0)'); // fully upaque at the bottom
  // Use this gradient for drawing lines
  ctx.strokeStyle = lineGradient;

  // other setup for the lines
  ctx.lineWidth = 2;
  ctx.shadowBlur = 8;
  ctx.shadowColor = 'green';
  
  let intervalBetweenLines = 10;
  drawHorizontalLines(ctx, verticalOffset, intervalBetweenLines, canvas.width, canvas.height);
  drawVerticalLines(ctx, verticalOffset, canvas.width, canvas.height, 2.8);

  // Setup for the triangle
  const triangleVerticalOffset = 10;
  const triangleHeight = 230;
  const triangleWidth = 500;
  drawTriangle(ctx, canvas.width, triangleVerticalOffset, triangleWidth, triangleHeight);
}

function createWhiteGrid() {
  const container = homeContent.value;
  if (!container) return;

  const canvas = container.querySelector('#canvasWhiteLines');
  canvas.width = container.offsetWidth;
  canvas.height = container.offsetHeight;

  const ctx = canvas.getContext('2d');

  // Set the background color for the canvas  
  ctx.fillStyle = 'black';
  ctx.fillRect(0, 0, canvas.width, canvas.height);

  // the lines draw until this offset
  let verticalCutoff = 250;

  // Create a vertical gradient for the lines
  const lineGradient = ctx.createLinearGradient(0, 0, 0, verticalCutoff);
  lineGradient.addColorStop(0, 'rgba(255, 255, 255, 1.0)');
  lineGradient.addColorStop(0.5, 'rgba(255, 255, 255, 0.8)');
  lineGradient.addColorStop(1, 'rgba(255, 255, 255, 0.075)');
  // Use this gradient for drawing lines
  ctx.strokeStyle = lineGradient;

  // other setup for the lines
  ctx.lineWidth = 2;
  ctx.shadowBlur = 8;
  ctx.shadowColor = 'purple';

  // Draw white glow lines
  const cellHeight = 60;
  const cellWidth = 40;
  drawWhiteGrid(ctx, canvas.width, canvas.height, cellHeight, cellWidth);
}

function drawHorizontalLines(ctx, verticalOffset, intervalBetweenLines, width, height) {
  let currentY = verticalOffset+40; // Starting position for the first line
  let interval = intervalBetweenLines+40; // Initial interval between lines

  while (currentY <= height) {
    ctx.beginPath();
    ctx.moveTo(0, currentY);
    ctx.lineTo(width, currentY);
    ctx.stroke();

    currentY += interval; // Move to the position for the next line
    interval += 5; // Increase the interval for the next line
  }
}

function drawVerticalLines(ctx, verticalOffset, width, height, angleFactor) {
  const centerX = width / 2;
  const lineSpacing = 50; // Spacing between each line
  const angleIncrease = Math.PI / 180; // Angle increase in radians for each line

  // Draw the central vertical line
  drawLine(ctx, centerX, verticalOffset, centerX, height);

  // Draw lines to the right and left of the center
  for (let i = 1; i <= (width - centerX) / lineSpacing; i++) {
    // Calculate the angle for the current line
    let angle = angleFactor * i * angleIncrease;

    // Lines to the right of the center
    drawAngledLine(ctx, centerX + i * lineSpacing, verticalOffset, angle, height);

    // Lines to the left of the center
    drawAngledLine(ctx, centerX - i * lineSpacing, verticalOffset, -angle, height);
  }
}

function drawAngledLine(ctx, x, y, angle, height) {
  ctx.beginPath();
  ctx.moveTo(x, y);
  // Calculate the end point of the line using the angle
  ctx.lineTo(x + Math.tan(angle) * height, height);
  ctx.stroke();
}

function drawLine(ctx, x1, y1, x2, y2) {
  ctx.beginPath();
  ctx.moveTo(x1, y1);
  ctx.lineTo(x2, y2);
  ctx.stroke();
}

function drawWhiteGrid(ctx, width, height, cellHeight, cellWidth) {
  // Draw horizontal and vertical lines
  for (let x = 25; x < width; x += cellWidth) {
    ctx.beginPath();
    ctx.moveTo(x, 0);
    ctx.lineTo(x, height);
    ctx.stroke();
  }
  for (let y = 25; y < height; y += cellHeight) {
    ctx.beginPath();
    ctx.moveTo(0, y);
    ctx.lineTo(width, y);
    ctx.stroke();
  }
}

function drawTriangle(ctx, canvasWidth, verticalOffset, triangleWidth, triangleHeight) {

  // Create a vertical gradient for the triangle
  const triangleGradientTop = ctx.createLinearGradient(0, verticalOffset, 0, verticalOffset + triangleHeight);
  triangleGradientTop.addColorStop(0, 'rgba(247, 0, 247, 1.0)'); 
  triangleGradientTop.addColorStop(0.4, 'rgba(255, 255, 255, 1.0)'); 
  triangleGradientTop.addColorStop(0.4, 'rgba(255, 255, 255, 0.0)'); 
  triangleGradientTop.addColorStop(1, 'rgba(255, 230, 255, 0.0)');
  // Use this gradient for drawing lines
  ctx.strokeStyle = triangleGradientTop;
  // other setup for the lines
  ctx.lineWidth = 3;
  ctx.shadowBlur = 4;
  ctx.shadowColor = 'deeppink';

  const centerX = canvasWidth / 2;

  ctx.beginPath();
  ctx.moveTo(centerX, verticalOffset + triangleHeight); // Start from the top center
  ctx.lineTo(centerX - triangleWidth/2, verticalOffset); // Draw to the bottom left
  ctx.lineTo(centerX + triangleWidth/2, verticalOffset); // Draw to the bottom right
  ctx.closePath();

  // Stroke the lines to create the triangle outline
  ctx.stroke();

  const triangleGradientBottom = ctx.createLinearGradient(0, verticalOffset, 0, verticalOffset + triangleHeight);
  triangleGradientBottom.addColorStop(0, 'rgba(247, 0, 247, 0.0)'); 
  triangleGradientBottom.addColorStop(0.4, 'rgba(255, 255, 255, 0.0)'); 
  triangleGradientBottom.addColorStop(0.4, 'rgba(255, 255, 255, 1.0)'); 
  triangleGradientBottom.addColorStop(1, 'rgba(255, 230, 255, 1.0)');
  // Use this gradient for drawing lines
  ctx.strokeStyle = triangleGradientBottom;
  ctx.shadowBlur = 0;
  ctx.shadowColor = 'white';
  ctx.beginPath();
  ctx.moveTo(centerX, verticalOffset + triangleHeight); // Start from the top center
  ctx.lineTo(centerX - triangleWidth/2, verticalOffset); // Draw to the bottom left
  ctx.lineTo(centerX + triangleWidth/2, verticalOffset); // Draw to the bottom right
  ctx.closePath();

  // Stroke the lines to create the triangle outline
  ctx.stroke();
  
}
</script>

<style scoped>
#home__content {
  position: relative;
  height: 100%;
  margin: 0;
}

#canvasWhiteLines {
  position: absolute;
  top: 0;
  left: 0;
  z-index: -2;
}

#canvasGreenLines {
  position: absolute;
  top: 0;
  left: 0;
  z-index: -1;
}

.content {
  position: relative;
  z-index: 1;
  /* Your content styling here */
}

.header {
  display: flex;
  justify-content: center;
}
</style>
