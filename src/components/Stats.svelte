<script>
  export let stats
  
  $: completionPercentage = stats.total > 0 ? Math.round((stats.completed / stats.total) * 100) : 0
</script>

{#if stats.total > 0}
  <div class="mb-6 bg-white bg-opacity-10 backdrop-blur-sm rounded-lg p-4 border border-current border-opacity-20">
    <div class="flex items-center justify-between mb-3">
      <h3 class="text-lg font-semibold">任务概览</h3>
      <div class="text-2xl font-bold opacity-80">
        {completionPercentage}%
      </div>
    </div>
    
    <!-- 进度条 -->
    <div class="mb-3">
      <div class="flex justify-between text-sm opacity-80 mb-1">
        <span>完成进度</span>
        <span>{stats.completed} / {stats.total}</span>
      </div>
      <div class="w-full bg-white bg-opacity-20 rounded-full h-2">
        <div 
          class="bg-gradient-to-r from-blue-400 to-blue-500 h-2 rounded-full transition-all duration-500 ease-out"
          style="width: {completionPercentage}%"
        ></div>
      </div>
    </div>
    
    <!-- 统计卡片 -->
    <div class="grid grid-cols-3 gap-3">
      <div class="text-center p-2 bg-white bg-opacity-10 backdrop-blur-sm rounded-lg border border-current border-opacity-20">
        <div class="text-lg font-bold" style="color: inherit;">{stats.total}</div>
        <div class="text-xs opacity-70">总任务</div>
      </div>
      <div class="text-center p-2 bg-white bg-opacity-10 backdrop-blur-sm rounded-lg border border-current border-opacity-20">
        <div class="text-lg font-bold text-orange-300">{stats.active}</div>
        <div class="text-xs opacity-70">进行中</div>
      </div>
      <div class="text-center p-2 bg-white bg-opacity-10 backdrop-blur-sm rounded-lg border border-current border-opacity-20">
        <div class="text-lg font-bold text-green-300">{stats.completed}</div>
        <div class="text-xs opacity-70">已完成</div>
      </div>
    </div>
  </div>
{/if}
