<script>
  import { createEventDispatcher } from 'svelte'
  
  const dispatch = createEventDispatcher()
  
  let inputText = ''
  let inputElement
  
  function handleSubmit() {
    if (inputText.trim()) {
      dispatch('add', inputText)
      inputText = ''
      inputElement.focus()
    }
  }
  
  function handleKeydown(event) {
    if (event.key === 'Enter') {
      handleSubmit()
    }
  }
</script>

<div class="mb-6">
  <div class="flex gap-3">
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
</div>
