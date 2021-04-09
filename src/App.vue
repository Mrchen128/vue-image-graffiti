<template>
  <div id="app">
    <ImageEdit
      width='800'
      height='800'
    />
  </div>
</template>

<script>
import ImageEdit from '@/components/ImageEdit'
export default {
  name: 'App',
  components: {
    ImageEdit
  }
}
</script>

<style>
  *{
    padding: 0;
    margin: 0;
    box-sizing: border-box;
  }
  canvas{
    display: block;
  }
</style>
