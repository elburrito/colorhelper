
<template>
  <h1>Color Picker Tool</h1>
  <div class="col-md-12" v-if="!image">
    <input type="file" @change="loadImage" accept="image/*" class="form-control mb-2">
  </div>
  <div v-if="image" class="row">
  <div class="col-md-8">
    <div class="wall-dimensions">              
      <label class="control-label">Wall dimensions</label>  
      <div class="input-group">
        <span class="input-group-text">Width(m)</span>
        <input type="number" step="0.1" min="0" v-model="wallDimensions.width" class="form-control" placeholder="Wall Width (meters)" @input="updateWallDimensions">
        <span class="input-group-text">Height(m)</span>
        <input type="number" step="0.1" min="0" v-model="wallDimensions.height" class="form-control" placeholder="Wall Height (meters)" @input="updateWallDimensions">
      </div>
    </div>

    <div v-if="currentSelection" class="mb-3">
        <div class="d-flex align-items-center">
            <div class="color-box me-2" :style="{ backgroundColor: currentSelection.color.hex }"></div>
            <span>{{ currentSelection.color.hex }}</span>
            <span class="ms-2">({{ currentSelection.percentage }}%)</span>
            <span class="ms-2">({{ currentSelection.area }} m²)</span>
        </div>
        <label for="thresholdSlider" class="form-label mt-2">Adjust Threshold:</label>
        <input type="range" class="form-range" v-model="currentSelection.threshold" min="0" max="255" @input="updateSelection">
        <button class="btn btn-success mt-2 me-2" @click="addToList">Add to List</button>
        <button class="btn btn-danger mt-2" @click="cancelSelection">Cancel</button>
    </div>

    <div class="canvas-wrapper">
        <canvas id="mainCanvas" ref="mainCanvas" @mousemove="updateCursor" @mouseleave="hideCursor" @click="selectColor"></canvas>
        <canvas id="selectionCanvas" ref="selectionCanvas"></canvas>
        <div v-show="cursor.visible" class="cursor-marker" :style="{ left: cursor.x + 'px', top: cursor.y + 'px', backgroundColor: cursor.color.hex }"></div>
    </div>
  </div>

  <div class="col-md-4">
    <h2>Selections</h2>
    <div class="list-group">
        <div v-for="(selection, index) in selections" :key="index" class="selection-item list-group-item list-group-item-action"
              @mouseenter="highlightSelection(selection, true)"
              @mouseleave="highlightSelection(selection, false)">
            <div class="d-flex align-items-center" style="gap: 10px">
                <div class="color-box" :style="{ backgroundColor: selection.color.hex}"></div>
                <div style="flex-grow: 1;">
                  <input v-model="selection.name" class="form-control form-control-sm " :placeholder="(selection.color.hex)" size="10" style="max-width: 100px;">
                </div>
                <span class="text-nowrap"> 
                  {{ selection.area }} m² <br />
                  <small class="text-muted">{{ selection.percentage }}%</small>
                </span>
                <button class="btn btn-sm btn-secondary" @click="removeSelection(index)">x</button>
            </div>
        </div>
        <div class="list-group-item list-group-item-dark">
            <strong>Total Selected: {{ totalSelectedPercentage }}%</strong>
        </div>
    </div>
  </div>
</div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
        image: null,
        mainCanvas: null,
        selectionCanvas: null,
        mainCtx: null,
        selectionCtx: null,
        originalImageData: null,
        selections: [],
        currentSelection: null,
        cursor: {
            visible: false,
            x: 0,
            y: 0,
            color: { r: 255, g: 255, b: 255, hex: '#FFFFFF' }
        },
        wallDimensions: {
            width: 0,
            height: 0
        }
    };
  },
  computed: {
      totalSelectedPercentage() {
          return this.selections.reduce((sum, sel) => sum + parseFloat(sel.percentage), 0).toFixed(2);
      },
      totalWallArea() {
          return (this.wallDimensions.width * this.wallDimensions.height).toFixed(2);
      }
  },
  methods: {
      loadImage(event) {
          const file = event.target.files[0];
          if (file) {
              const reader = new FileReader();
              reader.onload = (e) => {
                  this.image = new Image();
                  this.image.onload = this.renderImage;
                  this.image.src = e.target.result;
              };
              reader.readAsDataURL(file);
          }
      },
      renderImage() {
          this.mainCanvas = this.$refs.mainCanvas;
          this.selectionCanvas = this.$refs.selectionCanvas;

          this.mainCtx = this.mainCanvas.getContext('2d');
          this.selectionCtx = this.selectionCanvas.getContext('2d');

          this.mainCanvas.width = this.image.width;
          this.mainCanvas.height = this.image.height;

          this.selectionCanvas.width = this.image.width;
          this.selectionCanvas.height = this.image.height;

          this.mainCtx.drawImage(this.image, 0, 0);
          this.originalImageData = this.mainCtx.getImageData(0, 0, this.mainCanvas.width, this.mainCanvas.height);

          this.wallDimensions.width = this.image.width / 100; // Assume 100 pixels = 1 meter
          this.wallDimensions.height = this.image.height / 100;

          this.resizeCanvas();
          window.addEventListener('resize', this.resizeCanvas);
      },
      resizeCanvas() {
          const rect = this.mainCanvas.getBoundingClientRect();
          this.selectionCanvas.style.width = `${rect.width}px`;
          this.selectionCanvas.style.height = `${rect.height}px`;
      },
      updateCursor(event) {
          const rect = this.mainCanvas.getBoundingClientRect();
          const x = event.clientX - rect.left;
          const y = event.clientY - rect.top;

          this.cursor.visible = true;
          this.cursor.x = x;
          this.cursor.y = y;

          const scaledX = Math.floor((x / rect.width) * this.mainCanvas.width);
          const scaledY = Math.floor((y / rect.height) * this.mainCanvas.height);

          const imageData = this.mainCtx.getImageData(scaledX, scaledY, 1, 1).data;
          this.cursor.color = {
              r: imageData[0],
              g: imageData[1],
              b: imageData[2],
              hex: `#${((1 << 24) + (imageData[0] << 16) + (imageData[1] << 8) + imageData[2]).toString(16).slice(1).toUpperCase()}`
          };
      },
      hideCursor() {
          this.cursor.visible = false;
      },
      selectColor(event) {
          const rect = this.mainCanvas.getBoundingClientRect();
          const x = Math.floor(((event.clientX - rect.left) / rect.width) * this.mainCanvas.width);
          const y = Math.floor(((event.clientY - rect.top) / rect.height) * this.mainCanvas.height);

          const imageData = this.mainCtx.getImageData(x, y, 1, 1).data;
          const color = {
              r: imageData[0],
              g: imageData[1],
              b: imageData[2],
              hex: `#${((1 << 24) + (imageData[0] << 16) + (imageData[1] << 8) + imageData[2]).toString(16).slice(1).toUpperCase()}`
          };

          const threshold = 50;
          const { percentage, matchingPixels } = this.highlightSimilarColors(color, threshold, true);
          const area = ((percentage / 100) * this.totalWallArea).toFixed(2);

          this.currentSelection = { color, threshold, percentage, matchingPixels, area };
      },
      highlightSimilarColors(targetColor, threshold, draw) {
          const imageData = this.originalImageData;
          const data = imageData.data;
          const overlayData = this.selectionCtx.createImageData(this.mainCanvas.width, this.mainCanvas.height);
          const overlayPixels = overlayData.data;

          let matchingPixels = 0;
          const totalPixels = this.mainCanvas.width * this.mainCanvas.height;

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
                      overlayPixels[i + 3] = 128; // Semi-transparent
                  }
              }
          }

          if (draw) {
              this.selectionCtx.putImageData(overlayData, 0, 0);
          }

          const percentage = ((matchingPixels / totalPixels) * 100).toFixed(2);
          return { matchingPixels, percentage };
      },
      updateSelection() {
          if (this.currentSelection) {
              const { color, threshold } = this.currentSelection;
              const { percentage, matchingPixels } = this.highlightSimilarColors(color, threshold, true);
              this.currentSelection.percentage = percentage;
              this.currentSelection.area = ((percentage / 100) * this.totalWallArea).toFixed(2);
          }
      },
      addToList() {
          if (this.currentSelection) {
              this.selections.push(this.currentSelection);
              this.currentSelection = null;
              this.selectionCtx.clearRect(0, 0, this.selectionCanvas.width, this.selectionCanvas.height);
          }
      },
      cancelSelection() {
          this.selectionCtx.clearRect(0, 0, this.selectionCanvas.width, this.selectionCanvas.height);
          this.currentSelection = null;
      },
      removeSelection(index) {
          this.selections.splice(index, 1);
      },
      highlightSelection(selection, hover) {
          if (hover) {
              this.highlightSimilarColors(selection.color, selection.threshold, true);
          } else {
              this.selectionCtx.clearRect(0, 0, this.selectionCanvas.width, this.selectionCanvas.height);
          }
      },
      updateWallDimensions() {
          this.selections.forEach(selection => {
              selection.area = ((selection.percentage / 100) * this.totalWallArea).toFixed(2);
          });
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
.color-box {
  width: 20px;
  height: 20px;
  border-radius: 5px;
}
</style>
