<template>
  <div v-if="currentSelection" class="mb-3">
    <div class="d-flex align-items-center">
      <div class="color-box me-2" :style="{ backgroundColor: currentSelection.color.hex }"></div>
      <span>{{ currentSelection.color.hex }}</span>
      <span class="ms-2">({{ currentSelection.percentage }}%)</span>
      <span class="ms-2">({{ currentSelection.area }} m²)</span>
    </div>
    <label for="thresholdSlider" class="form-label mt-2">Adjust Threshold:</label>
    <input type="range" class="form-range" v-model="currentSelection.threshold" min="0" max="255" @input="$emit('update-selection')">
    <button class="btn btn-success mt-2 me-2" @click="$emit('add-to-list')">Add to List</button>
    <button class="btn btn-danger mt-2" @click="$emit('cancel-selection')">Cancel</button>
  </div>
</template>
<script>
export default {
  props: ['currentSelection']
}
</script>