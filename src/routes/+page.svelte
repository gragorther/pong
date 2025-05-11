<!-- src/Pong.svelte -->
<script lang="ts">
	import { onMount } from 'svelte';

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

	// Track the state of keys
	const keys: { [key: string]: boolean } = {
		ArrowUp: false,
		ArrowDown: false,
		w: false,
		s: false
	};

	onMount(() => {
		ctx = canvas.getContext('2d')!;
		window.addEventListener('keydown', handleKeyDown);
		window.addEventListener('keyup', handleKeyUp);
		requestAnimationFrame(update);
	});

	function update() {
		ctx.fillStyle = 'black';
		ctx.fillRect(0, 0, canvas.width, canvas.height);

		// Draw paddles
		ctx.fillStyle = 'white';
		ctx.fillRect(0, paddle1Y, paddleWidth, paddleHeight);
		ctx.fillRect(canvas.width - paddleWidth, paddle2Y, paddleWidth, paddleHeight);

		// Draw ball
		ctx.beginPath();
		ctx.arc(ballX, ballY, 10, 0, Math.PI * 2);
		ctx.fillStyle = 'white';
		ctx.fill();

		// Draw scores
		ctx.font = '32px Arial';
		ctx.fillText(score1.toString(), canvas.width / 4, 50);
		ctx.fillText(score2.toString(), (3 * canvas.width) / 4, 50);
		if (score1 == 2) {
			ctx.fillText('Player 1 has won!', canvas.width / 2, canvas.height / 3);
		}

		// Move ball
		ballX += ballSpeedX;
		ballY += ballSpeedY;

		// Ball collision with top and bottom
		if (ballY <= 0 || ballY >= canvas.height) {
			ballSpeedY = -ballSpeedY;
		}

		// Ball collision with paddles
		if (
			(ballX <= paddleWidth && ballY >= paddle1Y && ballY <= paddle1Y + paddleHeight) ||
			(ballX >= canvas.width - paddleWidth && ballY >= paddle2Y && ballY <= paddle2Y + paddleHeight)
		) {
			ballSpeedX = -ballSpeedX;
		}

		// Reset ball and update score if it goes out of bounds
		if (ballX < 0) {
			score2++;
			resetBall();
		} else if (ballX > canvas.width) {
			score1++;
			resetBall();
		}

		// Move paddles based on key state
		if (keys.w && paddle1Y > 0) {
			paddle1Y -= 7;
		}
		if (keys.s && paddle1Y < canvas.height - paddleHeight) {
			paddle1Y += 7;
		}
		if (keys.ArrowUp && paddle2Y > 0) {
			paddle2Y -= 7;
		}
		if (keys.ArrowDown && paddle2Y < canvas.height - paddleHeight) {
			paddle2Y += 7;
		}

		requestAnimationFrame(update);
	}

	function resetBall() {
		ballX = canvas.width / 2;
		ballY = canvas.height / 2;
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
	<canvas bind:this={canvas} width={800} height={600} tabindex="0" class="border-2 border-white"
	></canvas>
</div>
