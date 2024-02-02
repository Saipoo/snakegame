# snakegame
login ,register and snake game
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Registration, Login & Snake Game</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            font-family: Arial, sans-serif;
            background-color: #111;
            color: #fff;
        }

        .form-container {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            width: 100vw;
            height: 100vh;
            position: absolute;
            top: 0;
        }

        .form-box {
            display: none;
            background-color: #222;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(255, 255, 255, 0.3);
        }

        .form-box.show {
            display: block;
        }

        .form-box h2 {
            margin-bottom: 20px;
            text-align: center;
            color: #ff6600;
        }

        .form-box input[type="text"],
        .form-box input[type="password"],
        .form-box input[type="email"],
        .form-box select,
        .form-box input[type="date"],
        .form-box input[type="submit"] {
            width: 100%;
            padding: 10px;
            margin: 5px 0;
            border: none;
            border-radius: 5px;
            background-color: #444;
            color: #fff;
            font-size: 16px;
            box-sizing: border-box;
        }

        .form-box input[type="submit"] {
            cursor: pointer;
            background-color: #ff6600;
            transition: background-color 0.3s ease;
        }

        .form-box input[type="submit"]:hover {
            background-color: #ff944d;
        }

        #game-container {
            display: none;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            width: 100vw;
            height: 100vh;
            position: absolute;
            top: 0;
        }

        canvas {
            border: 2px solid #fff;
            background-color: #000;
        }

        #game-info {
            color: #fff;
            font-size: 24px;
            margin-top: 20px;
        }
    </style>
</head>
<body>
    <div class="form-container">
        <div class="form-box show" id="register-form">
            <h2>Registration</h2>
            <form id="registration-form">
                <input type="text" id="reg-username" placeholder="Username" required>
                <input type="password" id="reg-password" placeholder="Password" required>
                <input type="email" id="reg-email" placeholder="Email" required>
                <select id="reg-gender" required>
                    <option value="">Select Gender</option>
                    <option value="male">Male</option>
                    <option value="female">Female</option>
                    <option value="other">Other</option>
                </select>
                <input type="date" id="reg-dob" placeholder="Date of Birth" required>
                <input type="submit" value="Register">
            </form>
            <p>Already have an account? <a href="#" id="toggle-form">Login</a></p>
        </div>

        <div class="form-box" id="login-form">
            <h2>Login</h2>
            <form id="login-form">
                <input type="text" id="login-username" placeholder="Username" required>
                <input type="password" id="login-password" placeholder="Password" required>
                <input type="submit" value="Login">
            </form>
            <p>Don't have an account? <a href="#" id="toggle-form">Register</a></p>
        </div>
    </div>

    <div id="game-container">
        <canvas id="game-canvas"></canvas>
        <div id="game-info">Click "Start" to play</div>
        <button id="logout-btn">Logout</button>
        <button id="start-btn">Start</button>
    </div>

    <script>
        const registerForm = document.getElementById('register-form');
        const loginForm = document.getElementById('login-form');
        const gameContainer = document.getElementById('game-container');
        const toggleFormLinks = document.querySelectorAll('#toggle-form');
        const canvas = document.getElementById('game-canvas');
        const ctx = canvas.getContext('2d');
        const infoElement = document.getElementById('game-info');
        const logoutButton = document.getElementById('logout-btn');
        const startButton = document.getElementById('start-btn');
        const cellSize = 20;
        const gridSize = Math.floor(Math.min(window.innerWidth, window.innerHeight) / cellSize);
        canvas.width = gridSize * cellSize;
        canvas.height = gridSize * cellSize;
        let snake = [{ x: Math.floor(gridSize / 2), y: Math.floor(gridSize / 2) }];
        let food = { x: 0, y: 0 };
        let dx = 0;
        let dy = 0;
        let score = 0;
        let gameStarted = false;
        let intervalId;

        function getRandomFoodColor() {
            const colors = ['#ff0000', '#00ff00', '#0000ff', '#ffff00', '#ff00ff', '#00ffff'];
            return colors[Math.floor(Math.random() * colors.length)];
        }

        function drawCell(x, y, color) {
            ctx.fillStyle = color;
            ctx.fillRect(x * cellSize, y * cellSize, cellSize, cellSize);
        }

        function drawSnake() {
            snake.forEach(segment => drawCell(segment.x, segment.y, '#00ff00'));
        }

        function drawFood() {
            drawCell(food.x, food.y, getRandomFoodColor());
        }

        function moveSnake() {
            const head = { x: snake[0].x + dx, y: snake[0].y + dy };
            snake.unshift(head);
            if (head.x === food.x && head.y === food.y) {
                score++;
                infoElement.textContent = `Score: ${score}`;
                placeFood();
            } else {
                snake.pop();
            }
        }

        function placeFood() {
            food = { x: Math.floor(Math.random() * gridSize), y: Math.floor(Math.random() * gridSize) };
            if (snake.some(segment => segment.x === food.x && segment.y === food.y)) {
                placeFood();
            }
        }

        function checkCollision() {
            if (snake[0].x < 0 || snake[0].x >= gridSize || snake[0].y < 0 || snake[0].y >= gridSize || snake.slice(1).some(segment => segment.x === snake[0].x && segment.y === snake[0].y)) {
                clearInterval(intervalId);
                gameStarted = false;
                infoElement.textContent = `Game Over! Score: ${score}. Click "Start" to play again.`;
            }
        }

        function update() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            drawFood();
            drawSnake();
            moveSnake();
            checkCollision();
        }

        function startGame() {
            if (!gameStarted) {
                gameStarted = true;
                score = 0;
                infoElement.textContent = `Score: ${score}`;
                snake = [{ x: Math.floor(gridSize / 2), y: Math.floor(gridSize / 2) }];
                dx = 1;
                dy = 0;
                placeFood();
                intervalId = setInterval(update, 100);
            }
        }

        document.addEventListener('keydown', e => {
            if (!gameStarted) return;
            switch (e.key) {
                case 'ArrowUp':
                    if (dy === 0) {
                        dx = 0;
                        dy = -1;
                    }
                    break;
                case 'ArrowDown':
                    if (dy === 0) {
                        dx = 0;
                        dy = 1;
                    }
                    break;
                case 'ArrowLeft':
                    if (dx === 0) {
                        dx = -1;
                        dy = 0;
                    }
                    break;
                case 'ArrowRight':
                    if (dx === 0) {
                        dx = 1;
                        dy = 0;
                    }
                    break;
            }
        });

        startButton.addEventListener('click', startGame);

        // Function to toggle between registration and login forms
        function toggleForm(event) {
            event.preventDefault();
            registerForm.classList.toggle('show');
            loginForm.classList.toggle('show');
        }

        // Event listeners for toggle links
        toggleFormLinks.forEach(link => link.addEventListener('click', toggleForm));

        // Function to handle form submission (registration or login)
        function handleFormSubmit(event) {
            event.preventDefault();

            // Check if it's a registration or login form
            const formId = event.target.id;
            if (formId === 'registration-form') {
                // Handle registration
                // ...

                // After registration, automatically switch to login form
                registerForm.classList.remove('show');
                loginForm.classList.add('show');
            } else if (formId === 'login-form') {
                // Handle login
                // ...

                // After successful login, start the Snake game
                startGame();
                gameContainer.style.display = 'flex'; // Show game container
                document.querySelectorAll('.form-box').forEach(form => form.classList.remove('show'));
            }
        }

        // Function to handle user logout
        function logout() {
            gameContainer.style.display = 'none';
            document.querySelectorAll('.form-box').forEach(form => form.classList.add('show'));
            clearInterval(intervalId); // Stop Snake game interval
            gameStarted = false;
        }

        // Event listener for logout button
        logoutButton.addEventListener('click', logout);

        // Event listeners for form submissions
        registerForm.addEventListener('submit', handleFormSubmit);
        loginForm.addEventListener('submit', handleFormSubmit);
    </script>
</body>
</html>
