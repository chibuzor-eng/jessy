<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>Police Chase Game</title>

    <style>
        * {
            box-sizing: border-box;
        }

        body {
            margin: 0;
            min-height: 100vh;
            padding: 15px;
            display: flex;
            justify-content: center;
            align-items: center;
            background: #101827;
            color: white;
            font-family: Arial, sans-serif;
        }

        .game-container {
            width: 100%;
            max-width: 450px;
            text-align: center;
        }

        .header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            gap: 8px;
            margin-bottom: 12px;
            font-weight: bold;
        }

        .score-box {
            background: #263449;
            padding: 10px;
            border-radius: 10px;
        }

        .game-area {
            position: relative;
        }

        canvas {
            display: block;
            width: 100%;
            height: auto;
            background: #303943;
            border: 3px solid #52637a;
            border-radius: 12px;
            touch-action: none;
        }

        .message {
            position: absolute;
            inset: 0;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            padding: 20px;
            background: rgba(5, 12, 25, 0.9);
            border-radius: 10px;
        }

        .message h2 {
            font-size: 28px;
            margin-bottom: 5px;
        }

        .message p {
            line-height: 1.5;
        }

        button {
            padding: 13px 22px;
            border: none;
            border-radius: 12px;
            font-size: 18px;
            font-weight: bold;
            cursor: pointer;
            touch-action: manipulation;
        }

        .start-button {
            background: #35db83;
            color: #092b19;
            margin-top: 10px;
        }

        .controls {
            display: flex;
            justify-content: center;
            gap: 25px;
            margin: 15px 0;
        }

        .controls button {
            width: 100px;
            background: #e7edf6;
            color: #152033;
            font-size: 25px;
        }

        .restart-button {
            background: #ffcc55;
            color: #202020;
        }

        .help {
            color: #bdc9da;
            font-size: 13px;
            margin-bottom: 12px;
        }

        .hidden {
            display: none;
        }
    </style>
</head>

<body>

    <div class="game-container">

        <div class="header">
            <span>🚗 POLICE CHASE</span>

            <span class="score-box">
                Score: <span id="score">0</span>
            </span>

            <span class="score-box">
                🚔 <span id="level">1</span>
            </span>
        </div>

        <div class="game-area">

            <canvas
                id="gameCanvas"
                width="420"
                height="640">
            </canvas>

            <div class="message" id="message">

                <h2 id="messageTitle">🚗 Police Chase!</h2>

                <p id="messageText">
                    Escape the police, dodge traffic,
                    and increase your score!
                </p>

                <p>
                    Use the arrow keys or the buttons below.
                </p>

                <button class="start-button" id="startButton">
                    START GAME
                </button>

            </div>

        </div>

        <div class="controls">

            <button id="leftButton">◀</button>

            <button id="rightButton">▶</button>

        </div>

        <p class="help">
            Computer: Arrow keys or A/D<br>
            Phone: Press and hold the arrow buttons
        </p>

        <button class="restart-button" id="restartButton">
            RESTART GAME
        </button>

    </div>

    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");

        const scoreElement = document.getElementById("score");
        const levelElement = document.getElementById("level");

        const message = document.getElementById("message");
        const messageTitle = document.getElementById("messageTitle");
        const messageText = document.getElementById("messageText");

        const startButton = document.getElementById("startButton");
        const restartButton = document.getElementById("restartButton");

        const W = canvas.width;
        const H = canvas.height;

        let player;
        let police;
        let traffic;

        let score = 0;
        let speed = 3;
        let roadOffset = 0;
        let spawnTimer = 0;

        let running = false;
        let lastTime = 0;

        const keys = {
            left: false,
            right: false
        };

        const lanes = [92, 162, 232, 302];

        function resetGame() {

            player = {
                x: W / 2 - 19,
                y: H - 120,
                w: 38,
                h: 66
            };

            police = {
                x: W / 2 - 18,
                y: 45,
                w: 36,
                h: 62
            };

            traffic = [];

            score = 0;
            speed = 3;
            roadOffset = 0;
            spawnTimer = 0;

            keys.left = false;
            keys.right = false;

            scoreElement.textContent = "0";
            levelElement.textContent = "1";

            message.classList.add("hidden");

            running = true;
            lastTime = performance.now();

            requestAnimationFrame(gameLoop);
        }

        function drawCar(x, y, w, h, color, isPolice = false) {

            // Car shadow
            ctx.fillStyle = "rgba(0,0,0,0.3)";
            ctx.fillRect(x + 3, y + 5, w, h);

            // Car body
            ctx.fillStyle = color;
            ctx.fillRect(x, y, w, h);

            // Front window
            ctx.fillStyle = "#c8e4f5";
            ctx.fillRect(x + 6, y + 10, w - 12, 15);

            // Rear window
            ctx.fillStyle = "#182735";
            ctx.fillRect(x + 6, y + 34, w - 12, 16);

            // Headlights
            ctx.fillStyle = "#fff3b0";
            ctx.fillRect(x + 3, y + 3, 7, 5);
            ctx.fillRect(x + w - 10, y + 3, 7, 5);

            // Rear lights
            ctx.fillStyle = "#ff3030";
            ctx.fillRect(x + 3, y + h - 7, 7, 4);
            ctx.fillRect(x + w - 10, y + h - 7, 7, 4);

            // Police siren
            if (isPolice) {

                ctx.fillStyle = "#ff3030";
                ctx.fillRect(x + 5, y + 25, 12, 6);

                ctx.fillStyle = "#258aff";
                ctx.fillRect(x + w - 17, y + 25, 12, 6);

            }
        }

        function drawRoad() {

            // Road
            ctx.fillStyle = "#303943";
            ctx.fillRect(0, 0, W, H);

            // Green roadside
            ctx.fillStyle = "#27834e";
            ctx.fillRect(0, 0, 22, H);
            ctx.fillRect(W - 22, 0, 22, H);

            // Road boundaries
            ctx.fillStyle = "#e6dca4";
            ctx.fillRect(28, 0, 3, H);
            ctx.fillRect(W - 31, 0, 3, H);

            // Lane markings
            ctx.strokeStyle = "#ffffff";
            ctx.lineWidth = 3;

            ctx.setLineDash([30, 25]);
            ctx.lineDashOffset = -roadOffset;

            for (const x of [127, 197, 267]) {

                ctx.beginPath();
                ctx.moveTo(x, 0);
                ctx.lineTo(x, H);
                ctx.stroke();

            }

            ctx.setLineDash([]);

            roadOffset = (roadOffset + speed) % 55;
        }

        function spawnTraffic() {

            const lane =
                lanes[Math.floor(Math.random() * lanes.length)];

            const colors = [
                "#f0a23b",
                "#5ca5ed",
                "#c96be8",
                "#eeeeee",
                "#ff6565"
            ];

            const color =
                colors[Math.floor(Math.random() * colors.length)];

            traffic.push({
                x: lane - 17,
                y: -80,
                w: 34,
                h: 58,
                color: color,
                speed: speed * (0.7 + Math.random() * 0.4)
            });
        }

        function drawGame() {

            drawRoad();

            // Draw traffic
            traffic.forEach(car => {

                drawCar(
                    car.x,
                    car.y,
                    car.w,
                    car.h,
                    car.color
                );

            });

            // Draw player
            drawCar(
                player.x,
                player.y,
                player.w,
                player.h,
                "#39d5a2"
            );

            // Draw police
            drawCar(
                police.x,
                police.y,
                police.w,
                police.h,
                "#f5f7fb",
                true
            );

            ctx.fillStyle = "#ffffff";
            ctx.font = "bold 12px Arial";
            ctx.fillText("ESCAPE THE POLICE!", 40, 25);
        }

        function isColliding(a, b) {

            return (
                a.x + 5 < b.x + b.w - 5 &&
                a.x + a.w - 5 > b.x + 5 &&
                a.y + 7 < b.y + b.h - 7 &&
                a.y + a.h - 7 > b.y + 7
            );
        }

        function gameLoop(currentTime) {

            if (!running) return;

            const deltaTime = Math.min(
                (currentTime - lastTime) / 16.67,
                2
            );

            lastTime = currentTime;

            // Increase difficulty as the score grows
            speed = Math.min(7, 3 + score / 450);

            // Move player
            if (keys.left) {
                player.x -= 5.2 * deltaTime;
            }

            if (keys.right) {
                player.x += 5.2 * deltaTime;
            }

            // Keep player on the road
            player.x = Math.max(
                38,
                Math.min(W - player.w - 38, player.x)
            );

            // Generate traffic
            spawnTimer += deltaTime;

            if (spawnTimer > Math.max(22, 52 - score / 30)) {

                spawnTraffic();
                spawnTimer = 0;

            }

            // Move traffic
            traffic.forEach(car => {

                car.y += car.speed * deltaTime;

            });

            // Remove cars that have passed
            traffic = traffic.filter(car => {

                if (car.y > H) {

                    score += 10;
                    return false;

                }

                return true;

            });

            // Police chase
            police.y = Math.min(
                player.y - 80,
                police.y + (0.25 + score / 5000) * deltaTime
            );

            police.x += (player.x - police.x) * 0.006 * deltaTime;

            if (police.y > player.y - 85) {
                police.y = player.y - 85;
            }

            // Check for crashes
            for (const car of traffic) {

                if (isColliding(player, car)) {

                    endGame("💥 CRASH!");

                    return;
                }

            }

            // Check whether police catches player
            if (isColliding(player, police)) {

                endGame("🚔 CAUGHT!");

                return;
            }

            // Update score
            score += Math.floor(deltaTime);

            scoreElement.textContent = score;

            levelElement.textContent = Math.min(
                5,
                1 + Math.floor(score / 250)
            );

            drawGame();

            requestAnimationFrame(gameLoop);
        }

        function endGame(title) {

            running = false;

            message.classList.remove("hidden");

            messageTitle.textContent = title;

            messageText.innerHTML =
                "Your final score is <strong>" +
                score +
                "</strong>.<br>Can you escape for longer next time?";

            startButton.textContent = "PLAY AGAIN";

            drawGame();
        }

        // Start and restart buttons
        startButton.addEventListener("click", resetGame);
        restartButton.addEventListener("click", resetGame);

        // Keyboard controls
        window.addEventListener("keydown", function(event) {

            if (
                ["ArrowLeft", "ArrowRight", "a", "d", "A", "D"]
                .includes(event.key)
            ) {
                event.preventDefault();
            }

            if (
                event.key === "ArrowLeft" ||
                event.key === "a" ||
                event.key === "A"
            ) {
                keys.left = true;
            }

            if (
                event.key === "ArrowRight" ||
                event.key === "d" ||
                event.key === "D"
            ) {
                keys.right = true;
            }

            if (
                (event.key === "Enter" || event.key === " ") &&
                !running
            ) {
                resetGame();
            }

        });

        window.addEventListener("keyup", function(event) {

            if (
                event.key === "ArrowLeft" ||
                event.key === "a" ||
                event.key === "A"
            ) {
                keys.left = false;
            }

            if (
                event.key === "ArrowRight" ||
                event.key === "d" ||
                event.key === "D"
            ) {
                keys.right = false;
            }

        });

        // Mobile touch controls
        function setupButton(buttonId, direction) {

            const button = document.getElementById(buttonId);

            button.addEventListener("pointerdown", function(event) {

                event.preventDefault();
                keys[direction] = true;

            });

            function release() {
                keys[direction] = false;
            }

            button.addEventListener("pointerup", release);
            button.addEventListener("pointerleave", release);
            button.addEventListener("pointercancel", release);

        }

        setupButton("leftButton", "left");
        setupButton("rightButton", "right");

        // Draw the initial road
        player = {
            x: W / 2 - 19,
            y: H - 120,
            w: 38,
            h: 66
        };

        police = {
            x: W / 2 - 18,
            y: 45,
            w: 36,
            h: 62
        };

        traffic = [];

        drawGame();

    </script>

</body>
</html>
