<template>
  <div class="col-md-12">
    <input type="file" @change="$emit('load-image', $event)" accept="image/*" class="form-control mb-2">
  </div>
</template>
