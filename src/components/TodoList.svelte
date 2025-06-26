<script>
  import { onMount, createEventDispatcher } from 'svelte'
  import TodoItem from './TodoItem.svelte'
  import AddTodo from './AddTodo.svelte'
  import FilterTabs from './FilterTabs.svelte'
  import Stats from './Stats.svelte'
  
  const dispatch = createEventDispatcher()
  
  let todos = []
  let filter = 'all' // 'all', 'active', 'completed'
  
  onMount(() => {
    const savedTodos = localStorage.getItem('svelte-todos')
    if (savedTodos) {
      todos = JSON.parse(savedTodos)
    }
  })
  
  function saveTodos() {
    localStorage.setItem('svelte-todos', JSON.stringify(todos))
    dispatch('todosUpdate', todos)
  }

  function addTodo(event) {
    const { text, dueDate } = event.detail
    const newTodo = {
      id: Date.now().toString(),
      text: text.trim(),
      completed: false,
      createdAt: new Date().toISOString(),
      dueDate: dueDate
    }
    todos = [newTodo, ...todos]
    saveTodos()
  }

  function toggleTodo(id) {
    todos = todos.map(todo => 
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    )
    saveTodos()
  }

  function deleteTodo(id) {
    todos = todos.filter(todo => todo.id !== id)
    saveTodos()
  }

  function editTodo(id, newText) {
    todos = todos.map(todo => 
      todo.id === id ? { ...todo, text: newText.trim() } : todo
    )
    saveTodos()
  }

  function clearCompleted() {
    todos = todos.filter(todo => !todo.completed)
    saveTodos()
  }

  function toggleAll() {
    const allCompleted = todos.every(todo => todo.completed)
    todos = todos.map(todo => ({ ...todo, completed: !allCompleted }))
    saveTodos()
  }

  $: filteredTodos = todos.filter(todo => {
    if (filter === 'active') return !todo.completed
    if (filter === 'completed') return todo.completed
    return true
  })

  $: stats = {
    total: todos.length,
    active: todos.filter(todo => !todo.completed).length,
    completed: todos.filter(todo => todo.completed).length
  }
</script>

<div class="bg-white bg-opacity-10 backdrop-blur-lg rounded-2xl shadow-xl border border-white border-opacity-20 p-6 md:p-8 animate-slideIn">
  <!-- 统计信息 -->
  <Stats {stats} />
  
  <!-- 添加任务 -->
  <AddTodo on:add={addTodo} />
  
  <!-- 过滤选项卡 -->
  {#if todos.length > 0}
    <FilterTabs bind:filter {stats} on:clearCompleted={clearCompleted} />
    
    <!-- 全选/取消全选 -->
    <div class="mb-4">
      <button 
        class="text-sm opacity-80 hover:opacity-100 transition-colors"
        style="color: inherit;"
        on:click={toggleAll}
      >
        {todos.every(todo => todo.completed) ? '取消全选' : '全选'}
      </button>
    </div>
  {/if}
  
  <!-- 任务列表 -->
  <div class="space-y-2">
    {#each filteredTodos as todo (todo.id)}
      <TodoItem 
        {todo} 
        on:toggle={() => toggleTodo(todo.id)}
        on:delete={() => deleteTodo(todo.id)}
        on:edit={e => editTodo(todo.id, e.detail)}
      />
    {:else}
      <div class="text-center py-12 opacity-70" style="color: inherit;">
        {#if filter === 'active'}
          所有任务都已完成！
        {:else if filter === 'completed'}
          还没有已完成的任务
        {:else}
          还没有任务，赶紧添加一个吧！
        {/if}
      </div>
    {/each}
  </div>
</div>
