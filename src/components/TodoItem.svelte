<script>
  import { createEventDispatcher } from 'svelte'
  
  export let todo
  
  const dispatch = createEventDispatcher()
  
  let isEditing = false
  let editText = todo.text
  let editInput
  
  function startEdit() {
    isEditing = true
    editText = todo.text
    // 等待 DOM 更新后聚焦
    setTimeout(() => {
      if (editInput) {
        editInput.focus()
        editInput.select()
      }
    }, 0)
  }
  
  function saveEdit() {
    if (editText.trim()) {
      dispatch('edit', editText)
    }
    isEditing = false
  }
  
  function cancelEdit() {
    isEditing = false
    editText = todo.text
  }
  
  function handleKeydown(event) {
    if (event.key === 'Enter') {
      saveEdit()
    } else if (event.key === 'Escape') {
      cancelEdit()
    }
  }
  
  function formatDate(dateString) {
    const date = new Date(dateString)
    return date.toLocaleString('zh-CN', {
      month: 'short',
      day: 'numeric',
      hour: '2-digit',
      minute: '2-digit'
    })
  }
</script>

<div class="todo-item bg-white bg-opacity-10 backdrop-blur-sm rounded-lg p-4 border border-current border-opacity-20 hover:border-current hover:border-opacity-40 {todo.completed ? 'todo-completed' : ''}">>
  <div class="flex items-center gap-3">
    <!-- 完成状态复选框 -->
    <button
      on:click={() => dispatch('toggle')}
      class="flex-shrink-0 w-5 h-5 rounded border-2 border-current border-opacity-50 flex items-center justify-center transition-all
             {todo.completed ? 'bg-green-500 border-green-500' : 'hover:border-green-400'}"
    >
      {#if todo.completed}
        <svg class="w-3 h-3 text-white" fill="currentColor" viewBox="0 0 20 20">
          <path fill-rule="evenodd" d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z" clip-rule="evenodd"></path>
        </svg>
      {/if}
    </button>
    
    <!-- 任务内容 -->
    <div class="flex-1 min-w-0">
      {#if isEditing}
        <input
          bind:this={editInput}
          bind:value={editText}
          on:keydown={handleKeydown}
          on:blur={saveEdit}
          class="w-full bg-white bg-opacity-20 backdrop-blur-sm border border-current border-opacity-30 rounded px-2 py-1 focus:outline-none focus:ring-2 focus:ring-blue-400 placeholder-current placeholder-opacity-70"
          style="color: inherit;"
        />
      {:else}
        <div class="flex items-center justify-between">
          <span 
            class="cursor-pointer hover:opacity-80 transition-colors {todo.completed ? 'line-through opacity-50' : ''}"
            on:click={startEdit}
            on:keydown={(e) => e.key === 'Enter' && startEdit()}
            role="button"
            tabindex="0"
          >
            {todo.text}
          </span>
          <div class="flex items-center gap-2 ml-3">
            <!-- 创建时间 -->
            <span class="text-xs opacity-60 hidden sm:block">
              {formatDate(todo.createdAt)}
            </span>
            
            <!-- 编辑按钮 -->
            <button
              on:click={startEdit}
              class="opacity-60 hover:opacity-100 transition-colors p-1"
              title="编辑任务"
            >
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z"></path>
              </svg>
            </button>
            
            <!-- 删除按钮 -->
            <button
              on:click={() => dispatch('delete')}
              class="opacity-60 hover:text-red-400 transition-colors p-1"
              title="删除任务"
            >
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path>
              </svg>
            </button>
          </div>
        </div>
      {/if}
    </div>
  </div>
</div>
