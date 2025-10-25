<script lang="ts">
  import { onMount, onDestroy } from 'svelte';
  
  let videoElement: HTMLVideoElement;
  let canvasElement: HTMLCanvasElement;
  let stream: MediaStream | null = null;
  let error: string = '';
  let capturedImage: string = '';
  let isCapturing: boolean = false;
  let isGlowing: boolean = false;
  let discovering: boolean = false;
  let selectedDistro: Distro | null = null;

  interface Distro {
    name: string;
    logo: string;
    website: string;
    audio: string;
  }

  const thinkingAudio: string[] = [
    'thinking1.mp3',
    'thinking2.mp3',
    'thinking3.mp3',
    'thinking4.mp3'
  ];

  const distros: Distro[] = [
    { name: 'Ubuntu', logo: 'ubuntu-logo.png', website: 'https://ubuntu.com', audio: 'ubuntu.mp3' },
    { name: 'Arch Linux', logo: 'arch-logo.png', website: 'https://archlinux.org', audio: 'arch.mp3' },
    { name: 'Fedora', logo: 'fedora-logo.png', website: 'https://fedoraproject.org', audio: 'fedora.mp3' },
    { name: 'Debian', logo: 'debian-logo.png', website: 'https://debian.org', audio: 'debian.mp3' },
    { name: 'Linux Mint', logo: 'mint-logo.png', website: 'https://linuxmint.com', audio: 'mint.mp3' },
    { name: 'Manjaro', logo: 'manjaro-logo.png', website: 'https://manjaro.org', audio: 'manjaro.mp3' },
    { name: 'Pop!_OS', logo: 'popos-logo.png', website: 'https://pop.system76.com', audio: 'popos.mp3' },
    { name: 'Elementary OS', logo: 'elementary-logo.png', website: 'https://elementary.io', audio: 'elementary.mp3' },
    { name: 'Kali Linux', logo: 'kali-logo.png', website: 'https://kali.org', audio: 'kali.mp3' },
    { name: 'Gentoo', logo: 'gentoo-logo.png', website: 'https://gentoo.org', audio: 'gentoo.mp3' }
  ];

  onMount(async () => {
    try {
      stream = await navigator.mediaDevices.getUserMedia({ 
        video: { facingMode: 'user' }
      });
      
      if (videoElement) {
        videoElement.srcObject = stream;
      }
    } catch (err) {
      error = 'Unable to access camera: ' + (err as Error).message;
      console.error('Camera error:', err);
    }
  });

  onDestroy(() => {
    if (stream) {
      stream.getTracks().forEach(track => track.stop());
    }
  });

  async function takePicture() {
    if (!videoElement || !canvasElement) return;
    
    discovering = true;
    isGlowing = true;
    
    // Take the picture
    canvasElement.width = videoElement.videoWidth;
    canvasElement.height = videoElement.videoHeight;
    
    const context = canvasElement.getContext('2d');
    if (context) {
      context.drawImage(videoElement, 0, 0);
      capturedImage = canvasElement.toDataURL('image/png');
    }
    
    isCapturing = true;
    
    // Play random thinking audio
    // const randomThinking = thinkingAudio[Math.floor(Math.random() * thinkingAudio.length)];
    // const thinkingSound = new Audio(randomThinking);
    // thinkingSound.play().catch(err => console.error('Error playing thinking audio:', err));
    
    // Wait 5 seconds while "thinking"
    await new Promise(resolve => setTimeout(resolve, 5000));
    
    // Select random distro
    const randomDistro = distros[Math.floor(Math.random() * distros.length)];
    selectedDistro = randomDistro;
    
    // Stop glowing
    isGlowing = false;
    discovering = false;
    
    // Play distro audio
    // const distroSound = new Audio(randomDistro.audio);
    // distroSound.play().catch(err => console.error('Error playing distro audio:', err));
  }

  function resetCamera() {
    capturedImage = '';
    isCapturing = false;
    isGlowing = false;
    discovering = false;
    selectedDistro = null;
  }
</script>

<div class="page-container">
  <h1 class="title">Distro Selector Hat</h1>
  
  <div class="container">
    {#if error}
      <div class="error">{error}</div>
    {:else}
      <div class="video-wrapper">
 <!-- Sorting hat PNG overlay -->
          <img 
            src="/neutral-hat-removebg-preview.png" 
            alt="Sorting Hat" 
            class="sorting-hat-png"
            class:glowing={isGlowing}
          />

        <div class="video-container">
          {#if capturedImage}
            <img src={capturedImage} alt="Captured" class="captured-image" />
          {:else}
            <video
              bind:this={videoElement}
              autoplay
              playsinline
              muted
            />
          {/if}
          
                 </div>
        
        <div class="controls">
          {#if capturedImage && !discovering}
            <button on:click={resetCamera} class="btn">Take Another</button>
          {:else if !discovering}
            <button on:click={takePicture} class="btn btn-discover">Discover</button>
          {/if}
        </div>
        
        {#if discovering}
          <div class="result-text discovering-text">
            <p>🔮 The Sorting Hat is thinking...</p>
          </div>
        {/if}
        
        {#if selectedDistro && !discovering}
          <div class="result-text">
            <img src={selectedDistro.logo} alt={selectedDistro.name} class="distro-logo" />
            <h2>{selectedDistro.name}!</h2>
            <p>The Sorting Hat has chosen <strong>{selectedDistro.name}</strong> for you!</p>
            <a href={selectedDistro.website} target="_blank" rel="noopener noreferrer" class="distro-link">
              Visit {selectedDistro.name} →
            </a>
          </div>
        {/if}
      </div>
    {/if}
    
    <!-- Hidden canvas for capturing -->
    <canvas bind:this={canvasElement} style="display: none;"></canvas>
  </div>
</div>

<style>
  .page-container {
    min-height: 100vh;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    padding: 40px 20px;
  }

  .title {
    text-align: center;
    color: white;
    font-size: 3rem;
    font-weight: bold;
    margin-bottom: 40px;
    text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
    font-family: 'Georgia', serif;
  }

  .container {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
  }

  .video-wrapper {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 20px;
  }

  .video-container {
    position: relative;
    width: 400px;
    height: 400px;
    overflow: hidden;
    border-radius: 8px;
    box-shadow: 0 8px 16px rgba(0, 0, 0, 0.3);
    background: white;
  }

  video,
  .captured-image {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  /* Sorting hat PNG overlay */
  .sorting-hat-png {
    position: absolute;
    top: 60px;
    left: 50%;
    transform: translateX(-50%);
    width: 350px;
    height: auto;
    z-index: 10;
    animation: float 3s ease-in-out infinite;
    filter: drop-shadow(0 4px 8px rgba(0, 0, 0, 0.3));
  }

  /* Floating animation */
  @keyframes float {
    0%, 100% {
      transform: translateX(-50%) translateY(0px) rotate(-2deg);
    }
    50% {
      transform: translateX(-50%) translateY(-15px) rotate(2deg);
    }
  }

  /* Glowing effect class */
  .glowing {
    animation: float 3s ease-in-out infinite, glow 2s ease-in-out infinite;
    filter: drop-shadow(0 4px 8px rgba(0, 0, 0, 0.3)) 
            drop-shadow(0 0 20px rgba(255, 215, 0, 0.8))
            drop-shadow(0 0 40px rgba(255, 215, 0, 0.6))
            drop-shadow(0 0 60px rgba(255, 215, 0, 0.4));
  }

  @keyframes glow {
    0%, 100% {
      filter: drop-shadow(0 4px 8px rgba(0, 0, 0, 0.3))
              drop-shadow(0 0 20px rgba(255, 215, 0, 0.8))
              drop-shadow(0 0 40px rgba(255, 215, 0, 0.6))
              drop-shadow(0 0 60px rgba(255, 215, 0, 0.4));
    }
    50% {
      filter: drop-shadow(0 4px 8px rgba(0, 0, 0, 0.3))
              drop-shadow(0 0 30px rgba(255, 215, 0, 1))
              drop-shadow(0 0 60px rgba(255, 215, 0, 0.8))
              drop-shadow(0 0 90px rgba(255, 215, 0, 0.6));
    }
  }

  .controls {
    display: flex;
    gap: 10px;
  }

  .btn {
    padding: 12px 32px;
    font-size: 16px;
    font-weight: 600;
    border: none;
    border-radius: 8px;
    cursor: pointer;
    transition: all 0.3s ease;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }

  .btn-discover {
    background: linear-gradient(135deg, #FFA500 0%, #FF6347 100%);
    color: white;
  }

  .btn-discover:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  }

  .btn-discover:active {
    transform: translateY(0);
  }

  .btn:not(.btn-discover) {
    background: white;
    color: #667eea;
  }

  .btn:not(.btn-discover):hover {
    background: #f8f9fa;
  }

  .error {
    padding: 20px;
    background-color: #fee;
    border: 1px solid #fcc;
    border-radius: 8px;
    color: #c00;
  }

  /* Result text styling */
  .result-text {
    background: white;
    padding: 30px;
    border-radius: 12px;
    box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
    text-align: center;
    max-width: 400px;
    animation: fadeIn 0.5s ease-in;
  }

  .discovering-text {
    background: rgba(255, 255, 255, 0.95);
    animation: pulse 1.5s ease-in-out infinite;
  }

  @keyframes fadeIn {
    from {
      opacity: 0;
      transform: translateY(20px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }

  @keyframes pulse {
    0%, 100% {
      transform: scale(1);
    }
    50% {
      transform: scale(1.05);
    }
  }

  .result-text h2 {
    color: #667eea;
    font-size: 2rem;
    margin: 15px 0;
  }

  .result-text p {
    color: #333;
    font-size: 1.1rem;
    margin: 10px 0;
  }

  .distro-logo {
    width: 100px;
    height: 100px;
    object-fit: contain;
    margin: 0 auto;
  }

  .distro-link {
    display: inline-block;
    margin-top: 15px;
    padding: 10px 20px;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
    text-decoration: none;
    border-radius: 6px;
    font-weight: 600;
    transition: transform 0.2s ease;
  }

  .distro-link:hover {
    transform: translateY(-2px);
  }
</style>
