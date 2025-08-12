<template>
  <div class="card">
    <h1>📏 Screen Size Detector</h1>

    <!-- Default estimation -->
    <h2>Quick Estimate (Not Exact)</h2>
    <p>
      This is an <b>approximation</b> based on common device PPI.
      For accurate results, please calibrate with a credit card below.
    </p>
    <div v-html="estimateResults"></div>

    <!-- Calibration section -->
    <div class="section">
      <h2>Calibrate with a Credit Card</h2>
      <p>
        Place a credit card on your screen and adjust the slider
        until the blue box matches its size exactly.
      </p>
      <input
        type="range"
        v-model="scalePercent"
        min="50"
        max="200"
        @input="updateScale"
      />
      <p id="scaleValue">Scale: {{ scalePercent }}%</p>
      <button @click="calculatePrecise">✅ Get Accurate Size</button>

      <div id="preciseResults" style="margin-top:20px;" v-html="preciseResults"></div>
      <div
        id="creditCard"
        :style="{ transform: `scale(${scalePercent / 100})` }"
        ref="creditCard"
      >
        <div class="cornerMarker">
          <div class="cornerDot"></div>
          <span class="cornerText">Put your card here</span>
        </div>
      </div>
    </div>
  </div>
  <footer style="margin-top:40px; color:#666; font-size:15px;">
    &copy; 2025 Screen Size Detector. Released 2025-08. Created with help of ChatPGT, Copilot.
  </footer>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const scalePercent = ref(100);
const estimateResults = ref('');
const preciseResults = ref('');
const creditCard = ref(null);

const CREDIT_CARD_WIDTH_MM = 85.6;

function quickEstimate() {
  const physicalPixelWidth = screen.width * window.devicePixelRatio;
  const physicalPixelHeight = screen.height * window.devicePixelRatio;

  let estimatedPPI;
  if (physicalPixelWidth > 3000) {
    estimatedPPI = 300; // phone/tablet
  } else if (physicalPixelWidth > 2000) {
    estimatedPPI = 110; // laptop/HiDPI
  } else {
    estimatedPPI = 96; // desktop
  }

  const widthInInches = physicalPixelWidth / estimatedPPI;
  const heightInInches = physicalPixelHeight / estimatedPPI;
  const diagonalInInches = Math.sqrt(widthInInches ** 2 + heightInInches ** 2);

  const widthInCm = widthInInches * 2.54;
  const heightInCm = heightInInches * 2.54;
  const diagonalInCm = diagonalInInches * 2.54;

  estimateResults.value = `
    <p><b>Diagonal:</b> ~${diagonalInInches.toFixed(2)}" (${diagonalInCm.toFixed(1)} cm)</p>
    <p><b>Width:</b> ~${widthInInches.toFixed(2)}" (${widthInCm.toFixed(1)} cm) x <b>Height:</b> ~${heightInInches.toFixed(2)}" (${heightInCm.toFixed(1)} cm)</p>
    <p><b>Resolution:</b> ${screen.width} × ${screen.height} (CSS px), DPR: ${window.devicePixelRatio}</p>
    <p style="color:red;font-size:14px;">⚠ This is just an estimate. Use calibration below for accuracy.</p>
  `;
}

function updateScale() {
  // The scale is handled by binding in Vue, so nothing else needed here
}

function calculatePrecise() {
  const creditCardWidthInches = CREDIT_CARD_WIDTH_MM / 25.4;
  const measuredWidthPx = creditCard.value.offsetWidth;
  const scale = scalePercent.value / 100;
  const scaledWidthPx = measuredWidthPx * scale;

  const dpr = window.devicePixelRatio;
  const ppi = (scaledWidthPx * dpr) / creditCardWidthInches;

  const physicalPixelWidth = screen.width * dpr;
  const physicalPixelHeight = screen.height * dpr;

  const widthInInches = physicalPixelWidth / ppi;
  const heightInInches = physicalPixelHeight / ppi;
  const diagonalInInches = Math.sqrt(widthInInches ** 2 + heightInInches ** 2);

  const widthInCm = widthInInches * 2.54;
  const heightInCm = heightInInches * 2.54;
  const diagonalInCm = diagonalInInches * 2.54;

  preciseResults.value = `
    <p><b>Diagonal:</b> ${diagonalInInches.toFixed(2)}" (${diagonalInCm.toFixed(1)} cm)</p>
    <p><b>Width:</b> ${widthInInches.toFixed(2)}" (${widthInCm.toFixed(1)} cm) x <b>Height:</b> ${heightInInches.toFixed(2)}" (${heightInCm.toFixed(1)} cm)</p>
    <p><b>Resolution:</b> ${screen.width} × ${screen.height} (CSS px), DPR: ${dpr}</p>
    <p><b>Calculated PPI:</b> ${ppi.toFixed(1)}</p>
  `;
}

onMounted(() => {
  quickEstimate();
});
</script>

<style scoped>
body {
  font-family: Arial, sans-serif;
  background: #f5f5f5;
  padding: 20px;
  text-align: center;
}
.card {
  background: white;
  padding: 20px;
  border-radius: 12px;
  max-width: 650px;
  margin: auto;
  box-shadow: 0 2px 8px rgba(0,0,0,0.15);
}
h1 {
  margin-bottom: 10px;
}
p {
  font-size: 18px;
  margin: 8px 0;
}
#creditCard {
  background: lightblue;
  width: 90vw;
  max-width: 320px;
  min-width: 120px;
  aspect-ratio: 85.6 / 54;
  margin: 20px auto;
  border: 2px dashed #333;
  display: flex;
  align-items: flex-start;
  justify-content: flex-start;
  position: relative;
  transform-origin: top left;
  transform: scale(1);
}
input[type=range] {
  width: 95vw;
  max-width: 350px;
}
.section {
  border-top: 2px dashed #ddd;
  margin-top: 25px;
  padding-top: 20px;
}
.cornerMarker {
  position: absolute;
  top: 0;
  left: 0;
  display: flex;
  align-items: center;
  padding: 4px;
}
.cornerDot {
  width: 12px;
  height: 12px;
  background-color: red;
  border-radius: 50%;
  margin-right: 6px;
}
.cornerText {
  font-size: 14px;
  font-weight: bold;
  color: #333;
}
button {
  font-size: 18px;
  padding: 12px 24px;
  background-color: #0078D7;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.2s ease;
  margin-top: 12px;
  width: 95vw;
  max-width: 350px;
}
button:hover {
  background-color: #005fa3;
  transform: scale(1.05);
}
@media (min-width: 700px) {
  .card {
    max-width: 650px;
    padding: 20px;
  }
  #creditCard {
    width: 85.6mm;
    max-width: 400px;
    aspect-ratio: unset;
    height: 54mm;
  }
  input[type=range], button {
    width: 100%;
    max-width: 400px;
  }
  .cornerText {
    font-size: 16px;
  }
}
</style>