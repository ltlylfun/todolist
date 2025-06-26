<script>
  import { createEventDispatcher } from 'svelte'
  
  const dispatch = createEventDispatcher()
  
  let inputText = ''
  let selectedDate = new Date()
  let inputElement
  
  function handleSubmit() {
    if (inputText.trim()) {
      dispatch('add', {
        text: inputText,
        dueDate: selectedDate.toISOString()
      })
      inputText = ''
      selectedDate = new Date() // 重置为今天
      inputElement.focus()
    }
  }
  
  function handleKeydown(event) {
    if (event.key === 'Enter') {
      handleSubmit()
    }
  }
  
  function formatDateForInput(date) {
    return date.toISOString().split('T')[0]
  }
  
  function handleDateChange(event) {
    selectedDate = new Date(event.target.value)
  }
</script>

<div class="mb-6">
  <div class="flex gap-3 mb-3">
    <input
      bind:this={inputElement}
      bind:value={inputText}
      on:keydown={handleKeydown}
      type="text"
      placeholder="添加新任务..."
      class="input-field flex-1 text-lg"
      autofocus
    />
    <button
      on:click={handleSubmit}
      class="btn-primary whitespace-nowrap px-6"
      disabled={!inputText.trim()}
    >
      添加
    </button>
  </div>
  
  <!-- 日期选择器 -->
  <div class="flex items-center gap-2 text-sm">
    <label for="due-date" class="opacity-80 whitespace-nowrap" style="color: inherit;">截止日期:</label>
    <input
      id="due-date"
      type="date"
      value={formatDateForInput(selectedDate)}
      on:change={handleDateChange}
      class="input-field text-sm px-2 py-1 w-auto"
    />
  </div>
</div>
