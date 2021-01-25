<template>
  <div id="app">
    <img id="text-img" src="./assets/testocr.png" />
    <button v-on:click="recognize">recognize</button>

    <div v-if="status">
      <label for="ocr-progress">{{ status }}...</label>
      <progress id="ocr-progress" :value="progress" max="100">
        {{ progress }}%
      </progress>
    </div>

    <p v-if="result">{{ result }}</p>
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
      await worker.loadLanguage("eng");
      await worker.initialize("eng", OEM.LSTM_ONLY);
      await worker.setParameters({
        tessedit_pageseg_mode: PSM.SINGLE_BLOCK,
      });
      const {
        data: { text },
      } = await worker.recognize(img);
      this.result = text;
      this.status = "";
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
