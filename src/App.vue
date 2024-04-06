<script lang="ts" setup>
import { onMounted, ref } from 'vue';

import '@mediapipe/face_detection'
import '@tensorflow/tfjs-core'

import '@tensorflow/tfjs-backend-webgl'
import * as faceDetection from '@tensorflow-models/face-detection'

const video = ref()
const constraints = ref({
  video: {
    width: 640,
    height: 480
  },
  audio: false
})
const canvas = ref()

const setupCamera = async () => {
  const stream = await navigator.mediaDevices.getUserMedia(
    constraints.value
  )
  video.value.srcObject = stream
}

let faceDetector: any
const setupModel = async () => {
  const model = faceDetection.SupportedModels.MediaPipeFaceDetector
  const detectorConfig = {
    runtime: 'mediapipe',
    solutionPath: "https://cdn.jsdelivr.net/npm/@mediapipe/face_detection"
  }
  faceDetector = await faceDetection.createDetector(model, detectorConfig)
}

const setupCanvas = () => {
  canvas.value.width = constraints.value.video.width
  canvas.value.height = constraints.value.video.height
}

const drawingFaceBound = async () => {

  const ctx = canvas.value.getContext("2d")

  const faces = await faceDetector.estimateFaces(video.value)

  ctx.drawImage(video.value, 0, 0, video.value.offsetWidth, video.value.offsetHeight)
  faces.forEach((face: any) => {
    const { box } = face

    ctx.strokeStyle = "red"
    ctx.lineWidth = 5
    ctx.strokeRect(box.xMin, box.yMin, box.width, box.height)
  });

  window.requestAnimationFrame(drawingFaceBound)
}

const loadVideo = () => {
  window.requestAnimationFrame(drawingFaceBound)
}

onMounted(async () => {
  await setupModel()
  await setupCamera()
  setupCanvas()
})
</script>

<template>
  <div>
    <video id="video" ref="video" autoplay @loadeddata="loadVideo"></video>
    <canvas id="canvas" ref="canvas"></canvas>
  </div>
</template>

<style scoped>
#video {
  position: absolute;
  z-index: -1;
}
</style>
