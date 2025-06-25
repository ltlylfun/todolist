<script>
  import { createEventDispatcher } from 'svelte'
  
  const dispatch = createEventDispatcher()
  
  export let isOpen = false
  export let backgroundImage = ''
  export let textColor = '#ffffff'
  
  let fileInput
  let customUrl = ''
  let customTextColor = textColor
  
  // 预设字体颜色
  const presetTextColors = [
    '#ffffff', 
    '#000000', 
    '#1f2937', 
    '#374151', 
    '#6b7280', 
    '#ef4444', 
    '#f97316', 
    '#eab308', 
    '#22c55e', 
    '#3b82f6', 
    '#8b5cf6', 
    '#ec4899'  
  ]

  // 预设背景图片
  const presetBackgrounds = [
    'https://images.unsplash.com/photo-1506905925346-21bda4d32df4?ixlib=rb-4.0.3&auto=format&fit=crop&w=2070&q=80', 
    'https://images.unsplash.com/photo-1441974231531-c6227db76b6e?ixlib=rb-4.0.3&auto=format&fit=crop&w=2071&q=80', 
    'https://images.unsplash.com/photo-1507525428034-b723cf961d3e?ixlib=rb-4.0.3&auto=format&fit=crop&w=2073&q=80', 
    'https://images.unsplash.com/photo-1469474968028-56623f02e42e?ixlib=rb-4.0.3&auto=format&fit=crop&w=2070&q=80', 
    'https://images.unsplash.com/photo-1518837695005-2083093ee35b?ixlib=rb-4.0.3&auto=format&fit=crop&w=2070&q=80'  
  ]
  
  function selectTextColor(color) {
    customTextColor = color
    dispatch('textColorChange', { color })
  }

  function closeModal() {
    isOpen = false
    dispatch('close')
  }
  
  function selectBackground(url) {
    dispatch('backgroundChange', { url })
    closeModal()
  }
  
  function handleFileUpload(event) {
    const file = event.target.files[0]
    if (file && file.type.startsWith('image/')) {
      const reader = new FileReader()
      reader.onload = (e) => {
        dispatch('backgroundChange', { url: e.target.result })
        closeModal()
      }
      reader.readAsDataURL(file)
    }
  }
  
  function handleCustomUrl() {
    if (customUrl.trim()) {
      dispatch('backgroundChange', { url: customUrl.trim() })
      closeModal()
    }
  }
  
  function handleCustomTextColor() {
    if (customTextColor) {
      dispatch('textColorChange', { color: customTextColor })
    }
  }

  function handleKeydown(event) {
    if (event.key === 'Escape') {
      closeModal()
    }
  }
</script>

<svelte:window on:keydown={handleKeydown} />

{#if isOpen}
  <!-- svelte-ignore a11y-click-events-have-key-events -->
  <!-- svelte-ignore a11y-no-static-element-interactions -->
  <div class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50" on:click={closeModal}>
    <!-- svelte-ignore a11y-click-events-have-key-events -->
    <!-- svelte-ignore a11y-no-static-element-interactions -->
    <div class="bg-white rounded-lg shadow-xl p-6 w-full max-w-md mx-4" on:click|stopPropagation>
      <div class="flex justify-between items-center mb-4">
        <h2 class="text-xl font-semibold text-gray-800">设置</h2>
        <button on:click={closeModal} class="text-gray-400 hover:text-gray-600">
          <svg width="24" height="24" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
          </svg>
        </button>
      </div>
      
      <div class="space-y-6">
        <!-- 字体颜色设置 -->
        <div>
          <h3 class="text-sm font-medium text-gray-700 mb-3">字体颜色</h3>
          
          <!-- 预设颜色 -->
          <div class="mb-3">
            <h4 class="text-xs text-gray-600 mb-2">预设颜色</h4>
            <div class="grid grid-cols-6 gap-2">
              {#each presetTextColors as color}
                <button 
                  on:click={() => selectTextColor(color)}
                  class="w-8 h-8 rounded-full border-2 hover:scale-110 transition-transform duration-200"
                  class:ring-2={customTextColor === color}
                  class:ring-blue-500={customTextColor === color}
                  class:ring-offset-2={customTextColor === color}
                  style="background-color: {color};"
                  title={color}
                />
              {/each}
            </div>
          </div>
          
          <!-- 自定义颜色 -->
          <div class="mb-3">
            <h4 class="text-xs text-gray-600 mb-2">自定义颜色</h4>
            <div class="flex gap-2 items-center">
              <input 
                type="color" 
                bind:value={customTextColor}
                on:change={handleCustomTextColor}
                class="w-10 h-8 rounded border border-gray-300 cursor-pointer"
                title="选择自定义颜色"
              />
              <input 
                type="text" 
                placeholder="#ffffff"
                bind:value={customTextColor}
                on:blur={handleCustomTextColor}
                class="flex-1 px-3 py-1.5 border border-gray-300 rounded text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
              />
              <button 
                on:click={handleCustomTextColor}
                class="px-3 py-1.5 bg-blue-500 text-white rounded text-sm hover:bg-blue-600 transition-colors"
              >
                应用
              </button>
            </div>
          </div>
          
          <!-- 颜色预览 -->
          <div class="p-3 bg-gray-50 rounded-lg">
            <p class="text-sm text-gray-600 mb-1">预览效果：</p>
            <p style="color: {customTextColor};" class="font-medium">
              这是文字颜色预览效果
            </p>
          </div>
        </div>

        <!-- 背景设置 -->
        <div>
          <h3 class="text-sm font-medium text-gray-700 mb-3">背景设置</h3>
          
          <!-- 预设背景 -->
          <div class="mb-3">
            <h4 class="text-xs text-gray-600 mb-2">预设背景</h4>
            <div class="grid grid-cols-3 gap-2">
              {#each presetBackgrounds as bg, index}
                <button 
                  on:click={() => selectBackground(bg)}
                  class="w-full h-16 rounded-lg border-2 hover:border-blue-300 transition-colors bg-cover bg-center"
                  class:border-blue-500={backgroundImage === bg}
                  style="background-image: url('{bg}')"
                  title="预设背景 {index + 1}"
                />
              {/each}
            </div>
          </div>
        
          <!-- 上传图片 -->
          <div class="mb-3">
            <h4 class="text-xs text-gray-600 mb-2">上传图片</h4>
            <input 
              type="file" 
              accept="image/*" 
              on:change={handleFileUpload}
              bind:this={fileInput}
              class="w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-full file:border-0 file:text-sm file:font-semibold file:bg-blue-50 file:text-blue-700 hover:file:bg-blue-100"
            />
          </div>
        
          <!-- 自定义URL -->
          <div>
            <h4 class="text-xs text-gray-600 mb-2">自定义URL</h4>
            <div class="flex gap-2">
              <input 
                type="url" 
                placeholder="输入图片URL"
                bind:value={customUrl}
                class="flex-1 px-3 py-2 border border-gray-300 rounded-md text-sm focus:outline-none focus:ring-2 focus:ring-blue-500"
              />
              <button 
                on:click={handleCustomUrl}
                class="px-4 py-2 bg-blue-500 text-white rounded-md text-sm hover:bg-blue-600 transition-colors"
              >
                应用
              </button>
            </div>
          </div>
        </div>
      </div>
      
      <div class="mt-6 flex justify-end">
        <button 
          on:click={closeModal}
          class="px-4 py-2 text-gray-600 hover:text-gray-800 transition-colors"
        >
          取消
        </button>
      </div>
    </div>
  </div>
{/if}
