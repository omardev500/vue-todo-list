<script setup>
  import { ref, computed, watch } from 'vue'
  
  const todoDone = ref('done')
  const todoTitle = ref('')
  const hideCompleted = ref(false)
  
  const todosSortValue = ref('')
  const daysSortValue = ref('')
  
  
  if (!localStorage.getItem("theme")) { localStorage.setItem("theme", "light") }
  if (!localStorage.getItem("todos")) { localStorage.setItem("todos", JSON.stringify([])) }
  const localStorageTodos = JSON.parse(localStorage.getItem("todos"))
  let ID, todos;
  
  const themeIcon = localStorage.getItem("theme") == "dark" ? ref('fa-solid fa-moon') : ref('fa-solid fa-sun');
  if (localStorage.getItem("theme") == "dark") {
    document.querySelector("html").classList.add("dark");
  } else { document.querySelector("html").classList.remove("dark"); }
  
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
      localStorage.setItem("theme", "light")
    } else {
      document.querySelector("html").classList.add("dark");
      themeIcon.value = 'fa-solid fa-moon'
      localStorage.setItem("theme", "dark")
    }
    
  }
  function sortTodos() {
    return todosSortValue.value == "latest"
    ? todos.value.reverse()
    : todos.value[0].id < todos.value[todos.value.length - 1].id 
    ? todos.value
    : todos.value.reverse()
  }
  
  function sortDays() {
    switch(daysSortValue.value) {
      case "today":
        console.log("I'm here!")
        console.log("Today's date: ", getTodayDate())
        return todos.value.filter(t => { 
          console.log(t);
        })
        break;
      case "all-days":
        return todos.value
        break;
    }
  }
  
  watch(todos.value, updateStorage)
  watch(todosSortValue, sortTodos)
  watch(daysSortValue, sortDays)
  
</script>

<template>
  <header class="relative pt-8">
    <h1 class="text-center font-(family-name:--font-1) text-lg sm:text-2xl md:text-3xl font-bold uppercase">Todos Warrior</h1>
    <button class="absolute top-5 right-4 bg-gray-200 text-orange-700 dark:text-blue-200 text-md dark:bg-gray-500 cursor-pointer p-2 rounded-lg" @click="toggleTheme"><i :class="themeIcon"></i></button>
  </header>
  <main class="pb-5">
    <section class="relative font-(family-name:--font-1) text-sm sm:text-md max-w-[90%] w-screen mx-auto sm:max-w-150 2xl:text-2xl xl:text-xl lg:text-lg lg:max-w-170 lg:min-h-130 xl:max-w-200 lg:min-h-145 mt-4 rounded-lg min-h-110 p-3 bg-slate-200 dark:bg-slate-700 pb-14">
      <form @submit.prevent="addTodo" class="flex">
        <input type="text" placeholder="Enter todo title.." v-model="todoTitle" required autofocus class="w-7/10 sm:w-3/4 rounded-tl-md rounded-bl-md border-3 text-black dark:text-white border-r-0 border-purple-600 focus:border-purple-800 dark:border-slate-500 dark:focus:border-slate-400 outline-0 pl-2 bg-slate-150 placeholder:text-gray-600  dark:bg-slate-700 text-md sm:text-lg dark:placeholder:text-gray-300 transition duration-200" />
        <button class="w-3/10 sm:w-1/4 rounded-tr-md rounded-br-md p-2 sm:p-3 bg-purple-600 hover:bg-purple-800 text-white font-bold dark:bg-slate-500 transition duration-300 dark:hover:bg-slate-400 cursor-pointer">Add Todo</button>
      </form>

      <ul class="mt-4 mb-8 flex flex-col gap-1" v-if="ID > 0">
        <li v-for="todo in filteredTodos" :key="todo.id" class="animate-[todoAnime_0.5s_ease-out] w-full p-[2px] pl-2 z-40 rounded-lg hover:bg-gray-300 dark:hover:bg-gray-500 sm:flex relative" :class="todo.done ? 'bg-gray-300 dark:bg-gray-500' : ''">
          <input type="checkbox" class="accent-purple-600 inline-block dark:accent-slate-500 lg:text-xl" :checked="todo.done" :id="getTodoInputId(todo)" @click="todo.done = !todo.done"  />
          <label :for="getTodoInputId(todo)" class="inline-block cursor-pointer lg:p-[6px] p-[5px] w-9/10 sm:w-7/10 pl-2 cursor-pointer" :class="todo.done && todoDone" >{{ todo.title }}</label>
          
          <div class="flex items-center sm:justify-end justify-end w-full sm:w-3/10">
  
            <input v-bind="{value: todo.date || ''}" type="date" class="sm:w-full h-full opacity-0 top-0 left-0 cursor-pointer rounded-lg opacity-90" @change="todo.date = $event.target.value" />
            
            <button class="cursor-pointer rounded-sm z-41 hover:bg-red-400 dark:hover:bg-red-700 font-bold text-sm ml-3 p-1 mr-2" @click="removeTodo(todo)"><i class="fa-solid fa-x"></i></button>
          </div>
        </li>
      </ul>
      
      <p v-else class="animate-[todoAnime_0.5s_ease-out] absolute top-1/2 left-1/2 -translate-1/2 font-bold">You don't have any todos yet.</p>
      
      <button class="absolute bottom-5 left-3 cursor-pointer bg-blue-200 p-2 lg:p-3 rounded-sm mt-10  dark:bg-blue-800 border-2 border-gray-300 dark:border-gray-500" @click="hideCompleted = !hideCompleted">{{ hideCompleted ? "Show all todos?" : "Hide completed?" }}</button>
      
      <select class="dark:bg-pink-800 bg-lime-200 border-2 border-gray-300 dark:border-gray-500 absolute bottom-5 right-3 p-2 rounded-md dark:hover:bg-pink-700 cursor-pointer" @change="todosSortValue = $event.target.value">
        <option value="oldest" title="Oldest added todos">Oldest</option>
        <option value="latest" title="Latest added todos">Latest</option>
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
