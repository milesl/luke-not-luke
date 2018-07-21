<template>
  <section class="section">
    <div class="container">
      <h1 class="title" v-if="bestMatch" :class="getClass(bestMatch.distance)">{{ bestMatch.className }} <span class="is-size-6">{{ bestMatch.distance }}</span></h1>
      <h2 class="subtitle" v-if="loadingState">{{ loadingState }}</h2>
      <div class="columns">
        <div class="column is-half">
          <div class="file">
            <label class="file-label">
              <input class="file-input" type="file" @change="onImageChange($event)">
              <span class="file-cta">
                <span class="file-icon">
                  <i class="fas fa-upload"></i>
                </span>
                <span class="file-label">
                  Choose a image...
                </span>
              </span>
            </label>
            <button class="button" @click="removeImage()">Remove</button>
          </div>
        </div>
      </div>
      <div class="columns is-centered">
        <div class="column">
          <img :src="image" />
        </div>
      </div>
    </div>
  </section>
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
    computeMeanDistance: function (descriptorsOfClass, queryDescriptor) {
      return faceapi.round(
        descriptorsOfClass
          .map(d => faceapi.euclideanDistance(d, queryDescriptor))
          .reduce((d1, d2) => d1 + d2, 0) / (descriptorsOfClass.length || 1)
      )
    },
    getBestMatch: function getBestMatch (descriptorsByClass, queryDescriptor) {
      return descriptorsByClass
        .map(
          ({ descriptors, className }) => ({
            distance: this.computeMeanDistance(descriptors, queryDescriptor),
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
        var base64Img = e.target.result
        if (!base64Img) {
          return
        }
        this.resizedataURL(base64Img).then(async (result) => {
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
        img.onload = () => {
          var dimensions = this.calculateAspectRatioFit(img.width, img.height, 1000, 1000)
          var canvas = document.createElement('canvas')
          var ctx = canvas.getContext('2d')
          canvas.width = dimensions.width
          canvas.height = dimensions.height
          ctx.drawImage(img, 0, 0, dimensions.width, dimensions.height)
          var dataUrl = canvas.toDataURL()
          resolve(dataUrl)
        }
        img.src = base64Img
      })
    },
    removeImage: function () {
      this.image = null
      this.bestMatch = null
      this.loadingState = null
    },
    calculateAspectRatioFit: function (srcWidth, srcHeight, maxWidth, maxHeight) {
      var ratio = Math.min(maxWidth / srcWidth, maxHeight / srcHeight)
      return { width: srcWidth * ratio, height: srcHeight * ratio }
    },
    getClass: function (distance) {
      switch (true) {
        case distance <= 0.3:
          return 'has-text-danger'
        case distance <= 0.5:
          return 'has-text-warning'
        case distance <= 0.75:
          return 'has-text-info'
        case distance <= 1:
          return 'has-text-success'
      }
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

</style>
