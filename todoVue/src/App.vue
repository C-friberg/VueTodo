<script setup lang="ts">
import { ref, onMounted } from "vue";

interface TodoList {
  id: number;
  name: string;
}

interface TodoItem {
  id: number;
  name: string;
  isCompleted: boolean;
  todoListId: number;
}

const apiUrl = "http://localhost:5046/api/todoList";

const lists = ref<TodoList[]>([]);
const newListName = ref("");
const selectedListId = ref<number | null>(null);
const items = ref<TodoItem[]>([]);

async function getLists() {
  const res = await fetch(apiUrl);
  const data = await res.json();
  lists.value = data;
}

async function createList() {
  if (newListName.value.trim() === "") return;

  const res = await fetch(apiUrl, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      name: newListName.value,
    }),
  });

  const createdList = await res.json();

  lists.value.push(createdList);
  newListName.value = "";
}

async function getItems(listId: number) {
  const res = await fetch("http://localhost:5046/api/todoItem");
  const data = await res.json();

  items.value = data.filter((item: TodoItem) => item.todoListId === listId);
}

onMounted(() => {
  getLists();
});
</script>

<template>
  <h2>TodoList (Vue)</h2>

  <form @submit.prevent="createList">
    <input v-model="newListName" placeholder="Ny lista" />

    <button type="submit">Skapa lista</button>
  </form>

  <ul>
    <li
      v-for="list in lists"
      :key="list.id"
      @click="
        () => {
          selectedListId = list.id;
          getItems(list.id);
        }
      "
      style="cursor: pointer"
    >
      {{ list.name }}
    </li>
  </ul>

  <h3 v-if="selectedListId">Items</h3>

  <ul>
    <li v-for="item in items" :key="item.id">
      {{ item.name }} - {{ item.isCompleted ? "Completed" : "Not completed" }}
    </li>
  </ul>
</template>
