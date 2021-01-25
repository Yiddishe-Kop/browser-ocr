<template>
  <div id="app" class="bg-gray-500">
    <div>
      <button @click="lang = 'he'">Hebrew sample</button>
      <button @click="lang = 'en'">English sample</button>
    </div>

    <div class="relative">
      <img v-if="lang == 'he'" id="text-img" src="./assets/he.png" />
      <img v-else id="text-img" src="./assets/en.png" />
    </div>

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
