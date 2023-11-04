<template>
  <div class="search">
    <label for="search">{{ label }}</label>
    <input
      id="search"
      :placeholder="placeholder"
      :value="modelValue"
      @input="updateInputValue"
    />
  </div>
</template>

<script setup>
import { defineEmits, defineProps } from "vue";

const { modelValue, placeholder } = defineProps([
  "modelValue",
  "placeholder",
  "label",
]);

const emit = defineEmits(["update:modelValue"]);

// Обработчик для обновления значения инпута
const updateInputValue = (event) => {
  const newValue = event.target.value;
  emit("update:modelValue", newValue);
};
</script>

<style scoped>
.search input {
  width: 100%;
  height: 40px;
  padding: 5px 15px;
}
.search label {
  display: block;
  font-size: 14px;
  margin-bottom: 10px;
  opacity: 0.3;
}
.search input {
  width: 100%;
}
</style>
