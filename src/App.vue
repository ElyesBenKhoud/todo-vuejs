<script setup>
import { ref, onMounted, computed, watch,watchEffect } from 'vue'

const todos = ref([])
const name = ref('')

const input_content = ref('')
const input_category = ref(null)

const todos_asc = computed(() => todos.value.sort((a,b) =>{
	return a.createdAt - b.createdAt
}))

watch(name, (newVal) => {
	localStorage.setItem('name', newVal)
})

watch(todos, (newVal) => {
	localStorage.setItem('todos', JSON.stringify(newVal))
}, {
	deep: true
})

const addTodo = () => {
	if (input_content.value.trim() === '' || input_category.value === null) {
		return
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
}

const removeTodo = (todo) => {
	todos.value = todos.value.filter((t) => t !== todo)
}

onMounted(() => {
	name.value = localStorage.getItem('name') || ''
	todos.value = JSON.parse(localStorage.getItem('todos')) || []
})

// const groupedTodos = computed(() => {
//   const grouped = {}

//   todos.value.forEach((todo) => {
//     if (!grouped[todo.category]) {
      
//       grouped[todo.category] = {
//         name: todo.category,
//         logoPath: '',
//         todos: [],
//       }

//       // Asynchronously load the logo
//       import(`./assets/${todo.category}Logo.png`).then((logoModule) => {
//         grouped[todo.category].logoPath = logoModule.default;
//       });
//     }
// 	grouped[todo.category].todos.push(todo)
//   })

//   return Object.values(grouped)
// })



const loadLogo = async (category) => {
  const logoModule = await import(`./assets/${category}Logo.png`);
  return logoModule.default;
};

const groupedTodos = ref({});

const addTodoAndLoadLogo = async (todo) => {
  if (!groupedTodos.value[todo.category]) {
    groupedTodos.value[todo.category] = {
      name: todo.category,
      logoPath: '',
      todos: [],
    };

    // Asynchronously load the logo and update logoPath
    try {
      const logoPath = await loadLogo(todo.category);
      groupedTodos.value[todo.category].logoPath = logoPath;
    } catch (error) {
      console.error(`Error loading logo for ${todo.category}:`, error);
    }
  }

  groupedTodos.value[todo.category].todos.push(todo);
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
			<h3>Cumparaturi list !!</h3>

			<form id="new-todo-form" @submit.prevent="addTodo">
				<h4>Ce vrei sa cumperi ?</h4>
				<input 
					type="text" 
					name="content" 
					id="content" 
					placeholder="e.g. Fructe de mare"
					v-model="input_content" />
				
				<h4>Alege magazinul </h4>
				<div class="options">

					<label>
						<input 
							type="radio" 
							name="category" 
							id="category1" 
							value="kaufland"
							v-model="input_category" />
						<span class="bubble kaufland"></span>
						<div class="brand"> <img src='./assets/kauflandLogo.png' alt="kaufland logo" class="store-logo" /></div>
						
					</label>

					<label>
						<input 
							type="radio" 
							name="category" 
							id="category2" 
							value="lidl"
							v-model="input_category" />
						<span class="bubble lidl"></span>
						<div class="brand">	<img src='./assets/lidlLogo.png' alt="lidl logo" class="store-logo" /> </div>
					
					</label>

					<label>
						<input 
							type="radio" 
							name="category" 
							id="category3" 
							value="auchan"
							v-model="input_category" />
						<span class="bubble auchan"></span>
						<div class="brand"><img src='./assets/auchanLogo.png' alt="auchan logo" class="store-logo" />  </div>
							
					</label>

				</div>

				<input type="submit" value="Add todo" />
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
							<button class="delete" @click="removeTodo(todo)">Delete</button>
						</div>
					</div>
				</div>
			</div>
		</section>

	</main>
</template>
