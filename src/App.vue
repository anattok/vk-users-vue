<template>
  <div class="wrapper">
    <div class="wrapper__top">
      <Input v-model:value="searchText" placeholder="Пиши" />
      <Button label="Добавить в 'Исходный' " />
    </div>

    <ul v-if="searchText" class="users">
      <User
        v-for="user in filteredFriends"
        :key="user.id"
        :id="user.id"
        :img="user.photo_100"
        :first_name="user.first_name"
        :last_name="user.last_name"
      />
    </ul>
  </div>
</template>

<script setup>
import { onBeforeMount, ref, computed } from "vue";
import User from "./components/User.vue";
import Input from "./components/Input.vue";
import Button from "./components/Button.vue";

const APP_ID = 51784186;
const VERSION = "5.131";
let ID_USER;
const friendsList = ref([]);
const searchText = ref("");

onBeforeMount(async () => {
  VK.init({
    apiId: APP_ID,
  });

  await VK.Auth.getLoginStatus((response) => {
    if (response.status === "connected") {
      ID_USER = response.session.mid;
      console.log(`ID авторизованного пользователя: ${ID_USER}`);
    } else {
      VK.Auth.login();
    }
  });

  await VK.Api.call(
    "friends.get",
    {
      user_id: ID_USER,
      v: VERSION,
      fields: "photo_200_orig,photo_100,photo_50",
    },
    (response) => {
      if (response.response) {
        friendsList.value = response.response.items;
        console.log(friendsList.value);
      } else {
        console.error("Ошибка при получении списка друзей");
      }
    }
  );
});

const filteredFriends = computed(() => {
  return friendsList.value.filter((user) => {
    const fullName = `${user.first_name} ${user.last_name} ${user.id}`;
    return fullName.toLowerCase().includes(searchText.value.toLowerCase());
  });
});
</script>

<style scoped>
.wrapper__top {
  display: flex;
  align-items: flex-end;
}
.wrapper {
  display: flex;
  position: relative;
}
.users {
  position: absolute;
  top: 80px;
  left: 0;
  padding: 5px 10px;
  max-height: 285px;
  overflow-y: scroll;
}
</style>
