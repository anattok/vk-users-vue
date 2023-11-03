<template>
  <div class="wrapper">
    <div class="wrapper__top">
      <Input
        v-model:modelValue="searchText"
        placeholder="Пиши"
        label="Имя/Фамилия или id"
      />
      <Button
        label="Добавить или удалить из спиcка 'Исходный' "
        @click="addInOriginalList"
      />
      <Button
        label="Построить"
        v-if="originalList.length > 0"
        @click="addOriginalListAllFriends"
      />
    </div>
    <ul v-if="searchText" class="users">
      <User
        @data-updated="updateInput"
        v-for="user in filteredFriends"
        :key="user.id"
        :id="user.id"
        :img="user.photo_100"
        :first_name="user.first_name"
        :last_name="user.last_name"
        :sex="user.sex"
      />
    </ul>
    <div class="wrapper__main">
      <div class="original" v-if="originalList.length > 0">
        <p>
          Список "Исходный" <span>Количество {{ originalList.length }}</span>
        </p>
        <ul class="original__list">
          <li v-for="user in originalList" :key="user.id">
            <img :src="user.img" :alt="user.first_name" />
          </li>
        </ul>
      </div>
      <div class="all-friends" v-if="originalListAllFriends.length > 0">
        <div class="all-friends__top">
          <p>
            Список "Друзья "<span
              >Количество {{ originalListAllFriends.length }}</span
            >
          </p>
        </div>

        <ul class="all-friends__list">
          <li
            class="all-friends__item"
            v-for="user in originalListAllFriends"
            :key="user.id"
          >
            <img :src="user.photo_50" :alt="user.first_name" />
            <span>{{ user.first_name }} {{ user.last_name }}</span>
            <span>Пол: {{ user.sex === 1 ? "Ж" : "М" }}</span>
            <span>{{ user.friend_count }}</span>
          </li>
        </ul>
      </div>
    </div>
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
//список друзей аворизованного пользователя
const friendsList = ref([]);
//значение инпута
const searchText = ref("");
//cохраним пропсы кликнутого юзера
const clickedUser = ref({});
//списоку Исходный
const originalList = ref([]);
//друзья людей списка Исходный
const originalListAllFriends = ref([]);

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
      fields: "photo_200_orig,photo_100,photo_50,sex",
    },
    (response) => {
      if (response.response) {
        friendsList.value = response.response.items;
      } else {
        console.error("Ошибка при получении списка друзей");
      }
    }
  );
});
//фильтрация списка друзей при вводе в инпут
const filteredFriends = computed(() => {
  return friendsList.value.filter((user) => {
    const fullName = `${user.first_name} ${user.last_name} ${user.id}`;
    return fullName.toLowerCase().includes(searchText.value.toLowerCase());
  });
});

//функция добавления или удаления по кнопке
const addInOriginalList = () => {
  const userToAdd = clickedUser.value;

  // Проверяем, существует ли пользователь в originalList
  const existingUserIndex = originalList.value.findIndex(
    (user) => user.id === userToAdd.id
  );

  if (existingUserIndex !== -1) {
    // Если пользователь существует, удаляем его
    originalList.value.splice(existingUserIndex, 1);
  } else {
    // иначе удаляем
    originalList.value.push(userToAdd);
  }
  console.log(originalList.value);
  searchText.value = ""; // Очищаем инпут
};

//функция обновление инпута
const updateInput = (data) => {
  searchText.value = `${data.first_name} ${data.last_name}`;
  clickedUser.value = data;
};

const fetchDataFromVK = async (id) => {
  try {
    // Выполняем запрос к VK API для получения друзей
    const response = await new Promise((resolve, reject) => {
      VK.Api.call(
        "friends.get",
        {
          user_id: id,
          v: VERSION,
          fields: "photo_50,sex",
        },
        (response) => {
          resolve(response);
        }
      );
    });

    if (response && response.response) {
      const newFriends = response.response.items;
      console.log(newFriends);

      // Фильтруем новых друзей, чтобы исключить дубли
      const filteredNewFriends = newFriends.filter(
        (friend) =>
          !originalListAllFriends.value.some(
            (existingFriend) => existingFriend.id === friend.id
          )
      );

      console.log(filteredNewFriends);

      // Получаем количество друзей для каждого нового друга
      for (const friend of filteredNewFriends) {
        const friendId = friend.id;

        try {
          const friendResponse = await new Promise((resolve, reject) => {
            VK.Api.call(
              "friends.get",
              {
                user_id: friendId,
                v: VERSION,
              },
              (response) => {
                resolve(response);
              }
            );
          });

          if (friendResponse && friendResponse.response) {
            friend.friend_count = friendResponse.response.count;
          } else {
            console.error(
              "Ошибка при получении количества друзей: ",
              friendResponse
            );
            friend.friend_count = 0; // Обработка ошибки, если запрос к VK API не удался
          }
        } catch (error) {
          console.error(
            "Ошибка при выполнении запроса к VK API для получения друзей: ",
            error
          );
          friend.friend_count = 0; // Обработка ошибки, если запрос к VK API не удался
        }
      }

      // Добавляем новых друзей в общий список
      originalListAllFriends.value.push(...filteredNewFriends);

      console.log(originalListAllFriends.value);
    } else {
      console.error("Ошибка при получении списка друзей");
    }
  } catch (error) {
    console.error(
      "Ошибка при выполнении запроса к VK API для получения друзей: ",
      error
    );
  }
};

const addOriginalListAllFriends = async () => {
  const ids = originalList.value.map((user) => user.id);

  for (const id of ids) {
    await fetchDataFromVK(id);
  }
};
</script>

<style scoped>
.wrapper__top {
  display: flex;
  align-items: flex-end;
  flex-wrap: wrap;
  gap: 20px;
  margin-bottom: 40px;
}
.wrapper {
  position: relative;
}
.wrapper__main {
  display: flex;
  flex-direction: column;
}
.users {
  position: absolute;
  background-color: #535151;
  border: 1px solid #fff;
  top: 80px;
  left: 0;
  padding: 5px 10px;
  max-height: 280px;
  width: 300px;
  overflow-y: scroll;
}
.original {
  width: 100%;
  margin-bottom: 30px;
}
.original p {
  margin-bottom: 10px;
}

.original__list {
  display: flex;
  flex-wrap: wrap;
  list-style-type: none;
}
.original__list li {
  width: 100px;
  height: 100px;
}
.all-friends {
  display: flex;
  flex-direction: column;
  width: 100%;
}
.all-friends__top {
  margin-bottom: 10px;
}
.all-friends__list {
  max-height: 400px;
  overflow-y: scroll;
}

.all-friends__item {
  padding: 5px 10px;
  display: flex;
  align-items: center;
  margin-bottom: 5px;
  background-color: #535151;
  gap: 10px;
}
</style>
