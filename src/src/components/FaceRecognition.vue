<template>
  <div>
    <h1>Face-api.js</h1>
    <p v-if="loadingState">{{ loadingState }}</p>
    <input type="file" @change="onImageChange($event)" />
    <p>{{ bestMatch.className }}</p>
    <img :src="image" />
    <button @click="removeImage()">Remove</button>
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
      bestMatch: null
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
        this.image = e.target.result
        var img = document.createElement('img')
        img.src = this.image
        var descriptor = await faceapi.computeFaceDescriptor(img)
        this.bestMatch = this.getBestMatch(this.trainDescriptorsByClass, descriptor)
        this.loadingState = null
      }
      reader.readAsDataURL(files[0])
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

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="stylus">

</style>
