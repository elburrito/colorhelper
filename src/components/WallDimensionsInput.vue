<template>
  <div class="wall-dimensions">              
    <label class="control-label">Wall dimensions</label>  
    <div class="input-group">
      <span class="input-group-text">Width(m)</span>
      <input type="number" step="0.1" min="0" v-model="dimensions.width" class="form-control" placeholder="Wall Width (meters)" @input="update">
      <span class="input-group-text">Height(m)</span>
      <input type="number" step="0.1" min="0" v-model="dimensions.height" class="form-control" placeholder="Wall Height (meters)" @input="update">
    </div>
  </div>
</template>
<script>
export default {
  props: ['dimensions'],
  methods: {
    update() {
      this.$emit('update-dimensions');
    }
  }
}
</script>