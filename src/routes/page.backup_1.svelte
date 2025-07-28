<script>
  import { tweened } from 'svelte/motion';
  import { cubicOut } from 'svelte/easing';
  import { onMount } from 'svelte';

  let innerWidth;
  let innerHeight;

  const views = [
    { 
      name: 'India', 
      viewBox: [0, 0, 6000, 6750], 
      labelVisible: false,
      // No adjustments needed for India overview
      info: {
        title: "Exploring India",
        content: "Journey through India's diverse regions and discover the unique stories of each state. Scroll down to explore different regions in detail."
      }
    },
    { 
      name: 'Punjab+Haryana', 
      viewBox: [900, 1150, 1958, 1100], 
      labelVisible: true,
      // Custom adjustments for different screen sizes
      adjustments: {
        mobile: { offsetX: 0, offsetY: 0, scale: 1.0 }, // Keep as is
        tablet: { offsetX: 0, offsetY: 0, scale: 1.0 },
        desktop: { offsetX: 0, offsetY: 0, scale: 1.0 }
      },
      info: {
        title: "Punjab & Haryana",
        content: "Known as India's breadbasket, these northern states are famous for their agricultural productivity, especially wheat and rice. The Green Revolution transformed this region into the country's food bowl."
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
        content: "India's most populous state, home to the iconic Taj Mahal and ancient cities like Varanasi. This region holds immense cultural and historical significance in Indian civilization."
      }
    },
    { 
      name: 'Bihar', 
      viewBox: [3100, 2200, 1246, 700], 
      labelVisible: true,
      // Shrink it a little as requested
      adjustments: {
        mobile: { offsetX: -100, offsetY: 0, scale: 0.8 }, // Zoom out (shrink) more on mobile
        tablet: { offsetX: 0, offsetY: 0, scale: 1.15 },
        desktop: { offsetX: 0, offsetY: 0, scale: 1.1 }
      },
      info: {
        title: "Bihar",
        content: "The birthplace of Buddhism and Jainism, Bihar is steeped in ancient history. Bodh Gaya, where Buddha attained enlightenment, remains one of the most sacred pilgrimage sites in the world."
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
        content: "Famous for its tea gardens and rich biodiversity, Assam is home to the one-horned rhinoceros in Kaziranga National Park. The state's tea industry produces some of the world's finest black tea."
      }
    }
  ];

  let currentStep = 0;

  const DURATION = 1000;

  // ENHANCED: Helper function with custom adjustments per step and screen size
  function getResponsiveViewBox(viewBoxArray, windowWidth, windowHeight, step) {
    const [x, y, width, height] = viewBoxArray;
    
    console.log(`\n=== getResponsiveViewBox DEBUG ===`);
    console.log(`Step: ${step} (${views[step].name}), Screen: ${windowWidth}x${windowHeight}`);
    console.log(`Original viewBox: [${x}, ${y}, ${width}, ${height}]`);
    
    // For India overview (step 0), no zoom needed
    if (step === 0) {
      console.log(`Step 0: India overview - no zoom applied`);
      return adjustForAspectRatio(x, y, width, height, windowWidth, windowHeight);
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
    
    return result;
  }
  
  // Helper function to adjust viewBox for screen aspect ratio
  function adjustForAspectRatio(x, y, width, height, windowWidth, windowHeight) {
    const viewBoxAspectRatio = width / height;
    const windowAspectRatio = windowWidth / windowHeight;
    
    console.log(`[adjustForAspectRatio] ViewBox: ${width}x${height} (ratio: ${viewBoxAspectRatio.toFixed(2)})`);
    console.log(`[adjustForAspectRatio] Window: ${windowWidth}x${windowHeight} (ratio: ${windowAspectRatio.toFixed(2)})`);
    
    // If window is wider than viewBox, expand width to fit
    if (windowAspectRatio > viewBoxAspectRatio) {
      const adjustedWidth = height * windowAspectRatio;
      const widthDiff = adjustedWidth - width;
      const newX = x - widthDiff / 2;
      console.log(`[adjustForAspectRatio] Wide window: expanding width ${width} -> ${adjustedWidth}`);
      return [newX, y, adjustedWidth, height];
    } 
    // If window is taller than viewBox, expand height to fit
    else {
      const adjustedHeight = width / windowAspectRatio;
      const heightDiff = adjustedHeight - height;
      const newY = y - heightDiff / 2;
      console.log(`[adjustForAspectRatio] Tall window: expanding height ${height} -> ${adjustedHeight}`);
      return [x, newY, width, adjustedHeight];
    }
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
    console.log(`[updateViewBox] Called with step=${step}, width=${width}, height=${height}`);

    // Use the original viewBox coordinates with responsive adjustments
    const originalViewBox = views[step].viewBox;
    const responsiveViewBox = getResponsiveViewBox(originalViewBox, width, height, step);

    console.log(`[updateViewBox] Original viewBox: ${originalViewBox.join(' ')}`);
    console.log(`[updateViewBox] Responsive viewBox: ${responsiveViewBox.join(' ')}`);

    viewBoxStore.set(responsiveViewBox);
  }

  function onScroll() {
    const scrollY = window.scrollY;
    const height = window.innerHeight;
    const step = Math.min(views.length - 1, Math.floor(scrollY / height));

    console.log(`[onScroll] scrollY=${scrollY}, calculated step=${step}`);

    if (step !== currentStep) {
      currentStep = step;
      updateViewBox(currentStep, window.innerWidth, window.innerHeight);
    }
  }

  function onResize() {
    console.log(`[onResize] window.innerWidth=${window.innerWidth}, window.innerHeight=${window.innerHeight}`);
    updateViewBox(currentStep, window.innerWidth, window.innerHeight);
  }

  onMount(() => {
    window.addEventListener('scroll', onScroll);
    window.addEventListener('resize', onResize);

    // Initial update
    updateViewBox(currentStep, window.innerWidth, window.innerHeight);

    return () => {
      window.removeEventListener('scroll', onScroll);
      window.removeEventListener('resize', onResize);
    };
  });

  // Reactive statement to handle window size changes
  $: if (innerWidth && innerHeight) {
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
    top: 54%;
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
  }

  /* Hide background image when zoomed in to avoid double boundaries */
  .background-image {
    transition: opacity 0.5s ease;
  }
  
  .background-image.hidden {
    opacity: 0;
  }
  
  /* Alternative: Make background image less prominent when zoomed */
  .background-image.faded {
    opacity: 0.3;
  }
</style>

<svelte:window bind:innerWidth bind:innerHeight />

<div class="svg-wrapper">
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
      <image width="6000" height="6750" xlink:href="large_image.png"/>
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

{#each Array(views.length) as _, i}
  <section style="height: 100vh;"></section>
{/each}

<!-- Info Panel for Scrollytelling -->
<div class="info-panel" class:visible={views[currentStep].info}>
  {#if views[currentStep].info}
    <h2>{views[currentStep].info.title}</h2>
    <p>{views[currentStep].info.content}</p>
  {/if}
</div>