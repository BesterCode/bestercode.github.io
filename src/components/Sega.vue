<template>
  <div id="home__content" ref="homeContent">
    <canvas id="canvasWhiteLines"></canvas>
    <canvas id="canvasGreenLines"></canvas>    
    <div class="content">
      <div class="header">
        <div class="credits">
          <a href="https://github.com/BesterCode/bestercode.github.io">repository</a>
        </div>
        <img src="@/assets/Sega_Mega_Drive_Logo.png" alt="Sega Genesis Logo"/>
        <div class="subheader"><div>WORTHWHILE</div> <div>COLLECTION</div></div>        
      </div>
      <div class="games_container">        
        <div class="game" v-for="game in games" :key="game.Game">
          <div class="genre">{{game.Genre}}</div>
          <div class="gamedetails">
            <div class="screenshot_container">
              <img class="screenshot" :src="game.Screenshot"/>
            </div>
            <div class="gameTexts">
              <span class="gameTitle">{{game.Game}}</span>
              <span class="gameDescription" v-if="game.Description">{{game.Description}}</span>
              <span class="gameDescription" v-if="game.Description2">{{game.Description2}}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, onUpdated, onUnmounted, ref } from 'vue';
import yaml from 'js-yaml';
import segaYaml from '@/assets/sega.yaml';

const games = ref([]);

const homeContent = ref(null);
const resizeListener = ref(null);

onMounted(async () => {
  if (navigator.userAgent.toLowerCase().indexOf('firefox') > -1) {
    document.documentElement.classList.add('firefox');
  }

  resizeListener.value = () => {
    createWhiteGrid();
    createGreenGrid();    
  };
  window.addEventListener('resize', resizeListener.value);

  const response = await fetch(segaYaml);
  const text = await response.text();
  games.value = yaml.load(text);
});

onUpdated(async () => {
  createWhiteGrid();
  createGreenGrid();  
});

onUnmounted(() => {
  window.removeEventListener('resize', resizeListener.value);
});

function createGreenGrid() {
  console.log('createGreenGrid')
  const container = homeContent.value;
  if (!container) return;
  const canvas = container.querySelector('#canvasGreenLines');
  canvas.width =  container.parentElement.offsetWidth; //Math.max(container.offsetWidth, window.innerWidth);
  canvas.height =  Math.max(container.offsetHeight, window.innerHeight);
  const ctx = canvas.getContext('2d');  

  // the lines start drawing at this offset from the top
  let verticalOffset = 200;
  
  // Create a vertical gradient for the lines
  // the transition lasts for 300 pixels, i.e. from verticalOffset to verticalOffset+300
  const lineGradient = ctx.createLinearGradient(0, verticalOffset, 0, verticalOffset+2000);
  lineGradient.addColorStop(0, 'rgba(49, 255, 109, 0.0)'); // fully transparent at the top
  lineGradient.addColorStop(0.015, 'rgba(49, 255, 109, 0.1)'); // 0.1 transparency at 30 pixels from the top
  lineGradient.addColorStop(0.03, 'rgba(49, 255, 109, 0.3)'); // 0.3 at 60 px
  lineGradient.addColorStop(0.15, 'rgba(49, 255, 109, 1.0)'); // 1.0 at 300 px
  lineGradient.addColorStop(0.6, 'rgba(49, 255, 109, 1.0)'); // 1.0 at 1200 px
  lineGradient.addColorStop(1.0, 'rgba(49, 255, 109, 0.4)'); // 0.4 at 2000 px and further down
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
  canvas.width =  container.parentElement.offsetWidth;
  canvas.height =  Math.max(container.offsetHeight, window.innerHeight);

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
    angle = Math.min(Math.max(angle, 0), 1.57); // clamp to 90 degrees to avoid the line going into wild directions

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
  // 30 is the original horizontal offset
  for (let x = 32; x < width; x += cellWidth) {
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
  align-items: center;
  flex-direction: column;  
}

.credits {
  position: absolute;
  right: 10px;
  -webkit-filter: drop-shadow(0px 0px 10px rgb(0, 0, 0));
  filter: drop-shadow(0px 0px 10px rgb(0, 0, 0));
  text-shadow: 
    -1px -1px 0 #ff00dd6b,
    -1px  1px 0 #0000007a,
     1px  1px 0 #0000007a
     ;
  line-height: 1.4;
  font-size: 16px;
  a {
    color: #e165ff;
  }
  a:hover {
    color: #cc5ae8;
  }
}

@font-face {
  font-family: 'Ethnocentric Rg'; 
  src: url('@/assets/fonts/ethnocentric_rg.otf') format('truetype');
}

@font-face {
  font-family: 'Arcade'; 
  src: url('@/assets/fonts/arcade.otf') format('truetype');
}

.subheader {
  display: flex;
  align-items: center;
  gap: 50px;
  margin-top: -22px;
  div {
    width: 450px;
    &:first-child {
      text-align: right;
    }    
    
    /* text color gradient: */
    background: linear-gradient(to bottom, #220081 0%, #3A00FD 25%, white 49.9%, black 50%, #44005A 50.1%, #F1AA00 75%, white 100%);
    -webkit-background-clip: text; /* Webkit-based browsers */
    background-clip: text;
    /* stroke: */
    -webkit-text-stroke: 1px rgba(212, 212, 212, 0.897); /* Webkit-based browsers */
    text-stroke: 1px rgb(179, 179, 179); /* Standard syntax */
    /* outer glow: */
    -webkit-filter: drop-shadow(0px 0px 10px rgb(21, 75, 255));
    filter: drop-shadow(0px 0px 10px rgb(21, 75, 255));
    color: transparent; /* Make the text color transparent */
    display: inline-block; /* Ensure the gradient background spans the text */

    /* font */
    font-family: 'Ethnocentric Rg', sans-serif; /* Fallback to sans-serif if the custom font doesn't load */
    font-size: 43px;
  }
}

/* "worthwhile collection" is too wide of a text for mobile devices */
@media only screen and (max-width: 870px) {  
  .header {    
    overflow: hidden;
    img {
      margin-top: 15px;
      width: 100%;
    }
  }
  .subheader {
    gap: 0px;
    flex-direction: column;
    margin-top: 0px;
    div {
      text-align: center !important;
      font-size: 23px;
    }
  }
}

.games_container {
  display: flex;
  gap: 20px 10px;
  padding: 25px 5px 30px 5px;
  flex-wrap: wrap;
  justify-content: space-evenly;
}

.game {
  width: 374px;
  height: 300px;
  background-image: image-set(
    '@/assets/sega-template.png' 1x,
    '@/assets/sega-template@2x.png' 2x,
  );
  background-size: cover;
  display:flex;
}

.genre {
  writing-mode: vertical-rl;
  transform: rotate(180deg);
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  /* width is 54, the sum of Width and second padding */
  width: 39px;  
  padding: 28px 15px 14px 0px;
  font-family: 'Arcade', sans-serif;
  font-size: 36px;
  word-spacing: -10px;
  line-height: 0.6;
}

.firefox .genre {
  width: 49px;
  padding: 28px 5px 14px 0px;
}

.screenshot_container {
  width: 320px;
  height: 224px;
  /* crops the image's top and bottom if it exceeds the size */
  overflow: hidden; /* This hides the parts of the image that exceed the container's dimensions */
  position: relative;
  display: flex;
  align-items: center;
  /* glow */
  /*filter: drop-shadow(0 10px 10px rgba(128, 0, 128, 0.5));*/
}
.screenshot {
  /* helps with the crop */
  width: 100%; /* Makes the image fill the container's width */
  height: auto; /* Maintains the image's aspect ratio */
  /* helps to make it look crisp, like it's integer scaling on 4k */
  image-rendering: pixelated;
}

.gamedetails {
  width: calc(100% - 54px);
  text-align: center;
  display: flex;
  flex-direction: column;  
}

.gameTexts {
  height: 76px;
  display: flex;
  flex-direction: column;  
  align-items: center;
  justify-content: center;
}

.gameTitle {
  font-size: 18px;
  line-height: 1.1;
}

.gameDescription {
  font-size: 12px;
}
</style>
