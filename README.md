<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>Snake Game</title>

<style>
* {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
}

body {
    min-height: 100vh;
    background:
        radial-gradient(circle at top, #172554, #020617 55%);
    color: white;
    font-family: Arial, Helvetica, sans-serif;

    display: flex;
    justify-content: center;
    align-items: center;

    overflow: hidden;
}

.game {
    width: min(94vw, 650px);
    text-align: center;
}

h1 {
    font-size: 42px;
    margin-bottom: 8px;
    letter-spacing: 3px;
}

.subtitle {
    color: #94a3b8;
    margin-bottom: 18px;
}

.info {
    display: flex;
    justify-content: space-between;
    gap: 10px;
    margin-bottom: 12px;
}

.box {
    flex: 1;
    padding: 12px;
    border: 1px solid #334155;
    background: rgba(15, 23, 42, 0.8);
    border-radius: 12px;
}

.box span {
    display: block;
    color: #94a3b8;
    font-size: 13px;
    margin-bottom: 4px;
}

.box strong {
    font-size: 22px;
}

.game-area {
    position: relative;
    width: 100%;
    aspect-ratio: 1 / 1;

    background: #020617;
    border: 3px solid #334155;
    border-radius: 18px;

    overflow: hidden;
    box-shadow:
        0 0 40px rgba(34, 197, 94, 0.12),
        inset 0 0 40px rgba(0, 0, 0, 0.5);
}

canvas {
    width: 100%;
    height: 100%;
    display: block;
}

.overlay {
    position: absolute;
    inset: 0;

    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;

    background: rgba(2, 6, 23, 0.82);
    backdrop-filter: blur(5px);

    opacity: 0;
    pointer-events: none;

    transition: 0.25s;
}

.overlay.show {
    opacity: 1;
    pointer-events: auto;
}

.overlay h2 {
    font-size: 38px;
    margin-bottom: 8px;
}

.overlay p {
    color: #94a3b8;
    margin-bottom: 20px;
}

button {
    border: none;
    cursor: pointer;

    background: #22c55e;
    color: #052e16;

    font-size: 17px;
    font-weight: bold;

    padding: 12px 25px;
    border-radius: 10px;

    transition: 0.15s;
}

button:hover {
    transform: translateY(-2px);
    background: #4ade80;
}

button:active {
    transform: scale(0.95);
}

.controls {
    margin-top: 18px;

    display: grid;
    grid-template-columns: repeat(3, 60px);
    justify-content: center;
    gap: 7px;
}

.controls button {
    width: 60px;
    height: 50px;
    padding: 0;

    background: #1e293b;
    color: white;

    border: 1px solid #475569;
}

.controls button:hover {
    background: #334155;
}

.controls .empty {
    visibility: hidden;
}

.help {
    margin-top: 14px;
    color: #64748b;
    font-size: 13px;
}

@media (max-width: 500px) {

    h1 {
        font-size: 32px;
    }

    .controls {
        grid-template-columns: repeat(3, 55px);
    }

    .controls button {
        width: 55px;
        height: 46px;
    }
}
</style>
</head>

<body>

<div class="game">

    <h1>🐍 SNAKE</h1>

    <p class="subtitle">
        Classic Snake Game
    </p>

    <div class="info">

        <div class="box">
            <span>SCORE</span>
            <strong id="score">0</strong>
        </div>

        <div class="box">
            <span>BEST</span>
            <strong id="best">0</strong>
        </div>

        <div class="box">
            <span>SPEED</span>
            <strong id="speed">1x</strong>
        </div>

    </div>

    <div class="game-area">

        <canvas id="gameCanvas"></canvas>

        <div class="overlay" id="overlay">

            <h2>GAME OVER</h2>

            <p>
                Your score:
                <strong id="finalScore">0</strong>
            </p>

            <button id="restart">
                🔄 PLAY AGAIN
            </button>

        </div>

    </div>

    <div class="controls">

        <button class="empty"></button>

        <button data-direction="up">▲</button>

        <button class="empty"></button>

        <button data-direction="left">◀</button>

        <button data-direction="down">▼</button>

        <button data-direction="right">▶</button>

    </div>

    <p class="help">
        Use WASD or Arrow Keys to control the snake
    </p>

</div>

<script>

const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

const scoreElement = document.getElementById("score");
const bestElement = document.getElementById("best");
const speedElement = document.getElementById("speed");

const overlay = document.getElementById("overlay");
const finalScore = document.getElementById("finalScore");
const restartButton = document.getElementById("restart");

const GRID = 25;

let snake;
let food;

let direction;
let nextDirection;

let score = 0;
let best = Number(localStorage.getItem("snakeBest")) || 0;

let gameRunning = false;
let gameOver = false;

let lastTime = 0;
let accumulator = 0;

let speed = 130;

bestElement.textContent = best;


/* =========================
   CANVAS
========================= */

function resizeCanvas() {

    const size = canvas.clientWidth;

    const dpr = window.devicePixelRatio || 1;

    canvas.width = size * dpr;
    canvas.height = size * dpr;

    ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
}

window.addEventListener("resize", resizeCanvas);


/* =========================
   START GAME
========================= */

function startGame() {

    snake = [
        { x: 12, y: 12 },
        { x: 11, y: 12 },
        { x: 10, y: 12 },
        { x: 9, y: 12 }
    ];

    direction = {
        x: 1,
        y: 0
    };

    nextDirection = {
        x: 1,
        y: 0
    };

    score = 0;

    speed = 130;

    gameOver = false;
    gameRunning = true;

    accumulator = 0;

    scoreElement.textContent = score;
    speedElement.textContent = "1x";

    overlay.classList.remove("show");

    createFood();
}


/* =========================
   FOOD
========================= */

function createFood() {

    let valid = false;

    while (!valid) {

        food = {
            x: Math.floor(Math.random() * GRID),
            y: Math.floor(Math.random() * GRID)
        };

        valid = !snake.some(part =>
            part.x === food.x &&
            part.y === food.y
        );
    }
}


/* =========================
   UPDATE
========================= */

function update() {

    direction = nextDirection;

    const head = {
        x: snake[0].x + direction.x,
        y: snake[0].y + direction.y
    };


    /* WALL COLLISION */

    if (
        head.x < 0 ||
        head.x >= GRID ||
        head.y < 0 ||
        head.y >= GRID
    ) {

        endGame();
        return;
    }


    /* SELF COLLISION */

    const hitSelf = snake.some(part =>
        part.x === head.x &&
        part.y === head.y
    );

    if (hitSelf) {

        endGame();
        return;
    }


    snake.unshift(head);


    /* FOOD */

    if (
        head.x === food.x &&
        head.y === food.y
    ) {

        score++;

        scoreElement.textContent = score;

        if (score > best) {

            best = score;

            localStorage.setItem(
                "snakeBest",
                best
            );

            bestElement.textContent = best;
        }


        /* SPEED INCREASE */

        if (score % 5 === 0) {

            speed = Math.max(
                55,
                speed - 10
            );

            const multiplier =
                (130 / speed).toFixed(1);

            speedElement.textContent =
                multiplier + "x";
        }

        createFood();

    } else {

        snake.pop();
    }
}


/* =========================
   GAME OVER
========================= */

function endGame() {

    gameRunning = false;
    gameOver = true;

    finalScore.textContent = score;

    overlay.classList.add("show");
}


/* =========================
   DRAW
========================= */

function draw() {

    const size = canvas.clientWidth;

    const cell = size / GRID;


    /* BACKGROUND */

    ctx.fillStyle = "#020617";

    ctx.fillRect(
        0,
        0,
        size,
        size
    );


    /* GRID */

    ctx.strokeStyle = "rgba(51,65,85,0.25)";
    ctx.lineWidth = 1;

    for (let i = 0; i <= GRID; i++) {

        const p = i * cell;

        ctx.beginPath();

        ctx.moveTo(p, 0);
        ctx.lineTo(p, size);

        ctx.stroke();

        ctx.beginPath();

        ctx.moveTo(0, p);
        ctx.lineTo(size, p);

        ctx.stroke();
    }


    /* FOOD */

    const fx =
        food.x * cell + cell / 2;

    const fy =
        food.y * cell + cell / 2;

    ctx.beginPath();

    ctx.arc(
        fx,
        fy,
        cell * 0.32,
        0,
        Math.PI * 2
    );

    ctx.fillStyle = "#ef4444";

    ctx.shadowColor = "#ef4444";
    ctx.shadowBlur = 15;

    ctx.fill();

    ctx.shadowBlur = 0;


    /* SNAKE */

    snake.forEach((part, index) => {

        const padding = 2;

        const x =
            part.x * cell + padding;

        const y =
            part.y * cell + padding;

        const size2 =
            cell - padding * 2;

        ctx.fillStyle =
            index === 0
                ? "#4ade80"
                : "#22c55e";

        ctx.beginPath();

        ctx.roundRect(
            x,
            y,
            size2,
            size2,
            5
        );

        ctx.fill();


        /* EYES */

        if (index === 0) {

            ctx.fillStyle = "#052e16";

            const eyeSize = cell * 0.12;

            let eye1X;
            let eye1Y;

            let eye2X;
            let eye2Y;


            if (direction.x === 1) {

                eye1X = x + cell * 0.68;
                eye2X = x + cell * 0.68;

                eye1Y = y + cell * 0.30;
                eye2Y = y + cell * 0.70;

            } else if (direction.x === -1) {

                eye1X = x + cell * 0.32;
                eye2X = x + cell * 0.32;

                eye1Y = y + cell * 0.30;
                eye2Y = y + cell * 0.70;

            } else if (direction.y === -1) {

                eye1X = x + cell * 0.30;
                eye2X = x + cell * 0.70;

                eye1Y = y + cell * 0.32;
                eye2Y = y + cell * 0.32;

            } else {

                eye1X = x + cell * 0.30;
                eye2X = x + cell * 0.70;

                eye1Y = y + cell * 0.68;
                eye2Y = y + cell * 0.68;
            }


            ctx.beginPath();

            ctx.arc(
                eye1X,
                eye1Y,
                eyeSize,
                0,
                Math.PI * 2
            );

            ctx.fill();


            ctx.beginPath();

            ctx.arc(
                eye2X,
                eye2Y,
                eyeSize,
                0,
                Math.PI * 2
            );

            ctx.fill();
        }

    });
}


/* =========================
   GAME LOOP
========================= */

function gameLoop(time) {

    if (!lastTime) {
        lastTime = time;
    }

    const delta =
        time - lastTime;

    lastTime = time;

    if (gameRunning) {

        accumulator += delta;

        if (accumulator >= speed) {

            update();

            accumulator = 0;
        }
    }

    draw();

    requestAnimationFrame(gameLoop);
}


/* =========================
   CHANGE DIRECTION
========================= */

function changeDirection(dir) {

    if (!gameRunning) {
        return;
    }

    if (dir === "up") {

        if (direction.y !== 1) {

            nextDirection = {
                x: 0,
                y: -1
            };
        }

    }

    if (dir === "down") {

        if (direction.y !== -1) {

            nextDirection = {
                x: 0,
                y: 1
            };
        }

    }

    if (dir === "left") {

        if (direction.x !== 1) {

            nextDirection = {
                x: -1,
                y: 0
            };
        }

    }

    if (dir === "right") {

        if (direction.x !== -1) {

            nextDirection = {
                x: 1,
                y: 0
            };
        }
    }
}


/* =========================
   KEYBOARD
========================= */

document.addEventListener("keydown", event => {

    const key = event.key.toLowerCase();

    if (
        key === "arrowup" ||
        key === "w"
    ) {

        event.preventDefault();

        changeDirection("up");

    } else if (
        key === "arrowdown" ||
        key === "s"
    ) {

        event.preventDefault();

        changeDirection("down");

    } else if (
        key === "arrowleft" ||
        key === "a"
    ) {

        event.preventDefault();

        changeDirection("left");

    } else if (
        key === "arrowright" ||
        key === "d"
    ) {

        event.preventDefault();

        changeDirection("right");
    }

});


/* =========================
   MOBILE BUTTONS
========================= */

document.querySelectorAll(
    "[data-direction]"
).forEach(button => {

    button.addEventListener(
        "click",
        () => {

            changeDirection(
                button.dataset.direction
            );

        }
    );

});


/* =========================
   RESTART
========================= */

restartButton.addEventListener(
    "click",
    startGame
);


/* =========================
   START
========================= */

resizeCanvas();

startGame();

requestAnimationFrame(gameLoop);

</script>

</body>
</html>
