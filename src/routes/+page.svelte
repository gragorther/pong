<!-- src/Pong.svelte -->
<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { browser } from '$app/environment';

	let canvas: HTMLCanvasElement;
	let ctx: CanvasRenderingContext2D;
	let ballX = 50;
	let ballY = 50;
	let ballSpeedX = 5;
	let ballSpeedY = 5;
	let paddle1Y = 250;
	let paddle2Y = 250;
	const paddleHeight = 100;
	const paddleWidth = 10;
	let score1 = 0;
	let score2 = 0;

	// Track the state of keys for keyboard controls
	let keys: { [key: string]: boolean } = {
		w: false,
		s: false
	};

	// Track touch position for dragging the paddle
	let isDragging = false;
	let touchY = 0;

	// Fixed game dimensions
	const gameWidth = 800;
	const gameHeight = 600;
	let scale = 1;

	onMount(() => {
		if (browser) {
			// Calculate scale to fit the game within the window while maintaining aspect ratio
			const windowAspectRatio = window.innerWidth / window.innerHeight;
			const gameAspectRatio = gameWidth / gameHeight;

			if (windowAspectRatio > gameAspectRatio) {
				scale = window.innerHeight / gameHeight;
			} else {
				scale = window.innerWidth / gameWidth;
			}

			canvas.width = gameWidth * scale;
			canvas.height = gameHeight * scale;

			ctx = canvas.getContext('2d')!;
			ctx.scale(scale, scale);
			requestAnimationFrame(update);

			// Add touch event listeners for dragging the paddle
			canvas.addEventListener('touchstart', handleTouchStart, false);
			canvas.addEventListener('touchmove', handleTouchMove, false);
			canvas.addEventListener('touchend', handleTouchEnd, false);
		}
	});

	onDestroy(() => {
		if (browser) {
			// Remove touch event listeners when the component is destroyed
			canvas.removeEventListener('touchstart', handleTouchStart);
			canvas.removeEventListener('touchmove', handleTouchMove);
			canvas.removeEventListener('touchend', handleTouchEnd);
		}
	});

	function handleTouchStart(event: TouchEvent) {
		event.preventDefault();
		const touch = event.touches[0];
		const rect = canvas.getBoundingClientRect();
		touchY = (touch.clientY - rect.top) / scale;

		// Allow dragging from anywhere on the canvas
		isDragging = true;
	}

	function handleTouchMove(event: TouchEvent) {
		event.preventDefault();
		if (!isDragging) return;

		const touch = event.touches[0];
		const rect = canvas.getBoundingClientRect();
		touchY = (touch.clientY - rect.top) / scale;

		// Update paddle position based on touch position
		paddle1Y = touchY - paddleHeight / 2;
	}

	function handleTouchEnd(event: TouchEvent) {
		event.preventDefault();
		isDragging = false;
	}

	function update() {
		if (!browser) return;

		ctx.fillStyle = 'black';
		ctx.fillRect(0, 0, gameWidth, gameHeight);

		// Draw paddles
		ctx.fillStyle = 'white';
		ctx.fillRect(0, paddle1Y, paddleWidth, paddleHeight);
		ctx.fillRect(gameWidth - paddleWidth, paddle2Y, paddleWidth, paddleHeight);

		// Draw ball
		ctx.beginPath();
		ctx.arc(ballX, ballY, 10, 0, Math.PI * 2);
		ctx.fillStyle = 'white';
		ctx.fill();

		// Draw scores
		ctx.font = '32px Arial';
		ctx.fillText(score1.toString(), gameWidth / 4, 50);
		ctx.fillText(score2.toString(), (3 * gameWidth) / 4, 50);

		// Move ball
		ballX += ballSpeedX;
		ballY += ballSpeedY;

		// Ball collision with top and bottom
		if (ballY <= 0 || ballY >= gameHeight) {
			ballSpeedY = -ballSpeedY;
		}

		// Ball collision with paddles
		if (
			(ballX <= paddleWidth && ballY >= paddle1Y && ballY <= paddle1Y + paddleHeight) ||
			(ballX >= gameWidth - paddleWidth && ballY >= paddle2Y && ballY <= paddle2Y + paddleHeight)
		) {
			ballSpeedX = -ballSpeedX;

			// Add randomness to the ball's direction when hit by a paddle
			const randomFactor = 0.2; // Adjust this value to control the amount of randomness
			ballSpeedY += (Math.random() - 0.5) * randomFactor * gameHeight;
		}

		// Reset ball and update score if it goes out of bounds
		if (ballX < 0) {
			score2++;
			resetBall();
		} else if (ballX > gameWidth) {
			score1++;
			resetBall();
		}

		// Move left paddle based on keyboard input
		if (keys.w && paddle1Y > 0) {
			paddle1Y -= 7;
		}
		if (keys.s && paddle1Y < gameHeight - paddleHeight) {
			paddle1Y += 7;
		}

		requestAnimationFrame(update);
	}

	function resetBall() {
		ballX = gameWidth / 2;
		ballY = gameHeight / 2;
		ballSpeedX = -ballSpeedX;
		ballSpeedY = Math.random() > 0.5 ? 5 : -5;
	}

	function handleKeyDown(event: KeyboardEvent) {
		if (keys.hasOwnProperty(event.key)) {
			keys[event.key as keyof typeof keys] = true;
		}
	}

	function handleKeyUp(event: KeyboardEvent) {
		if (keys.hasOwnProperty(event.key)) {
			keys[event.key as keyof typeof keys] = false;
		}
	}
</script>

<div class="flex min-h-screen flex-col items-center justify-center bg-gray-900 text-white">
	<h1 class="mb-4 text-4xl">Pong Game</h1>
	<canvas bind:this={canvas} tabindex="0" class="border-2 border-white"></canvas>
	<div class="mt-4 text-center">
		<p>Drag anywhere on the screen or use keyboard (W/S) to move the left paddle</p>
	</div>
</div>

<svelte:window on:keydown={handleKeyDown} on:keyup={handleKeyUp} />
