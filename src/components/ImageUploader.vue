<template>
  <div id="imageUploadContainer" class="items-center justify-center min-h-screen">
    <div class="main">
      <input type="file" @change="$emit('load-image', $event)" accept="image/*" class="form-control mb-2">
    </div>
  </div>
</template>

<style lang="css" scoped>
#imageUploadContainer {
  display: flex;
}
  #imageUploadContainer .main{
    border: 10px solid #000;
    background: #fff;
    padding: 20px;
    text-align: center;
    border-radius: 4px;

  }
</style>
