<template>
  <h1>Color Helpers</h1>
  <ImageUploader v-if="!image" @load-image="loadImage" />
  <div v-if="image" class="row">
    <div class="col-md-8">
      <WallDimensionsInput :dimensions="wallDimensions" @update-dimensions="updateWallDimensions" />
      <SelectionDetails :currentSelection="currentSelection" @update-selection="updateSelection" @add-to-list="addToList" @cancel-selection="cancelSelection" />
      <CanvasArea
        ref="canvasArea"
        :image="image"
        :cursor="cursor"
        :currentSelection="currentSelection"
        @update-cursor="onUpdateCursor"
        @hide-cursor="onHideCursor"
        @select-color="onSelectColor"
        @canvas-ready="onCanvasReady"
      />
    </div>
    <div class="col-md-4">
      <SelectionsList :selections="selections" :totalSelectedPercentage="totalSelectedPercentage" @highlight-selection="highlightSelection" @remove-selection="removeSelection" />
    <Downloader :selections="selections" />
    </div>
  </div>
</template>

<script>
import ImageUploader from './components/ImageUploader.vue'
import WallDimensionsInput from './components/WallDimensionsInput.vue'
import CanvasArea from './components/CanvasArea.vue'
import SelectionDetails from './components/SelectionDetails.vue'
import SelectionsList from './components/SelectionsList.vue'
import Downloader from './components/Downloader.vue'

export default {
  name: 'App',
  components: {
    ImageUploader,
    WallDimensionsInput,
    CanvasArea,
    SelectionDetails,
    SelectionsList,
    Downloader
  },
  data() {
    return {
      image: null,
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
      },
      mainCtx: null,
      selectionCtx: null,
      originalImageData: null
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
          this.image = new window.Image();
          this.image.onload = () => {
            // CanvasArea will handle rendering
          };
          this.image.src = e.target.result;
        };
        reader.readAsDataURL(file);
      }
    },
    onCanvasReady({ mainCtx, selectionCtx, originalImageData, wallDimensions }) {
      this.mainCtx = mainCtx;
      this.selectionCtx = selectionCtx;
      this.originalImageData = originalImageData;
      this.wallDimensions = wallDimensions;
    },
    onUpdateCursor({ x, y, color }) {
      this.cursor.visible = true;
      this.cursor.x = x;
      this.cursor.y = y;
      this.cursor.color = color;
    },
    onHideCursor() {
      this.cursor.visible = false;
    },
    onSelectColor({ color }) {
      const threshold = 50;
      const { percentage, matchingPixels } = this.$refs.canvasArea.highlightSimilarColors(color, threshold, true);
      const area = ((percentage / 100) * this.totalWallArea).toFixed(2);
      this.currentSelection = { color, threshold, percentage, matchingPixels, area };
    },
    updateSelection() {
      if (this.currentSelection) {
        const { color, threshold } = this.currentSelection;
        const { percentage, matchingPixels } = this.$refs.canvasArea.highlightSimilarColors(color, threshold, true);
        this.currentSelection.percentage = percentage;
        this.currentSelection.area = ((percentage / 100) * this.totalWallArea).toFixed(2);
      }
    },
    addToList() {
      if (this.currentSelection) {
        this.selections.push(this.currentSelection);
        this.currentSelection = null;
        this.$refs.canvasArea.selectionCtx.clearRect(
          0, 0,
          this.$refs.canvasArea.$refs.selectionCanvas.width,
          this.$refs.canvasArea.$refs.selectionCanvas.height
        );
      }
    },
    cancelSelection() {
      this.$refs.canvasArea.selectionCtx.clearRect(
        0, 0,
        this.$refs.canvasArea.$refs.selectionCanvas.width,
        this.$refs.canvasArea.$refs.selectionCanvas.height
      );
      this.currentSelection = null;
    },
    removeSelection(index) {
      this.selections.splice(index, 1);
    },
    highlightSelection(selection, hover) {
      if (hover) {
        this.$refs.canvasArea.highlightSimilarColors(selection.color, selection.threshold, true);
      } else {
        this.$refs.canvasArea.selectionCtx.clearRect(
          0, 0,
          this.$refs.canvasArea.$refs.selectionCanvas.width,
          this.$refs.canvasArea.$refs.selectionCanvas.height
        );
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

<style>
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