<template>
  <div class="piece" :class="props.color"></div>
</template>

<script setup>
import { ref, reactive } from "vue";

const props = defineProps(["color"]);

console.log(props.color);
</script>
<style scoped>
.piece {
  height: 20px;
  width: 20px;
  border-radius: 20px;
  /* background-color: black; */
}

.black {
  background-color: black;
}

.white {
  background-color: white;
}
</style>
