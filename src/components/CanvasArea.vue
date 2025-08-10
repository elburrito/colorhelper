<template>
  <div class="canvas-wrapper">
    <canvas ref="mainCanvas"
            @mousemove="handleMouseMove"
            @mouseleave="handleMouseLeave"
            @click="handleClick"></canvas>
    <canvas ref="selectionCanvas" id="selectionCanvas"></canvas>
    <div v-show="cursor.visible"
         class="cursor-marker"
         :style="{ left: cursor.x + 'px', top: cursor.y + 'px', backgroundColor: cursor.color.hex }"></div>
  </div>
</template>

<script>
export default {
  props: ['image', 'cursor', 'currentSelection'],
  emits: ['update-cursor', 'hide-cursor', 'select-color', 'update-selection', 'canvas-ready', 'highlight-selection'],
  data() {
    return {
      mainCtx: null,
      selectionCtx: null,
      originalImageData: null,
      wallDimensions: { width: 0, height: 0 }
    }
  },
  mounted() {
    this.$nextTick(() => {
      this.initCanvas();
      window.addEventListener('resize', this.resizeCanvas);
    });
  },
  watch: {
    image(newImg) {
      if (newImg) {
        this.initCanvas();
      }
    }
  },
  methods: {
    initCanvas() {
      const mainCanvas = this.$refs.mainCanvas;
      const selectionCanvas = this.$refs.selectionCanvas;
      if (!mainCanvas || !selectionCanvas || !this.image) return;

      mainCanvas.width = this.image.width;
      mainCanvas.height = this.image.height;
      selectionCanvas.width = this.image.width;
      selectionCanvas.height = this.image.height;

      this.mainCtx = mainCanvas.getContext('2d');
      this.selectionCtx = selectionCanvas.getContext('2d');
      this.mainCtx.drawImage(this.image, 0, 0);
      this.originalImageData = this.mainCtx.getImageData(0, 0, mainCanvas.width, mainCanvas.height);

      this.wallDimensions.width = this.image.width / 100;
      this.wallDimensions.height = this.image.height / 100;

      this.resizeCanvas();
      this.$emit('canvas-ready', {
        mainCtx: this.mainCtx,
        selectionCtx: this.selectionCtx,
        originalImageData: this.originalImageData,
        wallDimensions: this.wallDimensions
      });
    },
    resizeCanvas() {
      const rect = this.$refs.mainCanvas.getBoundingClientRect();
      this.$refs.selectionCanvas.style.width = `${rect.width}px`;
      this.$refs.selectionCanvas.style.height = `${rect.height}px`;
    },
    handleMouseMove(event) {
      const rect = this.$refs.mainCanvas.getBoundingClientRect();
      const x = event.clientX - rect.left;
      const y = event.clientY - rect.top;
      const scaledX = Math.floor((x / rect.width) * this.$refs.mainCanvas.width);
      const scaledY = Math.floor((y / rect.height) * this.$refs.mainCanvas.height);

      const imageData = this.mainCtx.getImageData(scaledX, scaledY, 1, 1).data;
      const color = {
        r: imageData[0],
        g: imageData[1],
        b: imageData[2],
        hex: `#${((1 << 24) + (imageData[0] << 16) + (imageData[1] << 8) + imageData[2]).toString(16).slice(1).toUpperCase()}`
      };
      this.$emit('update-cursor', { x, y, color });
    },
    handleMouseLeave() {
      this.$emit('hide-cursor');
    },
    handleClick(event) {
      const rect = this.$refs.mainCanvas.getBoundingClientRect();
      const x = Math.floor(((event.clientX - rect.left) / rect.width) * this.$refs.mainCanvas.width);
      const y = Math.floor(((event.clientY - rect.top) / rect.height) * this.$refs.mainCanvas.height);

      const imageData = this.mainCtx.getImageData(x, y, 1, 1).data;
      const color = {
        r: imageData[0],
        g: imageData[1],
        b: imageData[2],
        hex: `#${((1 << 24) + (imageData[0] << 16) + (imageData[1] << 8) + imageData[2]).toString(16).slice(1).toUpperCase()}`
      };
      this.$emit('select-color', { color, mainCtx: this.mainCtx, selectionCtx: this.selectionCtx, originalImageData: this.originalImageData, wallDimensions: this.wallDimensions });
    },
    highlightSimilarColors(targetColor, threshold, draw) {
      const imageData = this.originalImageData;
      const data = imageData.data;
      const overlayData = this.selectionCtx.createImageData(this.$refs.mainCanvas.width, this.$refs.mainCanvas.height);
      const overlayPixels = overlayData.data;

      let matchingPixels = 0;
      const totalPixels = this.$refs.mainCanvas.width * this.$refs.mainCanvas.height;

      for (let i = 0; i < data.length; i += 4) {
        const color = {
          r: data[i],
          g: data[i + 1],
          b: data[i + 2]
        };

        const diffR = Math.abs(color.r - targetColor.r);
        const diffG = Math.abs(color.g - targetColor.g);
        const diffB = Math.abs(color.b - targetColor.b);

        if (diffR <= threshold && diffG <= threshold && diffB <= threshold) {
          matchingPixels++;
          if (draw) {
            overlayPixels[i] = 255;
            overlayPixels[i + 1] = 0;
            overlayPixels[i + 2] = 255;
            overlayPixels[i + 3] = 128;
          }
        }
      }

      if (draw) {
        this.selectionCtx.putImageData(overlayData, 0, 0);
      }

      const percentage = ((matchingPixels / totalPixels) * 100).toFixed(2);
      return { matchingPixels, percentage };
    }
  }
}
</script>

<style scoped>
.canvas-wrapper {
  position: relative;
  max-width: 100%;
}
canvas {
  display: block;
  width: 100%;
  cursor: crosshair;
}
#selectionCanvas {
  position: absolute;
  top: 0;
  left: 0;
  pointer-events: none;
}
.cursor-marker {
  position: absolute;
  width: 20px;
  height: 20px;
  border: 2px solid rgba(255, 0, 0, 0.8);
  border-radius: 50%;
  pointer-events: none;
  transform: translate(-50%, -50%);
  background-color: rgba(255, 255, 255, 0.5);
}
</style>