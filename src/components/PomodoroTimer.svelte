
<script>
  import { onMount, onDestroy } from 'svelte'
  
  let isRunning = false
  let isPaused = false
  let timeLeft = 25 * 60 
  let totalTime = 25 * 60
  let customMinutes = 25
  let interval = null
  let audio = null
  

  let notificationPermission = 'default'
  
  onMount(async () => {
    const savedMinutes = localStorage.getItem('pomodoro-minutes')
    if (savedMinutes) {
      customMinutes = parseInt(savedMinutes)
      timeLeft = customMinutes * 60
      totalTime = customMinutes * 60
    }
    
    if ('Notification' in window) {
      try {
        notificationPermission = await Notification.requestPermission()
        console.log('通知权限状态:', notificationPermission)
      } catch (error) {
        console.log('请求通知权限失败:', error)
      }
    }
    
    try {
      audio = new Audio('./4181.wav')
      audio.preload = 'auto'
      audio.volume = 0.8
      console.log('音频初始化成功')
    } catch (error) {
      console.log('音频初始化失败:', error)
    }
  })
  
  onDestroy(() => {
    if (interval) {
      clearInterval(interval)
    }
  })
  
  function toggleTimer() {
    if (isRunning) {
      pause()
    } else {
      start()
    }
  }
  
  function start() {

    isRunning = true
    isPaused = false
    
    interval = setInterval(() => {
      timeLeft--
      
      if (timeLeft <= 0) {
        complete()
      }
    }, 1000)
  }
  
  function pause() {
    isRunning = false
    isPaused = true
    if (interval) {
      clearInterval(interval)
      interval = null
    }
  }
  
  function reset() {
    isRunning = false
    isPaused = false
    timeLeft = totalTime
    if (interval) {
      clearInterval(interval)
      interval = null
    }
  }
  
  function complete() {
    isRunning = false
    isPaused = false
    timeLeft = 0
    
    if (interval) {
      clearInterval(interval)
      interval = null
    }
    
    console.log('番茄钟时间到！开始播放音频和发送通知...')
    

    if (audio) {
      audio.currentTime = 0 
      audio.loop = true 
      audio.play()
        .then(() => {
          console.log('音频播放成功')
          window.focus()
          
          const confirmed = confirm('🍅 番茄钟时间到！\n\n恭喜你完成了一个番茄钟周期！\n是时候休息一下了。\n\n点击确定停止提示音。')
          if (confirmed) {
            audio.pause()
            audio.currentTime = 0
          }
        })
        .catch(e => {
          console.log('音频播放失败:', e)
          window.focus()
          alert('🍅 番茄钟时间到！\n\n恭喜你完成了一个番茄钟周期！\n是时候休息一下了。')
        })
    } else {
      console.log('音频对象未初始化')
      window.focus()
      alert('🍅 番茄钟时间到！\n\n恭喜你完成了一个番茄钟周期！\n是时候休息一下了。')
    }
    
    if (notificationPermission === 'granted') {
      try {
        const notification = new Notification('🍅 番茄钟时间到！', {
          body: '恭喜你完成了一个番茄钟周期！是时候休息一下了。点击此通知返回页面。',
          icon: './todo-icon.svg',
          badge: './todo-icon.svg',
          tag: 'pomodoro-complete',
          requireInteraction: true,
          silent: false
        })
        
        notification.onclick = () => {
          window.focus()
          notification.close()
          if (audio && !audio.paused) {
            const shouldStop = confirm('🍅 番茄钟时间到！\n\n恭喜你完成了一个番茄钟周期！\n是时候休息一下了。\n\n点击确定停止提示音。')
            if (shouldStop) {
              audio.pause()
              audio.currentTime = 0
            }
          }
        }
        
        setTimeout(() => {
          notification.close()
        }, 10000)
        
        console.log('通知发送成功')
      } catch (error) {
        console.log('发送通知失败:', error)
      }
    } else {
      console.log('通知权限未授予，当前状态:', notificationPermission)
    }
    
    let originalTitle = document.title
    let flashCount = 0
    const flashTitle = setInterval(() => {
      document.title = flashCount % 2 === 0 ? '🍅 时间到！' : originalTitle
      flashCount++
      if (flashCount >= 20) { 
        clearInterval(flashTitle)
        document.title = originalTitle
      }
    }, 500)
  }
  
  function setCustomTime() {
    if (customMinutes >= 1 && customMinutes <= 999) {
      totalTime = customMinutes * 60
      timeLeft = totalTime
      localStorage.setItem('pomodoro-minutes', customMinutes.toString())
    }
  }
  
  function stopAudio() {
    if (audio && !audio.paused) {
      audio.pause()
      audio.currentTime = 0
      console.log('音频已手动停止')
    }
  }
  

  function formatTime(seconds) {
    const minutes = Math.floor(seconds / 60)
    const remainingSeconds = seconds % 60
    return `${minutes.toString().padStart(2, '0')}:${remainingSeconds.toString().padStart(2, '0')}`
  }
  

  $: progress = totalTime > 0 ? ((totalTime - timeLeft) / totalTime) * 100 : 0
  

  $: strokeDasharray = 2 * Math.PI * 90 
  $: strokeDashoffset = strokeDasharray - (strokeDasharray * progress) / 100
</script>

<div class="pomodoro-timer bg-white bg-opacity-10 backdrop-blur-sm rounded-2xl p-6 border border-white border-opacity-20">
  <h3 class="text-2xl font-bold text-center mb-6" style="color: inherit;">🍅 番茄钟</h3>
  
  <!-- 圆形进度条和时间显示 -->
  <div class="relative flex items-center justify-center mb-6">
    <svg class="w-48 h-48 transform -rotate-90" viewBox="0 0 200 200">
      <!-- 背景圆 -->
      <circle
        cx="100"
        cy="100"
        r="90"
        stroke="currentColor"
        stroke-width="8"
        fill="none"
        class="opacity-20"
      />
      <!-- 进度圆 -->
      <circle
        cx="100"
        cy="100"
        r="90"
        stroke="currentColor"
        stroke-width="8"
        fill="none"
        class="transition-all duration-1000 ease-in-out"
        style="stroke-dasharray: {strokeDasharray}; stroke-dashoffset: {strokeDashoffset};"
        stroke-linecap="round"
      />
    </svg>
    
    <!-- 时间显示 -->
    <div class="absolute inset-0 flex flex-col items-center justify-center">
      <div class="text-4xl font-mono font-bold mb-2" style="color: inherit;">
        {formatTime(timeLeft)}
      </div>
      <div class="text-sm opacity-70" style="color: inherit;">
        {#if timeLeft <= 0}
          时间到！
        {:else if isRunning}
          运行中
        {:else if isPaused}
          已暂停
        {:else}
          准备开始
        {/if}
      </div>
    </div>
  </div>
  
  <!-- 控制按钮 -->
  <div class="flex justify-center space-x-4 mb-6">
    <button
      class="px-6 py-3 bg-white bg-opacity-20 hover:bg-opacity-30 rounded-lg transition-all duration-200 font-medium"
      style="color: inherit;"
      on:click={toggleTimer}
      disabled={timeLeft <= 0}
    >
      {#if isRunning}
        暂停
      {:else if isPaused}
        继续
      {:else}
        开始
      {/if}
    </button>
    
    <button
      class="px-6 py-3 bg-white bg-opacity-20 hover:bg-opacity-30 rounded-lg transition-all duration-200 font-medium"
      style="color: inherit;"
      on:click={reset}
    >
      重置
    </button>
    
    {#if timeLeft <= 0}
      <button
        class="px-6 py-3 bg-red-500 bg-opacity-30 hover:bg-opacity-50 rounded-lg transition-all duration-200 font-medium"
        style="color: inherit;"
        on:click={stopAudio}
      >
        停止音频
      </button>
    {/if}
  </div>
  
  <!-- 时间设置 -->
  <div class="border-t border-white border-opacity-20 pt-4">
    <div class="flex items-center justify-center space-x-4">
      <label for="custom-minutes" class="text-sm font-medium" style="color: inherit;">自定义时间：</label>
      <input
        id="custom-minutes"
        type="number"
        min="1"
        max="999"
        bind:value={customMinutes}
        class="w-20 px-3 py-2 bg-white bg-opacity-20 border border-white border-opacity-30 rounded-lg text-center font-mono"
        style="color: inherit;"
        placeholder="25"
        disabled={isRunning}
      />
      <span class="text-sm" style="color: inherit;">分钟</span>
      <button
        class="px-4 py-2 bg-white bg-opacity-20 hover:bg-opacity-30 rounded-lg transition-all duration-200 text-sm font-medium"
        style="color: inherit;"
        on:click={setCustomTime}
        disabled={isRunning}
      >
        设置
      </button>
    </div>
    
    {#if customMinutes < 1 || customMinutes > 999}
      <p class="text-center text-sm mt-2 opacity-70" style="color: inherit;">
        请输入1-999之间的数字
      </p>
    {/if}
  </div>
  

</div>

<style>
  .pomodoro-timer {
    max-width: 400px;
    margin: 0 auto;
  }
  
  input[type="number"] {
    appearance: textfield;
    -moz-appearance: textfield;
  }
  
  input[type="number"]::-webkit-outer-spin-button,
  input[type="number"]::-webkit-inner-spin-button {
    -webkit-appearance: none;
    margin: 0;
  }
  
  input::placeholder {
    color: inherit;
    opacity: 0.5;
  }
  
  button:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
</style>
