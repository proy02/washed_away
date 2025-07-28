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
    
    console.log('Setting up progressive loading...');
    
    // Start loading high-res after a delay
    setTimeout(() => {
      console.log('Fading in high-res image...');
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
    console.time('Total Page Load');
    pageLoadStartTime = performance.now();
    console.log(`Script initialization: ${performance.now() - pageLoadStartTime}ms`);
  }
  
  const views = [
    { 
      name: 'India', 
      viewBox: [0, 0, 6000, 6750], 
      labelVisible: false,
      info: {
        title: "India",
        content: `Microsoft has just released a global dataset on floods and India stands out as the biggest hotspot over the past 10 years. More than <span style="background-color: #ca0020; color: white; padding: 4px;">0.3 million square kilometres (sq km) or 10%</span> of the country’s total land area experienced some form of flooding between 2014 and 2024.<br><strong style="color: #888; font-size: 14px; margin-top: 12px; display: block; opacity: 1; transition: opacity 0.3s; text-align: center;">Keep scrolling ↓</strong>`
      }
    },
    { 
      name: 'Punjab+Haryana', 
      viewBox: [900, 1150, 1958, 1100], 
      labelVisible: true,
      // Custom adjustments for different screen sizes
      adjustments: {
        mobile: { offsetX: 0, offsetY: 200, scale: 1.0 }, // Keep as is
        tablet: { offsetX: 0, offsetY: 0, scale: 1.0 },
        desktop: { offsetX: 0, offsetY: 0, scale: 1.0 }
      },
      info: {
        title: "Punjab & Haryana",
        content: `For these two northwestern states, the findings are tricky. Both rely heavily on large-scale paddy cultivation, which involves deliberately flooding vast tracts of farmland. As a result, the data reflects agricultural flooding rather than natural inundation. Even so, the numbers are astounding: In Punjab, <span style="background-color: #ca0020; color: white; padding: 4px;">68%</span> of the total area registered as flooded, while Haryana recorded <span style="background-color: #ca0020; color: white; padding: 4px;">38%</span>.`
      }
    },
    { 
      name: 'Uttar Pradesh', 
      viewBox: [1800, 1800, 1958, 1100], 
      labelVisible: true,
      // Shift west to show western parts, expand width slightly
      adjustments: {
        mobile: { offsetX: -90, offsetY: 0, scale: 0.8 }, // Move left, show more area
        tablet: { offsetX: -150, offsetY: 0, scale: 0.95 },
        desktop: { offsetX: -100, offsetY: 0, scale: 1.0 }
      },
      info: {
        title: "Uttar Pradesh",
        content: `India’s most populous state also recorded the largest flood-affected area in absolute figures: close to <span style="background-color: #ca0020; color: white; padding: 4px;">60,000 sq km</span> or around <span style="background-color: #ca0020; color: white; padding: 4px;">25%</span> of its total area. ` 
      }
    },
    { 
      name: 'Bihar', 
      viewBox: [3100, 2200, 1246, 700], 
      labelVisible: true,
      // Shrink it a little as requested
      adjustments: {
        mobile: { offsetX: -100, offsetY: 100, scale: 0.8 }, // Zoom out (shrink) more on mobile
        tablet: { offsetX: 0, offsetY: 0, scale: 1.15 },
        desktop: { offsetX: 0, offsetY: 0, scale: 1.1 }
      },
      info: {
        title: "Bihar",
        content: `Among India’s most flood-prone and socio-economically vulnerable states, over <span style="background-color: #ca0020; color: white; padding: 4px;">50%</span> of Bihar’s land area has reported natural inundation, underlining the widespread destruction experienced by the state.`
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
        content: `The final stretch of the Indo-Gangetic Plain before the Ganga River flows into Bangladesh, West Bengal has seen <span style="background-color: #ca0020; color: white; padding: 4px;">37%</span> of its territory affected by flooding. `
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
  let isScrollytellingActive = true; // Track if scrollytelling is active
  let showInfoPanel = true; // NEW: Separate control for info panel visibility
  
  const DURATION = 1000;
  
  // ENHANCED: Helper function with custom adjustments per step and screen size
  function getResponsiveViewBox(viewBoxArray, windowWidth, windowHeight, step) {
    if (browser) console.time(`getResponsiveViewBox-step-${step}`);
    
    const [x, y, width, height] = viewBoxArray;
    
    console.log(`\n=== getResponsiveViewBox DEBUG ===`);
    console.log(`Step: ${step} (${views[step].name}), Screen: ${windowWidth}x${windowHeight}`);
    console.log(`Original viewBox: [${x}, ${y}, ${width}, ${height}]`);
    
    // For India overview (step 0), no zoom needed
    if (step === 0) {
      console.log(`Step 0: India overview - no zoom applied`);
      const result = adjustForAspectRatio(x, y, width, height, windowWidth, windowHeight);
      if (browser) console.timeEnd(`getResponsiveViewBox-step-${step}`);
      return result;
    }
    
    // Get screen size category and base zoom factor
    let zoomFactor = 1;
    let screenCategory = 'desktop';
    
    if (windowWidth <= 480) {
      zoomFactor = 2.0;
      screenCategory = 'mobile';
      console.log(`Very small mobile (≤480px): base zoom factor = ${zoomFactor}`);
    } else if (windowWidth <= 768) {
      zoomFactor = 1.6;
      screenCategory = 'mobile';
      console.log(`Mobile (≤768px): base zoom factor = ${zoomFactor}`);
    } else if (windowWidth <= 1024) {
      zoomFactor = 1.3;
      screenCategory = 'tablet';
      console.log(`Tablet (≤1024px): base zoom factor = ${zoomFactor}`);
    } else {
      zoomFactor = 1.0;
      screenCategory = 'desktop';
      console.log(`Desktop (>1024px): base zoom factor = ${zoomFactor}`);
    }
    
    // Apply custom adjustments for this step and screen size
    const adjustments = views[step].adjustments?.[screenCategory] || { offsetX: 0, offsetY: 0, scale: 1.0 };
    console.log(`Custom adjustments for ${views[step].name} on ${screenCategory}:`, adjustments);
    
    // Apply custom scale to zoom factor
    const finalZoomFactor = zoomFactor * adjustments.scale;
    console.log(`Final zoom factor: ${zoomFactor} * ${adjustments.scale} = ${finalZoomFactor}`);
    
    // Apply zoom by reducing viewBox dimensions (zooms in)
    const zoomedWidth = width / finalZoomFactor;
    const zoomedHeight = height / finalZoomFactor;
    
    console.log(`Applying zoom: ${width} x ${height} -> ${zoomedWidth} x ${zoomedHeight}`);
    
    // Calculate center with custom offset
    const centerX = x + width / 2 + adjustments.offsetX;
    const centerY = y + height / 2 + adjustments.offsetY;
    
    const newX = centerX - zoomedWidth / 2;
    const newY = centerY - zoomedHeight / 2;
    
    console.log(`Original center: (${x + width / 2}, ${y + height / 2})`);
    console.log(`Adjusted center with offset: (${centerX}, ${centerY})`);
    console.log(`New zoomed position: (${newX}, ${newY})`);
    
    // Now adjust for aspect ratio
    const result = adjustForAspectRatio(newX, newY, zoomedWidth, zoomedHeight, windowWidth, windowHeight);
    console.log(`Final result after aspect adjustment: [${result.map(v => v.toFixed(2)).join(', ')}]`);
    console.log(`=== END DEBUG ===\n`);
    
    if (browser) console.timeEnd(`getResponsiveViewBox-step-${step}`);
    return result;
  }
  
  // Helper function to adjust viewBox for screen aspect ratio
  function adjustForAspectRatio(x, y, width, height, windowWidth, windowHeight) {
    if (browser) console.time('adjustForAspectRatio');
    
    const viewBoxAspectRatio = width / height;
    const windowAspectRatio = windowWidth / windowHeight;
    
    console.log(`[adjustForAspectRatio] ViewBox: ${width}x${height} (ratio: ${viewBoxAspectRatio.toFixed(2)})`);
    console.log(`[adjustForAspectRatio] Window: ${windowWidth}x${windowHeight} (ratio: ${windowAspectRatio.toFixed(2)})`);
    
    let result;
    // If window is wider than viewBox, expand width to fit
    if (windowAspectRatio > viewBoxAspectRatio) {
      const adjustedWidth = height * windowAspectRatio;
      const widthDiff = adjustedWidth - width;
      const newX = x - widthDiff / 2;
      console.log(`[adjustForAspectRatio] Wide window: expanding width ${width} -> ${adjustedWidth}`);
      result = [newX, y, adjustedWidth, height];
    } 
    // If window is taller than viewBox, expand height to fit
    else {
      const adjustedHeight = width / windowAspectRatio;
      const heightDiff = adjustedHeight - height;
      const newY = y - heightDiff / 2;
      console.log(`[adjustForAspectRatio] Tall window: expanding height ${height} -> ${adjustedHeight}`);
      result = [x, newY, width, adjustedHeight];
    }
    
    if (browser) console.timeEnd('adjustForAspectRatio');
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
    if (browser) console.time(`updateViewBox-step-${step}`);
    console.log(`[updateViewBox] Called with step=${step}, width=${width}, height=${height}`);
  
    // Use the original viewBox coordinates with responsive adjustments
    const originalViewBox = views[step].viewBox;
    const responsiveViewBox = getResponsiveViewBox(originalViewBox, width, height, step);
  
    console.log(`[updateViewBox] Original viewBox: ${originalViewBox.join(' ')}`);
    console.log(`[updateViewBox] Responsive viewBox: ${responsiveViewBox.join(' ')}`);
  
    viewBoxStore.set(responsiveViewBox);
    if (browser) console.timeEnd(`updateViewBox-step-${step}`);
  }
  
  function onScroll() {
    if (browser) console.time('onScroll');
    const scrollY = window.scrollY;
    const height = window.innerHeight;
    // Add one extra viewport height so the last step is fully visible
    const totalScrollytellingHeight = (views.length + 1) * height;
    
    // Check if we're still in the scrollytelling section
    if (scrollY < totalScrollytellingHeight) {
      isScrollytellingActive = true;
      const step = Math.min(views.length - 1, Math.floor(scrollY / height));
      console.log(`[onScroll] scrollY=${scrollY}, calculated step=${step}, totalHeight=${totalScrollytellingHeight}`);
  
      if (step !== currentStep) {
        console.log(`[onScroll] Step changed from ${currentStep} to ${step}`);
        currentStep = step;
        updateViewBox(currentStep, window.innerWidth, window.innerHeight);
      }
      
      // NEW: Hide info panel during the last viewport (the extra one for transition)
      const lastStepThreshold = views.length * height; // Start of the extra viewport
      showInfoPanel = scrollY < lastStepThreshold;
      
    } else {
      // We've scrolled past the scrollytelling section
      isScrollytellingActive = false;
      showInfoPanel = false; // Also hide info panel
      console.log(`[onScroll] Past scrollytelling section, hiding SVG`);
    }
    
    if (browser) console.timeEnd('onScroll');
  }
  
  function onResize() {
    if (browser) console.time('onResize');
    console.log(`[onResize] window.innerWidth=${window.innerWidth}, window.innerHeight=${window.innerHeight}`);
    updateViewBox(currentStep, window.innerWidth, window.innerHeight);
    if (browser) console.timeEnd('onResize');
  }
  
  // Monitor the loading time of the large image
  function monitorLargeImageLoad() {
    if (!browser) return;
    
    console.log('Starting to monitor SVG large image load...');
    imageLoadStart = performance.now();
    
    // Try to find the SVG and image element
    setTimeout(() => {
      const svgElement = document.querySelector('.svg-wrapper svg');
      if (svgElement) {
        // Try with various xlink:href syntaxes since SVG namespaces can be tricky
        const imageElement = 
          svgElement.querySelector('image[xlink\\:href="large_image.png"]') || 
          svgElement.querySelector('image[href="large_image.png"]') ||
          svgElement.querySelector('image');
        
        if (imageElement) {
          console.log('Found large image element in SVG');
          
          // Check if already loaded
          if (imageElement.complete) {
            imageLoadEnd = performance.now();
            const loadTime = imageLoadEnd - imageLoadStart;
            console.log(`❗ Large image (14MB) already loaded in approximately ${loadTime.toFixed(2)}ms`);
          } else {
            console.log('Large image is still loading, waiting...');
            
            // Add load event listener
            imageElement.addEventListener('load', () => {
              imageLoadEnd = performance.now();
              const loadTime = imageLoadEnd - imageLoadStart;
              console.log(`❗ Large image (14MB) loaded in ${loadTime.toFixed(2)}ms`);
            });
            
            // Add error event listener
            imageElement.addEventListener('error', () => {
              console.error(`❌ Error loading large image after ${performance.now() - imageLoadStart}ms`);
            });
          }
        } else {
          console.warn('Could not find the large image element in SVG');
        }
      } else {
        console.warn('Could not find SVG element');
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
        
        console.log('\n===== LARGE IMAGE PERFORMANCE METRICS =====');
        console.log(`URL: ${largeImage.name}`);
        console.log(`• Total load time: ${totalLoadTime.toFixed(2)}ms`);
        console.log(`• Network latency: ${networkTime.toFixed(2)}ms`);
        console.log(`• Download time: ${downloadTime.toFixed(2)}ms`);
        console.log(`• Transfer size: ${size}MB`);
        console.log(`• Decoded size: ${largeImage.decodedBodySize ? (largeImage.decodedBodySize / (1024 * 1024)).toFixed(2) : 'unknown'}MB`);
        console.log('===========================================\n');
        
        // If the image is over 1MB, log a warning
        if (largeImage.transferSize > 1024 * 1024) {
          console.warn(`⚠️ Large image is ${size}MB - consider optimizing for faster load times`);
          
          // Suggest optimizations if load time is significant
          if (totalLoadTime > 500) {
            console.warn(`⚠️ Image took ${(totalLoadTime/1000).toFixed(2)}s to load - consider optimizing:`);
            console.warn(`   1. Resize to appropriate dimensions`);
            console.warn(`   2. Use image compression`);
            console.warn(`   3. Consider WebP format`);
            console.warn(`   4. Implement progressive loading`);
          }
        }
      } else {
        console.warn('Could not find large_image.png in performance entries');
        
        // Log all image resources found
        const imageResources = resources.filter(r => 
          r.initiatorType === 'img' || 
          r.name.includes('.png') || 
          r.name.includes('.jpg') || 
          r.name.includes('.svg')
        );
        
        if (imageResources.length > 0) {
          console.log('Found these image resources instead:');
          imageResources.forEach(img => {
            const size = img.transferSize ? (img.transferSize / 1024).toFixed(2) : 'unknown';
            console.log(`- ${img.name} (${size}KB)`);
          });
        }
      }
      
      // Also find the largest resource overall
      if (resources.length > 0) {
        const largestResource = resources.reduce((prev, current) => 
          (prev.transferSize > current.transferSize) ? prev : current
        );
        
        if (largestResource.transferSize > 1024 * 1024) {
          console.log(`\nℹ️ Largest resource: ${largestResource.name}`);
          console.log(`• Size: ${(largestResource.transferSize / (1024 * 1024)).toFixed(2)}MB`);
          console.log(`• Load time: ${(largestResource.responseEnd - largestResource.startTime).toFixed(2)}ms\n`);
        }
      }
    }, 200);
  }
  
  onMount(() => {
    if (!browser) return;
    
    console.log('Component mounted at', performance.now() - pageLoadStartTime, 'ms');
    console.time('onMount-setupAndInitialRender');
    
    // Start monitoring the large image load
    monitorLargeImageLoad();
  
    setupProgressiveLoading();
    
    window.addEventListener('scroll', onScroll);
    window.addEventListener('resize', onResize);
  
    // Initial update
    console.time('initialViewBoxUpdate');
    updateViewBox(currentStep, window.innerWidth, window.innerHeight);
    console.timeEnd('initialViewBoxUpdate');
    
    // Monitor when DOM is fully loaded
    if (document.readyState === 'loading') {
      document.addEventListener('DOMContentLoaded', () => {
        console.log('DOMContentLoaded at', performance.now() - pageLoadStartTime, 'ms');
      });
    } else {
      console.log('DOMContentLoaded already fired at', performance.now() - pageLoadStartTime, 'ms');
    }
    
    // Monitor when all resources (images, SVGs) are loaded
    if (document.readyState === 'complete') {
      const loadTime = performance.now() - pageLoadStartTime;
      console.log(`All resources already loaded at ${loadTime.toFixed(2)}ms`);
      console.timeEnd('Total Page Load');
      checkResourcePerformance();
    } else {
      window.addEventListener('load', () => {
        const loadTime = performance.now() - pageLoadStartTime;
        console.log(`All resources loaded at ${loadTime.toFixed(2)}ms`);
        console.timeEnd('Total Page Load');
        checkResourcePerformance();
      });
    }
    
    console.timeEnd('onMount-setupAndInitialRender');
    
    return () => {
      window.removeEventListener('scroll', onScroll);
      window.removeEventListener('resize', onResize);
    };
  });
  
  // Reactive statement to handle window size changes
  $: if (browser && innerWidth && innerHeight) {
    updateViewBox(currentStep, innerWidth, innerHeight);
  }
  
</script>

<style>
  
  html, body {
    margin: 0;
    padding: 0;
    height: 100%;
    overflow-x: hidden;
  }

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
    transform: translateY(0%) translateX(0);
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

  .text-content h2 {
    font-size: 2rem;
    margin: 3rem 0 1.5rem 0;
    color: #2c2c2c;
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
    }

    .info-panel.visible {
      transform: none;
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

    .text-content h2 {
      font-size: 1.5rem;
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
</style>

<svelte:window bind:innerWidth bind:innerHeight />

<div class="svg-wrapper" class:hidden={!isScrollytellingActive}>
  <svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox={viewBoxString} preserveAspectRatio="xMidYMid meet">
    <!-- <defs>
      <filter id="luminosity-noclip" x="1690.9" y="1881.4" width="220" height="56" color-interpolation-filters="sRGB" filterUnits="userSpaceOnUse">
        <feFlood flood-color="#fff" result="bg"/>
        <feBlend in="SourceGraphic" in2="bg"/>
      </filter>
      <mask id="mask" x="1690.9" y="1881.4" width="220" height="56" maskUnits="userSpaceOnUse">
        <g filter="url(#luminosity-noclip)">
          <image width="220" height="56" transform="translate(1690.9 1881.4)" xlink:href="large_image.png"/>
        </g>
      </mask>
      <filter id="luminosity-noclip-2" x="1555.9" y="1557.7" width="192" height="56" color-interpolation-filters="sRGB" filterUnits="userSpaceOnUse">
        <feFlood flood-color="#fff" result="bg"/>
        <feBlend in="SourceGraphic" in2="bg"/>
      </filter>
      <mask id="mask-1" x="1555.9" y="1557.7" width="192" height="56" maskUnits="userSpaceOnUse">
        <g filter="url(#luminosity-noclip-2)">
          <image width="192" height="56" transform="translate(1555.9 1557.7)" xlink:href="large_image.png"/>
        </g>
      </mask>
      <filter id="luminosity-noclip-3" x="2590.3" y="2416.4" width="376" height="56" color-interpolation-filters="sRGB" filterUnits="userSpaceOnUse">
        <feFlood flood-color="#fff" result="bg"/>
        <feBlend in="SourceGraphic" in2="bg"/>
      </filter>
      <mask id="mask-2" x="2590.3" y="2416.4" width="376" height="56" maskUnits="userSpaceOnUse">
        <g filter="url(#luminosity-noclip-3)">
          <image width="376" height="56" transform="translate(2590.3 2416.4)" xlink:href="large_image.png"/>
        </g>
      </mask>
      <filter id="luminosity-noclip-4" x="3528.8" y="2556.9" width="149" height="56" color-interpolation-filters="sRGB" filterUnits="userSpaceOnUse">
        <feFlood flood-color="#fff" result="bg"/>
        <feBlend in="SourceGraphic" in2="bg"/>
      </filter>
      <mask id="mask-3" x="3528.8" y="2556.9" width="149" height="56" maskUnits="userSpaceOnUse">
        <g filter="url(#luminosity-noclip-4)">
          <image width="149" height="56" transform="translate(3528.8 2556.9)" xlink:href="large_image.png"/>
        </g>
      </mask>
      <filter id="luminosity-noclip-5" x="3745.2" y="3112.5" width="339" height="56" color-interpolation-filters="sRGB" filterUnits="userSpaceOnUse">
        <feFlood flood-color="#fff" result="bg"/>
        <feBlend in="SourceGraphic" in2="bg"/>
      </filter>
      <mask id="mask-4" x="3745.2" y="3112.5" width="339" height="56" maskUnits="userSpaceOnUse">
        <g filter="url(#luminosity-noclip-5)">
          <image width="339" height="56" transform="translate(3745.2 3112.5)" xlink:href="large_image.png"/>
        </g>
      </mask>
      <filter id="luminosity-noclip-6" x="4991.9" y="2336.9" width="169" height="56" color-interpolation-filters="sRGB" filterUnits="userSpaceOnUse">
        <feFlood flood-color="#fff" result="bg"/>
        <feBlend in="SourceGraphic" in2="bg"/>
      </filter>
      <mask id="mask-5" x="4991.9" y="2336.9" width="169" height="56" maskUnits="userSpaceOnUse">
        <g filter="url(#luminosity-noclip-6)">
          <image width="169" height="56" transform="translate(4991.9 2336.9)" xlink:href="large_image.png"/>
        </g>
      </mask>
    </defs> -->
    <g id="Layer_1" data-name="Layer 1">
      <image width="6000" height="6750" 
       xlink:href="large_image_placeholder.jpg" 
       class="placeholder-image"/>

      <image width="6000" height="6750" 
        xlink:href="large_image.png" 
        class="high-res-image" 
        style="opacity: 0;"
        bind:this={highResImage}/>

        <g class="chart-elements" class:visible={currentStep === 0}>
        <rect x="3127.2" y="1012.9" width="87.8" height="87.8" fill="#ca0020" stroke-width="0"/>
        <g>
          <path d="m3282.9,965.6h57.4v12.4h-43.9v46.4h43.9v12.2h-43.9v62.1h-13.5v-133.2Z" stroke-width="0"/>
          <path d="m3360.4,965.6h12.1v133.2h-12.1v-133.2Z" stroke-width="0"/>
          <path d="m3392.5,1049.7c0-27.9,11.5-51.1,37.8-51.1s37.6,23.2,37.6,51.3-11.5,51.3-37.6,51.3-37.8-23.2-37.8-51.3v-.2Zm12.6.4c0,25.9,8.5,39.8,25.4,39.8s24.8-13.9,24.8-40-8.5-40-25.2-40-25,13.9-25,40v.2Z" stroke-width="0"/>
          <path d="m3481.2,1049.7c0-27.9,11.5-51.1,37.8-51.1s37.6,23.2,37.6,51.3-11.5,51.3-37.6,51.3-37.8-23.2-37.8-51.3v-.2Zm12.6.4c0,25.9,8.5,39.8,25.4,39.8s24.8-13.9,24.8-40-8.5-40-25.2-40-25,13.9-25,40v.2Z" stroke-width="0"/>
          <path d="m3639.6,1098.8h-11.2c0-9,0-14.4.2-19.4-3.4,13.5-13.5,21.8-25.9,21.8-19.4,0-32.8-20.5-32.8-51.1s12.2-51.5,32.8-51.5,20,5.6,25.2,20.7c-.4-5.8-.4-11.2-.4-18.4v-35.3h12.1v133.2Zm-57.6-49c0,22.1,7.9,39.2,22.9,39.2s13.1-4.1,17.1-10.8c4.1-6.8,5.6-15.7,5.6-28.6s-1.4-21.8-5.6-28.6c-4-6.5-9.4-10.6-17.1-10.6-14.9,0-22.9,16.9-22.9,39.2v.2Z" stroke-width="0"/>
          <path d="m3664.8,965.6h12.1v17.6h-12.1v-17.6Zm0,35.3h12.1v97.9h-12.1v-97.9Z" stroke-width="0"/>
          <path d="m3702.1,1000.9h10.8c0,5.6,0,10.6-.4,16.7,5-13.9,16.4-19.1,27.7-19.1,16.7,0,28.1,9.7,28.1,33.5v66.8h-12.1v-63c0-9.9-1.4-25.4-18.7-25.4s-23.4,9.9-23.4,30.4v58h-12.1v-97.9Z" stroke-width="0"/>
          <path d="m3856,1097c0,14-2.9,23-8.3,29.3-6.1,7.2-14.9,10.8-26.3,10.8s-31.5-7-32.9-30.8h12.2c.7,6.3,2.2,10.1,5.6,13.7,4.3,4.3,10.1,5.9,15.7,5.9,10.3,0,22.1-4.7,22.1-28.1s0-11,.2-17.5c-3.6,13-12.6,20.7-25.2,20.7-21.1,0-32.8-21.2-32.8-51.1s11.7-51.5,32.8-51.5,20.5,5.4,26.5,21.2c-.2-5.9-.2-11.5-.2-18.9h10.6v96.1Zm-57.6-47c0,22.1,7.9,39.2,22.9,39.2s13.1-4.1,17.1-10.8c4.1-6.8,5.6-15.8,5.6-28.6s-1.4-22-5.6-28.6c-4-6.7-9.4-10.8-17.1-10.8-14.9,0-22.9,17.1-22.9,39.4v.2Z" stroke-width="0"/>
          <path d="m3923.1,1000.9h11.2c0,9,0,14.6-.4,20.5,4.9-19.1,17.5-22.5,26.1-22.9v14c-1.4-.9-2.9-1.3-4.5-1.3-8.3,0-20.3,10.3-20.3,34.9v52.6h-12.1v-97.9Z" stroke-width="0"/>
          <path d="m3978.6,1053.5c0,20.3,8.3,36.4,24.8,36.4s19.4-8.1,22-19.4h11.9c-1.8,10.1-6.3,17.8-12.2,23-5.9,5-13.3,7.7-21.6,7.7-27,0-37.4-25.9-37.4-51.1s14-51.5,37.1-51.5,20.5,5.2,26.3,13.9c7.9,11.7,9.2,26.8,9,41h-59.8Zm47.5-10.6c-.4-20.3-8.8-34-23.6-34s-23.6,14.6-23.9,34h47.5Z" stroke-width="0"/>
          <path d="m4056.9,1000.9h11.2c0,7.4,0,12.8-.2,18.7,5-14.8,14-21.1,25.7-21.1,19.6,0,32.9,20.3,32.9,51.5s-12.2,51.1-32.9,51.1-21.1-7.9-25-20.2c.4,4.3.4,9.4.4,17.8v36.2h-12.1v-134.1Zm57.6,48.8c0-22.1-7.7-39.2-22.9-39.2s-13,4.1-16.7,10.8c-4.3,6.7-5.9,15.7-5.9,28.6s1.6,21.8,5.9,28.6c3.8,6.7,9.2,10.8,16.7,10.8,15.1,0,22.9-17.1,22.9-39.4v-.2Z" stroke-width="0"/>
          <path d="m4139.8,1049.7c0-27.9,11.5-51.1,37.8-51.1s37.6,23.2,37.6,51.3-11.5,51.3-37.6,51.3-37.8-23.2-37.8-51.3v-.2Zm12.6.4c0,25.9,8.5,39.8,25.4,39.8s24.8-13.9,24.8-40-8.5-40-25.2-40-25,13.9-25,40v.2Z" stroke-width="0"/>
          <path d="m4233.8,1000.9h11.2c0,9,0,14.6-.4,20.5,4.9-19.1,17.5-22.5,26.1-22.9v14c-1.4-.9-2.9-1.3-4.5-1.3-8.3,0-20.3,10.3-20.3,34.9v52.6h-12.1v-97.9Z" stroke-width="0"/>
          <path d="m4289.2,1012.3h-13.7v-11.3h13.7v-31h12.1v31h13.5v11.3h-13.5v86.6h-12.1v-86.6Z" stroke-width="0"/>
          <path d="m4335,1053.5c0,20.3,8.3,36.4,24.8,36.4s19.4-8.1,22-19.4h11.9c-1.8,10.1-6.3,17.8-12.2,23-5.9,5-13.3,7.7-21.6,7.7-27,0-37.4-25.9-37.4-51.1s14-51.5,37.1-51.5,20.5,5.2,26.3,13.9c7.9,11.7,9.2,26.8,9,41h-59.8Zm47.5-10.6c-.4-20.3-8.8-34-23.6-34s-23.6,14.6-23.9,34h47.5Z" stroke-width="0"/>
          <path d="m4477.7,1098.8h-11.2c0-9,0-14.4.2-19.4-3.4,13.5-13.5,21.8-25.9,21.8-19.4,0-32.8-20.5-32.8-51.1s12.2-51.5,32.8-51.5,20,5.6,25.2,20.7c-.4-5.8-.4-11.2-.4-18.4v-35.3h12.1v133.2Zm-57.6-49c0,22.1,7.9,39.2,22.9,39.2s13.1-4.1,17.1-10.8c4.1-6.8,5.6-15.7,5.6-28.6s-1.4-21.8-5.6-28.6c-4-6.5-9.4-10.6-17.1-10.6-14.9,0-22.9,16.9-22.9,39.2v.2Z" stroke-width="0"/>
        </g>
        <rect x="3127.2" y="1242.8" width="87.8" height="87.8" fill="#fff" stroke="#231f20" stroke-miterlimit="10" stroke-width="5"/>
        <g>
          <path d="m3353.6,1306.6v-111.1h12.2v133.2h-15.1l-55.1-113.8v113.8h-12.2v-133.2h16.2l54,111.1Z" stroke-width="0"/>
          <path d="m3384.9,1279.6c0-27.9,11.5-51.1,37.8-51.1s37.6,23.2,37.6,51.3-11.5,51.3-37.6,51.3-37.8-23.2-37.8-51.3v-.2Zm12.6.4c0,25.9,8.5,39.8,25.4,39.8s24.8-13.9,24.8-40-8.5-40-25.2-40-25,13.9-25,40v.2Z" stroke-width="0"/>
          <path d="m3537.6,1242.2v86.6h-12.1v-86.6h-11.2v-11.3h11.2v-7.4c0-29.3,23.2-30.2,29.2-30.2v10.6c-9,.5-17.1,4.9-17.1,20.7v6.3h25.2v-35.3h12.1v133.2h-12.1v-86.6h-25.2Z" stroke-width="0"/>
          <path d="m3590.5,1279.6c0-27.9,11.5-51.1,37.8-51.1s37.6,23.2,37.6,51.3-11.5,51.3-37.6,51.3-37.8-23.2-37.8-51.3v-.2Zm12.6.4c0,25.9,8.5,39.8,25.4,39.8s24.8-13.9,24.8-40-8.5-40-25.2-40-25,13.9-25,40v.2Z" stroke-width="0"/>
          <path d="m3679.2,1279.6c0-27.9,11.5-51.1,37.8-51.1s37.6,23.2,37.6,51.3-11.5,51.3-37.6,51.3-37.8-23.2-37.8-51.3v-.2Zm12.6.4c0,25.9,8.5,39.8,25.4,39.8s24.8-13.9,24.8-40-8.5-40-25.2-40-25,13.9-25,40v.2Z" stroke-width="0"/>
          <path d="m3837.6,1328.8h-11.2c0-9,0-14.4.2-19.4-3.4,13.5-13.5,21.8-25.9,21.8-19.4,0-32.8-20.5-32.8-51.1s12.2-51.5,32.8-51.5,20,5.6,25.2,20.7c-.4-5.8-.4-11.2-.4-18.4v-35.3h12.1v133.2Zm-57.6-49c0,22.1,7.9,39.2,22.9,39.2s13.1-4.1,17.1-10.8c4.1-6.8,5.6-15.7,5.6-28.6s-1.4-21.8-5.6-28.6c-4-6.5-9.4-10.6-17.1-10.6-14.9,0-22.9,16.9-22.9,39.2v.2Z" stroke-width="0"/>
          <path d="m3862.8,1195.6h12.1v17.6h-12.1v-17.6Zm0,35.3h12.1v97.9h-12.1v-97.9Z" stroke-width="0"/>
          <path d="m3900.1,1230.9h10.8c0,5.6,0,10.6-.4,16.7,5-13.9,16.4-19.1,27.7-19.1,16.7,0,28.1,9.7,28.1,33.5v66.8h-12.1v-63c0-9.9-1.4-25.4-18.7-25.4s-23.4,9.9-23.4,30.4v58h-12.1v-97.9Z" stroke-width="0"/>
          <path d="m4054,1327c0,14-2.9,23-8.3,29.3-6.1,7.2-14.9,10.8-26.3,10.8s-31.5-7-32.9-30.8h12.2c.7,6.3,2.2,10.1,5.6,13.7,4.3,4.3,10.1,5.9,15.7,5.9,10.3,0,22.1-4.7,22.1-28.1s0-11,.2-17.5c-3.6,13-12.6,20.7-25.2,20.7-21.1,0-32.8-21.2-32.8-51.1s11.7-51.5,32.8-51.5,20.5,5.4,26.5,21.2c-.2-5.9-.2-11.5-.2-18.9h10.6v96.1Zm-57.6-47c0,22.1,7.9,39.2,22.9,39.2s13.1-4.1,17.1-10.8c4.1-6.8,5.6-15.8,5.6-28.6s-1.4-22-5.6-28.6c-4-6.7-9.4-10.8-17.1-10.8-14.9,0-22.9,17.1-22.9,39.4v.2Z" stroke-width="0"/>
        </g>
        <rect x="3127.2" y="1472.8" width="87.8" height="87.8" fill="#e5e5e5" stroke="#231f20" stroke-miterlimit="10" stroke-width="5"/>
        <g>
          <path d="m3277.6,1491.9c0-39.1,16.6-68.8,48.4-68.8s48.4,29.7,48.4,68.9-13.5,68.9-48.4,68.9-48.4-29.7-48.4-68.9v-.2Zm13.9.4c0,36.4,13.1,56.5,34.6,56.5s34.7-20.2,34.7-56.7-13.3-56.7-34.7-56.7-34.6,20.2-34.6,56.7v.2Z" stroke-width="0"/>
          <path d="m3393.2,1425.5h12.1v35.3c0,8.6,0,14-.4,18.7,3.6-12.6,12.6-21.1,25-21.1,20.7,0,32.9,21.2,32.9,51.1s-13.3,51.5-32.9,51.5-20.3-5.8-25.7-21.2c.2,5.9.2,11.5.2,18.9h-11.2v-133.2Zm57.6,84.1c0-21.1-7.4-39.2-22.7-39.2s-22.9,19.8-22.9,39.4,5.9,39.4,22.9,39.4,22.7-18.2,22.7-39.4v-.2Z" stroke-width="0"/>
          <path d="m3520.8,1486.2c0-12.4-6.5-16.9-15.8-16.9s-14.2,5.9-14.2,14.6,9.2,14.9,21.1,18.5c10.8,3.2,23.4,9.9,23.4,29.5s-12.4,29.2-29.2,29.2-29.7-10.6-30.6-31.1h11.9c.2,13.3,7.9,20,18.7,20s16.6-7.2,16.6-17.1-8.3-16.6-21.4-20.7c-12.4-4.1-23-10.4-23-27.9s11.3-25.7,27.4-25.7,26.6,9.5,27.2,27.7h-11.9Z" stroke-width="0"/>
          <path d="m3561.7,1513.4c0,20.3,8.3,36.4,24.8,36.4s19.4-8.1,22-19.4h11.9c-1.8,10.1-6.3,17.8-12.2,23-5.9,5-13.3,7.7-21.6,7.7-27,0-37.4-25.9-37.4-51.1s14-51.5,37.1-51.5,20.5,5.2,26.3,13.9c7.9,11.7,9.2,26.8,9,41h-59.8Zm47.5-10.6c-.4-20.3-8.8-34-23.6-34s-23.6,14.6-23.9,34h47.5Z" stroke-width="0"/>
          <path d="m3640,1460.8h11.2c0,9,0,14.6-.4,20.5,4.9-19.1,17.5-22.5,26.1-22.9v14c-1.4-.9-2.9-1.3-4.5-1.3-8.3,0-20.3,10.3-20.3,34.9v52.6h-12.1v-97.9Z" stroke-width="0"/>
          <path d="m3722.1,1558.7h-12.1l-29.5-97.9h12.2l23.2,81.7,22.5-81.7h12.1l-28.4,97.9Z" stroke-width="0"/>
          <path d="m3825.7,1558.7h-11.2c0-8.6,0-14,.2-18.7-4.3,13.3-13.5,21.1-25.9,21.1-19.4,0-32.8-20.5-32.8-51.1s12.2-51.5,32.8-51.5,20.7,5.6,25.9,21.2c-.2-5.9-.2-11.5-.2-18.9h11.2v97.9Zm-57.6-48.8c0,22.1,7.9,39.2,22.9,39.2s13.1-4.1,17.1-10.8c4.1-6.8,5.6-15.8,5.6-28.6s-1.4-22-5.6-28.6c-4-6.7-9.4-10.8-17.1-10.8-14.9,0-22.9,17.1-22.9,39.4v.2Z" stroke-width="0"/>
          <path d="m3854.2,1472.1h-13.7v-11.3h13.7v-31h12.1v31h13.5v11.3h-13.5v86.6h-12.1v-86.6Z" stroke-width="0"/>
          <path d="m3895.9,1425.5h12.1v17.6h-12.1v-17.6Zm0,35.3h12.1v97.9h-12.1v-97.9Z" stroke-width="0"/>
          <path d="m3928,1509.6c0-27.9,11.5-51.1,37.8-51.1s37.6,23.2,37.6,51.3-11.5,51.3-37.6,51.3-37.8-23.2-37.8-51.3v-.2Zm12.6.4c0,25.9,8.5,39.8,25.4,39.8s24.8-13.9,24.8-40-8.5-40-25.2-40-25,13.9-25,40v.2Z" stroke-width="0"/>
          <path d="m4021.9,1460.8h10.8c0,5.6,0,10.6-.4,16.7,5-13.9,16.4-19.1,27.7-19.1,16.7,0,28.1,9.7,28.1,33.5v66.8h-12.1v-63c0-9.9-1.4-25.4-18.7-25.4s-23.4,9.9-23.4,30.4v58h-12.1v-97.9Z" stroke-width="0"/>
          <path d="m4154.8,1460.8h11.2c0,9,0,14.6-.4,20.5,4.9-19.1,17.5-22.5,26.1-22.9v14c-1.4-.9-2.9-1.3-4.5-1.3-8.3,0-20.3,10.3-20.3,34.9v52.6h-12.1v-97.9Z" stroke-width="0"/>
          <path d="m4210.2,1513.4c0,20.3,8.3,36.4,24.8,36.4s19.4-8.1,22-19.4h11.9c-1.8,10.1-6.3,17.8-12.2,23-5.9,5-13.3,7.7-21.6,7.7-27,0-37.4-25.9-37.4-51.1s14-51.5,37.1-51.5,20.5,5.2,26.3,13.9c7.9,11.7,9.2,26.8,9,41h-59.8Zm47.5-10.6c-.4-20.3-8.8-34-23.6-34s-23.6,14.6-23.9,34h47.5Z" stroke-width="0"/>
          <path d="m4327.9,1486.2c0-12.4-6.5-16.9-15.8-16.9s-14.2,5.9-14.2,14.6,9.2,14.9,21.1,18.5c10.8,3.2,23.4,9.9,23.4,29.5s-12.4,29.2-29.2,29.2-29.7-10.6-30.6-31.1h11.9c.2,13.3,7.9,20,18.7,20s16.6-7.2,16.6-17.1-8.3-16.6-21.4-20.7c-12.4-4.1-23-10.4-23-27.9s11.3-25.7,27.4-25.7,26.6,9.5,27.2,27.7h-11.9Z" stroke-width="0"/>
          <path d="m4368.8,1513.4c0,20.3,8.3,36.4,24.8,36.4s19.4-8.1,22-19.4h11.9c-1.8,10.1-6.3,17.8-12.2,23-5.9,5-13.3,7.7-21.6,7.7-27,0-37.4-25.9-37.4-51.1s14-51.5,37.1-51.5,20.5,5.2,26.3,13.9c7.9,11.7,9.2,26.8,9,41h-59.8Zm47.5-10.6c-.4-20.3-8.8-34-23.6-34s-23.6,14.6-23.9,34h47.5Z" stroke-width="0"/>
          <path d="m4447.1,1460.8h11.2c0,9,0,14.6-.4,20.5,4.9-19.1,17.5-22.5,26.1-22.9v14c-1.4-.9-2.9-1.3-4.5-1.3-8.3,0-20.3,10.3-20.3,34.9v52.6h-12.1v-97.9Z" stroke-width="0"/>
          <path d="m4529.2,1558.7h-12.1l-29.5-97.9h12.2l23.2,81.7,22.5-81.7h12.1l-28.4,97.9Z" stroke-width="0"/>
          <path d="m4576.1,1513.4c0,20.3,8.3,36.4,24.8,36.4s19.4-8.1,22-19.4h11.9c-1.8,10.1-6.3,17.8-12.2,23-5.9,5-13.3,7.7-21.6,7.7-27,0-37.4-25.9-37.4-51.1s14-51.5,37.1-51.5,20.5,5.2,26.3,13.9c7.9,11.7,9.2,26.8,9,41h-59.8Zm47.5-10.6c-.4-20.3-8.8-34-23.6-34s-23.6,14.6-23.9,34h47.5Z" stroke-width="0"/>
          <path d="m4718.9,1558.7h-11.2c0-9,0-14.4.2-19.4-3.4,13.5-13.5,21.8-25.9,21.8-19.4,0-32.8-20.5-32.8-51.1s12.2-51.5,32.8-51.5,20,5.6,25.2,20.7c-.4-5.8-.4-11.2-.4-18.4v-35.3h12.1v133.2Zm-57.6-49c0,22.1,7.9,39.2,22.9,39.2s13.1-4.1,17.1-10.8c4.1-6.8,5.6-15.7,5.6-28.6s-1.4-21.8-5.6-28.6c-4-6.5-9.4-10.6-17.1-10.6-14.9,0-22.9,16.9-22.9,39.2v.2Z" stroke-width="0"/>
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

<!-- Info Panel for Scrollytelling - now uses showInfoPanel instead of isScrollytellingActive -->
{#if showInfoPanel && views[currentStep]?.info}
<div class="info-panel" class:visible={views[currentStep].info}>
  <h2>{views[currentStep].info.title}</h2>
  <p>{@html views[currentStep].info.content}</p>
</div>
{/if}

<div class="text-content">
  <div class="container">
    <h1>Washed Away!</h1>

    <p style="font-family: Georgia, serif; font-style: oblique; font-weight: 300; color: #555; line-height: 1.5;">Microsoft's new AI-powered flood dataset highlights India's growing vulnerability, revealing that one in every ten square kilometres has experienced flooding over the past decade. Globally, the satellite-based analysis shows a steady 5% annual increase in flood extent, a troubling trend with far-reaching implications</p>
    
    <p>South Asia — and India in particular — has emerged as a hotspot for flooding in the last decade. A new analysis by Down To Earth (DTE) of a global flood mapping project led by Microsoft, using artificial intelligence and cloud-penetrating radar satellite imagery, reveals that over 10 per cent of India’s landmass has experienced flooding at least once in the past decade.</p>
    
    <p>The worst-hit states, predictably, include Bihar, Uttar Pradesh, Assam and West Bengal — but leading the charts in raw percentages is Punjab, where 68 per cent of the state’s land showed up as flooded.</p>
    
    <p>At first glance, the figures are startling. But researchers caution that not all floods are the same. Punjab’s unusually high numbers are largely the result of controlled, deliberate inundation of paddy fields, which require standing water through much of the agricultural cycle. But what’s happening in Bihar is a very different story.</p>
    
    <p>In Bihar, the water is far from welcome. More than half the state’s area, 51 per cent, has faced flooding over the past ten years, much of it from destructive, recurrent river floods. The state’s geography places it at the mercy of the Kosi and Gandak rivers, among others, which frequently burst their banks during the monsoon, displacing thousands and damaging crops, roads and homes.</p>

    <p>Uttar Pradesh, India’s most populous state, isn’t far behind, with flooding across 25 per cent of its area. However, in sheer numbers, close to 60,000 square kilometres in the state have been impacted by flooding — the biggest numbers so far. Bihar stood at a little over 48,000 sq km and Punjab saw over 34,000 sq km impacted. </p>

    <p>Assam (over 23,000 sq km) and West Bengal (over 31,000 sq km) clocked in at 30 per cent and 37 per cent, respectively. Haryana recorded 38 per cent, with roughly 17,000 sq km impacted. </p>

    <p>These figures are drawn from a new global flood dataset developed by Microsoft’s AI for Good Lab, which combines deep learning models with Sentinel-1 radar satellite imagery — a technology that can see through clouds and operate both day and night. The results are a part of a wider study in the journal Nature Communications.</p>

    <p>By applying their model to ten years of satellite data (2014-2024), the team has produced the most comprehensive flood mapping dataset available globally, covering both past floods and areas at continued risk. What makes this dataset especially powerful is its public availability. The flood maps, covering every Sentinel-1 radar image from October 2014 to September 2024, have been released online along with the full codebase used to generate them.</p>

    <h3>What the researchers found</h3>

    <p>India is far from alone in facing a rising tide — the study revealed the extent of flooding around the world has been significantly underestimated and Africa has shown some of the sharpest increases in flood-prone areas. </p>

    <p>Globally, the model identified 71 per cent more flood-prone land than older methods had captured. In Africa, the detected flood extent nearly doubled; in Ethiopia, it nearly tripled.</p>

    <p>The same tool was used in Kenya’s deadly 2024 floods, where it provided real-time flood maps to emergency responders, helping estimate that 75,000 hectares of cropland had been affected; this figure was nearly identical to official ground reports.</p>

    <p>By layering flood detection data with land use maps, the researchers turned their attention to agriculture. In Semera, Ethiopia, around 19 per cent of cropland was found to lie in historically flooded zones. That’s nearly three times higher than what previous satellite records, such as Landsat (7 per cent) and MODIS (2 per cent), had shown.</p>

    <p>But it was in Dolo Ado, further south along the Ganale River, where the contrast was most striking: more than half — 52 per cent — of cropland was flagged as flood-prone by the new model. Earlier datasets had captured just 1 to 3 per cent. In a region where families depend on rain-fed staples like maize and sorghum, such underestimation carries heavy consequences — both for food security and rural livelihoods.</p>

    <h3>Bigger, unsettling picture </h3>
    
    <p>When the researchers charted flood data across a ten-year timeline, from 2014 to 2024, they spotted what appears to be a slow but steady rise: A 5 per cent annual increase in flood extent. If that trend continues, it could mean a 60 per cent rise each decade — a shift with far-reaching implications.</p>

    <p>The map of growing risk is not evenly drawn. A belt stretching from Nigeria to Ethiopia has seen a clear uptick in flooding, in line with projections from global climate models that predict heavier rainfall in parts of Africa. Eastern Australia, too, stands out, reflecting a string of extreme weather events in recent years. </p>

    <p>However, the dataset covers just a decade — not quite long enough to draw definitive conclusions about climate-driven change, the authors have warned.</p>

    <div class="credits">
      Source: Mapping global floods with 10 years of satellite radar data | Copernicus GLO-30 Digital Elevation Model <br><br>
      By Pulaha Roy and Nandita Banerjee
    </div>
  </div>
</div>