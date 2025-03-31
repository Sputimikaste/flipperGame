<script lang="ts">
  import { onMount } from 'svelte';

  let canvas: HTMLCanvasElement;
  let ctx: CanvasRenderingContext2D;

  let ball = { x: 200, y: 450, radius: 10, speedY: 0, speedX: 0 };
  let gravity = 0.4;
  let flipperLeft = { x: 100, y: 500, width: 60, height: 10, angle: 0 };
  let flipperRight = { x: 240, y: 500, width: 60, height: 10, angle: 0 };

  let charging = false;
  let chargePower = 0;
  let maxCharge = 20;
  let tries = 3;
  let score = 0;
  let highScores: number[] = [];

  function drawFlippers() {
    ctx.save();
    ctx.translate(flipperLeft.x + flipperLeft.width / 2, flipperLeft.y);
    ctx.rotate(-flipperLeft.angle);
    ctx.fillStyle = '#00bcd4';
    ctx.fillRect(-flipperLeft.width / 2, 0, flipperLeft.width, flipperLeft.height);
    ctx.restore();

    ctx.save();
    ctx.translate(flipperRight.x + flipperRight.width / 2, flipperRight.y);
    ctx.rotate(flipperRight.angle);
    ctx.fillStyle = '#00bcd4';
    ctx.fillRect(-flipperRight.width / 2, 0, flipperRight.width, flipperRight.height);
    ctx.restore();
  }

  function drawBall() {
    ctx.beginPath();
    ctx.arc(ball.x, ball.y, ball.radius, 0, Math.PI * 2);
    ctx.fillStyle = '#ff4081';
    ctx.fill();
    ctx.closePath();
  }

  function draw() {
    ctx.clearRect(0, 0, canvas.width, canvas.height);

    drawFlippers();
    drawBall();

    ctx.fillStyle = '#fff';
    ctx.fillText(`Score: ${score}`, 10, 20);
    ctx.fillText(`Tries left: ${tries}`, 10, 40);
  }

  function updateBall() {
    if (ball.y + ball.radius < canvas.height) {
      ball.speedY += gravity;
    }
    ball.y += ball.speedY;
    ball.x += ball.speedX;

    if (ball.y > canvas.height) {
      tries--;
      if (tries > 0) {
        resetBall();
      } else {
        highScores.push(score);
        highScores.sort((a, b) => b - a);
        alert('Game over!');
        score = 0;
        tries = 3;
        resetBall();
      }
    }
  }

  function resetBall() {
    ball = { x: 200, y: 450, radius: 10, speedY: 0, speedX: 0 };
  }

  function gameLoop() {
    updateBall();
    draw();
    requestAnimationFrame(gameLoop);
  }

  function handleKeyDown(e: KeyboardEvent) {
    if (e.code === 'ArrowLeft') {
      flipperLeft.angle = 0.5;
    } else if (e.code === 'ArrowRight') {
      flipperRight.angle = 0.5;
    } else if (e.code === 'Space') {
      charging = true;
    }
  }

  function handleKeyUp(e: KeyboardEvent) {
    if (e.code === 'ArrowLeft') {
      flipperLeft.angle = 0;
    } else if (e.code === 'ArrowRight') {
      flipperRight.angle = 0;
    } else if (e.code === 'Space') {
      charging = false;
      ball.speedY = -chargePower;
      ball.speedX = 0;
      chargePower = 0;
    }
  }

  onMount(() => {
    ctx = canvas.getContext('2d')!;
    ctx.font = '16px Arial';
    gameLoop();

    window.addEventListener('keydown', handleKeyDown);
    window.addEventListener('keyup', handleKeyUp);

    const chargeInterval = setInterval(() => {
      if (charging && chargePower < maxCharge) {
        chargePower += 0.5;
      }
    }, 30);

    return () => {
      window.removeEventListener('keydown', handleKeyDown);
      window.removeEventListener('keyup', handleKeyUp);
      clearInterval(chargeInterval);
    };
  });
</script>

<style>
  canvas {
    background: linear-gradient(135deg, #1d2671, #c33764);
    border-radius: 12px;
    box-shadow: 0 0 20px rgba(0, 0, 0, 0.3);
    display: block;
    margin: auto;
  }
</style>

<canvas bind:this={canvas} width="400" height="600"></canvas>

<h2 style="text-align:center; color:white;">Highscores</h2>
<ul style="color:white; text-align:center;">
  {#each highScores.slice(0, 5) as hs, i}
    <li>{i + 1}. {hs}</li>
  {/each}
</ul>
