<template>
  <button class="btn" @click="handleClick">
    {{ label }}
  </button>
</template>

<script setup>
const props = defineProps({
  label: {
    type: String,
    default: "Button",
  },
  id: Number,
  img: String,
  first_name: String,
  last_name: String,
  sex: Number,
});

const emit = defineEmits(["click"]);

const handleClick = () => {
  emit("click", props);
};
</script>

<style scoped>
.btn {
  cursor: pointer;
  padding: 4px 12px;
  height: 40px;
}
</style>
