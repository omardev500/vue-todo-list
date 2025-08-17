<script setup>
  import { ref, computed, watch } from 'vue'
  
  const todoDone = ref('done')
  const todoTitle = ref('')
  const hideCompleted = ref(false)
  const themeIcon = ref('fa-solid fa-moon')
  
  if (!localStorage.getItem("todos")) { localStorage.setItem("todos", JSON.stringify([])) }
  const localStorageTodos = JSON.parse(localStorage.getItem("todos"))
  let ID, todos;
  
  try {
    ID = localStorageTodos[localStorageTodos.length - 1].id + 1;
    todos = ref(localStorageTodos)
  } catch(e) {
    ID = 0
    todos = ref([])
  }
  
  function getTodoInputId(todo) {
    return "todo-checkbox-" + todo.id
  }
  
  function updateStorage() {
    localStorage.setItem('todos', JSON.stringify(todos.value))
  }
  
  watch(todos.value, updateStorage)
  
  function addTodo() {
    todos.value.push({ id: ID++, title: todoTitle.value, done: false, date: "" })
    updateStorage()
    todoTitle.value = ''
  }
  
  function removeTodo(todo) {
    todos.value = todos.value.filter(t => t !== todo)
    updateStorage()
  }
  
  const filteredTodos = computed(() => {
    return hideCompleted.value
    ? todos.value.filter(t => !t.done)
    : todos.value
  })
  
  function getTodayDate() {
    const today = new Date();
    const year = today.getFullYear();
    const month = String(today.getMonth() + 1).padStart(2, '0'); // Zero-padded (08)
    const day = String(today.getDate()).padStart(2, '0'); // Zero-padded (16)

    const formattedDate = `${year}-${month}-${day}`;
    return formattedDate
  }
  const todayDate = getTodayDate();
  
  function toggleTheme() {
    if (document.querySelector("html").classList.contains("dark")) {
      document.querySelector("html").classList.remove("dark");
      themeIcon.value = 'fa-solid fa-sun'
    } else {
      document.querySelector("html").classList.add("dark");
      themeIcon.value = 'fa-solid fa-moon'
    }
    
  }
  // sorting days
  
  
</script>

<template>
  <header class="relative pt-8">
    <h1 class="text-center text-3xl font-bold uppercase">Todos Warrior</h1>
    <button class="absolute top-5 right-4 bg-gray-200 text-orange-700 dark:text-blue-200 text-md dark:bg-gray-500 cursor-pointer p-2 rounded-lg" @click="toggleTheme"><i :class="themeIcon"></i></button>
  </header>
  <main class="pb-5">
    <section class="relative w-screen mx-auto max-w-150 mt-4 rounded-lg min-h-110 p-3 bg-slate-200 dark:bg-slate-700 pb-14">
      <form @submit.prevent="addTodo" class="flex">
        <input type="text" placeholder="Enter todo title.." v-model="todoTitle" required autofocus class="w-3/4  rounded-tl-md rounded-bl-md border-3 text-black dark:text-white border-r-0 border-purple-600 focus:border-purple-800 dark:border-slate-500 dark:focus:border-slate-400 outline-0 pl-2 bg-slate-150 placeholder:text-gray-600  dark:bg-slate-700 text-lg dark:placeholder:text-gray-300 transition duration-200" />
        <button class="w-1/4 rounded-tr-md rounded-br-md p-3 bg-purple-600 hover:bg-purple-800 text-white font-bold dark:bg-slate-500 transition duration-300 dark:hover:bg-slate-400 cursor-pointer">Add Todo</button>
      </form>
      
      <h2 class="font-mono underline text-lg pt-3 italic">Todos</h2>
      <ul class="mt-2 mb-4 flex flex-col gap-2" v-if="ID > 0">
        <li v-for="todo in filteredTodos" :key="todo.id" class="w-full p-1 pl-2 cursor-pointer z-40 rounded-lg flex hover:bg-gray-300 dark:hover:bg-gray-500 items-center justify-between relative" @click="todo.done = !todo.done" :class="todo.done ? 'bg-gray-300 dark:bg-gray-500' : ''">
          <div>
            <input type="checkbox" class="accent-purple-600 dark:accent-slate-500 text-md" :checked="todo.done" :id="getTodoInputId(todo)" @click="todo.done = !todo.done"  />
            <label :for="getTodoInputId(todo)" class="pl-2 text-md cursor-pointer" :class="todo.done && todoDone" >{{ todo.title }}</label>
          </div>
          <div class="flex items-center justify-end">
            <label class="relative inline-block z-43 cursor-pointer">
              <i class="fa-solid fa-calendar-days text-lg cursor-pointer"></i>
              <input v-bind="{value: todo.date !== '' ? todo.date : todayDate}" type="date" class="absolute w-full h-full opacity-0 top-0 left-0 cursor-pointer dark:bg-gray-600 rounded-lg opacity-90" @change="todo.date = $event.target.value" />
            </label>
            
            <button class="cursor-pointer rounded-sm z-41 hover:bg-red-400 dark:hover:bg-red-700 font-bold text-sm ml-3 p-1 mr-2" @click="removeTodo(todo)"><i class="fa-solid fa-x"></i></button>
          </div>
        </li>
      </ul>
      
      <p v-else class="absolute top-1/2 left-1/2 -translate-1/2 text-xl font-bold">You don't have any todos yet.</p>
      
      <button class="absolute bottom-5 left-3 cursor-pointer bg-blue-200 p-2 rounded-sm text-sm mt-10  dark:bg-blue-800 border-2 border-gray-300 dark:border-gray-500" @click="hideCompleted = !hideCompleted">{{ hideCompleted ? "Show all todos?" : "Hide completed?" }}</button>
      
      <select id="sort-days" class="dark:bg-gray-600 bg-gray-500 text-white border-2 border-gray-300 dark:border-gray-500 absolute bottom-5 left-1/2 -translate-x-1/2 p-2 rounded-md dark:hover:bg-gray-500 cursor-pointer">
        <option value="all-days">All Days</option>
        <option value="today">Today</option>
        <option value="future">Future</option>
        <option value="past">Past</option>
      </select>
      
      <select id="sort-todos" class="dark:bg-pink-800 bg-lime-200 border-2 border-gray-300 dark:border-gray-500 absolute bottom-5 right-3 p-2 rounded-md dark:hover:bg-pink-700 cursor-pointer">
        <option value="latest" title="latest added todos not in date box">Latest</option>
        <option value="today" title="oldest added todos not in date box">Oldest</option>
        <option value="future" title="sort todos by alphabets">Ascending</option>
        <option value="past" title="reverse todos by alphabets">Descending</option>
      </select>
      
    </section>
  </main>
</template>

<style scoped>
  .done {
    text-decoration: line-through;
    opacity: 0.5;
  }
</style>
