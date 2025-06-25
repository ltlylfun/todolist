<script>
  import { onMount } from 'svelte'
  import TodoItem from './TodoItem.svelte'
  import AddTodo from './AddTodo.svelte'
  import FilterTabs from './FilterTabs.svelte'
  import Stats from './Stats.svelte'
  
  let todos = []
  let filter = 'all' // 'all', 'active', 'completed'
  
  // 从 localStorage 加载任务
  onMount(() => {
    const savedTodos = localStorage.getItem('svelte-todos')
    if (savedTodos) {
      todos = JSON.parse(savedTodos)
    }
  })
  
  // 保存到 localStorage
  function saveTodos() {
    localStorage.setItem('svelte-todos', JSON.stringify(todos))
  }
  
  // 添加新任务
  function addTodo(text) {
    const newTodo = {
      id: Date.now().toString(),
      text: text.trim(),
      completed: false,
      createdAt: new Date().toISOString()
    }
    todos = [newTodo, ...todos]
    saveTodos()
  }
  
  // 切换任务完成状态
  function toggleTodo(id) {
    todos = todos.map(todo => 
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    )
    saveTodos()
  }
  
  // 删除任务
  function deleteTodo(id) {
    todos = todos.filter(todo => todo.id !== id)
    saveTodos()
  }
  
  // 编辑任务
  function editTodo(id, newText) {
    todos = todos.map(todo => 
      todo.id === id ? { ...todo, text: newText.trim() } : todo
    )
    saveTodos()
  }
  
  // 清除所有已完成任务
  function clearCompleted() {
    todos = todos.filter(todo => !todo.completed)
    saveTodos()
  }
  
  // 切换所有任务状态
  function toggleAll() {
    const allCompleted = todos.every(todo => todo.completed)
    todos = todos.map(todo => ({ ...todo, completed: !allCompleted }))
    saveTodos()
  }
  
  // 过滤任务
  $: filteredTodos = todos.filter(todo => {
    if (filter === 'active') return !todo.completed
    if (filter === 'completed') return todo.completed
    return true
  })
  
  // 统计信息
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
  <AddTodo on:add={e => addTodo(e.detail)} />
  
  <!-- 过滤选项卡 -->
  {#if todos.length > 0}
    <FilterTabs bind:filter {stats} on:clearCompleted={clearCompleted} />
    
    <!-- 全选/取消全选 -->
    <div class="mb-4">
      <button 
        class="text-sm text-white text-opacity-80 hover:text-white transition-colors"
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
      <div class="text-center py-12 text-white text-opacity-70">
        {#if filter === 'active'}
          🎉 所有任务都已完成！
        {:else if filter === 'completed'}
          还没有已完成的任务
        {:else}
          还没有任务，赶紧添加一个吧！
        {/if}
      </div>
    {/each}
  </div>
</div>
