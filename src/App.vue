<script setup>
import { ref, onMounted, computed, watch } from 'vue';
import Toastify from 'toastify-js';
import 'toastify-js/src/toastify.css';

const showToast = (message, duration, color) => {
  Toastify({
    text: message,
    duration: duration,
    gravity: 'top',
    position: 'right',
    backgroundColor: color,
  }).showToast();
};

const todos = ref([]);
const name = ref('');

const input_content = ref('');
const input_category = ref(null);

watch(name, (newVal) => {
  localStorage.setItem('name', newVal);
});

const addTodo = () => {
  if (input_content.value.trim() === '' || input_category.value === null) {
    return;
  }

  const newTodo = {
    content: input_content.value,
    category: input_category.value,
    done: false,
    editable: false,
    createdAt: new Date().getTime(),
  };

  todos.value.push(newTodo);
  addTodoAndLoadLogo(newTodo);
  showToast('Cumpărătură adăugat cu succes!', 2500, 'linear-gradient(90deg, rgba(106,215,73,1) 0%, rgba(126,236,79,1) 26%, rgba(176,255,251,1) 100%)');
  input_content.value = '';
  input_category.value = null;
};

const removeTodoAndLogo = (todo) => {
  todos.value = todos.value.filter((t) => t !== todo);

  if (groupedTodosLocalStorage.value[todo.category]) {
    const todoIndex = groupedTodosLocalStorage.value[todo.category].todos.indexOf(todo);
    if (todoIndex !== -1) {
      groupedTodosLocalStorage.value[todo.category].todos.splice(todoIndex, 1);

      if (groupedTodosLocalStorage.value[todo.category].todos.length === 0) {
        delete groupedTodosLocalStorage.value[todo.category];
      }
    }

    // Save the updated groupedTodosLocalStorage to localStorage
    localStorage.setItem('groupedTodos', JSON.stringify(groupedTodosLocalStorage.value));
  }
};


const removeTodo = (todo) => {
  removeTodoAndLogo(todo);
  showToast('Cumpărătură a fost șters cu succes!', 2500, 'linear-gradient(90deg, rgba(241,133,133,1) 0%, rgba(240,0,0,1) 99%, rgba(176,255,251,1) 100%)');
};

const groupedTodosLocalStorage = ref(JSON.parse(localStorage.getItem('groupedTodos')) || {});

watch(groupedTodosLocalStorage, (newVal) => {
  localStorage.setItem('groupedTodos', JSON.stringify(newVal));
});

onMounted(() => {
  name.value = localStorage.getItem('name') || '';
  groupedTodosLocalStorage.value = JSON.parse(localStorage.getItem('groupedTodos')) || {};
});

const loadLogo = async (category) => {
  const logoModule = await import(`./assets/${category}Logo.png`);
  return logoModule.default;
};

const groupedTodos = computed(() => Object.values(groupedTodosLocalStorage.value));

const addTodoAndLoadLogo = async (todo) => {
  if (!groupedTodosLocalStorage.value[todo.category]) {
    groupedTodosLocalStorage.value[todo.category] = {
      name: todo.category,
      logoPath: '',
      todos: [],
    };

    try {
      const logoPath = await loadLogo(todo.category);
      groupedTodosLocalStorage.value[todo.category].logoPath = logoPath;
    } catch (error) {
      console.error(`Error loading logo for ${todo.category}:`, error);
    }
  }

  groupedTodosLocalStorage.value[todo.category].todos.push(todo);
};

</script>

<template>
  <main class="app">
    <section class="greeting">
      <h2 class="title">
        Bine ati venit, <input type="text" id="name" placeholder="numele tale" v-model="name">
      </h2>
    </section>

    <section class="create-todo">

      <form id="new-todo-form" @submit.prevent="addTodo">
        <h4>Ce vrei sa cumperi ?</h4>
        <input
          type="text"
          name="content"
          id="content"
          placeholder="e.g. Fructe de mare"
          v-model="input_content"
        />

        <h4>Alege magazinul </h4>
        <div class="options">
          <label>
            <input
              type="radio"
              name="category"
              id="category1"
              value="kaufland"
              v-model="input_category"
            />
            <span class="bubble kaufland"></span>
            <div class="brand"> <img src='./assets/kauflandLogo.png' alt="kaufland logo" class="store-logo" /></div>

          </label>

          <label>
            <input
              type="radio"
              name="category"
              id="category2"
              value="lidl"
              v-model="input_category"
            />
            <span class="bubble lidl"></span>
            <div class="brand"> <img src='./assets/lidlLogo.png' alt="lidl logo" class="store-logo" /> </div>
          </label>

          <label>
            <input
              type="radio"
              name="category"
              id="category3"
              value="auchan"
              v-model="input_category"
            />
            <span class="bubble auchan"></span>
            <div class="brand"> <img src='./assets/auchanLogo.png' alt="auchan logo" class="store-logo" /> </div>
          </label>
        </div>
        <input type="submit" value="Adăuga" />
      </form>
    </section>

    <section class="todo-list">
      <div class="list" id="todo-list">
        <div v-for="magazine in groupedTodos" :key="magazine.name">
          <img :src="magazine.logoPath" alt="Magazine Logo" class="store-logo" />

          <div v-for="todo in magazine.todos" :class="`todo-item ${todo.done && 'done'}`">
            <label>
              <input type="checkbox" v-model="todo.done" />
              <span :class="`bubble ${todo.category}`"></span>
            </label>

            <div class="todo-content">
              <input type="text" v-model="todo.content" />
            </div>

            <div class="actions">
              <button class="delete" @click="removeTodo(todo)">șterge</button>
            </div>
          </div>
        </div>
      </div>
    </section>
  </main>
</template>
