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
	let hatImage: string = '/neutral-hat.png';

	interface Distro {
		name: string;
		logo: string;
		website: string;
		audio: string;
	}

	const thinkingAudio: string[] = [
		'/audio/thinking/thinking1.ogg',
		'/audio/thinking/thinking2.ogg',
		'/audio/thinking/thinking3.ogg',
		'/audio/thinking/thinking4.ogg',
		'/audio/thinking/thinking5.ogg'
	];

	const distros: Distro[] = [
		{
			name: 'Ubuntu',
			logo: '/imgs/distro-logo/ubuntu.png',
			website: 'https://ubuntu.com',
			audio: '/audio/distros/ubuntu.ogg'
		},
		{
			name: 'Arch Linux',
			logo: '/imgs/distro-logo/arch.png',
			website: 'https://archlinux.org',
			audio: '/audio/distros/arch-linux.ogg'
		},
		{
			name: 'Fedora',
			logo: '/imgs/distro-logo/fedora.png',
			website: 'https://fedoraproject.org',
			audio: '/audio/distros/fedora.ogg'
		},
		{
			name: 'Debian',
			logo: '/imgs/distro-logo/debian.png',
			website: 'https://debian.org',
			audio: '/audio/distros/debian.ogg'
		},
		{
			name: 'Pop!_OS',
			logo: '/imgs/distro-logo/popos.png',
			website: 'https://pop.system76.com',
			audio: '/audio/distros/popos.ogg'
		}
	];

	async function initCamera() {
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
	}

	onMount(async () => {
		initCamera();
	});

	onDestroy(() => {
		if (stream) {
			stream.getTracks().forEach((track) => track.stop());
		}
	});

	async function discoverDistro() {
		if (!videoElement || !canvasElement) return;

		discovering = true;
		isGlowing = true;
		isCapturing = true;

		// Play random thinking audio
		const randomThinking = thinkingAudio[Math.floor(Math.random() * thinkingAudio.length)];
		const thinkingSound = new Audio(randomThinking);
		thinkingSound.play().catch((err) => console.error('Error playing thinking audio:', err));
		hatImage = '/eyes-closed-hat.png';

		// Wait 5 seconds while "thinking"
		await new Promise((resolve) => setTimeout(resolve, 5000));

		// Take the picture
		canvasElement.width = videoElement.videoWidth;
		canvasElement.height = videoElement.videoHeight;

		const context = canvasElement.getContext('2d');
		if (context) {
			context.drawImage(videoElement, 0, 0);
			capturedImage = canvasElement.toDataURL('image/png');
		}

		// Select random distro
		const randomDistro = distros[Math.floor(Math.random() * distros.length)];
		selectedDistro = randomDistro;

		// Stop glowing
		isGlowing = false;
		discovering = false;

		hatImage = '/speaking-hat.png';
		// Play distro audio
		const distroSound = new Audio(randomDistro.audio);
		distroSound.addEventListener('ended', () => {
			hatImage = '/neutral-hat.png';
		});
		distroSound.play().catch((err) => console.error('Error playing distro audio:', err));
	}

	function resetDiscovery() {
		capturedImage = '';
		isCapturing = false;
		isGlowing = false;
		discovering = false;
		selectedDistro = null;
		hatImage = '/neutral-hat.png';

		initCamera();
	}
</script>

<svelte:head>
	<link rel="preconnect" href="https://fonts.googleapis.com" />
	<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="anonymous" />
	<link
		href="https://fonts.googleapis.com/css2?family=Cinzel:wght@400;600;700&display=swap"
		rel="stylesheet"
	/>
</svelte:head>

<div class="page-container">
	<h1 class="title">Distro Selector Hat</h1>

	<div class="container">
		{#if error}
			<div class="error">{error}</div>
		{:else}
			<div class="video-wrapper">
				<!-- Sorting hat PNG overlay -->
				<img src={hatImage} alt="Sorting Hat" class="sorting-hat-png" class:glowing={isGlowing} />

				<div class="video-container">
					{#if capturedImage}
						<img src={capturedImage} alt="Captured" class="captured-image" />
					{:else}
						<video bind:this={videoElement} autoplay playsinline muted></video>
					{/if}
				</div>

				<div class="controls">
					{#if capturedImage && !discovering}
						<button on:click={resetDiscovery} class="btn">Try again</button>
					{:else if !discovering}
						<button on:click={discoverDistro} class="btn btn-discover">Discover</button>
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
						<p>
							The Distro Selector Hat has chosen <strong>{selectedDistro.name}</strong> for you!
						</p>
						<a
							href={selectedDistro.website}
							target="_blank"
							rel="noopener noreferrer"
							class="distro-link"
						>
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
	:global(body) {
		font-family: 'Courier New', 'Courier', monospace;
		margin: 0;
		padding: 0;
	}

	.page-container {
		display: flex;
		flex-direction: column;
		gap: 6.66rem;
		min-height: 100vh;
		background: linear-gradient(135deg, #1a1a2e 0%, #16213e 50%, #0f3460 100%);
		color: #d4af37;
	}

	.title {
		text-align: center;
		color: #d4af37;
		font-size: 3.5rem;
		font-weight: 700;
		margin-bottom: 40px;
		text-shadow:
			0 0 20px rgba(212, 175, 55, 0.5),
			2px 2px 4px rgba(0, 0, 0, 0.8);
		font-family: 'Cinzel', serif;
		letter-spacing: 2px;
	}

	.container {
		display: flex;
		justify-content: center;
		align-items: center;
		flex-direction: column;
	}

	.video-wrapper {
		position: relative;
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
		box-shadow:
			0 8px 16px rgba(0, 0, 0, 0.5),
			0 0 30px rgba(212, 175, 55, 0.2);
		background: #0f0f0f;
		border: 2px solid #d4af37;
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
		top: -7rem;
		left: 50%;
		transform: translateX(-50%);
		width: 350px;
		height: 350px;
		z-index: 10;
		animation: float 3s ease-in-out infinite;
		filter: drop-shadow(0 4px 8px rgba(0, 0, 0, 0.5));
	}

	/* Floating animation */
	@keyframes float {
		0%,
		100% {
			transform: translateX(-50%) translateY(0px) rotate(-2deg);
		}
		50% {
			transform: translateX(-50%) translateY(-15px) rotate(2deg);
		}
	}

	/* Glowing effect class */
	.glowing {
		animation:
			float 3s ease-in-out infinite,
			glow 2s ease-in-out infinite;
		filter: drop-shadow(0 4px 8px rgba(0, 0, 0, 0.5)) drop-shadow(0 0 20px rgba(212, 175, 55, 0.8))
			drop-shadow(0 0 40px rgba(212, 175, 55, 0.6)) drop-shadow(0 0 60px rgba(212, 175, 55, 0.4));
	}

	@keyframes glow {
		0%,
		100% {
			filter: drop-shadow(0 4px 8px rgba(0, 0, 0, 0.5))
				drop-shadow(0 0 20px rgba(212, 175, 55, 0.8)) drop-shadow(0 0 40px rgba(212, 175, 55, 0.6))
				drop-shadow(0 0 60px rgba(212, 175, 55, 0.4));
		}
		50% {
			filter: drop-shadow(0 4px 8px rgba(0, 0, 0, 0.5)) drop-shadow(0 0 30px rgba(212, 175, 55, 1))
				drop-shadow(0 0 60px rgba(212, 175, 55, 0.8)) drop-shadow(0 0 90px rgba(212, 175, 55, 0.6));
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
		border: 2px solid #d4af37;
		border-radius: 8px;
		cursor: pointer;
		transition: all 0.3s ease;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
		font-family: 'Courier New', 'Courier', monospace;
		text-transform: uppercase;
		letter-spacing: 1px;
	}

	.btn-discover {
		background: linear-gradient(135deg, #d4af37 0%, #f4d03f 100%);
		color: #1a1a2e;
	}

	.btn-discover:hover {
		transform: translateY(-2px);
		box-shadow:
			0 4px 8px rgba(212, 175, 55, 0.4),
			0 0 20px rgba(212, 175, 55, 0.3);
	}

	.btn-discover:active {
		transform: translateY(0);
	}

	.btn:not(.btn-discover) {
		background: rgba(26, 26, 46, 0.8);
		color: #d4af37;
	}

	.btn:not(.btn-discover):hover {
		background: rgba(26, 26, 46, 0.95);
		box-shadow: 0 4px 8px rgba(212, 175, 55, 0.2);
	}

	.error {
		padding: 20px;
		background-color: rgba(139, 0, 0, 0.2);
		border: 2px solid #8b0000;
		border-radius: 8px;
		color: #ff6b6b;
		font-family: 'Courier New', 'Courier', monospace;
	}

	/* Result text styling */
	.result-text {
		background: rgba(26, 26, 46, 0.95);
		padding: 30px;
		border-radius: 12px;
		box-shadow:
			0 8px 16px rgba(0, 0, 0, 0.5),
			0 0 30px rgba(212, 175, 55, 0.2);
		text-align: center;
		max-width: 400px;
		animation: fadeIn 0.5s ease-in;
		border: 2px solid #d4af37;
	}

	.discovering-text {
		background: rgba(26, 26, 46, 0.95);
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
		0%,
		100% {
			transform: scale(1);
			box-shadow:
				0 8px 16px rgba(0, 0, 0, 0.5),
				0 0 30px rgba(212, 175, 55, 0.2);
		}
		50% {
			transform: scale(1.05);
			box-shadow:
				0 8px 16px rgba(0, 0, 0, 0.5),
				0 0 40px rgba(212, 175, 55, 0.4);
		}
	}

	.result-text h2 {
		color: #d4af37;
		font-size: 2rem;
		margin: 15px 0;
		font-family: 'Cinzel', serif;
		text-shadow: 0 0 10px rgba(212, 175, 55, 0.5);
	}

	.result-text p {
		color: #c9c9c9;
		font-size: 1.1rem;
		margin: 10px 0;
		font-family: 'Courier New', 'Courier', monospace;
	}

	.result-text strong {
		color: #d4af37;
	}

	.distro-logo {
		width: 100px;
		height: 100px;
		object-fit: contain;
		margin: 0 auto;
		filter: drop-shadow(0 0 10px rgba(212, 175, 55, 0.3));
	}

	.distro-link {
		display: inline-block;
		margin-top: 15px;
		padding: 10px 20px;
		background: linear-gradient(135deg, #d4af37 0%, #f4d03f 100%);
		color: #1a1a2e;
		text-decoration: none;
		border-radius: 6px;
		font-weight: 600;
		transition: all 0.2s ease;
		font-family: 'Courier New', 'Courier', monospace;
		text-transform: uppercase;
		letter-spacing: 1px;
		border: 2px solid #d4af37;
	}

	.distro-link:hover {
		transform: translateY(-2px);
		box-shadow:
			0 4px 8px rgba(212, 175, 55, 0.4),
			0 0 20px rgba(212, 175, 55, 0.3);
	}
</style>
