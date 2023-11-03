<script setup>
const props = defineProps({
  id: Number,
  img: String,
  first_name: String,
  last_name: String,
  sex: Number,
});

const emit = defineEmits(["data-updated"]);

const handleClick = () => {
  emit("data-updated", props);
};
</script>

<template>
  <li class="user" @click="handleClick">
    <img class="user__img" :src="img" alt="User Photo" />
    <span class="user__name"> {{ first_name }}</span>
    <span class="user__last-name">{{ last_name }}</span>
  </li>
</template>

<style scoped>
.user {
  width: 250px;
  display: flex;
  align-items: center;
  background: #373737;
  margin: 5px 0;
  gap: 5px;
  border-radius: 5px;
  overflow: hidden;
  cursor: pointer;
  transition: all 0.3s;
}
.user:hover {
  background-color: #272727;
}
.user__img {
  display: block;
  width: 50px;
  height: 50px;
}
</style>
