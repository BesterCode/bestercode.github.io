<template>
  <div id="home__content" ref="homeContent">
    <canvas id="canvasWhiteLines"></canvas>
    <!-- <canvas id="canvasGreenLines"></canvas> -->
    <div class="content">
      <div class="header">
        <div class="credits">
          <a href="https://github.com/BesterCode/bestercode.github.io">repository</a>
        </div>
        <div class="logo1container pixelated">
          <img src="@/assets/NesLogo1.png" 
                srcset="@/assets/NesLogo1.png 1x, 
                 @/assets/NesLogo1@2x.png 2x"
                alt="NES Logo"/>          
        </div>
        <div class="logo2container pixelated">
          <img src="@/assets/NesLogo2.png" 
                srcset="@/assets/NesLogo2.png 1x, 
                 @/assets/NesLogo2@2x.png 2x"
                alt="NES Logo"/>          
        </div>
        <div class="subheader"><div>WORTHWHILE</div> <div>COLLECTION</div></div>
        <div class="toggleDiv">
          <span>NES</span>
          <Toggler :checked="true"
            @update:checked="toggleFamicom"
            label=""
          />
          <span>FAMICOM</span>
        </div>
      </div>
      <div class="games_container">

        <!-- NES cartridge -->
        <template v-if="!famicom">
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
        </template>

        <!-- Famicom cartridge -->
        <template v-else>
          <div class="gameFamicom" v-for="(game, index) in games" :key="game.Game" :style="{ 'background-color': getRandomColor(index) }">
              <!-- <div class="genre">{{game.Genre}}</div> -->
              <div class="gamedetailsFamicom">
                <div class="gameTextsFamicom">
                  <span class="gameTitleFamicom" :class="{bright: useBrightFont(index)}">{{game.Game}}</span>
                  <span class="gameDescriptionFamicom" v-if="game.Description" :class="{bright: useBrightFont(index)}">{{game.Description}}</span>
                  <span class="gameDescriptionFamicom" v-if="game.Description2" :class="{bright: useBrightFont(index)}">{{game.Description2}}</span>
                </div>
                <div class="screenshot_containerFamicom">
                  <img class="screenshotFamicom" :src="game.Screenshot"/>
                </div>
              </div>
          </div>
        </template>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, onUpdated, onUnmounted, ref } from 'vue';
import yaml from 'js-yaml';
import nesYaml from '@/assets/nes.yaml';
import Toggler from './Toggler.vue';

const games = ref([]);

// switch between NES and Famicom cartridges display
let famicom = ref(true);
function toggleFamicom() {
  famicom.value = !famicom.value;
}

const possibleColors = [
  '#ff770c',
  '#ffb200',
  '#042526ab', // rare
  '#6e76c6',
  '#c6ea4b',
  '#787878',
  '#e4e4e4',
  '#262626',
  '#e95241',
  '#e68379',
  '#7acdd7',
  '#7de9e3',
  '#e4382f',
  '#e8e08e',
  '#1087e1',
  '#b262ff',
  '#fbea66',
  '#0fa563',
  '#936144',
  '#fdc651',
];

const cartridgeColors = new Map();

// Return a random color for a cartridge
// Occasionally return a transparency, because some cartridges were see-through
function getRandomColor(cartridgeIndex) {
  const randomIndex = Math.floor(Math.random() * possibleColors.length);  
  const result = possibleColors[randomIndex];
  cartridgeColors.set(cartridgeIndex, result);
  return result;
  //#042526 - dark green
  const letters = '0123456789ABCDEF';
  let color = '#';
  for (let i = 0; i < 6; i++) {
    color += letters[Math.floor(Math.random() * 16)];
  }
  return color;
}

function useBrightFont(cartridgeIndex)
{
  const hexColor = cartridgeColors.get(cartridgeIndex);
  const rgb = hexToRgb(hexColor);
  const luminance = calculateLuminance(rgb);

  console.log(`cartridge ${cartridgeIndex} luminance: ${luminance}`);

  // A common threshold for determining if the color is bright or dark is 0.5
  // This threshold can be adjusted based on desired sensitivity
  return luminance < 0.5 ? true : false;
}

function hexToRgb(hex) {
    // Remove the "#" if present
    if (hex.charAt(0) === '#') {
        hex = hex.substr(1);
    }

    // Parse the R, G, B values
    let bigint = parseInt(hex, 16);
    let r = (bigint >> 16) & 255;
    let g = (bigint >> 8) & 255;
    let b = bigint & 255;

    return [r, g, b];
}

function calculateLuminance([r, g, b]) {
    // Normalize RGB values to the range 0 - 1
    r /= 255;
    g /= 255;
    b /= 255;

    // Calculate luminance
    const luminance = 0.2126 * r + 0.7152 * g + 0.0722 * b;
    return luminance;
}

const homeContent = ref(null);
const resizeListener = ref(null);

onMounted(async () => {
  if (navigator.userAgent.toLowerCase().indexOf('firefox') > -1) {
    document.documentElement.classList.add('firefox');
  }

  resizeListener.value = () => {
    createWhiteGrid();
    //createGreenGrid();    
  };
  window.addEventListener('resize', resizeListener.value);

  const response = await fetch(nesYaml);
  const text = await response.text();
  games.value = yaml.load(text);
});

onUpdated(async () => {
  createWhiteGrid();
  //createGreenGrid();  
});

onUnmounted(() => {
  window.removeEventListener('resize', resizeListener.value);
});

function createGreenGrid() {
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

  // the lines draw until this offset
  let verticalCutoff = 250;

  // Create a linear gradient
// The gradient starts at the top (y=0) and ends at y=verticalCutoffpixels
  const backgroundGradient = ctx.createLinearGradient(0, 0, 0, 250);
  backgroundGradient.addColorStop(0, '#000000'); // Start with black at the top
  backgroundGradient.addColorStop(0.85, '#000000'); // Continue with black towards 85% of the height
  backgroundGradient.addColorStop(1.0, '#0f0619'); // Transition to #0f0619 towards verticalCutoff

  // Fill the rest of the canvas with #0f0619
  // This ensures a smooth transition in the first 250 pixels
  // and a solid color (#0f0619) afterwards
  ctx.fillStyle = backgroundGradient;
  ctx.fillRect(0, 0, canvas.width, verticalCutoff); // Apply the gradient only to the first 250 pixels

  // If needed, fill the rest of the canvas height with a solid color (#0f0619)
  // This step may be optional if the gradient's last color stop suffices for the visual effect
  ctx.fillStyle = '#0f0619';
  ctx.fillRect(0, verticalCutoff, canvas.width, canvas.height - verticalCutoff);

  // Create a vertical gradient for the lines
  const lineGradient = ctx.createLinearGradient(0, 0, 0, verticalCutoff);
  lineGradient.addColorStop(0, 'rgba(255, 255, 255, 0.0)');
  lineGradient.addColorStop(0.8, 'rgba(255, 255, 255, 0.0)');
  lineGradient.addColorStop(1, 'rgba(255, 255, 255, 0.15)');
  // Use this gradient for drawing lines
  ctx.strokeStyle = lineGradient;

  // other setup for the lines
  ctx.lineWidth = 3;
  ctx.shadowBlur = 2;
  ctx.shadowColor = 'purple';

  // Draw white glow lines
  const cellHeight = 60;
  const cellWidth = 60;
  drawWhiteGrid(ctx, canvas.width, canvas.height, cellHeight, cellWidth);

  // Draw 500 stars
  drawStars(canvas, ctx, 500, verticalCutoff);
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

function drawStars(canvas, ctx, count, verticalCutoff) {
  for (let i = 0; i < count; i++) {      
    const x = Math.random() * canvas.width;
    const y = Math.random() * verticalCutoff; // draw stars only in the upper 250 pixels of the canvas
    const size = Math.random() * 2 + 1; // Random size for the stars
    // Calculate opacity based on y position
    let opacity = 1;
    if (y > 150) {
      // Opacity decreases from 1 to 0 as y goes from 150 to 250
      opacity = 1 - (y - 150) / 100;
    }
    ctx.fillStyle = `rgba(255, 255, 255, ${opacity})`; // Slightly transparent white
    ctx.fillRect(x, y, size, size); // Draw a pixelated star
  }
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
  padding-bottom: 30px;
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
  font-family: 'PixelEmulator', sans-serif;
  font-size: 14px;
  a {
    color: #e165ff;
  }
  a:hover {
    color: #cc5ae8;
  }
}

@font-face {
  font-family: 'Yoster Island'; 
  src: url('@/assets/fonts/yoster-island.ttf') format('truetype');
}

@font-face {
  font-family: 'Permanent Marker'; 
  src: url('@/assets/fonts/PermanentMarker.ttf') format('truetype');
}

@font-face {
  font-family: 'Nusaliver'; 
  src: url('@/assets/fonts/nusaliver.ttf') format('truetype');
}

@font-face {
  font-family: 'PixelEmulator'; 
  src: url('@/assets/fonts/PixelEmulator.ttf') format('truetype');
}

.subheader {
  display: flex;
  align-items: center;
  gap: 50px;
  line-height: 1.2;

  div {
    width: 450px;
    &:first-child {
      text-align: right;
    }    
    
    /* text color gradient: */
    background: linear-gradient(to bottom, #ffe565 0%, #ffe565 25%, #ff8400 40%, #f32a1a 55%, #c2267a 70%, #8721e6 80%);
    -webkit-background-clip: text; /* Webkit-based browsers */
    background-clip: text;
    
    /* stroke: */
    /* -webkit-text-stroke: 1px rgba(212, 212, 212, 0.897);
    text-stroke: 1px rgb(179, 179, 179); */

    /* outer glow: */
    /* -webkit-filter: drop-shadow(0px 0px 10px rgb(21, 75, 255));
    filter: drop-shadow(0px 0px 10px rgb(21, 75, 255)); */
    color: transparent;
    display: inline-block;

    /* font */
    font-family: 'Yoster Island', sans-serif; /* Fallback to sans-serif if the custom font doesn't load */
    font-size: 45px;
  }
}

/* "worthwhile collection" is too wide of a text for mobile devices */
@media only screen and (max-width: 870px) {  
  .header {    
    overflow: hidden;
    img {
      width: 100%;
    }
    padding-bottom: 0px;
  }
  .subheader {
    gap: 0px;
    flex-direction: column;
    margin-top: 0px;
    div {
      text-align: center !important;
      font-size: 35px;
    }
  }
}

.games_container {
  display: flex;
  gap: 20px 10px;
  padding: 30px 5px 30px 5px;
  flex-wrap: wrap;
  justify-content: space-evenly;
}

.game {
  width: 323px;
  height: 330px;
  background-image: image-set(
    '@/assets/nes-template.png' 1x,
    '@/assets/nes-template@2x.png' 2x,
  );
  background-size: cover;
  display:flex;
}

.gameFamicom {
  width: 497px;
  height: 331px;
  background-size: cover;
  display:flex;
  background-image: url('@/assets/famicom-cartridge-top-layer.png');
  -webkit-mask-image: url('@/assets/famicom-cartridge-bottom-layer.png');
  mask-image: url('@/assets/famicom-cartridge-bottom-layer.png');
  background-color: orange; /* The color you want */
}

.genre {
  writing-mode: vertical-rl;
  transform: rotate(180deg);
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;  
  width: 50px;
  padding: 53px 0px 10px 0px;
  font-family: 'PixelEmulator', sans-serif;
  font-size: 26px;
  word-spacing: -9px;
  line-height: 0.8;
}

.screenshot_container {
  width: 256px;
  height: 224px;
  /* crops the image's top and bottom if it exceeds the size */
  overflow: hidden; /* This hides the parts of the image that exceed the container's dimensions */
  position: relative;
  display: flex;
  align-items: center;
  padding-top: 16px;
  /* glow */
  /*filter: drop-shadow(0 10px 10px rgba(128, 0, 128, 0.5));*/
}

.screenshot_containerFamicom {
  width: 256px;
  height: 224px;
  /* crops the image's top and bottom if it exceeds the size */
  overflow: hidden; /* This hides the parts of the image that exceed the container's dimensions */
  position: relative;
  display: flex;
  align-items: center;
  padding-top: 61px;
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

.screenshotFamicom {
  /* helps with the crop */
  width: 100%; /* Makes the image fill the container's width */
  height: auto; /* Maintains the image's aspect ratio */
  /* helps to make it look crisp, like it's integer scaling on 4k */
  image-rendering: pixelated;

  border-top-right-radius: 12px;
  border-bottom-right-radius: 12px;
}

.gamedetails {
  width: calc(100% - 54px);
  text-align: center;
  display: flex;
  flex-direction: column;  
}

.gamedetailsFamicom {
  display: flex;
}

.gameTexts {
  height: 86px;
  display: flex;
  flex-direction: column;  
  align-items: center;
  justify-content: center;
  padding-left: 70px;
  padding-right: 23px;
}

.gameTextsFamicom {
  width: 171px;
  height: 217px;
  display: flex;
  flex-direction: column;  
  align-items: center;
  justify-content: center;
  padding-left: 35px;
  padding-right: 1px;
  padding-top: 67px;

  span {
  text-align: center;
  }
}

.gameTitle {
  font-size: 18px;
  line-height: 1.1;
}

.gameTitleFamicom {
  font-size: 22px;
  line-height: 1.1;
  font-family: 'Permanent Marker', monospace;
  color: rgb(34, 34, 34);
  &.bright {
    color: rgb(226, 226, 226);
  }
}

.gameDescription {
  font-size: 12px;
}

.gameDescriptionFamicom {
  font-size: 12px;
  font-family: 'Nusaliver', monospace;
  color: rgb(34, 34, 34);
  &.bright {
    color: rgb(226, 226, 226);
  }
}

.logo1container {
  padding-top: 30px;  
}

.logo2container {
  padding-top: 10px;
}

.pixelated img {
  image-rendering: pixelated;
}

.logo1container img {
  /* Ensure the image fits within the container. Adjust as necessary. */
  width: 100%;
  height: auto;
  display: block; /* Remove space below the image */  

  /* Create and apply the gradient mask */
  -webkit-mask-image: linear-gradient(to right, rgba(0,0,0,0) 0%, rgba(0,0,0,1) 10%, rgba(0,0,0,1) 90%, rgba(0,0,0,0) 100%);
  mask-image: linear-gradient(to right, rgba(0,0,0,0) 0%, rgba(0,0,0,1) 10%, rgba(0,0,0,1) 90%, rgba(0,0,0,0) 100%);    
}

.toggleDiv {
  :first-child {
    width: 73px;
    text-align: right;
  }
  :last-child {
    color: orange;
  }
  color: #e878ff;
  display: flex;
  width: 217px;
  justify-content: space-between;
}
</style>
