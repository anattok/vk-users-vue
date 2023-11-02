<template>
  <button class="btn">
    {{ label }}
  </button>
</template>

<script setup>
import { defineProps } from "vue";
const props = defineProps({
  label: {
    type: String,
    default: "Button",
  },
});
</script>

<style scoped>
.btn {
  cursor: pointer;
  padding: 4px 12px;
  height: 40px;
}
</style>
