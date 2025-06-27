<script>
  import { onMount } from 'svelte'
  import TodoList from './components/TodoList.svelte'
  import Calendar from './components/Calendar.svelte'
  import Header from './components/Header.svelte'
  import Footer from './components/Footer.svelte'
  import Settings from './components/Settings.svelte'
  import PomodoroTimer from './components/PomodoroTimer.svelte'
  
  let currentView = 'todo' // 'todo', 'calendar', or 'pomodoro'
  let showSettings = false
  let backgroundImage = ''
  let textColor = '#ffffff'
  let todos = []
  
  const defaultBackground = 'https://images.unsplash.com/photo-1506905925346-21bda4d32df4?ixlib=rb-4.0.3&auto=format&fit=crop&w=2070&q=80'
  
  onMount(() => {
    const savedBackground = localStorage.getItem('todolist-background')
    if (savedBackground) {
      backgroundImage = savedBackground
    } else {
      backgroundImage = defaultBackground
      localStorage.setItem('todolist-background', backgroundImage)
    }
    
    const savedTextColor = localStorage.getItem('todolist-textcolor')
    if (savedTextColor) {
      textColor = savedTextColor
    }
    
    // 加载todos数据
    const savedTodos = localStorage.getItem('svelte-todos')
    if (savedTodos) {
      todos = JSON.parse(savedTodos)
    }
  })
  
  function handleOpenSettings() {
    showSettings = true
  }
  
  function handleCloseSettings() {
    showSettings = false
  }
  
  function handleBackgroundChange(event) {
    backgroundImage = event.detail.url
    localStorage.setItem('todolist-background', backgroundImage)
  }
  
  function handleTextColorChange(event) {
    textColor = event.detail.color
    localStorage.setItem('todolist-textcolor', textColor)
  }
  
  // 任务操作函数
  function saveTodos() {
    localStorage.setItem('svelte-todos', JSON.stringify(todos))
  }
  
  function handleToggleTodo(event) {
    const id = event.detail
    todos = todos.map(todo => 
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    )
    saveTodos()
  }
  
  function handleDeleteTodo(event) {
    const id = event.detail
    todos = todos.filter(todo => todo.id !== id)
    saveTodos()
  }
  
  function handleEditTodo(event) {
    const { id, text } = event.detail
    todos = todos.map(todo => 
      todo.id === id ? { ...todo, text: text.trim() } : todo
    )
    saveTodos()
  }
  
  function handleTodosUpdate(event) {
    todos = event.detail
    saveTodos()
  }
  
  $: backgroundStyle = backgroundImage 
    ? `background-image: url('${backgroundImage}'); background-size: cover; background-position: center; background-attachment: fixed;`
    : ''
    
  $: textColorStyle = `color: ${textColor};`
</script>

<main 
  class="min-h-screen flex flex-col"
  class:bg-gradient-to-br={!backgroundImage}
  class:from-blue-50={!backgroundImage}
  class:to-indigo-100={!backgroundImage}
  style={backgroundStyle}
>
  <div class="container mx-auto px-4 py-8 max-w-4xl flex-1 flex flex-col" 
       class:bg-black={backgroundImage}
       class:bg-opacity-20={backgroundImage}
       class:bg-white={!backgroundImage}
       class:bg-opacity-10={!backgroundImage}
       class:backdrop-blur-lg={!backgroundImage}
       style={textColorStyle}>
    <Header on:openSettings={handleOpenSettings} />
    
    <!-- 视图切换选项卡 -->
    <div class="mb-6">
      <div class="flex bg-white bg-opacity-10 backdrop-blur-sm rounded-lg p-1 border border-current border-opacity-20 w-fit">
        <button
          class="px-6 py-2 rounded-md text-sm font-medium transition-all duration-200
                 {currentView === 'todo' 
                   ? 'bg-white bg-opacity-20 shadow-sm' 
                   : 'opacity-70 hover:opacity-100'}"
          style="color: inherit;"
          on:click={() => currentView = 'todo'}
        >
          待办
        </button>
        <button
          class="px-6 py-2 rounded-md text-sm font-medium transition-all duration-200
                 {currentView === 'calendar' 
                   ? 'bg-white bg-opacity-20 shadow-sm' 
                   : 'opacity-70 hover:opacity-100'}"
          style="color: inherit;"
          on:click={() => currentView = 'calendar'}
        >
          日历
        </button>
        <button
          class="px-6 py-2 rounded-md text-sm font-medium transition-all duration-200
                 {currentView === 'pomodoro' 
                   ? 'bg-white bg-opacity-20 shadow-sm' 
                   : 'opacity-70 hover:opacity-100'}"
          style="color: inherit;"
          on:click={() => currentView = 'pomodoro'}
        >
          番茄钟
        </button>
      </div>
    </div>
    
    <div class="flex-1">
      {#if currentView === 'todo'}
        <TodoList on:todosUpdate={handleTodosUpdate} />
      {:else if currentView === 'calendar'}
        <Calendar 
          {todos}
          on:toggle={handleToggleTodo}
          on:delete={handleDeleteTodo}
          on:edit={handleEditTodo}
        />
      {:else if currentView === 'pomodoro'}
        <PomodoroTimer />
      {/if}
    </div>
    <Footer />
  </div>
</main>

<Settings 
  bind:isOpen={showSettings}
  {backgroundImage}
  {textColor}
  on:close={handleCloseSettings}
  on:backgroundChange={handleBackgroundChange}
  on:textColorChange={handleTextColorChange}
/>
