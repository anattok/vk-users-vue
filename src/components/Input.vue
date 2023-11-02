<template>
  <div class="search">
    <label for="search"> Пиши имя друга или фамилию или id </label>
    <input
      autocomplete="on"
      id="search"
      v-model="internalValue"
      :placeholder="placeholder"
      @input="updateValue"
    />
  </div>
</template>

<script setup>
import { defineEmits, defineProps } from "vue";
const emit = defineEmits(["update:value"]);
const props = defineProps({
  value: String,
  placeholder: String,
});

let internalValue = props.value;

const updateValue = (e) => {
  internalValue = e.target.value;
  emit("update:value", e.target.value);
};
</script>

<style scoped>
.search input {
  width: 300px;
  height: 45px;
  padding: 5px 15px;
}
.search label {
  display: block;
}
</style>
