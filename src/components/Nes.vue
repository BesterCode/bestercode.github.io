<template>
  <div id="home__content" ref="homeContent">
    <canvas id="canvasGrid"></canvas>
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
                </div>
                <div class="famicom_images_container">
                  <div class="cover_containerFamicom">
                    <img class="coverFamicom" :src="game.Cover"/>
                  </div>
                  <div class="screenshot_containerFamicom">
                    <img class="screenshotFamicom" :src="game.Screenshot"/>
                  </div>
                </div>
                <div class="gameDescriptionFamicom" v-if="game.Description" :class="{bright: useBrightFont(index)}">{{game.Description}}</div>
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
}

function useBrightFont(cartridgeIndex)
{
  const hexColor = cartridgeColors.get(cartridgeIndex);
  const rgb = hexToRgb(hexColor);
  const luminance = calculateLuminance(rgb);

  console.log(`cartridge ${cartridgeIndex} luminance: ${luminance}`);

  // A common threshold for determining if the color is bright or dark is 0.5
  // This threshold can be adjusted based on desired sensitivity
  return luminance < 0.4 ? true : false;
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
    drawGridOnCanvas();
  };
  window.addEventListener('resize', resizeListener.value);

  const response = await fetch(nesYaml);
  const text = await response.text();
  games.value = yaml.load(text);
});

onUpdated(async () => {
  drawGridOnCanvas();
});

onUnmounted(() => {
  window.removeEventListener('resize', resizeListener.value);
});

function drawGridOnCanvas() {
  const container = homeContent.value;
  if (!container) return;

  const canvas = container.querySelector('#canvasGrid');
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
</script>

<style scoped>
#home__content {
  margin: 0;
}

#canvasGrid {
  position: absolute;
  top: 0;
  left: 0;
  z-index: -2;
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
  background-image: url('@/assets/famicom-cartridge-top-layer3.png');
  -webkit-mask-image: url('@/assets/famicom-cartridge-bottom-layer2.png');
  mask-image: url('@/assets/famicom-cartridge-bottom-layer2.png');
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
}

.screenshot_containerFamicom {
  width: 256px;
  height: 224px;
  /* crops the image's top and bottom if it exceeds the size */
  overflow: hidden; /* This hides the parts of the image that exceed the container's dimensions */
  position: relative;
  display: flex;
  align-items: center;
}

.cover_containerFamicom {
  width: 172px;
  height: 224px;
  /* crops the image's top and bottom if it exceeds the size */
  overflow: hidden; /* This hides the parts of the image that exceed the container's dimensions */
  position: relative;
  display: flex;
  align-items: center;
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
}

.coverFamicom {
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

.gamedetailsFamicom {
  display: flex;
  flex-direction: column;
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
  width: 427px;
  height: 48px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: end;
  padding-left: 35px;
  padding-right: 35px;
  padding-top: 0px;
  padding-bottom: 10px;

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
  padding-left: 40px;
  padding-right: 40px;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 40px;
  text-align: center;
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

.famicom_images_container {
  display: flex;
  flex-direction: row;
  margin-left: 34px;
  background-color: black;
  margin-right: 35px;
  border-radius: 8px;
  overflow: hidden;
}
</style>
