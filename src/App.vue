<script setup>
  import { ref, computed, watch } from 'vue'
  
  const todoDone = ref('done')
  const todoTitle = ref('')
  const hideCompleted = ref(false)
  
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
  
  // sorting days
  
  
</script>

<template>
  <header class="relative">
    <h1 class="text-center text-3xl font-bold uppercase pt-4">Todos Warrior</h1>
    <button class="absolute top-4 right-4 dark:bg-gray-600 cursor-pointer p-3 rounded-lg">Dark</button>
  </header>
  <main class="pb-5">
    <section class="relative w-screen mx-auto max-w-150 mt-4 rounded-lg min-h-110 p-3 bg-slate-700 pb-14">
      <form @submit.prevent="addTodo" class="flex">
        <input type="text" placeholder="Enter todo title.." v-model="todoTitle" required autofocus class="w-3/4  rounded-tl-md rounded-bl-md border-3 border-r-0 border-purple-200 focus:border-purple-400 outline-0 pl-2 dark:bg-slate-700 text-lg placeholder:text-gray-300 transition duration-200" />
        <button class="w-1/4 rounded-tr-md rounded-br-md p-3 bg-purple-700 transition duration-300 hover:bg-purple-600 active:bg-purple-500 active:text-purple-100 cursor-pointer">Add Todo</button>
      </form>
      
      <h2 class="font-mono underline text-lg pt-3 italic">Todos</h2>
      <ul class="mt-2 mb-4 flex flex-col gap-2" v-if="ID > 0">
        <li v-for="todo in filteredTodos" :key="todo.id" class="w-full flex items-center relative">
          <input type="checkbox" class="accent-purple-700 text-md" :checked="todo.done" :id="getTodoInputId(todo)" @click="todo.done = !todo.done"  />
          <label :for="getTodoInputId(todo)" class="w-16/20 pl-2 cursor-pointer text-md" :class="todo.done && todoDone" >{{ todo.title }}</label>
          <button class="cursor-pointer rounded-full hover:bg-red-700 font-bold w-1/10 text-2xl ml-3 px-2 " @click="removeTodo(todo)">&times;</button>
          <input v-bind="{value: todo.date !== '' ? todo.date : todayDate}" type="date" class="w-3/10 bg-gray-600 rounded-lg opacity-90" @change="todo.date = $event.target.value" />
        </li>
      </ul>
      
      <p v-else class="absolute top-1/2 left-1/2 -translate-1/2 text-xl font-bold">You don't have any todos yet.</p>
      
      <button class="absolute bottom-5 left-3 cursor-pointer bg-slate-200 p-2 rounded-sm text-sm mt-10 dark:bg-blue-800 dark:text-white" @click="hideCompleted = !hideCompleted">{{ hideCompleted ? "See All" : "Hide completed" }}</button>
      <select id="sort-days" class="dark:bg-gray-600 absolute bottom-5 left-1/2 -translate-x-1/2 p-2 rounded-md dark:hover:bg-gray-500 cursor-pointer">
        <option value="all-days">All Days</option>
        <option value="today">Today</option>
        <option value="future">Future</option>
        <option value="past">Past</option>
      </select>
      
      <select id="sort-todos" class="dark:bg-pink-600 absolute bottom-5 right-3 p-2 rounded-md dark:hover:bg-pink-500 cursor-pointer">
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
