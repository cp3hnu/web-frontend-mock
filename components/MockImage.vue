<template>
  <div class="container">
    <img src="/1.png" width="800" class="mock-image" />
    <div>
      <div class="red-frame"></div>
      <div v-click>
        <div
          v-for="i in 8"
          class="red-frame"
          :style="{
            top: 70 + Math.floor(i / 3) * 30 + 'px',
            left: computeLeft(i) + 'px',
            width: computeWidth(i) + 'px'
          }"
        ></div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";

const isShowAll = ref(false);

function computeLeft(i) {
  return i % 3 === 0 ? 192 : i % 3 === 1 ? 386 : 588;
}
function computeWidth(i) {
  return i % 3 === 0 ? 192 : i % 3 === 1 ? 200 : 202;
}
</script>

<style scoped>
.container {
  position: relative;
}
.mock-image {
  position: absolute;
  top: 0;
  left: 0;
}
.red-frame {
  background-color: transparent;
  border: 2px solid red;
  position: absolute;
  top: 70px;
  left: 192px;
  width: 192px;
  height: 30px;
  z-index: 10;
}
</style>
