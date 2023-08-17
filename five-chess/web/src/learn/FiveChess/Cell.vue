<template>
  <div class="cell">
    <slot></slot>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from "vue";
</script>
<style scoped>
.cell {
  height: 40px;
  width: 40px;
  border: 1px #000 solid;
  background-color: antiquewhite;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
