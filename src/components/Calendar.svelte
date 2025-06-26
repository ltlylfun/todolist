<script>
  import { createEventDispatcher } from 'svelte'
  import TodoItem from './TodoItem.svelte'
  
  export let todos = []
  
  const dispatch = createEventDispatcher()
  
  let currentDate = new Date()
  let selectedDate = new Date()
  
  // 获取当前月份的日历网格数据
  function getCalendarGrid(date) {
    const year = date.getFullYear();
    const month = date.getMonth();
    
    // 获取当月第一天和最后一天
    const firstDay = new Date(year, month, 1);
    const lastDay = new Date(year, month + 1, 0);
    
    // 获取第一天是星期几（0=周日，1=周一...）
    const startDay = firstDay.getDay();
    
    // 创建日历网格
    const days = [];
    
    // 添加上个月的日期（灰色显示）
    const prevMonth = new Date(year, month - 1, 0);
    for (let i = startDay - 1; i >= 0; i--) {
      days.push({
        date: new Date(year, month - 1, prevMonth.getDate() - i),
        isCurrentMonth: false,
        isToday: false,
        isSelected: false
      });
    }
    
    // 添加当月的日期
    for (let day = 1; day <= lastDay.getDate(); day++) {
      const dayDate = new Date(year, month, day);
      const today = new Date();
      days.push({
        date: dayDate,
        isCurrentMonth: true,
        isToday: isSameDay(dayDate, today),
        isSelected: isSameDay(dayDate, selectedDate)
      });
    }
    
    // 添加下个月的日期（填满42格，6行7列）
    const remainingDays = 42 - days.length;
    for (let day = 1; day <= remainingDays; day++) {
      days.push({
        date: new Date(year, month + 1, day),
        isCurrentMonth: false,
        isToday: false,
        isSelected: false
      });
    }
    
    return days;
  }
  
  function isSameDay(date1, date2) {
    return date1.getFullYear() === date2.getFullYear() &&
           date1.getMonth() === date2.getMonth() &&
           date1.getDate() === date2.getDate();
  }
  
  function formatDate(date) {
    return date.toLocaleDateString('zh-CN', {
      year: 'numeric',
      month: 'long'
    });
  }
  
  function formatSelectedDate(date) {
    return date.toLocaleDateString('zh-CN', {
      year: 'numeric',
      month: 'long',
      day: 'numeric'
    });
  }
  
  function getTodosForDate(date) {
    return todos.filter(todo => {
      if (!todo.dueDate) return false;
      const todoDate = new Date(todo.dueDate);
      return isSameDay(todoDate, date);
    });
  }
  
  function getTodoCountForDate(date) {
    return getTodosForDate(date).length;
  }
  
  function selectDate(day) {
    if (day.isCurrentMonth) {
      selectedDate = new Date(day.date);
    }
  }
  
  function prevMonth() {
    currentDate = new Date(currentDate.getFullYear(), currentDate.getMonth() - 1, 1);
  }
  
  function nextMonth() {
    currentDate = new Date(currentDate.getFullYear(), currentDate.getMonth() + 1, 1);
  }
  
  function goToToday() {
    currentDate = new Date();
    selectedDate = new Date();
  }
  
  // 任务操作函数
  function toggleTodo(id) {
    dispatch('toggle', id);
  }
  
  function deleteTodo(id) {
    dispatch('delete', id);
  }
  
  function editTodo(id, newText) {
    dispatch('edit', { id, text: newText });
  }
  
  $: calendarDays = getCalendarGrid(currentDate);
  $: selectedDateTodos = getTodosForDate(selectedDate);
  
  const weekdays = ['日', '一', '二', '三', '四', '五', '六'];
</script>

<div class="bg-white bg-opacity-10 backdrop-blur-lg rounded-2xl shadow-xl border border-white border-opacity-20 p-6 md:p-8 animate-slideIn">
  <!-- 日历标题和导航 -->
  <div class="flex items-center justify-between mb-6">
    <h2 class="text-2xl font-bold" style="color: inherit;">
      {formatDate(currentDate)}
    </h2>
    <div class="flex items-center gap-2">
      <button
        on:click={goToToday}
        class="btn-secondary text-sm px-3 py-1"
      >
        今天
      </button>
      <button
        on:click={prevMonth}
        class="p-2 rounded-lg bg-white bg-opacity-10 hover:bg-opacity-20 transition-all"
      >
        <svg class="w-5 h-5" style="color: inherit;" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7"></path>
        </svg>
      </button>
      <button
        on:click={nextMonth}
        class="p-2 rounded-lg bg-white bg-opacity-10 hover:bg-opacity-20 transition-all"
      >
        <svg class="w-5 h-5" style="color: inherit;" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"></path>
        </svg>
      </button>
    </div>
  </div>
  
  <!-- 星期标题 -->
  <div class="grid grid-cols-7 gap-1 mb-2">
    {#each weekdays as weekday}
      <div class="text-center text-sm font-medium opacity-70 py-2" style="color: inherit;">
        {weekday}
      </div>
    {/each}
  </div>
  
  <!-- 日历网格 -->
  <div class="grid grid-cols-7 gap-1 mb-6">
    {#each calendarDays as day}
      <button
        class="relative aspect-square p-1 rounded-lg transition-all duration-200 text-sm
               {day.isCurrentMonth 
                 ? 'hover:bg-white hover:bg-opacity-10' 
                 : 'opacity-30'}
               {day.isToday 
                 ? 'bg-blue-500 bg-opacity-50 font-bold' 
                 : ''}
               {day.isSelected && day.isCurrentMonth 
                 ? 'bg-white bg-opacity-20 ring-2 ring-white ring-opacity-50' 
                 : ''}"
        style="color: inherit;"
        on:click={() => selectDate(day)}
      >
        <div class="flex flex-col items-center justify-center h-full">
          <span>{day.date.getDate()}</span>
          {#if getTodoCountForDate(day.date) > 0}
            <div class="w-1.5 h-1.5 bg-red-400 rounded-full mt-0.5"></div>
          {/if}
        </div>
      </button>
    {/each}
  </div>
  
  <!-- 选中日期的任务 -->
  <div class="border-t border-white border-opacity-20 pt-6">
    <h3 class="text-lg font-semibold mb-4" style="color: inherit;">
      {formatSelectedDate(selectedDate)} 的任务
    </h3>
    
    {#if selectedDateTodos.length > 0}
      <div class="space-y-2">
        {#each selectedDateTodos as todo (todo.id)}
          <TodoItem 
            {todo} 
            on:toggle={() => toggleTodo(todo.id)}
            on:delete={() => deleteTodo(todo.id)}
            on:edit={e => editTodo(todo.id, e.detail)}
          />
        {/each}
      </div>
    {:else}
      <div class="text-center py-8 opacity-70" style="color: inherit;">
        这一天还没有任务
      </div>
    {/if}
  </div>
</div>
