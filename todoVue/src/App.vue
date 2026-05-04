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

const editingListId = ref<number | null>(null);
const editingName = ref("");

const selectedListId = ref<number | null>(null);
const items = ref<TodoItem[]>([]);

async function getLists() {
  const res = await fetch(apiUrl);
  const data = await res.json();
  lists.value = data;
}

async function createList() {
  const trimmedName = newListName.value.trim();

  if (!trimmedName) return;

  await fetch(apiUrl, {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      name: trimmedName,
    }),
  });

  newListName.value = "";
  await getLists();
}

function startEdit(list: TodoList) {
  editingListId.value = list.id;
  editingName.value = list.name;
}

function cancelEdit() {
  editingListId.value = null;
  editingName.value = "";
}

async function updateList(list: TodoList) {
  const trimmedName = editingName.value.trim();

  if (!trimmedName) return;

  await fetch(`${apiUrl}/${list.id}`, {
    method: "PUT",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({
      name: trimmedName,
    }),
  });

  cancelEdit();
  await getLists();
}

async function deleteList(id: number) {
  await fetch(`${apiUrl}/${id}`, {
    method: "DELETE",
  });

  if (editingListId.value === id) {
    cancelEdit();
  }

  await getLists();
}

async function getItems(listId: number) {
  const res = await fetch("http://localhost:5046/api/todoItem");
  const data = await res.json();

  selectedListId.value = listId;
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

  <p>Antal listor: {{ lists.length }}</p>

  <ul>
    <li v-for="list in lists" :key="list.id">
      <template v-if="editingListId === list.id">
        <form @submit.prevent="updateList(list)">
          <input v-model="editingName" />
          <button type="submit">Spara</button>
          <button type="button" @click="cancelEdit">Avbryt</button>
        </form>
      </template>

      <template v-else>
        <span @click="getItems(list.id)" style="cursor: pointer">
          {{ list.name }}
        </span>

        <button type="button" @click="startEdit(list)">Ändra</button>
        <button type="button" @click="deleteList(list.id)">Ta bort</button>
      </template>
    </li>
  </ul>

  <h3 v-if="selectedListId">Items</h3>

  <ul>
    <li v-for="item in items" :key="item.id">
      {{ item.name }} -
      {{ item.isCompleted ? "Completed" : "Not completed" }}
    </li>
  </ul>
</template>