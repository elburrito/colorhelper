
<template>
  <div class="container-fluid h-100">
    <div class="row h-100">
      <div class="col-6 bg-secondary-subtle align-self-center h-100">
        <a class="btn btn-primary">Choose file</a>
        <!-- <img src="/example_images/2022_05_maiezyt-8-scaled.jpg" style="max-width: 100%;"> -->
        <canvas id="canvas">

        </canvas>
      </div>
      <div class="col-6 bg-light-subtle">
        <h1>Palette</h1>
        <p class="lead">Click somewhere in your image to pick a color.</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {

    }
  },
  mounted() {
    var canvas = document.getElementById('canvas'),
        context = canvas.getContext('2d');

    make_base();

    function make_base()
    {
      let base_image = new Image();
      base_image.src = '/example_images/propart_2017_composition_03-03.jpg';
      base_image.onload = function(){
        context.drawImage(base_image,0,0, 100, 100);
      }
      console.log('called', canvas, base_image );
    }
  },
  computed: {
  }
}

</script>

<style scoped>
#canvas {
  max-width: 100%;
  height: 100%;
  background: orange;
}
</style>
