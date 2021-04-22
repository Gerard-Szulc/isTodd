<template>
  <header>
    <h3>Teachable Machine Todd Howard Image Model</h3>
    <h3>Fully based only on memes from the internet</h3>
    <div>Its a tribute for best video game designer and producer Todd Howard.</div>
  </header>

  <div class="buttons-container">
    <button type="button" @click.prevent="init()">Start webcam</button>
    <button type="button" @click.prevent="startUploadFile">Start image</button>
  </div>
  <div v-if="state.progress !== 0 && state.progress !== 200" :style="`width: ${state.progress}px; background: red; height: 20px;`">{{state.progress}}</div>

  <input v-show="false" id="file-uploader" type="file" @change="handleUploadFile" accept="image/*">
  <div id="webcam-container" v-show="!state.loading"></div>
  <div v-show="state.loading" class="loading"></div>
  <div v-for="container in state.classes" :key="container" class="prediction-classes">{{ container }}</div>


</template>

<script setup>
import {reactive} from 'vue'

const FILE_MODE = 'FILE_MODE'
const WEBCAM_MODE = 'WEBCAM_MODE'

const state = reactive({
  classes: [],
  canvas: null,
  canvasContainer: null,
  mode: '',
  progress: 0,
  loading: false,
})

const URL = '/';
const modelURL = URL + "model.json";
const metadataURL = URL + "metadata.json";

let model, webcam, maxPredictions, loadedData;

async function init() {
  state.loading = true
  state.mode = WEBCAM_MODE

  model = await tmImage.load(modelURL, metadataURL)
  maxPredictions = model.getTotalClasses()

  const flip = true
  webcam = new tmImage.Webcam(400, 400, flip)

  await webcam.setup()
  state.canvasContainer = document.getElementById("webcam-container")
  if (state.canvas) state.canvasContainer.removeChild(state.canvas)
  state.canvas = webcam.canvas
  state.canvasContainer.appendChild(state.canvas)
  await webcam.play()

  window.requestAnimationFrame(loop)

  state.classes = maxPredictions
  state.loading = false
}

async function loop() {
  if (state.mode !== WEBCAM_MODE) {
    return
  }
  webcam.update()
  await predict(webcam.canvas, true)
  window.requestAnimationFrame(loop)
}

async function predict(element, flip = false) {
  const prediction = await model.predict(element, flip)
  state.classes = prediction.map(predicitonElement => {
    return predicitonElement.className + ": " + (predicitonElement.probability.toFixed(2) * 100).toFixed(0) + '%'
  })
}

const loaded = async (fileData) => {

  model = await tmImage.load(modelURL, metadataURL)
  maxPredictions = model.getTotalClasses()

  state.canvasContainer = document.getElementById("webcam-container")
  if (state.canvas) state.canvasContainer.removeChild(state.canvas)
  let image = new Image()

  image.onload = async () => {
    let canvas = document.createElement("CANVAS")
    let ctx = canvas.getContext("2d")
    canvas.height = canvas.width * (image.height / image.width);
    ctx.drawImage(image, 0, 0, canvas.width, canvas.height);
    state.canvas = canvas
    state.canvas.style.maxHeight = '60vh'
    state.canvasContainer.appendChild(state.canvas)
    state.classes = maxPredictions
    await predict(canvas, false)
    state.loading = false
  }
  image.src = fileData.target.result
}

const getAsBase64 = (readFile) => {

  let reader = new FileReader();

  reader.readAsDataURL(readFile);
  reader.onprogress = updateProgress;
  reader.onload = loaded;
  reader.onerror = () => {
    console.log('error')
  }
}

function updateProgress(evt) {
  console.log(evt.type, evt.loaded)
  if (evt.lengthComputable) {
    // evt.loaded and evt.total are ProgressEvent properties
    loadedData = (evt.loaded / evt.total);
    state.progress = loadedData * 200;
    if (loadedData < 1) {
      console.log(loadedData)
      // Increase the prog bar length
      state.progress = loadedData * 200;
    }
  }
}

const handleUploadFile = (e) => {
  state.loading = true

  if (webcam) {
    webcam.stop();
  }

  state.mode = FILE_MODE

  let file = e.target.files[0]
  if (file) {
    getAsBase64(file);
  }
}
const startUploadFile = () => {
  state.progress = 0
  document.getElementById('file-uploader').click()
}
</script>

<style scoped lang="scss">

header div, header h3 {
  text-transform: uppercase;
  color: #17fc1c;
}

.buttons-container {
  margin-top: 50px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
.buttons-container button {
  font-weight: bold;
  border-radius: 0;
  background-color: rgba(0,0,0,0);
  border: 0;
  color: #17fc1c;
  width: 10rem;
  height: 3rem;
  text-transform: uppercase;
  outline: 0;
}
.buttons-container button:hover {
  background-color: #17fc1c;
  color: #073207;

}
.prediction-classes {
  color: #17fc1c;

}
#webcam-container {

}

a {
  color: #42b983;
}

* {
  &, &:before, &:after {
    box-sizing: border-box;
    &, &:focus, &:active, &:focus:active {
      outline: none;
    }
  }
}

div.loading {
  left: 50%;
  position: absolute;
  top: 50%;
  transform: translate(-50%, -50%);
}
$color-a: #222;
$color-b: #32cd32;
$line-width: 1rem;
$font-base-size: 10px;

@function makeShadowBars($color, $direction: (-1, 0), $size: .2, $cnt: 3, $length: (10, 15, 10), $offsetX: (0, 0, 0), $offsetY: (-2, 0, 2)) {
  $x: nth($direction, 1);
  $y: nth($direction, 2);
  $val: 0 0 0 0 $color;
  @for $i from 1 through $cnt {
    $len: strip-unit(rem2px(nth($length, $i)));
    @for $a from -$len through $len {
      $val: #{$val}, #{($a * $x) + strip-unit(rem2px(nth($offsetX, $i)))}px #{($a * $y) + strip-unit(rem2px(nth($offsetY, $i)))}px 0 #{$size}rem #{$color};
    }
  }
  @return $val;
}

@function rem2px($rems, $unit: false) {
  $val: strip-unit($rems) * strip-unit($font-base-size);
  @if $unit {
    $val: #{$val}px;
  }
  @return $val;
}

@function strip-unit($number) {
  @if type-of($number) == 'number' and not unitless($number) {
    @return $number / ($number * 0 + 1);
  }
  @return $number;
}

div.loading {
  $size: $line-width * 7;
  $l-width: strip-unit($line-width);
  $l-length: strip-unit($size);
  $shadow-recess: strip-unit(($size - $line-width) / -2);
  height: $size;
  width: $size;
  &:before,
  &:after {
    border: {
      radius: 50%;
    };
    content: '';
    height: $size;
    left: 50%;
    position: absolute;
    top: 50%;
    transform: translate(-50%, -50%);
    width: $size;
    z-index: -1;
  }
  &:before {
    box-shadow: makeShadowBars($color-b, (-1, 0), $shadow-recess, 3, ($l-length, $l-length * 1.5, $l-length), (0, 0, 0), ($l-width * -2, 0, $l-width * 2));
  }
  &:after {
    animation: radarPulse 1.8s ease-out infinite;
    border: {
      radius: 50%;
    };
    box-shadow:
        inset 0 0 0 rem2px(.5rem, true) $color-b,
        inset 0 0 0 rem2px($size / 2.2, true) $color-a,
        inset 0 0 0 rem2px($size / 2, true) $color-b,
  ;
    z-index: 1;
  }

  @keyframes radarPulse {
    $x: strip-unit($size);
    $x2: $x / 2.2;
    from {
      box-shadow:
          inset 0 0 0 ceil(rem2px(.5))+px $color-b,
          inset 0 0 0 ceil(rem2px($x2))+px $color-a,
          inset 0 0 0 ceil(rem2px($x2 + .3))+px $color-b,
          inset 0 0 0 ceil(rem2px($x2))+px $color-a,
          inset 0 0 0 ceil(rem2px($x / 2))+px $color-b,
    ;
    }
    to {
      box-shadow:
          inset 0 0 0 ceil(rem2px(.5))+px $color-b,
          inset 0 0 0 0 $color-a,
          inset 0 0 0 ceil(rem2px(.3))+px $color-b,
          inset 0 0 0 ceil(rem2px($x2))+px $color-a,
          inset 0 0 0 ceil(rem2px($x / 2))+px $color-b,
    ;
    }
  }
}


</style>
