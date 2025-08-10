<template>
  <div>
    <h2>Selections</h2>
    <div class="list-group">
      <div v-for="(selection, index) in selections" :key="index" class="selection-item list-group-item list-group-item-action"
        @mouseenter="$emit('highlight-selection', selection, true)"
        @mouseleave="$emit('highlight-selection', selection, false)">
        <div class="d-flex align-items-center" style="gap: 10px">
          <div class="color-box" :style="{ backgroundColor: selection.color.hex}"></div>
          <div style="flex-grow: 1;">
            <input v-model="selection.name" class="form-control form-control-sm " :placeholder="(selection.color.hex)" size="10" style="max-width: 100px;">
          </div>
          <span class="text-nowrap"> 
            {{ selection.area }} m² <br />
            <small class="text-muted">{{ selection.percentage }}%</small>
          </span>
          <button class="btn btn-sm btn-secondary" @click="$emit('remove-selection', index)">x</button>
        </div>
      </div>
      <div class="list-group-item list-group-item-dark">
        <strong>Total Selected: {{ totalSelectedPercentage }}%</strong>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  props: ['selections', 'totalSelectedPercentage']
}
</script>