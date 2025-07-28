<script>
  import { onMount } from 'svelte';
  import { browser } from '$app/environment';
  
  // Props with default values
  export let title = "Welcome to Our Platform";
  export let content = "This is a responsive placeholder text that will remain centered on the page and adjust to different screen sizes.";
  
  let innerWidth;
  let innerHeight;
</script>

<svelte:window bind:innerWidth bind:innerHeight />

<div class="text-container">
  <div class="content-wrapper">
    <h2>{title}</h2>
    <p>{content}</p>
  </div>
</div>

<style>
  .text-container {
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 100;
    pointer-events: none;
  }
  
  .content-wrapper {
    background: rgba(255, 255, 255, 0.95);
    backdrop-filter: blur(10px);
    border-radius: 12px;
    padding: 24px;
    max-width: 600px;
    width: calc(100% - 40px);
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
    border: 1px solid rgba(255, 255, 255, 0.2);
    pointer-events: auto;
    transition: all 0.3s ease;
    text-align: center;
  }
  
  h2 {
    margin: 0 0 16px 0;
    font-size: clamp(1.5rem, 5vw, 2.5rem);
    font-weight: 700;
    color: #1a1a1a;
    line-height: 1.2;
  }
  
  p {
    margin: 0;
    font-size: clamp(1rem, 3vw, 1.2rem);
    line-height: 1.6;
    color: #4a4a4a;
  }
  
  /* Responsive adjustments */
  @media (max-width: 1024px) {
    .content-wrapper {
      max-width: 500px;
      padding: 20px;
    }
  }
  
  @media (max-width: 768px) {
    .content-wrapper {
      max-width: none;
      width: calc(100% - 32px);
      padding: 20px;
    }
  }
  
  @media (max-width: 480px) {
    .content-wrapper {
      padding: 16px;
    }
    
    h2 {
      font-size: clamp(1.125rem, 4vw, 1.5rem);
      margin-bottom: 10px;
    }
    
    p {
      font-size: clamp(0.875rem, 2.5vw, 1rem);
    }
  }
</style>