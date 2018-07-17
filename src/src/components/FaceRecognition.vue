<template>
    <h1>Face-api.js</h1>
</template>

<script>
import * as faceapi from 'face-api.js'

export default {
  name: 'FaceRecognition',
  props: {
  },
  data: () => {
    return {
      modelsUrl: '/models',
      classes: ['luke', 'not-luke'],
      trainDescriptorsByClass: [ ]
    }
  },
  method: {
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
    }
  },
  async mounted () {
    faceapi.loadModels(this.modelsUrl)
    this.trainDescriptorsByClass = await this.initTrainDescriptorsByClass(faceapi.recognitionNet)
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="stylus">

</style>
