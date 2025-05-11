<!-- src/Pong.svelte -->
<script lang="ts">
	import { onMount } from 'svelte';

	let leftPaddleY = 250;
	let rightPaddleY = 250;
	let ballX = 395;
	let ballY = 250;
	let ballSpeedX = 5;
	let ballSpeedY = 5;
	let score1 = 0;
	let score2 = 0;

	const paddleHeight = 100;
	const paddleWidth = 10;
	const gameWidth = 800;
	const gameHeight = 600;
	const paddleMovementSpeed = 7;

	// Track the state of keys for keyboard controls
	let keys: { [key: string]: boolean } = {
		w: false,
		s: false
	};

	onMount(() => {
		// Start the game loop
		requestAnimationFrame(update);

		// Add keyboard event listeners
	});

	function update() {
		// Move ball
		ballX += ballSpeedX;
		ballY += ballSpeedY;

		// Ball collision with top and bottom
		if (ballY <= 0 || ballY >= gameHeight - 10) {
			ballSpeedY = -ballSpeedY;
		}

		// Ball collision with paddles
		if (
			(ballX <= paddleWidth && ballY >= leftPaddleY && ballY <= leftPaddleY + paddleHeight) ||
			(ballX >= gameWidth - paddleWidth - 10 &&
				ballY >= rightPaddleY &&
				ballY <= rightPaddleY + paddleHeight)
		) {
			ballSpeedX = -ballSpeedX;
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

		if (keys.w && leftPaddleY > 0) {
			leftPaddleY -= paddleMovementSpeed;
		}
		if (keys.s && leftPaddleY < gameHeight - paddleHeight) {
			leftPaddleY += paddleMovementSpeed;
		}

		// Update positions
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

<svelte:window on:keydown={handleKeyDown} on:keyup={handleKeyUp} />
<div class="flex min-h-screen flex-col items-center justify-center bg-gray-900 text-white">
	<h1 class="mb-4 text-4xl">Pong Game</h1>
	<div class="game-container max-w-screen" style="width: {gameWidth}px; height: {gameHeight}px;">
		<div class="paddle left-paddle" style="top: {leftPaddleY}px;"></div>
		<div class="paddle right-paddle" style="top: {rightPaddleY}px;"></div>
		<div class="ball" style="left: {ballX}px; top: {ballY}px;"></div>
		<div class="score">{score1} - {score2}</div>
	</div>
	<div class="mt-4 text-center">
		<p>Use keyboard (W/S) to move the left paddle</p>
	</div>
</div>

<style>
	.game-container {
		position: relative;
		background-color: black;
		border: 2px solid white;
		overflow: hidden;
	}

	.paddle {
		position: absolute;
		width: 10px;
		height: 100px;
		background-color: white;
	}

	.left-paddle {
		left: 0;
	}

	.right-paddle {
		right: 0;
	}

	.ball {
		position: absolute;
		width: 10px;
		height: 10px;
		background-color: white;
		border-radius: 50%;
	}

	.score {
		position: absolute;
		top: 20px;
		width: 100%;
		text-align: center;
		color: white;
		font-size: 32px;
	}
</style>
