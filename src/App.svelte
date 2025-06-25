<script>
  import { onMount } from 'svelte'
  import TodoList from './components/TodoList.svelte'
  import Header from './components/Header.svelte'
  import Footer from './components/Footer.svelte'
  import Settings from './components/Settings.svelte'
  
  let showSettings = false
  let backgroundImage = ''
  let textColor = '#ffffff'
  
  const defaultBackground = 'https://images.unsplash.com/photo-1506905925346-21bda4d32df4?ixlib=rb-4.0.3&auto=format&fit=crop&w=2070&q=80'
  
  onMount(() => {
    const savedBackground = localStorage.getItem('todolist-background')
    if (savedBackground) {
      backgroundImage = savedBackground
    } else {
      backgroundImage = defaultBackground
      localStorage.setItem('todolist-background', backgroundImage)
    }
    
    const savedTextColor = localStorage.getItem('todolist-textcolor')
    if (savedTextColor) {
      textColor = savedTextColor
    }
  })
  
  function handleOpenSettings() {
    showSettings = true
  }
  
  function handleCloseSettings() {
    showSettings = false
  }
  
  function handleBackgroundChange(event) {
    backgroundImage = event.detail.url
    localStorage.setItem('todolist-background', backgroundImage)
  }
  
  function handleTextColorChange(event) {
    textColor = event.detail.color
    localStorage.setItem('todolist-textcolor', textColor)
  }
  
  $: backgroundStyle = backgroundImage 
    ? `background-image: url('${backgroundImage}'); background-size: cover; background-position: center; background-attachment: fixed;`
    : ''
    
  $: textColorStyle = `color: ${textColor};`
</script>

<main 
  class="min-h-screen flex flex-col"
  class:bg-gradient-to-br={!backgroundImage}
  class:from-blue-50={!backgroundImage}
  class:to-indigo-100={!backgroundImage}
  style={backgroundStyle}
>
  <div class="container mx-auto px-4 py-8 max-w-2xl flex-1 flex flex-col" 
       class:bg-black={backgroundImage}
       class:bg-opacity-20={backgroundImage}
       class:bg-white={!backgroundImage}
       class:bg-opacity-10={!backgroundImage}
       class:backdrop-blur-lg={!backgroundImage}
       style={textColorStyle}>
    <Header on:openSettings={handleOpenSettings} />
    <div class="flex-1">
      <TodoList />
    </div>
    <Footer />
  </div>
</main>

<Settings 
  bind:isOpen={showSettings}
  {backgroundImage}
  {textColor}
  on:close={handleCloseSettings}
  on:backgroundChange={handleBackgroundChange}
  on:textColorChange={handleTextColorChange}
/>
