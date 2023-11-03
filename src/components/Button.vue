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
});

const emit = defineEmits(["click"]);

const handleClick = () => {
  emit("click");
};
</script>

<style scoped>
.btn {
  cursor: pointer;
  padding: 4px 12px;
  height: 40px;
}
</style>
