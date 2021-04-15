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
  *{
    padding: 0;
    margin: 0;
    box-sizing: border-box;
    list-style: none;
  }
  canvas{
    display: block;
  }
</style>
