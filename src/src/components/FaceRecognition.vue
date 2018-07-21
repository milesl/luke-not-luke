<template>
  <div class="face-recognition">
    <h1 class="face-recognition__header">Luke-Not-Luke</h1>
    <p class="face-recognition__loading" v-if="loadingState">{{ loadingState }}</p>
    <input class="face-recognition__image-upload" type="file" @change="onImageChange($event)" />
    <p class="face-recognition__match">{{ bestMatch.className }}<span>{{ bestMatch.distance }}</span></p>
    <img class="face-recognition__image" :src="image" />
    <button class="face-recognition__remove-image" @click="removeImage()">Remove</button>
  </div>
</template>

<script>
import * as faceapi from 'face-api.js'

export default {
  name: 'FaceRecognition',
  props: {
  },
  data: () => {
    return {
      modelsUrl: '/models/face_recognition_model-weights_manifest.json',
      classes: ['luke', 'not-luke'],
      trainDescriptorsByClass: [ ],
      loadingState: null,
      image: null,
      bestMatch: {
        className: null
      }
    }
  },
  methods: {
    getFaceImageUri: function (className, idx) {
      return `img/classes/${className}/${className}${idx}.png`
    },
    fetchImage: async function (uri) {
      return (await fetch(uri)).blob()
    },
    initTrainDescriptorsByClass: async function (net, numImagesForTraining = 1) {
      const maxAvailableImagesPerClass = 5
      numImagesForTraining = Math.min(numImagesForTraining, maxAvailableImagesPerClass)
      return Promise.all(this.classes.map(
        async className => {
          const descriptors = []
          for (let i = 1; i < (numImagesForTraining + 1); i++) {
            const img = await faceapi.bufferToImage(
              await this.fetchImage(this.getFaceImageUri(className, i))
            )
            descriptors.push(await net.computeFaceDescriptor(img))
          }
          return {
            descriptors,
            className
          }
        }
      ))
    },
    getBestMatch: function getBestMatch (descriptorsByClass, queryDescriptor) {
      function computeMeanDistance (descriptorsOfClass) {
        return faceapi.round(
          descriptorsOfClass
            .map(d => faceapi.euclideanDistance(d, queryDescriptor))
            .reduce((d1, d2) => d1 + d2, 0) / (descriptorsOfClass.length || 1)
        )
      }
      return descriptorsByClass
        .map(
          ({ descriptors, className }) => ({
            distance: computeMeanDistance(descriptors),
            className
          })
        )
        .reduce((best, curr) => best.distance < curr.distance ? best : curr)
    },
    onImageChange: function (e) {
      var files = e.target.files || e.dataTransfer.files
      if (!files.length) {
        return
      }
      this.loadingState = 'Calculating match...'
      var reader = new FileReader()
      reader.onload = async (e) => {
        this.resizedataURL(e.target.result).then(async (result) => {
          this.image = result
          var img = document.createElement('img')
          img.src = this.image
          var descriptor = await faceapi.computeFaceDescriptor(img)
          this.bestMatch = this.getBestMatch(this.trainDescriptorsByClass, descriptor)
          this.loadingState = null
        })
      }
      reader.readAsDataURL(files[0])
    },
    resizedataURL: function (base64Img, callback) {
      return new Promise((resolve, reject) => {
        var img = document.createElement('img')
        img.onload = function () {
          var width = img.width / 5
          var height = img.height / 5
          var canvas = document.createElement('canvas')
          var ctx = canvas.getContext('2d')
          canvas.width = width
          canvas.height = height
          ctx.drawImage(this, 0, 0, width, height)
          var dataUrl = canvas.toDataURL()
          resolve(dataUrl)
        }
        img.src = base64Img
      })
    },
    removeImage: function () {
      this.image = null
    }
  },
  async mounted () {
    try {
      this.loadingState = 'Loading Models....'
      await faceapi.loadFaceRecognitionModel(this.modelsUrl)
      this.loadingState = 'Training....'
      this.trainDescriptorsByClass = await this.initTrainDescriptorsByClass(faceapi.recognitionNet)
      this.loadingState = 'Loading Complete'
    } catch (error) {
      this.loadingState = 'Error whilst loading....'
      console.error(error)
    }
  }
}
</script>

<style scoped lang="stylus">
  .face-recognition
    display block
    .face-recognition__header
      font-size 1em
    .face-recognition__loading
      font-size 0.8em
    .face-recognition__match
      font-size 1.2em
      font-weight bold
      span
        font-size 0.7em
        font-weight normal
    .face-recognition__image
      max-width 100%
      height auto

</style>
