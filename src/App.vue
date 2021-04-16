<template>
  <div id="app">
    <ImageGraffiti
      :width='800'
      :height='800'
    />
  </div>
</template>

<script>
import ImageGraffiti from '../packages/image-graffiti/ImageGraffiti'
export default {
  name: 'App',
  components: {
    ImageGraffiti
  }
}
</script>

<style>
</style>
