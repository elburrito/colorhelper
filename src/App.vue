<template>
  <div class="container min-vh-100">
    <!-- STEP 1: Upload Image-->
      <div v-if="!image" class="row min-vh-100">
        <div class="col-md-4">
          <img src="/src/assets/Logo@2x.png" alt="El Burritos Mural Helper Logo" width="350px" style="max-width: 100%;" class="mt-2"/>
          <hr></hr>
          <p>What you can do here:</p>
          <ul>
            <li>Drop your mural sketch and calculcate the squaremeters per color</li>
            <li class="crossed">More</li>
          </ul>
        </div>

        <div class="col-md-8 flex-fill d-flex">
            <ImageUploader v-if="!image" @load-image="loadImage" style="align-self: anchor-center;"/>
        </div>
      </div>

    <!-- STEP 2: Pick Colors-->
      <div v-else class="row min-vh-100">
        <div class="col-md-12">
          <!-- main content -->
          <div class="row">
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
        </div>    
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
body {
  margin: 0;
  padding: 0;
  overflow: hidden;
  background: #FFC904;
  background-image: url('assets/squiggles_and_doodles_background.png');
  background-blend-mode: soft-light;
  background-size: 500px;
}

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