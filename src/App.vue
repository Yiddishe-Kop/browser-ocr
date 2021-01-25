<template>
  <div id="app" class="py-8">
    <h1 class="text-3xl font-bold text-center">OCR in the browser 🤯</h1>
    <div class="flex items-center justify-center mt-8 space-x-2">
      <button @click="lang = 'he'">Hebrew sample</button>
      <button @click="lang = 'en'">English sample</button>
    </div>

    <div class="relative mt-8">
      <img
        v-if="lang == 'he'"
        id="text-img"
        src="./assets/he.png"
        class="max-w-full"
      />
      <img v-else id="text-img" src="./assets/en.png" class="max-w-full" />
      <canvas id="canvas" class="absolute top-0 max-w-full"/>
    </div>

    <div class="mt-8 text-center">
      <button v-on:click="recognize">recognize</button>
    </div>

    <div v-if="status" class="flex flex-col items-center mt-8 space-y-2">
      <label for="ocr-progress">{{ status }}...</label>
      <div class="relative h-4 overflow-hidden bg-gray-200 rounded-full w-96">
        <div
          class="absolute left-0 h-full bg-green-500 transition-width"
          :style="`width: ${progress}%`"
        ></div>
      </div>
    </div>

    <div v-show="result" class="mt-8">
      <h1 class="text-2xl font-bold text-center">Result</h1>
      <canvas id="result-canvas" class="mt-2"/>
      <p
        class="max-w-md mx-auto mt-2"
        :dir="lang == 'en' ? 'ltr' : 'rtl'"
        contenteditable
      >
        {{ result }}
      </p>
    </div>
  </div>
</template>

<script>
/* eslint-disable */
import { createWorker, PSM, OEM } from "tesseract.js";
let worker;

export default {
  name: "app",
  data() {
    return {
      lang: "he",
      progress: 0,
      status: "",
      result: "",
    };
  },
  methods: {
    async recognize() {
      const img = document.getElementById("text-img");
      console.log(img);
      await worker.load();
      const lang = this.lang == "he" ? "heb" : "eng";
      await worker.loadLanguage(lang);
      await worker.initialize(lang, OEM.LSTM_ONLY);
      await worker.setParameters({
        tessedit_pageseg_mode: PSM.SINGLE_BLOCK,
      });
      const { data } = await worker.recognize(img);
      console.log(data);
      this.result = data.text;
      this.status = "";
      this.drawOverlay(data);
    },
    drawOverlay(result) {
      const img = document.getElementById("text-img");
      const canvas = document.getElementById("canvas");
      const resultCanvas = document.getElementById("result-canvas");

      canvas.width = resultCanvas.width = img.naturalWidth;
      canvas.height = resultCanvas.height = img.naturalHeight;

      // resultCanvas.setAttribute('dir', this.lang == 'he' ? 'rtl' : 'ltr');

      const ctx = canvas.getContext("2d");
      const resultCtx = resultCanvas.getContext("2d");

      result.words.forEach(function (w) {
        var b = w.bbox;

        ctx.strokeWidth = 2;

        ctx.strokeStyle = "red";
        ctx.strokeRect(b.x0, b.y0, b.x1 - b.x0, b.y1 - b.y0);
        ctx.beginPath();
        ctx.moveTo(w.baseline.x0, w.baseline.y0);
        ctx.lineTo(w.baseline.x1, w.baseline.y1);
        ctx.strokeStyle = "green";
        ctx.stroke();

        resultCtx.font = '20px Times';
        resultCtx.font = 20 * (b.x1 - b.x0) / resultCtx.measureText(w.text).width + "px Times";
        resultCtx.fillText(w.text, b.x0, w.baseline.y0);
      });
    },
  },
  created() {
    worker = createWorker({
      logger: (m) => {
        console.log(m);
        this.progress = m.progress * 100;
        this.status = m.status;
      },
    });
  },
};
</script>

<style>
.transition-width {
  transition: width 0.3s ease-in;
}
</style>
