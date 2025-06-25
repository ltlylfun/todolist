<script>
  import { createEventDispatcher } from 'svelte'
  
  export let filter
  export let stats
  
  const dispatch = createEventDispatcher()
  
  const filters = [
    { key: 'all', label: '全部', count: stats.total },
    { key: 'active', label: '进行中', count: stats.active },
    { key: 'completed', label: '已完成', count: stats.completed }
  ]
</script>

<div class="mb-6">
  <div class="flex flex-wrap items-center justify-between gap-4">
    <!-- 过滤选项卡 -->
    <div class="flex bg-white bg-opacity-10 backdrop-blur-sm rounded-lg p-1 border border-current border-opacity-20">
      {#each filters as filterOption}
        <button
          class="px-4 py-2 rounded-md text-sm font-medium transition-all duration-200
                 {filter === filterOption.key 
                   ? 'bg-white bg-opacity-20 shadow-sm' 
                   : 'opacity-70 hover:opacity-100'}"
          style="color: inherit;"
          on:click={() => filter = filterOption.key}
        >
          {filterOption.label}
          <span class="ml-1 text-xs bg-white bg-opacity-20 px-1.5 py-0.5 rounded-full
                       {filter === filterOption.key ? 'bg-white bg-opacity-30' : ''}"
                style="color: inherit;">
            {filterOption.count}
          </span>
        </button>
      {/each}
    </div>
    
    <!-- 清除已完成按钮 -->
    {#if stats.completed > 0}
      <button
        on:click={() => dispatch('clearCompleted')}
        class="text-sm font-medium transition-colors opacity-70 hover:opacity-100 hover:text-red-400"
        style="color: inherit;"
      >
        清除已完成 ({stats.completed})
      </button>
    {/if}
  </div>
</div>
