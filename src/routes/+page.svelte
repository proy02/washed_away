
<script>
  import { tweened } from 'svelte/motion';
  import { cubicOut } from 'svelte/easing';
  import { onMount } from 'svelte';
  import { browser } from '$app/environment';
  
  let innerWidth;
  let innerHeight;
  
  //lazy loading
  let highResImage;
  let isHighResLoaded = false;
  
  function setupProgressiveLoading() {
    if (!browser || !highResImage) return;
    
    // Start loading high-res after a delay
    setTimeout(() => {
      highResImage.style.transition = 'opacity 0.8s ease-in-out';
      highResImage.style.opacity = '1';
      isHighResLoaded = true;
    }, 1000); // Wait 1 second, then fade in high-res
  }
  
  // Performance measurement variables
  let pageLoadStartTime;
  let imageLoadStart;
  let imageLoadEnd;
  
  // Only start timing in the browser, not during SSR
  if (browser) {
    pageLoadStartTime = performance.now();
  }
  
  const views = [
    { 
      name: 'India', 
      viewBox: [0, 0, 6000, 6750], 
      labelVisible: false,
      adjustments: {
        mobile: { offsetX: 0, offsetY: 800, scale: 1.0 },
        tablet: { offsetX: 0, offsetY: 0, scale: 1.0 },
        desktop: { offsetX: 0, offsetY: 0, scale: 1.0 }
      },
      info: {
        title: "India",
        content: `Microsoft has just released a global dataset on floods and India stands out as the biggest hotspot over the past 10 years. More than <span style="background-color: #ca0020; color: white; padding: 4px;">0.3 million square kilometres (sq km) or 10%</span> of the country's total land area experienced some form of flooding between 2014 and 2024.<br><strong style="color: #888; font-size: 14px; margin-top: 12px; display: block; opacity: 1; transition: opacity 0.3s; text-align: center;">Keep scrolling ↓</strong>`
      }
    },
    { 
      name: 'Punjab+Haryana', 
      viewBox: [900, 1150, 1958, 1100], 
      labelVisible: true,
      adjustments: {
        mobile: { offsetX: 0, offsetY: 200, scale: 1.0 },
        tablet: { offsetX: 0, offsetY: 0, scale: 1.0 },
        desktop: { offsetX: 0, offsetY: 0, scale: 1.0 }
      },
      info: {
        title: "Punjab & Haryana",
        content: `For these two northwestern states, the findings are tricky. Both rely heavily on large-scale paddy cultivation, which involves deliberately flooding vast tracts of farmland. As a result, the data reflects agricultural flooding rather than natural inundation. Even so, the numbers are astounding: In Punjab, <span style="background-color: #ca0020; color: white; padding: 4px;">68%</span> of the total area registered as flooded, while Haryana recorded <span style="background-color: #ca0020; color: white; padding: 4px;">38%</span>.<br><strong style="color: #888; font-size: 14px; margin-top: 12px; display: block; opacity: 1; transition: opacity 0.3s; text-align: center;">Keep scrolling ↓</strong>`
      }
    },
    { 
      name: 'Uttar Pradesh', 
      viewBox: [1800, 1800, 1958, 1100], 
      labelVisible: true,
      adjustments: {
        mobile: { offsetX: -90, offsetY: 0, scale: 0.8 },
        tablet: { offsetX: -150, offsetY: 0, scale: 0.95 },
        desktop: { offsetX: -100, offsetY: 0, scale: 1.0 }
      },
      info: {
        title: "Uttar Pradesh",
        content: `India's most populous state also recorded the largest flood-affected area in absolute figures: close to <span style="background-color: #ca0020; color: white; padding: 4px;">60,000 sq km</span> or around <span style="background-color: #ca0020; color: white; padding: 4px;">25%</span> of its total area.<br><strong style="color: #888; font-size: 14px; margin-top: 12px; display: block; opacity: 1; transition: opacity 0.3s; text-align: center;">Keep scrolling ↓</strong>`
      }
    },
    { 
      name: 'Bihar', 
      viewBox: [3100, 2200, 1246, 700], 
      labelVisible: true,
      adjustments: {
        mobile: { offsetX: -100, offsetY: 100, scale: 0.8 },
        tablet: { offsetX: 0, offsetY: 0, scale: 1.15 },
        desktop: { offsetX: 0, offsetY: 0, scale: 1.1 }
      },
      info: {
        title: "Bihar",
        content: `Among India's most flood-prone and socio-economically vulnerable states, over <span style="background-color: #ca0020; color: white; padding: 4px;">50%</span> of Bihar's land area has reported natural inundation, underlining the widespread destruction experienced by the state.<br><strong style="color: #888; font-size: 14px; margin-top: 12px; display: block; opacity: 1; transition: opacity 0.3s; text-align: center;">Keep scrolling ↓</strong>`
      }
    },
    { 
      name: 'West Bengal', 
      viewBox: [3300, 2800, 1246, 700], 
      labelVisible: true,
      adjustments: {
        mobile: { offsetX: 80, offsetY: -150, scale: 0.7 },
        tablet: { offsetX: 0, offsetY: 0, scale: 1.0 },
        desktop: { offsetX: 0, offsetY: -180, scale: 0.7 }
      },
      info: {
        title: "West Bengal",
        content: `The final stretch of the Indo-Gangetic Plain before the Ganga River flows into Bangladesh, West Bengal has seen <span style="background-color: #ca0020; color: white; padding: 4px;">37%</span> of its territory affected by flooding.<br><strong style="color: #888; font-size: 14px; margin-top: 12px; display: block; opacity: 1; transition: opacity 0.3s; text-align: center;">Keep scrolling ↓</strong>`
      }
    },
    { 
      name: 'Assam', 
      viewBox: [4200, 2250, 1246, 700], 
      labelVisible: true,
      adjustments: {
        mobile: { offsetX: 120, offsetY: -150, scale: 0.6 },
        tablet: { offsetX: 0, offsetY: 0, scale: 1.0 },
        desktop: { offsetX: 0, offsetY: -180, scale: 0.7 }
      },
      info: {
        title: "Assam",
        content: `Tucked into the foothills of the Eastern Himalayas and lying at the mercy of the raging Brahmaputra River, Assam has long lived under the threat of seasonal inundation. Around <span style="background-color: #ca0020; color: white; padding: 4px;">30%</span> of its area has been flooded at least once in the past decade, with regular events reported in April.<br><strong style="color: #888; font-size: 14px; margin-top: 12px; display: block; opacity: 1; transition: opacity 0.3s; text-align: center;">Keep scrolling ↓</strong>`
      }
    }
  ];
  
  let currentStep = 0;
  let isScrollytellingActive = true;
  let showInfoPanel = true;
  let lastScrollDirection = 'down';
  
  const DURATION = 1000;
  
  function getResponsiveViewBox(viewBoxArray, windowWidth, windowHeight, step) {
    const [x, y, width, height] = viewBoxArray;
    
    // Determine screen category
    let screenCategory = 'desktop';
    if (windowWidth <= 480) {
      screenCategory = 'mobile';
    } else if (windowWidth <= 768) {
      screenCategory = 'mobile';
    } else if (windowWidth <= 1024) {
      screenCategory = 'tablet';
    } else {
      screenCategory = 'desktop';
    }
    
    // Get adjustments for this step and screen size
    const adjustments = views[step].adjustments?.[screenCategory] || { offsetX: 0, offsetY: 0, scale: 1.0 };
    
    // For India overview (step 0), apply adjustments without zoom
    if (step === 0) {
      const adjustedX = x + adjustments.offsetX;
      const adjustedY = y + adjustments.offsetY;
      const result = adjustForAspectRatio(adjustedX, adjustedY, width, height, windowWidth, windowHeight);
      return result;
    }
    
    // Get base zoom factor for non-India steps
    let zoomFactor = 1;
    
    if (windowWidth <= 480) {
      zoomFactor = 2.0;
    } else if (windowWidth <= 768) {
      zoomFactor = 1.6;
    } else if (windowWidth <= 1024) {
      zoomFactor = 1.3;
    } else {
      zoomFactor = 1.0;
    }
    
    // Apply custom scale to zoom factor
    const finalZoomFactor = zoomFactor * adjustments.scale;
    
    // Apply zoom by reducing viewBox dimensions (zooms in)
    const zoomedWidth = width / finalZoomFactor;
    const zoomedHeight = height / finalZoomFactor;
    
    // Calculate center with custom offset
    const centerX = x + width / 2 + adjustments.offsetX;
    const centerY = y + height / 2 + adjustments.offsetY;
    
    const newX = centerX - zoomedWidth / 2;
    const newY = centerY - zoomedHeight / 2;
    
    // Now adjust for aspect ratio
    const result = adjustForAspectRatio(newX, newY, zoomedWidth, zoomedHeight, windowWidth, windowHeight);
    
    return result;
  }
  
  // Helper function to adjust viewBox for screen aspect ratio
  function adjustForAspectRatio(x, y, width, height, windowWidth, windowHeight) {
    const viewBoxAspectRatio = width / height;
    const windowAspectRatio = windowWidth / windowHeight;
    
    let result;
    // If window is wider than viewBox, expand width to fit
    if (windowAspectRatio > viewBoxAspectRatio) {
      const adjustedWidth = height * windowAspectRatio;
      const widthDiff = adjustedWidth - width;
      const newX = x - widthDiff / 2;
      result = [newX, y, adjustedWidth, height];
    } 
    // If window is taller than viewBox, expand height to fit
    else {
      const adjustedHeight = width / windowAspectRatio;
      const heightDiff = adjustedHeight - height;
      const newY = y - heightDiff / 2;
      result = [x, newY, width, adjustedHeight];
    }
    
    return result;
  }
  
  const viewBoxStore = tweened(views[0].viewBox, {
    duration: DURATION,
    easing: cubicOut
  });
  
  let viewBoxString = '';
  
  viewBoxStore.subscribe(value => {
    viewBoxString = value.map(v => v.toFixed(2)).join(' ');
  });
  
  function updateViewBox(step, width, height) {
    const originalViewBox = views[step].viewBox;
    const responsiveViewBox = getResponsiveViewBox(originalViewBox, width, height, step);
    viewBoxStore.set(responsiveViewBox);
  }
  
  function onScroll() {
    const scrollY = window.scrollY;
    const height = window.innerHeight;
    const totalScrollytellingHeight = (views.length + 1) * height;
    
    // Determine scroll direction with better tracking
    const lastScrollY = window.lastScrollY || 0;
    const scrollDirection = scrollY > lastScrollY ? 'down' : 'up';
    const scrollDelta = Math.abs(scrollY - lastScrollY);
    
    // Store last scroll position for direction detection
    window.lastScrollY = scrollY;
    lastScrollDirection = scrollDirection;
    
    if (scrollY < totalScrollytellingHeight) {
      // We're in the scrollytelling section
      const wasScrollytellingActive = isScrollytellingActive;
      isScrollytellingActive = true;
      
      // Calculate the step based on scroll position
      const rawStep = Math.floor(scrollY / height);
      const calculatedStep = Math.min(views.length - 1, Math.max(0, rawStep));
      
      // Improved step changing - allow direct step changes when scrolling up from end
      let newStep;
      
      // If we're coming back from the end (scrolling up from outside scrollytelling area)
      if (!wasScrollytellingActive && scrollDirection === 'up') {
        // Allow jumping to the calculated step directly
        newStep = calculatedStep;
      } else {
        // Normal step progression - prevent excessive jumping
        if (scrollDirection === 'up' && calculatedStep < currentStep - 1) {
          // When scrolling up, limit to one step back unless there's a big scroll delta (fast scroll)
          newStep = scrollDelta > height * 0.5 ? calculatedStep : Math.max(0, currentStep - 1);
        } else if (scrollDirection === 'down' && calculatedStep > currentStep + 1) {
          // When scrolling down, limit to one step forward unless there's a big scroll delta
          newStep = scrollDelta > height * 0.5 ? calculatedStep : Math.min(views.length - 1, currentStep + 1);
        } else {
          newStep = calculatedStep;
        }
      }
      
      // Ensure step is within bounds
      newStep = Math.min(views.length - 1, Math.max(0, newStep));
      
      // Update step if changed
      if (newStep !== currentStep) {
        currentStep = newStep;
        updateViewBox(currentStep, window.innerWidth, window.innerHeight);
      }
      
      // Info panel visibility logic
      const lastStepThreshold = views.length * height; // Start of the extra viewport
      const newShowInfoPanel = scrollY < lastStepThreshold;
      showInfoPanel = newShowInfoPanel;
      
    } else {
      // We've scrolled past the scrollytelling section
      isScrollytellingActive = false;
      showInfoPanel = false;
    }
  }
  
  // Throttled resize handler for better mobile performance
  let resizeTimeout;
  function onResize() {
    clearTimeout(resizeTimeout);
    resizeTimeout = setTimeout(() => {
      updateViewBox(currentStep, window.innerWidth, window.innerHeight);
    }, 100); // Debounce resize events
  }
  
  // Monitor the loading time of the large image
  function monitorLargeImageLoad() {
    if (!browser) return;
    
    imageLoadStart = performance.now();
    
    // Try to find the SVG and image element
    setTimeout(() => {
      const svgElement = document.querySelector('.svg-wrapper svg');
      if (svgElement) {
        const imageElement = 
          svgElement.querySelector('image[xlink\\:href="large_image.png"]') || 
          svgElement.querySelector('image[href="large_image.png"]') ||
          svgElement.querySelector('image');
        
        if (imageElement) {
          // Check if already loaded
          if (imageElement.complete) {
            imageLoadEnd = performance.now();
            const loadTime = imageLoadEnd - imageLoadStart;
          } else {
            // Add load event listener
            imageElement.addEventListener('load', () => {
              imageLoadEnd = performance.now();
              const loadTime = imageLoadEnd - imageLoadStart;
            });
            
            // Add error event listener
            imageElement.addEventListener('error', () => {
              // Error handling
            });
          }
        }
      }
    }, 0);
  }
  
  // Use Performance API to get detailed resource loading metrics
  function checkResourcePerformance() {
    if (!browser || !window.performance || !window.performance.getEntriesByType) return;
    
    setTimeout(() => {
      const resources = window.performance.getEntriesByType('resource');
      
      // Look for the large PNG image
      const largeImage = resources.find(r => 
        r.name.includes('large_image.png') || 
        r.name.includes('large_image')
      );
      
      if (largeImage) {
        const totalLoadTime = largeImage.responseEnd - largeImage.startTime;
        const networkTime = largeImage.responseStart - largeImage.startTime;
        const downloadTime = largeImage.responseEnd - largeImage.responseStart;
        const size = largeImage.transferSize ? (largeImage.transferSize / (1024 * 1024)).toFixed(2) : 'unknown';
      }
    }, 200);
  }
  
  onMount(() => {
    if (!browser) return;
    
    // Start monitoring the large image load
    monitorLargeImageLoad();
    setupProgressiveLoading();
    
    // Use passive scroll listeners for better mobile performance
    window.addEventListener('scroll', onScroll, { passive: true });
    window.addEventListener('resize', onResize, { passive: true });
  
    // Initial update
    updateViewBox(currentStep, window.innerWidth, window.innerHeight);
    
    // Monitor when DOM is fully loaded
    if (document.readyState === 'loading') {
      document.addEventListener('DOMContentLoaded', () => {
        // DOM loaded
      });
    }
    
    // Monitor when all resources (images, SVGs) are loaded
    if (document.readyState === 'complete') {
      const loadTime = performance.now() - pageLoadStartTime;
      checkResourcePerformance();
    } else {
      window.addEventListener('load', () => {
        const loadTime = performance.now() - pageLoadStartTime;
        checkResourcePerformance();
      });
    }
    
    return () => {
      window.removeEventListener('scroll', onScroll);
      window.removeEventListener('resize', onResize);
      clearTimeout(resizeTimeout);
    };
  });
  
  // Reactive statement with bounds checking
  $: if (browser && innerWidth && innerHeight) {
    // Ensure currentStep is within bounds before updating
    const safeStep = Math.min(views.length - 1, Math.max(0, currentStep));
    if (safeStep !== currentStep) {
      currentStep = safeStep;
    }
    
    updateViewBox(currentStep, innerWidth, innerHeight);
  }
  
</script>

<style>
  .svg-wrapper {
    position: fixed;
    top: 0; left: 0;
    width: 100vw; height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    background: white;
    overflow: hidden;
    opacity: 1;
    transition: opacity 0.5s ease-out;
    z-index: 10;
  }

  /* Hide the SVG wrapper when scrollytelling is not active */
  .svg-wrapper.hidden {
    opacity: 0;
    pointer-events: none;
  }

  svg {
    width: 100%;
    height: 100%;
    display: block;
  }

  #Labels {
    opacity: 0;
    pointer-events: none;
    transition: opacity 0.5s ease;
  }

  #Labels.visible {
    opacity: 1;
    pointer-events: auto;
  }

  /* Each section fills the viewport to create scroll */
  section {
    height: 100vh;
  }

  /* Info Panel Styles */
  .info-panel {
    position: fixed;
    top: 45%;
    left: 20px;
    transform: translateY(-50%);
    background: rgba(255, 255, 255, 0.95);
    backdrop-filter: blur(10px);
    border-radius: 12px;
    padding: 24px;
    max-width: 350px;
    width: calc(100% - 40px);
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
    border: 1px solid rgba(255, 255, 255, 0.2);
    opacity: 0;
    transform: translateY(-50%) translateX(-30px);
    transition: all 0.6s cubic-bezier(0.4, 0, 0.2, 1);
    pointer-events: none;
    z-index: 1000;
    font-family: Georgia, serif;
  }

  .info-panel.visible {
    opacity: 0.8;
    transform: translateY(-50%) translateX(0);
    pointer-events: auto;
  }

  .info-panel h2 {
    margin: 0 0 16px 0;
    font-size: 24px;
    font-weight: 700;
    color: #1a1a1a;
    line-height: 1.2;
  }

  .info-panel p {
    margin: 0;
    font-size: 16px;
    line-height: 1.6;
    color: #4a4a4a;
  }

  /* Text content section that appears after scrollytelling */
  .text-content {
    position: relative;
    background: white;
    padding: 80px 20px;
    min-height: 100vh;
    z-index: 20;
    font-family: Georgia, serif;
  }

  .text-content .container {
    max-width: 800px;
    margin: 0 auto;
    line-height: 1.8;
  }

  .text-content h1 {
    font-size: 3rem;
    margin-bottom: 2rem;
    color: #1a1a1a;
    text-align: center;
  }

  .text-content p {
    font-size: 1.1rem;
    margin-bottom: 1.5rem;
    color: #4a4a4a;
  }

  .credits {
    font-family: Georgia, serif;
    font-style: oblique;
    font-weight: 300;
    margin-top: 2rem;
    font-size: 1rem;
    color: #4a4a4a;
  }

  /* Mobile Responsive */
  @media (max-width: 768px) {
    .info-panel {
      left: 16px;
      right: 16px;
      max-width: none;
      width: auto;
      padding: 20px;
      top: auto;
      bottom: 20px;
      transform: none;
      border-radius: 16px;
      backdrop-filter: blur(15px);
      -webkit-backdrop-filter: blur(15px); /* iOS Safari support */
    }

    .info-panel.visible {
      transform: none;
      opacity: 0.9; /* Slightly more opaque on mobile */
    }

    .info-panel h2 {
      font-size: 20px;
      margin-bottom: 12px;
    }

    .info-panel p {
      font-size: 14px;
      line-height: 1.5;
    }

    .text-content {
      padding: 60px 16px;
    }

    .text-content h1 {
      font-size: 2rem;
    }

    .text-content p {
      font-size: 1rem;
    }

    .credits {
      font-size: 0.9rem;
      margin-top: 1.5rem;
    }
  }

  /* Very small mobile */
  @media (max-width: 480px) {
    .info-panel {
      left: 12px;
      right: 12px;
      bottom: 16px;
      padding: 16px;
    }

    .info-panel h2 {
      font-size: 18px;
      margin-bottom: 10px;
    }

    .info-panel p {
      font-size: 13px;
    }

    .text-content {
      padding: 40px 12px;
    }

    .text-content h1 {
      font-size: 1.75rem;
    }
    
    .credits {
      font-size: 0.85rem;
      margin-top: 1rem;
    }
  }

  .chart-elements {
    opacity: 0;
    transition: opacity 0.3s ease;
  }

  .chart-elements.visible {
    opacity: 1;
  }

  /* Fix for iOS Safari scrolling issues */
  @supports (-webkit-touch-callout: none) {
    .svg-wrapper {
      -webkit-transform: translate3d(0, 0, 0);
      transform: translate3d(0, 0, 0);
    }
    
    .info-panel {
      -webkit-transform: translate3d(0, 0, 0);
      transform: translate3d(0, 0, 0);
    }
  }
</style>

<svelte:window bind:innerWidth bind:innerHeight />


<div class="svg-wrapper" class:hidden={!isScrollytellingActive}>
  <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox={viewBoxString} preserveAspectRatio="xMidYMid meet">
    <g id="Layer_1" data-name="Layer 1">
      <image width="6000" height="6750" 
       xlink:href="large_image_placeholder.jpg" 
       class="placeholder-image"/>

      <image width="6000" height="6750" 
        xlink:href="large_image_ultra_10.webp" 
        class="high-res-image" 
        style="opacity: 0;"
        bind:this={highResImage}/>

      <g class="chart-elements" class:visible={currentStep === 0}>
        <g id="_Legend_" data-name="&amp;lt;Legend&amp;gt;">
          <g>
            <path d="m3317,4688.1v-2.3c14.1-2.5,15.4-4,15.4-18.4v-75.5c0-15.3-1.3-16.3-15.4-18.5v-2.3h45.5v2.3c-14.3,2-15.6,3.2-15.6,18.5v75.4c0,14.3,1.2,15.8,15.6,18.5v2.3h-45.5Z" stroke-width="0"/>
            <path d="m3416.2,4688.1v-2.3c10.7-1.4,11.4-2.7,11.4-17v-28.2c0-11.3-3.8-20.7-15.5-20.7s-13.5,3.6-18.4,8v42.3c0,13,1.2,13.8,11.1,15.6v2.3h-36v-2.3c11-1.7,11.6-3.2,11.6-15.8v-36.5c0-10.8-.7-11.9-9-14.2v-1.9c7.5-1.1,16-4,22.7-7.5-.2,3.4-.5,8.2-.4,14.3,3.6-2.7,7.6-5.7,11.9-8.6,5.3-3.2,8.5-5.1,13.5-5.1,13.9,0,21.7,10.6,21.7,26.8v33.2c0,12.6,1,13.6,10,15.2v2.3h-34.6Z" stroke-width="0"/>
            <path d="m3527.6,4688.4c-2.4,1.3-4.5,1.8-5.8,1.8-10.9,0-16.5-7-16.5-20.6v-51.9h-11.7l-.4-.8,2.3-4.2h9.7v-16.7c3.6-2.8,8.8-7.2,12.3-10.8l1.4.2c-.2,3.6-.6,13.7-.5,27.3h18.7c.4.9,0,3.7-1,5h-17.7v47c0,14.5,5.2,17.3,10.8,17.3s7.4-1.3,9.6-2.5l.8,2.4-12,6.5Z" stroke-width="0"/>
            <path d="m3590.7,4688.1v-2.3c10.1-1.7,11.4-2.9,11.4-15.5v-29.6c0-14.5-4.4-21.9-15.9-21.9s-13,3.1-17.9,8.1v43.4c0,12,1,13.9,11.2,15.5v2.3h-35.2v-2.3c9.7-1.4,10.9-2.7,10.9-15.2v-88.8c0-13.4-2-13.5-11.2-15.3v-1.9c9.9-1.2,19.7-3.7,24.9-6.1-.3,3.2-.6,14-.6,22.7v42.1c5.8-6.1,14.6-12.7,25.4-12.7s21.8,8.1,21.8,28.5v32.6c0,11.5.8,12.3,10,14.2v2.3h-34.7Z" stroke-width="0"/>
            <path d="m3697.1,4672.5c-10.5,13.7-22.3,17.8-27.6,17.8-21.5,0-33.9-17.8-33.9-37.4s3.9-22,11.2-30.1c7.2-7.7,16.7-12.2,25.1-12.2,15.6,0,25.4,12.7,25.5,24.4,0,2.7-.2,4.3-.9,4.4-3.8,1.2-25.2,2.4-48,2.8-.4,27.6,14.8,37.4,27.6,37.4s13-2.4,19.7-9l1.5,1.9Zm-30.2-57.1c-8.4,0-15.9,6.8-17.9,22.1,10.2.3,28.1.1,32.3-.5,1.2-.2,1.8-1,1.8-2.8,0-9.2-5.2-18.7-16.2-18.7Z" stroke-width="0"/>
            <path d="m3737.1,4688.1v-2.3c10.3-1.6,11.6-2.5,11.6-15.9v-86.4c0-13.8-.8-14.9-11.2-16.6v-2.2c8-.7,19.2-3.6,24.9-6.1-.2,4.4-.6,12.3-.6,25v86.5c0,13.1.7,14.1,11.2,15.9v2.3h-35.9Z" stroke-width="0"/>
            <path d="m3835.8,4690.3c-2.9,0-5.8-1.1-8.5-3.6-1.8-2-3.3-4-4-8.1-6.4,4.6-15.7,11.7-19.7,11.7-12.2,0-20.1-10.4-20.1-20.1s4.3-12.9,12.6-15.8c9.7-3.5,22.6-7.3,27.2-10.5v-6.3c0-12.6-6.4-20.5-15.3-20.5s-5.9,1.1-7.8,3.4c-1.9,2.1-3.4,6.7-4.7,11.4-.8,2.7-2.6,3.2-3.6,3.2-3.1,0-6.9-3.1-6.9-6.7s1.3-3.1,4.5-5.4c5.5-4,17.8-10.8,25.1-12.4,5.2,0,10.1,1.8,13.7,4.4,6.2,5.5,8,11.6,8,22.2v32c0,8.4,2.8,11.2,6.2,11.2s3.8-.5,5.3-1.6l1,2.4-13.1,9Zm-12.4-43.1c-3,1.9-11.2,5-15.9,7.3-6.1,2.9-10.2,6.2-10.2,12.7s6.6,13.6,12.7,13.6,9.4-1.5,13-5.5c0-9,.3-18.5.3-28Z" stroke-width="0"/>
            <path d="m3900.8,4631.5c-3.3-12.2-9.9-17.3-18.5-17.3s-11.6,4.7-11.6,12,6.1,12.3,14.6,16.8c12.6,6.4,20.4,12.2,20.4,23.5s-13.5,23.9-26.9,23.9-14.8-2.2-17.9-3.5c-.8-3-2.8-14.3-3.4-19.3l2.2-.5c3.1,10.7,10.1,19.6,21.8,19.6s12.8-5.1,12.8-12.7-4.6-12.7-13.9-17.9c-9.9-5.6-19.9-11.3-20.1-23.7-.1-12.3,10.2-21.8,25.3-21.8,7.1,0,12.5,1.7,15.5,1.8.2,3.9,1.7,15.5,2.2,18.4l-2.4.7Z" stroke-width="0"/>
            <path d="m3947.3,4688.4c-2.4,1.3-4.5,1.8-5.8,1.8-10.9,0-16.5-7-16.5-20.6v-51.9h-11.7l-.4-.8,2.3-4.2h9.7v-16.7c3.6-2.8,8.8-7.2,12.3-10.8l1.4.2c-.2,3.6-.6,13.7-.5,27.3h18.7c.4.9,0,3.7-1,5h-17.7v47c0,14.5,5.2,17.3,10.8,17.3s7.4-1.3,9.6-2.5l.8,2.4-12,6.5Z" stroke-width="0"/>
            <path d="m4007.9,4688.1v-2.3c19.3-2.2,20.2-3.9,20.2-16.5v-72c0-9.1-.9-10.5-10.2-11.6l-6-.7v-2.1c10.7-1.8,22.2-5.1,30.4-9.1v95.5c0,12.8.7,13.9,20.3,16.5v2.3h-54.7Z" stroke-width="0"/>
            <path d="m4116.5,4573.8c26.4,0,36.1,30.4,36.1,58.1s-9.8,58.3-36.2,58.3-35.9-30.8-35.9-58.2,9.9-58.3,36-58.3Zm-.1,3.7c-16.3,0-20.2,29-20.2,54.4s4.2,54.7,20.5,54.7,20.3-29.2,20.3-54.5-4.1-54.6-20.6-54.6Z" stroke-width="0"/>
            <path d="m4268.4,4614.8c-8.2,2-9.4,3.5-13.5,12.8-5.1,11.8-10,25.2-22.5,56.7-10.8,27.7-14.8,39-17.5,48.1-1,3.7-2.6,5-4.8,5-5.3,0-9.6-3.7-9.6-7.6s.7-2.8,4.4-5.5c5.6-4.4,9.5-8,13.7-17.7,3.4-7.8,5.4-12.2,6.7-15.8.7-1.6.2-3.2-.5-5.4-7.4-20.2-17-43.5-21.9-56.3-4.2-10.6-5-12.3-14.4-14.3v-2.1h34.2v2.1c-8.7,2.3-9.4,3.4-6.7,10.5l17.1,45.7c5.3-14.3,13.5-36.1,16.3-45.5,2.2-6.7.9-8.4-10-10.7v-2.1h29v2.1Z" stroke-width="0"/>
            <path d="m4333.7,4672.5c-10.5,13.7-22.3,17.8-27.6,17.8-21.5,0-33.9-17.8-33.9-37.4s3.9-22,11.2-30.1c7.2-7.7,16.7-12.2,25.1-12.2,15.6,0,25.4,12.7,25.5,24.4,0,2.7-.2,4.3-.9,4.4-3.8,1.2-25.2,2.4-48,2.8-.4,27.6,14.8,37.4,27.6,37.4s13-2.4,19.7-9l1.5,1.9Zm-30.2-57.1c-8.4,0-15.9,6.8-17.9,22.1,10.2.3,28.1.1,32.3-.5,1.2-.2,1.8-1,1.8-2.8,0-9.2-5.2-18.7-16.2-18.7Z" stroke-width="0"/>
            <path d="m4397.9,4690.3c-2.9,0-5.8-1.1-8.5-3.6-1.8-2-3.3-4-4-8.1-6.3,4.6-15.7,11.7-19.7,11.7-12.2,0-20.1-10.4-20.1-20.1s4.3-12.9,12.7-15.8c9.7-3.5,22.6-7.3,27.2-10.5v-6.3c0-12.6-6.4-20.5-15.3-20.5s-5.9,1.1-7.8,3.4c-1.8,2.1-3.4,6.7-4.7,11.4-.8,2.7-2.6,3.2-3.6,3.2-3.1,0-6.8-3.1-6.8-6.7s1.3-3.1,4.5-5.4c5.5-4,17.8-10.8,25.1-12.4,5.2,0,10.1,1.8,13.7,4.4,6.2,5.5,8,11.6,8,22.2v32c0,8.4,2.8,11.2,6.2,11.2s3.8-.5,5.3-1.6l1,2.4-13.1,9Zm-12.4-43.1c-3,1.9-11.2,5-15.9,7.3-6.1,2.9-10.2,6.2-10.2,12.7s6.6,13.6,12.7,13.6,9.4-1.5,13-5.5c0-9,.3-18.5.3-28Z" stroke-width="0"/>
            <path d="m4455.8,4688.1h-37.6v-2.3c10.2-1.6,10.9-2.5,10.9-15.6v-36.6c0-11.1-.3-12.2-9-14.4v-1.9c8.3-1.5,15.6-3.8,22.8-7.5-.2,3.2-.2,12.7-.5,19.3,4.4-7.7,13.2-18.7,21.6-18.7s9.1,3.6,9.1,7.4-2.5,7.5-5.3,9.2c-1.9,1.2-3.2.8-4.2-.1-2.8-2.8-4.9-4.7-8.1-4.7s-8.6,3.5-13,12v35.8c0,12.8,1,13.6,13.4,15.6v2.3Z" stroke-width="0"/>
            <path d="m4524.9,4631.5c-3.3-12.2-9.9-17.3-18.5-17.3s-11.6,4.7-11.6,12,6.1,12.3,14.5,16.8c12.6,6.4,20.4,12.2,20.4,23.5s-13.5,23.9-26.9,23.9-14.8-2.2-17.9-3.5c-.8-3-2.8-14.3-3.4-19.3l2.2-.5c3.1,10.7,10.1,19.6,21.8,19.6s12.8-5.1,12.8-12.7-4.6-12.7-13.9-17.9c-9.9-5.6-19.9-11.3-20.1-23.7-.1-12.3,10.2-21.8,25.3-21.8,7.1,0,12.5,1.7,15.5,1.8.2,3.9,1.7,15.5,2.2,18.4l-2.4.7Z" stroke-width="0"/>
            <path d="m4651.3,4726.9c-27-20-41.4-53.6-41.4-87.5s14.5-67.2,41.4-87.2l1.9,2.3c-19.1,17.6-29.9,49.1-29.9,85.1s11,67.5,29.9,84.8l-1.9,2.5Z" stroke-width="0"/>
            <path d="m4735.2,4665.8c-2.1,5.6-5,14.8-6.7,22.3h-65.1v-2.4c10-10,20.5-21.6,28.8-31.7,11.2-13.6,21.3-28.6,21.3-44.1s-7.8-25.4-22-25.4-19.1,9.1-23.6,14.7l-2.2-1.8,9.2-12.7c5-6.1,13.1-10.8,23.6-10.8,15.2,0,29,11.5,29,30.5s-6,24.4-22.8,43c-8.7,9.4-18,19.5-26.7,28.4h36.3c9.7,0,12.6-.8,18.6-10.8l2.2.9Z" stroke-width="0"/>
            <path d="m4782.9,4573.8c26.4,0,36.1,30.4,36.1,58.1s-9.8,58.3-36.2,58.3-35.9-30.8-35.9-58.2,9.9-58.3,36-58.3Zm-.1,3.7c-16.3,0-20.2,29-20.2,54.4s4.2,54.7,20.5,54.7,20.3-29.2,20.3-54.5-4.1-54.6-20.6-54.6Z" stroke-width="0"/>
            <path d="m4840.2,4688.1v-2.3c19.3-2.2,20.2-3.9,20.2-16.5v-72c0-9.1-.9-10.5-10.2-11.6l-6-.7v-2.1c10.7-1.8,22.2-5.1,30.4-9.1v95.5c0,12.8.7,13.9,20.3,16.5v2.3h-54.7Z" stroke-width="0"/>
            <path d="m4970,4655.5v15.2c0,11.6,1.2,13.5,14.4,15.2v2.3h-41.6v-2.3c12-2.2,13.3-3.3,13.3-15.3v-15.1h-45v-4.1c17.2-24.9,36.5-52.6,54.2-77.6h4.8v74.9h16.4v6.8h-16.4Zm-14-62.1c-11,16-25.2,36.9-37.5,55.3h37.5v-55.3Z" stroke-width="0"/>
            <path d="m5068.3,4652.3h-37.3l.5-7.6h37.6l-.8,7.6Z" stroke-width="0"/>
            <path d="m5186.5,4665.8c-2.1,5.6-5,14.8-6.7,22.3h-65.1v-2.4c10-10,20.5-21.6,28.8-31.7,11.2-13.6,21.3-28.6,21.3-44.1s-7.8-25.4-22-25.4-19.1,9.1-23.6,14.7l-2.2-1.8,9.2-12.7c5-6.1,13.1-10.8,23.6-10.8,15.2,0,29,11.5,29,30.5s-6,24.4-22.8,43c-8.7,9.4-18.1,19.5-26.7,28.4h36.3c9.7,0,12.6-.8,18.6-10.8l2.2.9Z" stroke-width="0"/>
            <path d="m5234.1,4573.8c26.4,0,36.1,30.4,36.1,58.1s-9.8,58.3-36.2,58.3-35.9-30.8-35.9-58.2,9.9-58.3,36-58.3Zm-.1,3.7c-16.3,0-20.2,29-20.2,54.4s4.2,54.7,20.5,54.7,20.3-29.2,20.3-54.5-4.1-54.6-20.6-54.6Z" stroke-width="0"/>
            <path d="m5352.4,4665.8c-2.1,5.6-5,14.8-6.7,22.3h-65.1v-2.4c10-10,20.5-21.6,28.8-31.7,11.2-13.6,21.3-28.6,21.3-44.1s-7.8-25.4-22-25.4-19.1,9.1-23.6,14.7l-2.2-1.8,9.2-12.7c5-6.1,13.1-10.8,23.6-10.8,15.2,0,29,11.5,29,30.5s-6,24.4-22.8,43c-8.7,9.4-18.1,19.5-26.7,28.4h36.3c9.7,0,12.6-.8,18.6-10.8l2.2.9Z" stroke-width="0"/>
            <path d="m5421.2,4655.5v15.2c0,11.6,1.2,13.5,14.4,15.2v2.3h-41.6v-2.3c12-2.2,13.3-3.3,13.3-15.3v-15.1h-45v-4.1c17.2-24.9,36.5-52.6,54.2-77.6h4.8v74.9h16.4v6.8h-16.4Zm-14-62.1c-11,16-25.2,36.9-37.5,55.3h37.5v-55.3Z" stroke-width="0"/>
            <path d="m5448.9,4552.2c26.7,20.2,41.5,53.7,41.5,87.6s-14.9,67.2-41.5,87.1l-1.9-2.4c18.9-17.5,29.9-49.1,29.9-84.9s-10.9-67.5-29.9-84.9l1.9-2.5Z" stroke-width="0"/>
          </g>
          <rect x="3311.2" y="4822.7" width="87.8" height="87.8" fill="#ca0020" stroke-width="0"/>
          <g>
            <path d="m3590.4,4833c-.8-2.5-1.2-4.2-2.1-6.8-3-7.8-5.8-9.1-16.8-9.1h-15.4c-4,0-4.5.5-4.5,4.5v36.1h19.6c11.1,0,11.8-1,14.2-11.7h1.9v28h-1.9c-2.2-10.8-3.5-12-14.5-12h-19.3v31.5c0,11.8.6,13.3,14.1,15.1v1.8h-37.8v-1.8c11.7-1.5,13-3,13-15.1v-63.6c0-12.3-1-13.3-13-15.1v-1.8h44.2c13.9,0,17.8-.2,19.5-.3-.2,3.9.3,14.4.9,19.9l-2.1.3Z" stroke-width="0"/>
            <path d="m3598.9,4910.6v-1.8c8.7-1.4,9.9-2.1,9.9-13.2v-72.1c0-12-.8-12.9-9.6-14.2v-1.8c6.4-.6,15.7-3.1,19.6-5.1-.1,3.9-.4,10.8-.4,20.8v72.4c0,10.8.6,11.7,9.6,13.2v1.8h-29.1Z" stroke-width="0"/>
            <path d="m3667,4846.1c17.8,0,30.3,14.4,30.3,31.9s-16.3,34.3-30.4,34.3-30.7-15.7-30.7-31.9c0-23.5,18.4-34.3,30.9-34.3Zm-1.8,3.2c-10.5,0-17.7,11.2-17.7,26.5s9,33.1,21.3,33.1,17.1-7,17.1-28.2-7-31.5-20.7-31.5Z" stroke-width="0"/>
            <path d="m3739.2,4846.1c17.8,0,30.3,14.4,30.3,31.9s-16.3,34.3-30.4,34.3-30.7-15.7-30.7-31.9c0-23.5,18.4-34.3,30.9-34.3Zm-1.8,3.2c-10.5,0-17.7,11.2-17.7,26.5s9,33.1,21.3,33.1,17.1-7,17.1-28.2-7-31.5-20.7-31.5Z" stroke-width="0"/>
            <path d="m3847.2,4907c-3.3.5-13.2,2.2-20.5,5.4.1-3.3.3-6.9.3-10.2-3.9,2.6-8.7,5.5-12.7,7.7-3.9,1.9-6.5,2.5-8.1,2.5-11.2,0-25.6-11.4-25.6-31s17.2-35.2,36.1-35.2,7,.2,10.3,1.4v-26.1c0-9.7-.8-11.4-11.2-12.6v-1.7c6.4-.6,16.6-2.4,21.6-4.8-.3,3.2-.6,12.9-.6,18.7v73c0,7.5,1,9.6,6.3,10.5l4.2.8v1.6Zm-20.2-49.2c-3.5-5.1-10.5-7.2-17.1-7.2s-18.6,5-18.6,25.9,10.8,28.3,21,28.3,10.8-2.1,14.7-5.5v-41.5Z" stroke-width="0"/>
            <path d="m3853.5,4910.6v-1.8c9.3-1.5,9.9-2.2,9.9-13v-30.1c0-10.2-.6-11-8.5-12.5v-1.5c6-1.2,12.6-3,18.6-5.8-.2,3-.3,11.8-.3,18.6v31.2c0,11.1.9,11.7,9.9,13.2v1.8h-29.5Zm14.1-84c-3.2,0-6.3-2.8-6.3-6.3s3.1-6.4,6.4-6.4,6.3,2.5,6.3,6.4-2.8,6.3-6.4,6.3Z" stroke-width="0"/>
            <path d="m3929.1,4910.6v-1.8c9.3-1.2,9.9-2.2,9.9-14v-23.4c0-9.1-2.8-18-13.5-18s-12,3.4-16.2,7.2v34.9c0,11,1,11.7,9.6,13.2v1.8h-29.2v-1.8c9.4-1.4,9.9-2.7,9.9-13v-30.3c0-9.3-.5-10.2-7.6-12.1v-1.5c5.7-.9,12.3-3.3,17.7-6.3-.2,2.7-.5,6.8-.3,12.1,2.8-2.2,6.4-4.8,10.2-7.3,4.3-2.5,7-4.2,11.2-4.2,11.4,0,18,8.7,18,22.2v27.7c0,10.5.9,11.4,8.5,12.8v1.8h-28.2Z" stroke-width="0"/>
            <path d="m3984.6,4892c-4.5,3.6-5.2,6.5-5.2,8.2,0,3.4,2.8,6.3,8.5,6.3s11-.2,15.5-.2c11.8,0,21.7,4.8,21.7,18.1s-16.2,28.8-34.5,28.8-25.8-10.3-25.8-18.6.9-5.8,2.8-7.8c2.5-2.7,9.2-8.4,13-11.7-5.4-1.6-9-4.6-10.8-8.1-1-2-1.6-4.2-1.5-5.7,3.9-2.2,11-7,13.8-10.7l2.4,1.2Zm7.4,1.7c-14.8,0-24-11.1-24-22.6s12.6-24.9,25.2-24.9,9.8,1.7,13.8,4c5.4,0,14.4-1.8,17.8-3.3l.8.9c-1.4,2.4-3.2,5.7-5.1,7.2-2.7,0-6.4.2-9.1-.1,3.2,3,5.1,9,5.1,14.2,0,16.3-13.9,24.6-24.4,24.6Zm4.3,21.4c-9.8,0-11.5.4-12.9,1.3-6.6,4.5-9.6,8.8-9.6,14.4,0,9.1,10.2,16.6,22,16.6,17.7,0,21-9.5,21-18s-2.7-10.2-7.9-12.6c-3.2-1.5-7-1.8-12.6-1.8Zm-5.1-66.1c-7,0-12.6,6.4-12.6,19s6.1,22.9,15.3,22.6c7.5-.3,12.1-6,12.1-19.2s-6-22.5-14.8-22.5Z" stroke-width="0"/>
            <path d="m4087.5,4910.6h-30.6v-1.8c8.7-1.4,9.3-2.1,9.3-13v-30.1c0-9.6,0-10.5-7.6-12.3v-1.5c6.7-1.3,12.1-3.3,17.8-6.3-.1,2.7-.1,10.6-.4,16.2,3.6-6.3,11.2-15.6,18.1-15.6s7.4,3,7.4,5.9-1.7,5.4-3.8,6.9c-1.5,1-2.7.8-3.5,0-2.2-2.2-4-3.8-6.8-3.8s-7.6,3.3-11.5,10.8v29.8c0,10.6.8,11.4,11.5,13v1.8Z" stroke-width="0"/>
            <path d="m4156.5,4898c-8.7,11.1-18.1,14.4-22.3,14.4-17.7,0-27.9-14.8-27.9-31.2s3-18,9-24.7c5.8-6.3,13.8-10.3,20.8-10.3,12.8,0,20.7,10.7,20.7,19.9s-.2,3.6-.8,3.8c-2.8,1-20.4,2-40.5,2.2-.5,24,12.7,32.1,23.4,32.1s10.8-2.1,16.3-7.6l1.2,1.5Zm-24.6-47.8c-7.4,0-14,5.5-15.9,18,9,.3,25.9.1,28.9-.5.8-.2,1-.9,1-2.4,0-7.3-4.3-15.1-14.1-15.1Z" stroke-width="0"/>
            <path d="m4166.1,4850.9c6.2-1,14-3.6,19.2-6.3-.1,2.2-.4,7.2-.4,11.7,3-1.8,9.3-5.5,11.8-7,3.4-2,6.6-3.2,9-3.2,15.9,0,24.6,13.8,24.6,27.1,0,21.3-16.8,35.8-35.8,39.1-3.9-.2-8-1.7-9.6-2.7v22.6c0,13.9,1.3,14.5,11.7,16.2v1.8h-31.6v-1.8c8.7-1.7,10.3-2.2,10.3-13.8v-70.8c0-9.1-1.7-9.9-9.2-11.5v-1.5Zm18.8,49.8c3.3,3.7,9.1,5.4,14.4,5.4,13.3,0,21-8.4,21-27.1s-9.5-25.5-19.6-25.5-11.8,2.7-15.7,5.8v41.4Z" stroke-width="0"/>
            <path d="m4272.1,4846.1c17.8,0,30.3,14.4,30.3,31.9s-16.3,34.3-30.4,34.3-30.7-15.7-30.7-31.9c0-23.5,18.4-34.3,30.9-34.3Zm-1.8,3.2c-10.5,0-17.7,11.2-17.7,26.5s9,33.1,21.3,33.1,17.1-7,17.1-28.2-7-31.5-20.7-31.5Z" stroke-width="0"/>
            <path d="m4342,4910.6h-30.6v-1.8c8.7-1.4,9.3-2.1,9.3-13v-30.1c0-9.6,0-10.5-7.6-12.3v-1.5c6.7-1.3,12.1-3.3,17.8-6.3-.1,2.7-.1,10.6-.4,16.2,3.6-6.3,11.2-15.6,18.1-15.6s7.4,3,7.4,5.9-1.7,5.4-3.8,6.9c-1.5,1-2.7.8-3.5,0-2.2-2.2-4-3.8-6.8-3.8s-7.6,3.3-11.5,10.8v29.8c0,10.6.8,11.4,11.5,13v1.8Z" stroke-width="0"/>
            <path d="m4391.2,4910.9c-2,1-3.6,1.5-4.6,1.5-8.8,0-13.2-6-13.2-17.1v-43.3h-9.9l-.3-.6,1.8-3.4h8.4v-14.2c2.5-2.2,6.4-5.7,9-8.5l1,.2c-.1,2.7-.4,10.6-.4,22.6h16c.3.8,0,3-.9,4h-15.1v39c0,12.3,4.3,14.7,9.3,14.7s6.6-1.2,8.5-2.2l.6,2-10.2,5.5Z" stroke-width="0"/>
            <path d="m4455.6,4898c-8.7,11.1-18.1,14.4-22.3,14.4-17.7,0-27.9-14.8-27.9-31.2s3-18,9-24.7c5.8-6.3,13.8-10.3,20.8-10.3,12.8,0,20.7,10.7,20.7,19.9s-.2,3.6-.8,3.8c-2.8,1-20.4,2-40.5,2.2-.5,24,12.7,32.1,23.4,32.1s10.8-2.1,16.3-7.6l1.2,1.5Zm-24.6-47.8c-7.4,0-14,5.5-15.9,18,9,.3,25.9.1,28.9-.5.8-.2,1-.9,1-2.4,0-7.3-4.3-15.1-14.1-15.1Z" stroke-width="0"/>
            <path d="m4534,4907c-3.3.5-13.2,2.2-20.5,5.4.1-3.3.3-6.9.3-10.2-3.9,2.6-8.7,5.5-12.7,7.7-3.9,1.9-6.5,2.5-8.1,2.5-11.2,0-25.6-11.4-25.6-31s17.2-35.2,36.1-35.2,7,.2,10.3,1.4v-26.1c0-9.7-.8-11.4-11.2-12.6v-1.7c6.4-.6,16.6-2.4,21.6-4.8-.3,3.2-.6,12.9-.6,18.7v73c0,7.5,1,9.6,6.3,10.5l4.2.8v1.6Zm-20.2-49.2c-3.5-5.1-10.5-7.2-17.1-7.2s-18.6,5-18.6,25.9,10.8,28.3,21,28.3,10.8-2.1,14.7-5.5v-41.5Z" stroke-width="0"/>
          </g>
          <rect x="3311.2" y="5052.6" width="87.8" height="87.8" fill="#fff" stroke="#231f20" stroke-miterlimit="10" stroke-width="5"/>
          <g>
            <path d="m3627.7,5044.8c-10.6,2.4-11.5,4.2-11.8,14.4-.1,3.9-.5,6.6-.5,19.6v63.3h-2.4l-67.3-82.9v43c0,14.4.3,19,.5,23.8.3,8.7,2.7,10.6,14.4,12.6v1.8h-33.9v-1.8c11.1-2.5,13.5-3.9,13.8-12.7.2-3.9.5-9.5.5-23.7v-44.4c0-2.7-.6-4.5-2.1-6.3-3.6-4.5-8.1-5.7-13.6-6.8v-1.8h21l64.6,78.4v-42.6c0-13-.2-15.1-.5-20.2-.6-9.6-2.5-12.3-15-13.8v-1.8h32.4v1.8Z" stroke-width="0"/>
            <path d="m3664.8,5076c17.8,0,30.3,14.4,30.3,31.9s-16.3,34.3-30.4,34.3-30.7-15.7-30.7-31.9c0-23.5,18.4-34.3,30.9-34.3Zm-1.8,3.1c-10.5,0-17.7,11.2-17.7,26.5s9,33.1,21.3,33.1,17.1-7,17.1-28.2-7-31.5-20.7-31.5Z" stroke-width="0"/>
            <path d="m3767.7,5140.5v-1.8c8.5-1.2,9.9-2.2,9.9-12.1v-82.5c-2.8-4.2-7.9-6.1-12.9-6.1s-7.3,2.1-9.6,5.2c-2.7,3.9-4.6,11.8-4.6,25.3v9.3h15c.6.8.6,3.2-.9,3.9h-14.1v44.8c0,9.9,1.5,10.8,10.2,12.1v1.8h-29.8v-1.8c8.7-1.3,9.9-2.2,9.9-12.1v-44.8h-9.8l-.3-.9,2-3h8.1c0-10.3.6-15,1.3-18,.9-3.6,3.9-13.2,12.1-20.1,5.7-4.5,9.8-6.9,16.6-7.5,3.8.1,7.5.9,10.7,3,1.5-.4,3.1-1.2,4.9-2.4l1,.8c-.3,5.1-.3,10.3-.3,15.6v77.4c0,10,1.3,10.8,10,12.1v1.8h-29.5Z" stroke-width="0"/>
            <path d="m3836.2,5076c17.8,0,30.3,14.4,30.3,31.9s-16.3,34.3-30.4,34.3-30.7-15.7-30.7-31.9c0-23.5,18.4-34.3,30.9-34.3Zm-1.8,3.1c-10.5,0-17.7,11.2-17.7,26.5s9,33.1,21.3,33.1,17.1-7,17.1-28.2-7-31.5-20.7-31.5Z" stroke-width="0"/>
            <path d="m3908.4,5076c17.8,0,30.3,14.4,30.3,31.9s-16.3,34.3-30.4,34.3-30.7-15.7-30.7-31.9c0-23.5,18.4-34.3,30.9-34.3Zm-1.8,3.1c-10.5,0-17.7,11.2-17.7,26.5s9,33.1,21.3,33.1,17.1-7,17.1-28.2-7-31.5-20.7-31.5Z" stroke-width="0"/>
            <path d="m4016.4,5136.9c-3.3.5-13.2,2.2-20.5,5.4.1-3.3.3-6.9.3-10.2-3.9,2.5-8.7,5.5-12.7,7.6-3.9,2-6.5,2.5-8.1,2.5-11.2,0-25.6-11.4-25.6-31s17.2-35.2,36.1-35.2,7,.2,10.3,1.3v-26.1c0-9.8-.8-11.4-11.2-12.6v-1.6c6.4-.6,16.6-2.4,21.6-4.8-.3,3.1-.6,12.9-.6,18.7v73c0,7.5,1,9.6,6.3,10.5l4.2.7v1.7Zm-20.2-49.2c-3.5-5.1-10.5-7.2-17.1-7.2s-18.6,4.9-18.6,25.9,10.8,28.3,21,28.3,10.8-2.1,14.7-5.5v-41.5Z" stroke-width="0"/>
            <path d="m4022.7,5140.5v-1.8c9.3-1.5,9.9-2.2,9.9-13v-30.1c0-10.2-.6-10.9-8.5-12.4v-1.5c6-1.2,12.6-3,18.6-5.8-.2,3-.3,11.8-.3,18.6v31.2c0,11.1.9,11.7,9.9,13.2v1.8h-29.5Zm14.1-84c-3.2,0-6.3-2.9-6.3-6.3s3.1-6.5,6.4-6.5,6.3,2.5,6.3,6.5-2.8,6.3-6.4,6.3Z" stroke-width="0"/>
            <path d="m4098.3,5140.5v-1.8c9.3-1.2,9.9-2.2,9.9-13.9v-23.4c0-9.1-2.8-18-13.5-18s-12,3.5-16.2,7.2v34.9c0,10.9,1,11.7,9.6,13.2v1.8h-29.2v-1.8c9.4-1.3,9.9-2.7,9.9-13v-30.3c0-9.3-.5-10.2-7.6-12.2v-1.5c5.7-.9,12.3-3.3,17.7-6.3-.2,2.7-.5,6.7-.3,12.1,2.8-2.2,6.4-4.8,10.2-7.3,4.3-2.6,7-4.2,11.2-4.2,11.4,0,18,8.7,18,22.2v27.7c0,10.5.9,11.4,8.5,12.7v1.8h-28.2Z" stroke-width="0"/>
            <path d="m4153.8,5121.9c-4.5,3.6-5.2,6.4-5.2,8.2,0,3.5,2.8,6.3,8.5,6.3s10.9-.1,15.4-.1c11.8,0,21.7,4.8,21.7,18.1s-16.2,28.8-34.5,28.8-25.8-10.3-25.8-18.6.9-5.8,2.8-7.8c2.5-2.7,9.2-8.4,13-11.7-5.4-1.7-9-4.7-10.8-8.1-1-2-1.6-4.2-1.5-5.7,3.9-2.2,11-7,13.8-10.6l2.4,1.2Zm7.4,1.6c-14.8,0-24-11.1-24-22.6s12.6-24.9,25.2-24.9,9.8,1.6,13.8,4c5.4,0,14.4-1.8,17.8-3.3l.8.9c-1.4,2.4-3.2,5.7-5.1,7.2-2.7,0-6.4.2-9.1-.2,3.2,3,5.1,9,5.1,14.2,0,16.3-13.9,24.6-24.4,24.6Zm4.3,21.4c-9.8,0-11.5.5-12.9,1.3-6.6,4.5-9.6,8.8-9.6,14.4,0,9.2,10.2,16.6,22,16.6,17.7,0,21-9.4,21-18s-2.7-10.2-7.9-12.6c-3.2-1.5-7-1.8-12.6-1.8Zm-5.1-66.1c-7,0-12.6,6.5-12.6,19s6.1,22.9,15.3,22.6c7.5-.3,12.1-6,12.1-19.2s-6-22.5-14.8-22.5Z" stroke-width="0"/>
          </g>
          <rect x="3311.2" y="5282.6" width="87.8" height="87.8" fill="#e5e5e5" stroke="#231f20" stroke-miterlimit="10" stroke-width="5"/>
          <g>
            <path d="m3577.6,5270.8c25.2,0,47.4,20.2,47.4,49.5s-21.7,52.3-48.6,52.3-47.5-21.6-47.5-49.5,18.6-52.3,48.7-52.3Zm-2.9,3.3c-17.4,0-33,15.6-33,44.7s15.3,50.4,37.9,50.4,32.4-15.3,32.4-44.2-15.7-50.8-37.3-50.8Z" stroke-width="0"/>
            <path d="m3666.4,5307c1.7-.8,3.6-1,4.8-1,15.1,0,27.9,12,27.9,30s-16.9,36.3-35.7,36.3-12.7-1.8-20.2-4.8c.3-3.7.3-7.6.3-11.4v-73.6c0-10.3-.6-11.8-9.7-13.5v-1.7c7.5-.8,15-3.1,19.8-5.1-.3,4.3-.5,13.7-.5,19.6v31l13.3-5.8Zm-13.3,47.4c0,2.2.2,3.8.8,5.4,1.2,3.8,7.5,7.9,16.2,7.9,12.8,0,18-11.4,18-27.6s-9.5-27.1-23.1-27.1-9,.9-11.8,2.2v39.1Z" stroke-width="0"/>
            <path d="m3744.1,5323.2c-2.7-10.5-8.4-14.2-15.3-14.2s-9.9,4-9.9,10.2,5.2,10.5,12.4,14.2c10.5,5.2,16.6,10,16.6,19.2s-10.8,19.6-21.9,19.6-12-1.8-14.5-2.8c-.8-2.4-2.5-12-3-15.8l1.8-.4c2.4,8.7,8.1,16,18,16s11.1-4.3,11.1-10.9-4.3-10.9-11.7-15c-8.7-4.9-16.5-9.4-16.6-19.5-.1-9.9,8.4-17.8,20.5-17.8s10.2,1.3,12.6,1.5c.2,2.8,1.5,12.7,1.8,15.1l-1.9.6Z" stroke-width="0"/>
            <path d="m3809.5,5357.8c-8.7,11.1-18.1,14.4-22.3,14.4-17.7,0-27.9-14.8-27.9-31.2s3-18,9-24.7c5.8-6.3,13.8-10.3,20.8-10.3,12.8,0,20.7,10.6,20.7,19.9s-.1,3.6-.8,3.8c-2.8,1-20.4,1.9-40.5,2.2-.5,24,12.7,32.1,23.4,32.1s10.8-2.1,16.3-7.6l1.2,1.5Zm-24.6-47.8c-7.4,0-14,5.6-15.9,18,9,.3,25.9.2,28.9-.5.8-.1,1-.9,1-2.4,0-7.3-4.3-15.2-14.1-15.2Z" stroke-width="0"/>
            <path d="m3849.4,5370.4h-30.6v-1.8c8.7-1.3,9.3-2.1,9.3-13v-30.1c0-9.6,0-10.5-7.6-12.3v-1.5c6.7-1.3,12.1-3.3,17.8-6.3-.1,2.7-.1,10.6-.4,16.2,3.6-6.3,11.2-15.6,18.1-15.6s7.4,3,7.4,5.8-1.7,5.4-3.8,6.9c-1.5,1-2.7.8-3.5,0-2.2-2.2-4-3.7-6.8-3.7s-7.6,3.3-11.5,10.8v29.8c0,10.6.8,11.4,11.5,13v1.8Z" stroke-width="0"/>
            <path d="m3936.6,5309.5c-7.3,1.5-9.1,3-12.7,11.7-4.8,11.5-16,39.6-20.5,51h-1.9c-4.7-13-12.5-33.3-19.6-51.7-3.3-8.2-4-9.6-11.5-10.9v-1.8h26.8v1.8c-7.5,1.7-7.7,3-5.7,8.7,3.9,11.5,9.6,26.1,14.4,38.5,5.7-14.1,10.9-28.2,14.1-37.2,2.7-7.6,1-8.4-7.8-10v-1.8h24.6v1.8Z" stroke-width="0"/>
            <path d="m3983.1,5372.2c-2.1,0-4.5-.9-6.8-3-1.5-1.7-2.7-3.1-3.3-6.8-5.4,3.9-13.2,9.8-16.6,9.8-10,0-16.5-8.5-16.5-16.5s3.6-10.8,10.6-13.2c8.1-2.8,18.7-6.1,22.6-8.8v-4.5c0-10.8-5.2-17.8-13.3-17.8s-5,.9-6.6,2.7c-1.7,1.8-3,5.8-4,9.4-.6,2.1-2.2,2.4-2.9,2.4-2.1,0-5.2-2.2-5.2-5.2s1-2.4,3.6-4.3c3.9-3,14.4-9,20.5-10.3,3.9,0,8.1,1.3,10.9,3.6,5,4.5,6.5,9.4,6.5,18.3v26.8c0,7.2,2.4,9.6,5.2,9.6s3.3-.5,4.6-1.5l.9,1.9-10.3,7.5Zm-9.9-35.8c-2.6,1.6-10.3,4.6-14.4,6.6-5,2.2-8.5,5.1-8.5,10.5s5.7,11.2,10.9,11.2,8.5-1.5,11.7-5.1c0-7.2.3-15.4.3-23.2Z" stroke-width="0"/>
            <path d="m4024.5,5370.7c-2,1.1-3.6,1.5-4.6,1.5-8.8,0-13.2-6-13.2-17.1v-43.3h-9.9l-.3-.6,1.8-3.5h8.4v-14.2c2.6-2.2,6.5-5.7,9-8.6l1,.2c-.1,2.7-.5,10.7-.5,22.6h16c.3.8,0,3-.9,4h-15.1v39c0,12.3,4.4,14.7,9.3,14.7s6.6-1.2,8.5-2.2l.6,2-10.2,5.5Z" stroke-width="0"/>
            <path d="m4039,5370.4v-1.8c9.3-1.5,9.9-2.2,9.9-13v-30.1c0-10.2-.6-11-8.5-12.5v-1.5c6-1.2,12.6-3,18.6-5.8-.2,3-.3,11.8-.3,18.6v31.2c0,11.1.9,11.7,9.9,13.2v1.8h-29.5Zm14.1-84c-3.2,0-6.3-2.8-6.3-6.3s3.1-6.5,6.4-6.5,6.3,2.5,6.3,6.5-2.8,6.3-6.4,6.3Z" stroke-width="0"/>
            <path d="m4107.1,5305.9c17.8,0,30.3,14.4,30.3,31.9s-16.3,34.3-30.4,34.3-30.7-15.7-30.7-31.9c0-23.5,18.4-34.3,30.9-34.3Zm-1.8,3.1c-10.5,0-17.7,11.2-17.7,26.5s9,33.1,21.3,33.1,17.1-7.1,17.1-28.2-7-31.5-20.7-31.5Z" stroke-width="0"/>
            <path d="m4185.3,5370.4v-1.8c9.3-1.2,9.9-2.2,9.9-13.9v-23.4c0-9.1-2.8-18-13.5-18s-12,3.4-16.2,7.2v34.9c0,10.9,1,11.7,9.6,13.2v1.8h-29.2v-1.8c9.4-1.3,9.9-2.7,9.9-13v-30.3c0-9.3-.5-10.2-7.6-12.2v-1.5c5.7-.9,12.3-3.3,17.7-6.3-.2,2.7-.5,6.7-.3,12.1,2.8-2.2,6.4-4.8,10.2-7.3,4.3-2.5,7-4.2,11.2-4.2,11.4,0,18,8.7,18,22.2v27.7c0,10.5.9,11.4,8.5,12.8v1.8h-28.2Z" stroke-width="0"/>
            <path d="m4282.8,5370.4h-30.6v-1.8c8.7-1.3,9.3-2.1,9.3-13v-30.1c0-9.6,0-10.5-7.6-12.3v-1.5c6.7-1.3,12.1-3.3,17.8-6.3-.1,2.7-.1,10.6-.4,16.2,3.6-6.3,11.2-15.6,18.1-15.6s7.4,3,7.4,5.8-1.7,5.4-3.8,6.9c-1.5,1-2.7.8-3.5,0-2.2-2.2-4-3.7-6.8-3.7s-7.6,3.3-11.5,10.8v29.8c0,10.6.8,11.4,11.5,13v1.8Z" stroke-width="0"/>
            <path d="m4351.8,5357.8c-8.7,11.1-18.1,14.4-22.3,14.4-17.7,0-27.9-14.8-27.9-31.2s3-18,9-24.7c5.8-6.3,13.8-10.3,20.8-10.3,12.8,0,20.7,10.6,20.7,19.9s-.2,3.6-.8,3.8c-2.8,1-20.4,1.9-40.5,2.2-.5,24,12.7,32.1,23.4,32.1s10.8-2.1,16.3-7.6l1.2,1.5Zm-24.6-47.8c-7.4,0-14,5.6-15.9,18,9,.3,25.9.2,28.9-.5.8-.1,1-.9,1-2.4,0-7.3-4.3-15.2-14.1-15.2Z" stroke-width="0"/>
            <path d="m4398,5323.2c-2.7-10.5-8.4-14.2-15.3-14.2s-9.9,4-9.9,10.2,5.2,10.5,12.4,14.2c10.5,5.2,16.6,10,16.6,19.2s-10.8,19.6-21.9,19.6-12-1.8-14.5-2.8c-.8-2.4-2.5-12-3-15.8l1.8-.4c2.4,8.7,8.1,16,18,16s11.1-4.3,11.1-10.9-4.3-10.9-11.7-15c-8.7-4.9-16.5-9.4-16.6-19.5-.1-9.9,8.4-17.8,20.5-17.8s10.2,1.3,12.6,1.5c.2,2.8,1.5,12.7,1.8,15.1l-1.9.6Z" stroke-width="0"/>
            <path d="m4463.4,5357.8c-8.7,11.1-18.1,14.4-22.3,14.4-17.7,0-27.9-14.8-27.9-31.2s3-18,9-24.7c5.8-6.3,13.8-10.3,20.8-10.3,12.8,0,20.7,10.6,20.7,19.9s-.1,3.6-.8,3.8c-2.8,1-20.4,1.9-40.5,2.2-.5,24,12.7,32.1,23.4,32.1s10.8-2.1,16.3-7.6l1.2,1.5Zm-24.6-47.8c-7.4,0-14,5.6-15.9,18,9,.3,25.9.2,28.9-.5.8-.1,1-.9,1-2.4,0-7.3-4.3-15.2-14.1-15.2Z" stroke-width="0"/>
            <path d="m4503.2,5370.4h-30.6v-1.8c8.7-1.3,9.3-2.1,9.3-13v-30.1c0-9.6,0-10.5-7.6-12.3v-1.5c6.7-1.3,12.1-3.3,17.8-6.3-.1,2.7-.1,10.6-.4,16.2,3.6-6.3,11.2-15.6,18.1-15.6s7.4,3,7.4,5.8-1.7,5.4-3.8,6.9c-1.5,1-2.7.8-3.5,0-2.2-2.2-4-3.7-6.8-3.7s-7.6,3.3-11.5,10.8v29.8c0,10.6.8,11.4,11.5,13v1.8Z" stroke-width="0"/>
            <path d="m4590.4,5309.5c-7.3,1.5-9.1,3-12.7,11.7-4.8,11.5-16,39.6-20.5,51h-1.9c-4.7-13-12.5-33.3-19.6-51.7-3.3-8.2-4-9.6-11.5-10.9v-1.8h26.8v1.8c-7.5,1.7-7.7,3-5.7,8.7,3.9,11.5,9.6,26.1,14.4,38.5,5.7-14.1,10.9-28.2,14.1-37.2,2.7-7.6,1-8.4-7.8-10v-1.8h24.6v1.8Z" stroke-width="0"/>
            <path d="m4642.6,5357.8c-8.7,11.1-18.1,14.4-22.3,14.4-17.7,0-27.9-14.8-27.9-31.2s3-18,9-24.7c5.8-6.3,13.8-10.3,20.8-10.3,12.8,0,20.7,10.6,20.7,19.9s-.1,3.6-.8,3.8c-2.8,1-20.4,1.9-40.5,2.2-.5,24,12.7,32.1,23.4,32.1s10.8-2.1,16.3-7.6l1.2,1.5Zm-24.6-47.8c-7.4,0-14,5.6-15.9,18,9,.3,25.9.2,28.9-.5.8-.1,1-.9,1-2.4,0-7.3-4.3-15.2-14.1-15.2Z" stroke-width="0"/>
            <path d="m4721,5366.8c-3.3.5-13.2,2.2-20.5,5.4.1-3.3.3-6.9.3-10.2-3.9,2.5-8.7,5.5-12.7,7.6-3.9,2-6.5,2.5-8.1,2.5-11.2,0-25.6-11.4-25.6-31s17.2-35.2,36.1-35.2,7,.2,10.3,1.3v-26.1c0-9.7-.8-11.4-11.2-12.6v-1.6c6.4-.6,16.6-2.4,21.6-4.8-.3,3.2-.6,12.9-.6,18.8v73c0,7.5,1,9.6,6.3,10.5l4.2.7v1.6Zm-20.2-49.2c-3.5-5.1-10.5-7.2-17.1-7.2s-18.6,4.9-18.6,25.9,10.8,28.3,21,28.3,10.8-2.1,14.7-5.5v-41.5Z" stroke-width="0"/>
          </g>
        </g>
      </g>
    </g>
    <g id="Labels" class:visible={views[currentStep].labelVisible}>
      <g>
        <g>
          <g opacity=".5">
            <g mask="url(#mask)">
              <rect x="1693.5" y="1883.6" width="215.2" height="50.9" fill="#fff" stroke-width="0"/>
            </g>
          </g>
          <g>
            <path d="m1699.5,1927.6v-37h3.8v16.4h13.2v-16.4h3.8v37h-3.8v-17.2h-13.2v17.2h-3.8Z" stroke-width="0"/>
            <path d="m1732.3,1927.6h-3.9l10.6-37h3.7l10.7,37h-3.8l-3.2-11h-11.1l-3.1,11Zm8.6-32.2l-4.6,17.8h9.1l-4.6-17.8Z" stroke-width="0"/>
            <path d="m1761.7,1890.6h7.1c5.5,0,7.8,1,9.2,2.2,2.4,2,3.9,5.1,3.9,9s-.4,4.7-1.6,6.7c-1.5,2.3-3.7,3.9-6.4,4.3l7.4,14.9h-4.2l-8.3-17.2c.5,0,.9,0,1.3,0,2.6,0,4.4-.6,5.8-2,1.5-1.5,2.1-3.8,2.1-6.3s-.6-5.2-2.8-6.8c-1.6-1.2-4.1-1.3-6.5-1.3h-3.2v33.6h-3.8v-37Z" stroke-width="0"/>
            <path d="m1797.1,1914.1l-9.5-23.6h3.9l7.5,20,7.5-20h3.9l-9.5,23.6v13.5h-3.8v-13.5Z" stroke-width="0"/>
            <path d="m1817.1,1927.6h-3.9l10.6-37h3.7l10.7,37h-3.8l-3.2-11h-11.1l-3,11Zm8.6-32.2l-4.6,17.8h9.1l-4.6-17.8Z" stroke-width="0"/>
            <path d="m1866,1921.4v-30.9h3.4v37h-4.2l-15.3-31.6v31.6h-3.4v-37h4.5l15,30.9Z" stroke-width="0"/>
            <path d="m1881.6,1927.6h-3.9l10.6-37h3.7l10.7,37h-3.8l-3.2-11h-11.1l-3.1,11Zm8.6-32.2l-4.6,17.8h9.1l-4.6-17.8Z" stroke-width="0"/>
          </g>
        </g>
        <g>
          <g opacity=".5">
            <g mask="url(#mask-1)">
              <rect x="1558.4" y="1560.6" width="186.8" height="50.9" fill="#fff" stroke-width="0"/>
            </g>
          </g>
          <g>
            <path d="m1573,1567.2c4.3,0,6.9.7,9.3,2.6,2.2,1.8,3.6,5,3.6,8.5s-1.5,6.8-3.6,8.6c-2.4,1.9-5.1,2.5-9.3,2.5h-3.2v14.9h-3.8v-37h7Zm-3.2,18.7h3.2c3.4,0,5.1-.3,6.8-1.8,1.4-1,2.2-3.4,2.2-5.9s-1-4.9-2.3-6c-1.6-1.4-3.4-1.7-6.8-1.7h-3.2v15.3Z" stroke-width="0"/>
            <path d="m1598.9,1567.2v27c0,2.1.3,4.1,1.9,5.6,1.2,1.2,3.1,1.6,5,1.6s3.8-.4,5.1-1.6c1.6-1.5,1.9-3.4,1.9-5.6v-27h3.8v27.3c0,3.5-.8,5.8-2.6,7.6-1.8,1.8-4.4,2.8-8,2.8s-6.3-1-8.1-2.8c-1.8-1.8-2.6-4.1-2.6-7.6v-27.3h3.8Z" stroke-width="0"/>
            <path d="m1647.5,1598.1v-30.9h3.4v37h-4.2l-15.3-31.6v31.6h-3.4v-37h4.5l15,30.9Z" stroke-width="0"/>
            <path d="m1677.4,1594c0,1.8-.1,3.8-.8,5.6-.7,1.4-1.6,2.7-3,3.6-1.5,1.2-3.4,1.6-5.1,1.6-4.5,0-8.2-2.7-8.9-8,0-.5-.1-1-.1-1.5h3.7c0,.2.1.5.1.8.8,4.5,3.4,5.3,5.3,5.3,2.1-.1,3.6-1.6,4.2-3.1.6-1.3.7-2.8.7-4.9v-26.3h3.8v26.8Z" stroke-width="0"/>
            <path d="m1688.7,1604.2h-3.9l10.6-37h3.7l10.7,37h-3.8l-3.2-11h-11.1l-3.1,11Zm8.6-32.2l-4.6,17.8h9.1l-4.6-17.8Z" stroke-width="0"/>
            <path d="m1718,1567.2h7.7c3.6,0,5.5.7,6.9,1.9,2.5,1.7,3.2,4.8,3.2,7.5,0,4.9-2.6,7-5.2,7.9,5.4,1.6,6.9,6.2,6.9,9.6s-1.5,6.8-4.1,8.4c-1.5,1.2-3.8,1.8-6.5,1.8h-9v-37Zm3.8,16h4.1c1.6,0,2.9-.3,3.8-.9,1.9-1.4,2.3-3.3,2.3-5.5s-1.1-4.5-2.3-5.2c-.8-.6-1.9-1-4.7-1h-3.3v12.7Zm0,17.7h4.6c2.1,0,3.5-.3,4.5-1,1.9-1,2.9-3,3-6.1,0-2.9-1.2-5.2-3-6.3-1.1-.8-2.6-1-4.9-1h-4.1v14.3Z" stroke-width="0"/>
          </g>
        </g>
        <g>
          <g opacity=".5">
            <g mask="url(#mask-2)">
              <rect x="2592.9" y="2419" width="370.9" height="50.9" fill="#fff" stroke-width="0"/>
            </g>
          </g>
          <g>
            <path d="m2602.2,2425.9v27c0,2.1.3,4.1,1.9,5.6,1.3,1.2,3.2,1.6,5,1.6s3.8-.4,5.1-1.6c1.6-1.5,1.9-3.4,1.9-5.6v-27h3.8v27.3c0,3.5-.8,5.8-2.6,7.6-1.8,1.8-4.4,2.8-8,2.8s-6.3-1-8.1-2.8c-1.8-1.8-2.6-4.1-2.6-7.6v-27.3h3.8Z" stroke-width="0"/>
            <path d="m2636.3,2429.4h-7.5v-3.5h18.8v3.5h-7.5v33.6h-3.8v-33.6Z" stroke-width="0"/>
            <path d="m2661.9,2429.4h-7.5v-3.5h18.8v3.5h-7.5v33.6h-3.8v-33.6Z" stroke-width="0"/>
            <path d="m2679.2,2462.9h-3.9l10.6-37h3.7l10.7,37h-3.8l-3.2-11h-11.1l-3.1,11Zm8.6-32.2l-4.6,17.8h9.1l-4.6-17.8Z" stroke-width="0"/>
            <path d="m2708.6,2425.9h7.1c5.5,0,7.8,1,9.2,2.2,2.4,2,3.9,5.1,3.9,9s-.4,4.7-1.6,6.7c-1.5,2.3-3.7,3.9-6.4,4.3l7.4,14.9h-4.2l-8.3-17.2c.5,0,.9,0,1.3,0,2.6,0,4.4-.6,5.8-2,1.6-1.5,2.2-3.8,2.2-6.3s-.6-5.2-2.8-6.8c-1.6-1.2-4.2-1.3-6.5-1.3h-3.2v33.6h-3.8v-37Z" stroke-width="0"/>
            <path d="m2762.9,2425.9c4.3,0,6.9.7,9.3,2.6,2.2,1.8,3.6,5,3.6,8.5s-1.5,6.8-3.6,8.6c-2.4,1.9-5.1,2.5-9.3,2.5h-3.2v14.9h-3.8v-37h7Zm-3.2,18.7h3.2c3.4,0,5.1-.3,6.8-1.8,1.4-1,2.2-3.4,2.2-5.9s-1-4.9-2.3-6c-1.6-1.4-3.4-1.7-6.8-1.7h-3.2v15.3Z" stroke-width="0"/>
            <path d="m2785.2,2425.9h7.1c5.5,0,7.8,1,9.2,2.2,2.4,2,3.9,5.1,3.9,9s-.4,4.7-1.6,6.7c-1.5,2.3-3.7,3.9-6.4,4.3l7.4,14.9h-4.2l-8.3-17.2c.5,0,.9,0,1.3,0,2.6,0,4.4-.6,5.8-2,1.6-1.5,2.2-3.8,2.2-6.3s-.6-5.2-2.8-6.8c-1.6-1.2-4.2-1.3-6.5-1.3h-3.2v33.6h-3.8v-37Z" stroke-width="0"/>
            <path d="m2815.7,2462.9h-3.9l10.6-37h3.7l10.7,37h-3.8l-3.2-11h-11.1l-3.1,11Zm8.6-32.2l-4.6,17.8h9.1l-4.6-17.8Z" stroke-width="0"/>
            <path d="m2845,2425.9h8.5c5,0,9.3,1.3,12.1,5.4,2,3,3.2,7.4,3.2,13.2,0,14.2-6.9,18.5-15.3,18.5h-8.6v-37Zm3.8,33.6h4.6c8.7,0,11.8-5.6,11.8-15.1s-.8-8.9-2.6-11.3c-1.9-2.6-4.9-3.8-9.3-3.8h-4.5v30.1Z" stroke-width="0"/>
            <path d="m2879.3,2425.9h17.4v3.5h-13.6v12.9h13.6v3.5h-13.6v13.8h13.6v3.5h-17.4v-37Z" stroke-width="0"/>
            <path d="m2910,2452.7c.5,5.8,3.9,7.6,6.7,7.6s7-2.6,7-7.2-4.6-6.7-8.3-7.9c-3.1-1.1-8.5-3-8.5-9.6,0-6.5,4-10.4,9.8-10.4s9.1,2.9,9.9,9.4h-3.6c-.4-2.8-1.6-6.2-6.2-6.2s-6.1,2.7-6.1,6.6,3.7,6,8.5,7.6c2.9,1,8.3,3,8.3,10.4s-4.7,10.7-10.7,10.7-10.2-3.5-10.4-10.9h3.6Z" stroke-width="0"/>
            <path d="m2937.6,2462.9v-37h3.8v16.4h13.2v-16.4h3.8v37h-3.8v-17.2h-13.2v17.2h-3.8Z" stroke-width="0"/>
          </g>
        </g>
        <g>
          <g opacity=".5">
            <g mask="url(#mask-3)">
              <rect x="3530.9" y="2559.8" width="144.7" height="50.9" fill="#fff" stroke-width="0"/>
            </g>
          </g>
          <g>
            <path d="m3539.5,2566.7h7.7c3.6,0,5.5.7,6.9,1.9,2.5,1.7,3.2,4.8,3.2,7.5,0,4.9-2.6,7-5.2,7.9,5.4,1.6,6.9,6.2,6.9,9.6s-1.5,6.8-4.1,8.4c-1.5,1.2-3.8,1.8-6.5,1.8h-9v-37Zm3.8,16h4.1c1.6,0,2.9-.3,3.8-.9,1.9-1.4,2.3-3.3,2.3-5.5s-1.1-4.5-2.3-5.2c-.8-.6-1.9-1-4.6-1h-3.3v12.7Zm0,17.7h4.6c2.1,0,3.5-.3,4.5-1,1.9-1,2.9-3,3-6.1,0-2.9-1.2-5.2-3-6.3-1.1-.8-2.5-1-4.9-1h-4.1v14.3Z" stroke-width="0"/>
            <path d="m3569.4,2566.7h3.8v37h-3.8v-37Z" stroke-width="0"/>
            <path d="m3584.9,2603.7v-37h3.8v16.4h13.2v-16.4h3.8v37h-3.8v-17.2h-13.2v17.2h-3.8Z" stroke-width="0"/>
            <path d="m3617.7,2603.7h-3.9l10.6-37h3.7l10.7,37h-3.8l-3.2-11h-11.1l-3.1,11Zm8.6-32.2l-4.6,17.8h9.1l-4.6-17.8Z" stroke-width="0"/>
            <path d="m3647,2566.7h7.1c5.5,0,7.8,1,9.2,2.2,2.4,2,3.9,5.1,3.9,9s-.4,4.7-1.6,6.7c-1.5,2.3-3.7,3.9-6.4,4.3l7.4,14.9h-4.2l-8.3-17.2c.5,0,.9,0,1.3,0,2.6,0,4.4-.6,5.8-2,1.6-1.5,2.2-3.8,2.2-6.3s-.6-5.2-2.8-6.8c-1.6-1.2-4.2-1.3-6.5-1.3h-3.2v33.6h-3.8v-37Z" stroke-width="0"/>
          </g>
        </g>
        <g>
          <g opacity=".5">
            <g mask="url(#mask-4)">
              <rect x="3747.4" y="3115.2" width="333.8" height="50.9" fill="#fff" stroke-width="0"/>
            </g>
          </g>
          <g>
            <path d="m3753.2,3122.2h3.7l7.3,31.4,6.7-31.4h3.8l6.7,31.4,7.2-31.4h3.7l-8.8,37h-3.9l-6.8-31.9-6.6,31.9h-3.9l-8.9-37Z" stroke-width="0"/>
            <path d="m3800.7,3122.2h17.4v3.5h-13.6v12.9h13.6v3.5h-13.6v13.8h13.6v3.5h-17.4v-37Z" stroke-width="0"/>
            <path d="m3831.4,3149c.5,5.8,3.9,7.6,6.7,7.6s7-2.6,7-7.2-4.6-6.7-8.3-7.9c-3.1-1.1-8.5-3-8.5-9.6,0-6.5,4-10.4,9.8-10.4s9.1,2.9,9.9,9.4h-3.6c-.4-2.8-1.7-6.2-6.2-6.2s-6.1,2.7-6.1,6.6,3.7,6,8.5,7.6c2.9,1,8.3,3,8.3,10.4s-4.7,10.7-10.7,10.7-10.2-3.5-10.4-10.9h3.6Z" stroke-width="0"/>
            <path d="m3864.1,3125.6h-7.5v-3.5h18.8v3.5h-7.5v33.6h-3.8v-33.6Z" stroke-width="0"/>
            <path d="m3901.7,3122.2h7.7c3.6,0,5.5.7,6.9,1.9,2.5,1.7,3.2,4.8,3.2,7.5,0,4.9-2.6,7-5.2,7.9,5.4,1.6,6.9,6.2,6.9,9.6s-1.5,6.8-4.1,8.4c-1.5,1.2-3.8,1.8-6.5,1.8h-9v-37Zm3.8,16h4.1c1.6,0,2.9-.3,3.8-.9,1.9-1.4,2.3-3.3,2.3-5.5s-1.1-4.5-2.3-5.2c-.8-.6-1.9-1-4.7-1h-3.3v12.7Zm0,17.7h4.6c2.1,0,3.5-.3,4.5-1,1.9-1,2.9-3,3-6.1,0-2.9-1.2-5.2-3-6.3-1.1-.8-2.6-1-4.9-1h-4.1v14.3Z" stroke-width="0"/>
            <path d="m3931.7,3122.2h17.4v3.5h-13.6v12.9h13.6v3.5h-13.6v13.8h13.6v3.5h-17.4v-37Z" stroke-width="0"/>
            <path d="m3979.9,3153v-30.9h3.4v37h-4.2l-15.3-31.6v31.6h-3.4v-37h4.5l15,30.9Z" stroke-width="0"/>
            <path d="m4016,3134.4c0-1.8-.7-4.1-1.7-5.7-1.3-2.2-3.3-3.8-6.8-3.8-6.1,0-10.1,5.4-10.1,15.8s4.4,15.8,10.1,15.8,5.7-1.3,7.3-4.4c1.4-2.7,1.6-5.5,1.6-8.4h-10.4v-3.4h14.3c0,1.7,0,3.7-.1,5.5-.2,6.3-3.5,14.1-12.6,14.1s-14-8.1-14-19.2,5.4-19.2,14-19.2,11.8,6,12.2,12.9h-3.8Z" stroke-width="0"/>
            <path d="m4030.4,3159.2h-3.9l10.6-37h3.7l10.7,37h-3.8l-3.2-11h-11.1l-3.1,11Zm8.6-32.2l-4.6,17.8h9.1l-4.6-17.8Z" stroke-width="0"/>
            <path d="m4059.9,3122.2h3.8v33.6h11.8v3.5h-15.6v-37Z" stroke-width="0"/>
          </g>
        </g>
        <g>
          <g opacity=".5">
            <g mask="url(#mask-5)">
              <rect x="4994.6" y="2339.8" width="163.4" height="50.9" fill="#fff" stroke-width="0"/>
            </g>
          </g>
          <g>
            <path d="m5004.6,2383.8h-3.9l10.6-37h3.7l10.7,37h-3.8l-3.2-11h-11.1l-3.1,11Zm8.6-32.2l-4.6,17.8h9.1l-4.6-17.8Z" stroke-width="0"/>
            <path d="m5036.1,2373.6c.5,5.8,3.9,7.6,6.7,7.6s7-2.6,7-7.2-4.6-6.7-8.3-7.9c-3.1-1.1-8.5-3-8.5-9.6,0-6.5,4-10.4,9.8-10.4s9.1,2.9,9.9,9.4h-3.6c-.4-2.8-1.7-6.2-6.2-6.2s-6.1,2.7-6.1,6.6,3.7,6,8.5,7.6c2.9,1,8.3,3,8.3,10.4s-4.7,10.7-10.7,10.7-10.2-3.5-10.4-10.9h3.6Z" stroke-width="0"/>
            <path d="m5065.8,2373.6c.5,5.8,3.9,7.6,6.7,7.6s7-2.6,7-7.2-4.6-6.7-8.3-7.9c-3.1-1.1-8.5-3-8.5-9.6,0-6.5,4-10.4,9.8-10.4s9.1,2.9,9.9,9.4h-3.6c-.4-2.8-1.6-6.2-6.2-6.2s-6.1,2.7-6.1,6.6,3.7,6,8.5,7.6c2.9,1,8.3,3,8.3,10.4s-4.7,10.7-10.7,10.7-10.2-3.5-10.4-10.9h3.6Z" stroke-width="0"/>
            <path d="m5093.8,2383.8h-3.9l10.6-37h3.7l10.7,37h-3.8l-3.2-11h-11.1l-3.1,11Zm8.6-32.2l-4.6,17.8h9.1l-4.6-17.8Z" stroke-width="0"/>
            <path d="m5123.2,2383.8v-37h5.8l8.7,31.6,8.1-31.6h5.9v37h-3.4v-33.6l-8.8,33.6h-3.5l-9.4-33.6v33.6h-3.4Z" stroke-width="0"/>
          </g>
        </g>
      </g>
    </g>
  </svg>
</div>

<!-- Scrollytelling sections - add one extra section for the last step to be fully visible -->
{#each Array(views.length + 1) as _, i}
  <section style="height: 100vh;"></section>
{/each}

Info Panel for Scrollytelling - now uses showInfoPanel instead of isScrollytellingActive
{#if showInfoPanel && views[currentStep]?.info}
<div class="info-panel" class:visible={views[currentStep].info}>
  <h2>{views[currentStep].info.title}</h2>
  <p>{@html views[currentStep].info.content}</p>
</div>
{/if}

<div class="text-content">
  <div class="container">
    <h1>Washed Away!</h1>

    <p style="font-family: Georgia, serif; font-style: oblique; text-align: center; font-weight: 300; color: #555; width: 100%; display: block;"> By Pulaha Roy and Nandita Banerjee</p>

    <p style="font-family: Georgia, serif; font-style: oblique; font-weight: 300; color: #555; line-height: 1.5;">Microsoft&#39;s new AI-powered flood dataset highlights India&#39;s growing vulnerability,
      revealing that one in every 10 square kilometres has experienced flooding over the past
      decade. Globally, the satellite-based analysis shows a steady 5% annual increase in
      flood extent, a troubling trend with far-reaching implications</p>
    
    <p>South Asia — and India in particular — has emerged as a hotspot for flooding in the last
      decade. A new analysis by Down To Earth (DTE) of a global flood mapping project led
      by Microsoft, using artificial intelligence and cloud-penetrating radar satellite imagery,
      reveals that over 10 per cent of India’s landmass has experienced flooding at least once
      in the past decade.</p>
    
    <p>The worst-hit states, predictably, include Bihar, Uttar Pradesh, Assam and West Bengal
      — but leading the charts in raw percentages is Punjab, where 68 per cent of the state’s
      land showed up as flooded.</p>
    
    <p>At first glance, the figures are startling. But researchers caution that not all floods are
      the same. Punjab’s unusually high numbers are largely the result of controlled,
      deliberate inundation of paddy fields, which require standing water through much of the
      agricultural cycle. But what’s happening in Bihar is a very different story.</p>
    
    <p>In Bihar, the water is far from welcome. More than half the state’s area, 51 per cent, has
      faced flooding over the past 10 years, much of it from destructive, recurrent river floods.
      The state’s geography places it at the mercy of the Kosi and Gandak rivers, among
      others, which frequently burst their banks during the monsoon, displacing thousands
      and damaging crops, roads and homes.</p>

    <p>Uttar Pradesh, India’s most populous state, isn’t far behind, with flooding across 25 per
      cent of its area. However, in sheer numbers, close to 60,000 square kilometres in the
      state have been impacted by flooding — the biggest numbers so far. Bihar stood at a
      little over 48,000 sq km and Punjab saw over 34,000 sq km impacted.</p>

    <p>Assam (over 23,000 sq km) and West Bengal (over 31,000 sq km) clocked in at 30 per
      cent and 37 per cent, respectively. Haryana recorded 38 per cent, with roughly 17,000 sq
      km impacted.</p>

    <p>These figures are drawn from a new global flood dataset developed by Microsoft’s AI for
      Good Lab, which combines deep learning models with Sentinel-1 radar satellite imagery
      — a technology that can see through clouds and operate both day and night. The results
      are a part of a wider study in the journal Nature Communications.</p>

    <p>By applying their model to 10 years of satellite data (2014-2024), the team has produced
      the most comprehensive flood mapping dataset available globally, covering both past
      floods and areas at continued risk. What makes this dataset especially powerful is its
      public availability. The flood maps, covering every Sentinel-1 radar image from October
      2014 to September 2024, have been released online along with the full codebase used to
      generate them.</p>

    <h3>What the researchers found</h3>

    <p>India is far from alone in facing a rising tide — the study revealed the extent of flooding
      around the world has been significantly underestimated and Africa has shown some of
      the sharpest increases in flood-prone areas.</p>

    <p>Globally, the model identified 71 per cent more flood-prone land than older methods
      had captured. In Africa, the detected flood extent nearly doubled; in Ethiopia, it nearly
      tripled.</p>

    <p>The same tool was used in Kenya’s deadly 2024 floods, where it provided real-time flood
      maps to emergency responders, helping estimate that 75,000 hectares of cropland had
      been affected; this figure was nearly identical to official ground reports.</p>

    <p>By layering flood detection data with land use maps, the researchers turned their
      attention to agriculture. In Semera, Ethiopia, around 19 per cent of cropland was found
      to lie in historically flooded zones. That’s nearly three times higher than what previous
      satellite records, such as Landsat (7 per cent) and MODIS (2 per cent), had shown.</p>

    <p>But it was in Dolo Ado, further south along the Ganale River, where the contrast was
      most striking: more than half — 52 per cent — of cropland was flagged as flood-prone by
      the new model. Earlier datasets had captured just 1 to 3 per cent. In a region where
      families depend on rain-fed staples like maize and sorghum, such underestimation
      carries heavy consequences — both for food security and rural livelihoods.</p>

    <h3>Bigger, unsettling picture</h3>
    
    <p>When the researchers charted flood data across a 10-year timeline, from 2014 to 2024,
      they spotted what appears to be a slow but steady rise: A 5 per cent annual increase in
      flood extent. If that trend continues, it could mean a 60 per cent rise each decade — a
      shift with far-reaching implications.</p>

    <p>The map of growing risk is not evenly drawn. A belt stretching from Nigeria to Ethiopia
      has seen a clear uptick in flooding, in line with projections from global climate models
      that predict heavier rainfall in parts of Africa. Eastern Australia, too, stands out,
      reflecting a string of extreme weather events in recent years.</p>

    <p>However, the dataset covers just a decade — not quite long enough to draw definitive
      conclusions about climate-driven change, the authors have warned.</p>

    <div class="credits">
      Source: Mapping global floods with 10 years of satellite radar data | Copernicus GLO-30 Digital Elevation Model
    </div>
  </div>
</div>