<template>
  <div class="wrapper">
    <div class="wrapper__top">
      <Input
        v-model:modelValue="searchText"
        placeholder="Пиши"
        label="Имя/Фамилия или id"
      />
    </div>
    <ul v-if="searchText" class="users">
      <li class="users__item" v-for="user in filteredFriends">
        <User
          :key="user.id"
          :id="user.id"
          :img="user.photo_100"
          :first_name="user.first_name"
          :last_name="user.last_name"
          :sex="user.sex"
        />
        <Button
          :label="isAdded(user.id) ? 'Удалить из списка' : 'Добавить в список'"
          @click="
            () => {
              handlerButtonDropList(user);
            }
          "
        />
      </li>
      <li v-if="filteredFriends.length === 0">Совпадений не найдено</li>
    </ul>
    <div class="wrapper__main">
      <!-- Список "Исходный" -->

      <div class="original" v-if="originalList.length > 0">
        <div class="original__top">
          <h2>Список "Исходный"</h2>
          <Button
            label="Построить"
            v-if="originalList.length > 0"
            @click="addOriginalListAllFriends"
          />
        </div>
        <ul class="original__list">
          <li
            class="original__list__item"
            v-for="user in originalList"
            :key="user.id"
          >
            <img :src="user.photo_100" :alt="user.first_name" />
            <p>{{ user.first_name }}</p>
            <p>{{ user.last_name }}</p>
          </li>
        </ul>
      </div>

      <!-- Список "Друзья" -->

      <div class="all-friends" v-if="originalListAllFriends.length > 0">
        <div class="all-friends__top">
          <h2>
            Список "Друзья"
            <span>Всего:{{ originalListAllFriends.length }}</span>
          </h2>

          <div class="sort">
            <b>Сортировать по:</b>
            <span @click="toggleSort">{{ typeSort }}</span>
            <ul class="sort__list" :class="{ active: isOpenedSort }">
              <li v-for="item in sortList" @click="selectSort(item)">
                {{ item }}
              </li>
            </ul>
          </div>
        </div>

        <ul class="all-friends__list">
          <li
            class="all-friends__item"
            v-for="user in sortedListAllFriends"
            :key="user.id"
            ref="userItems"
          >
            <img :src="user.photo_50" :alt="user.first_name" />
            <span>{{ user.first_name }} {{ user.last_name }}</span>
            <span>Пол: {{ user.sex === 1 ? "Ж" : "М" }}</span>
            <span>{{ user.friend_count }}</span>
          </li>
        </ul>
        <Button @click="loadMore" label="Показать еще 5 человек" />
      </div>
    </div>
  </div>
</template>

<script setup>
import {
  onBeforeMount,
  ref,
  computed,
  onMounted,
  onUnmounted,
  withDefaults,
  watch,
} from "vue";
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
//список Исходный
const originalList = ref([]);
//Список Друзья
const originalListAllFriends = ref([]);
// Отображаемые элементы
const displayedFriends = ref([]);
//сортировка
const typeSort = ref("Имя");
const isOpenedSort = ref(false);
const sortList = ["Имя", "Фамилия"];

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

//закрывавем открываем сортировку
const toggleSort = () => {
  isOpenedSort.value = !isOpenedSort.value;
};

//меняем typeSort и закрываем
const selectSort = (item) => {
  typeSort.value = item;
  isOpenedSort.value = !isOpenedSort.value;
};

//сортировка массива
const sortedListAllFriends = computed(() => {
  if (typeSort.value === "Имя") {
    console.log(displayedFriends);
    console.log(sortedListAllFriends);
    return displayedFriends.value.sort((a, b) =>
      a.first_name.localeCompare(b.first_name)
    );
  } else if (typeSort.value === "Фамилия") {
    return displayedFriends.value.sort((a, b) =>
      a.last_name.localeCompare(b.last_name)
    );
  }
});

//фильтрация списка друзей при вводе в инпут
const filteredFriends = computed(() => {
  return friendsList.value.filter((user) => {
    const fullName = `${user.first_name} ${user.last_name} ${user.id}`;
    return fullName.toLowerCase().includes(searchText.value.toLowerCase());
  });
});

//проверка добавленности пользователя в списаок друзья для изменения надписи кнопки
const isAdded = (id) => {
  return originalList.value.some((user) => user.id === id);
};

// добавления или удаления по кнопке
const handlerButtonDropList = (user) => {
  const isAddedUser = isAdded(user.id);

  if (isAddedUser) {
    const indexToRemove = originalList.value.findIndex(
      (item) => item.id === user.id
    );
    if (indexToRemove !== -1) {
      originalList.value.splice(indexToRemove, 1);
    }
  } else {
    originalList.value.push(user);
  }
  searchText.value = ""; // Сбрасываем текст поиска

  console.log(originalList);
};

//Подгрузка
let intersectionObserver = null;

const loadMore = () => {
  const nextBatch = originalListAllFriends.value.slice(
    displayedFriends.value.length,
    displayedFriends.value.length + 5
  );
  displayedFriends.value = [...displayedFriends.value, ...nextBatch];
};

const observerCallback = (entries) => {
  entries.forEach((entry) => {
    if (entry.isIntersecting) {
      const userItem = entry.target.__userItem;
      const index = displayedFriends.value.findIndex(
        (user) => user.id === userItem.id
      );
      if (index !== -1) {
        displayedFriends.value.splice(index, 1);
      }
    }
  });
};

onMounted(() => {
  displayedFriends.value = originalListAllFriends.value.slice(0, 5); // Отображаем первые 5 элементов при начальной загрузке
  intersectionObserver = new IntersectionObserver(observerCallback);
  document.querySelectorAll(".all-friends__item").forEach((item) => {
    intersectionObserver.observe(item);
    item.__userItem = item; // Сохраняем элемент пользователя как атрибут
  });
});

onUnmounted(() => {
  intersectionObserver.disconnect();
});

const addOriginalListAllFriends = async () => {
  const ids = originalList.value.map((user) => user.id);

  for (const id of ids) {
    await getFriends(id);
  }
  loadMore();
};

const getFriends = async (id) => {
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

    const filteredNewFriends = newFriends.filter((friend) => {
      return !originalListAllFriends.value.some((existingFriend) => {
        return existingFriend.id === friend.id;
      });
    });

    originalListAllFriends.value.push(...filteredNewFriends);
  }
};
</script>

<style scoped>
.wrapper__top {
  margin-bottom: 40px;
  width: 400px;
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
  top: 80px;
  left: 0;
  padding: 5px 10px;
  max-height: 290px;
  width: 480px;
  overflow-y: scroll;
  border-radius: 5px;
  box-shadow: -8px 8px 5px 0px rgba(0, 0, 0, 0.2);
}

.users::-webkit-scrollbar {
  width: 6px;
}
.users::-webkit-scrollbar-track {
  background: #ffa500;
}
.users::-webkit-scrollbar-thumb {
  background-color: #373737;
  border-radius: 3px;
}
.users__item {
  height: 50px;
  box-sizing: border-box;
  display: flex;
  align-items: center;
  gap: 10px;
  background: #373737;
  margin: 5px 0;
  border-radius: 5px;
  overflow: hidden;
  transition: all 0.3s;
  padding-right: 30px;
}
.original {
  width: 100%;
  height: auto;
  margin-bottom: 30px;
}
.original__top {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  gap: 30px;
  margin-bottom: 30px;
}
.original__list {
  display: flex;
  flex-wrap: wrap;
  gap: 5px;
  list-style-type: none;
  background-color: #373737;
  padding: 10px 30px;
  border-radius: 5px;
}
.original__list__item {
  padding: 10px 20px;
  width: 140px;
  height: 180px;
  text-align: center;
  background-color: #555454;
  box-shadow: -8px 8px 5px 0px rgba(0, 0, 0, 0.2);
  border-radius: 5px;
}
.original__list__item img {
  border-radius: 5px;
}
.all-friends {
  display: flex;
  flex-direction: column;
  width: 100%;
}
.all-friends__top {
  margin-bottom: 30px;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.all-friends__list {
  max-height: 300px;
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
.sort {
  position: relative;
}
.sort span {
  cursor: pointer;
}
.sort__list {
  display: none;
  position: absolute;
  top: 30px;
  right: 0;
  width: 100px;
  list-style-type: none;
  background-color: #555454;
  border-radius: 5px;
  padding: 5px 10px;
  box-shadow: -8px 8px 5px 0px rgba(0, 0, 0, 0.2);
}
.sort__list.active {
  display: block;
}
.sort__list li {
  cursor: pointer;
}
.sort__list li:hover {
  color: #373737;
}
.sort span {
  margin-left: 5px;
  border-bottom: 1px dotted #555454;
}
</style>
