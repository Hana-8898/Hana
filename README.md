<!DOCTYPE html>
<html>
<head>
    <title>趣味时钟大挑战</title>
    <style>
        body {
            display: flex;
            flex-direction: column;
            align-items: center;
            font-family: 'Comic Sans MS', cursive;
            background: #f0f9ff;
        }
        #clock-container {
            position: relative;
            margin: 20px;
        }
        #clock {
            border: 4px solid #ff6b6b;
            border-radius: 50%;
            box-shadow: 0 0 20px rgba(255,107,107,0.3);
            background: white;
        }
        .character {
            position: absolute;
            width: 40px;
            height: 40px;
            background-size: contain;
            animation: float 3s ease-in-out infinite;
        }
        @keyframes float {
            0%, 100% { transform: translateY(0); }
            50% { transform: translateY(-20px); }
        }
        #char1 { 
            top: -30px;
            left: 130px;
            background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 36 36"><path fill="#FFAC33" d="M18 2c8 0 15 6 15 14 0 8-7 14-15 14S3 24 3 16 10 2 18 2z"/><circle fill="#292F33" cx="12" cy="14" r="3"/><circle fill="#292F33" cx="24" cy="14" r="3"/><path fill="#664500" d="M18 23c-3 0-4-1-4-1 0 1 1 2 4 2s4-1 4-2c0 0-1 1-4 1z"/></svg>');
        }
        #char2 {
            bottom: -30px;
            right: 130px;
            background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 36 36"><path fill="#77B255" d="M36 18c0 9.941-8.059 18-18 18S0 27.941 0 18 8.059 0 18 0s18 8.059 18 18z"/><path fill="#5C913B" d="M25 16c0-1-3-4-7-4s-7 3-7 4c0 1 1 2 7 2s7-1 7-2z"/><circle fill="#FFF" cx="12" cy="14" r="3"/><circle fill="#FFF" cx="24" cy="14" r="3"/></svg>');
        }
        .controls {
            margin: 10px;
        }
        select, button {
            margin: 5px;
            padding: 8px 15px;
            font-size: 16px;
            border-radius: 25px;
            border: 2px solid #ff6b6b;
            background: white;
        }
        button {
            background-color: #4CAF50;
            color: white;
            border: none;
            cursor: pointer;
            transition: transform 0.2s;
        }
        button:hover {
            background-color: #45a049;
            transform: scale(1.1);
        }
        #result {
            margin-top: 10px;
            font-size: 24px;
            min-height: 40px;
            text-align: center;
        }
        .correct-animation {
            animation: correctScale 0.8s ease-in-out;
            color: #4CAF50;
            font-weight: bold;
        }
        @keyframes correctScale {
            0% { transform: scale(0.5); opacity: 0; }
            50% { transform: scale(1.2); }
            100% { transform: scale(1); opacity: 1; }
        }
        @keyframes shake {
            0% { transform: translateX(0); }
            25% { transform: translateX(-10px); }
            50% { transform: translateX(10px); }
            75% { transform: translateX(-5px); }
            100% { transform: translateX(0); }
        }
    </style>
</head>
<body>
    <div id="clock-container">
        <div class="character" id="char1"></div>
        <canvas id="clock" width="300" height="300"></canvas>
        <div class="character" id="char2"></div>
    </div>
    <div class="controls">
        <select id="hourSelect">
            <option value="">选择小时</option>
        </select>
        <select id="minuteSelect">
            <option value="">选择分钟</option>
        </select>
    </div>
    <div>
        <button id="practiceBtn">新的题目</button>
        <button id="submitBtn">提交答案</button>
    </div>
    <div id="result"></div>
    
    <!-- 音效元素 -->
    <audio id="correctSound" src="https://assets.mixkit.co/active_storage/sfx/2571/2571-preview.mp3"></audio>
    <audio id="errorSound" src="https://assets.mixkit.co/active_storage/sfx/2996/2996-preview.mp3"></audio>

    <script>
        let currentHour, currentMinute;
        const canvas = document.getElementById('clock');
        const ctx = canvas.getContext('2d');
        const correctSound = document.getElementById('correctSound');
        const errorSound = document.getElementById('errorSound');

        // 初始化下拉菜单
        const hourSelect = document.getElementById('hourSelect');
        for (let i = 1; i <= 12; i++) {
            const option = document.createElement('option');
            option.value = i;
            option.text = i;
            hourSelect.appendChild(option);
        }

        const minuteSelect = document.getElementById('minuteSelect');
        for (let i = 0; i < 60; i++) {
            const option = document.createElement('option');
            option.value = i;
            option.text = i < 10 ? '0' + i : i;
            minuteSelect.appendChild(option);
        }

        function generateNewQuestion() {
            currentHour = Math.floor(Math.random() * 12) + 1;
            currentMinute = Math.floor(Math.random() * 60);
            drawClock(currentHour, currentMinute);
            hourSelect.value = '';
            minuteSelect.value = '';
            document.getElementById('result').textContent = '';
        }

        function drawClock(hour, minute) {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            const radius = canvas.width / 2;
            ctx.translate(radius, radius);
            const drawingRadius = radius * 0.9;

            // 白色表盘背景
            ctx.fillStyle = 'white';
            ctx.beginPath();
            ctx.arc(0, 0, drawingRadius, 0, Math.PI * 2);
            ctx.fill();

            // 彩色刻度
            ctx.lineWidth = 3;
            for (let i = 0; i < 60; i++) {
                ctx.strokeStyle = i % 5 === 0 ? '#ff7675' : '#a5a5a5';
                const angle = (i * 6) * Math.PI / 180;
                const lineLength = i % 5 === 0 ? 20 : 10;
                ctx.beginPath();
                ctx.moveTo((drawingRadius-20) * Math.cos(angle), (drawingRadius-20) * Math.sin(angle));
                ctx.lineTo(drawingRadius * Math.cos(angle), drawingRadius * Math.sin(angle));
                ctx.stroke();
            }

            // 立体数字
            ctx.font = 'bold 22px Arial Rounded MT Bold';
            ctx.fillStyle = '#2d3436';
            ctx.strokeStyle = '#f8f9fa';
            ctx.lineWidth = 2;
            for (let i = 1; i <= 12; i++) {
                const angle = (i * 30 - 90) * Math.PI / 180;
                const textRadius = drawingRadius * 0.75;
                ctx.save();
                ctx.translate(textRadius * Math.cos(angle), textRadius * Math.sin(angle));
                ctx.rotate(angle + Math.PI/2);
                ctx.strokeText(i.toString(), 0, 0);
                ctx.fillText(i.toString(), 0, 0);
                ctx.restore();
            }

            // 绘制指针
            function createPointerGradient(length) {
                const grd = ctx.createLinearGradient(0, 0, length, 0);
                grd.addColorStop(0, '#ff6b6b');
                grd.addColorStop(1, '#ff8e8e');
                return grd;
            }

            // 时针
            const hourAngle = ((hour % 12) * 30 + minute * 0.5 - 90) * Math.PI / 180;
            ctx.shadowColor = 'rgba(0,0,0,0.2)';
            ctx.shadowBlur = 8;
            ctx.shadowOffsetY = 3;
            drawHand(ctx, hourAngle, drawingRadius * 0.6, 8, createPointerGradient(100));

            // 分针
            const minuteAngle = (minute * 6 - 90) * Math.PI / 180;
            ctx.shadowBlur = 5;
            drawHand(ctx, minuteAngle, drawingRadius * 0.8, 5, '#4b7bec');
            ctx.setLineDash([5, 3]);
            ctx.strokeStyle = 'white';
            ctx.lineWidth = 2;
            drawHand(ctx, minuteAngle, drawingRadius * 0.8, 5, 'transparent');
            ctx.setLineDash([]);
            
            ctx.translate(-radius, -radius);
        }

        function drawHand(ctx, angle, length, width, color) {
            ctx.save();
            ctx.strokeStyle = color;
            ctx.lineWidth = width;
            ctx.lineCap = 'round';
            ctx.beginPath();
            ctx.moveTo(0, 0);
            ctx.lineTo(Math.cos(angle) * length, Math.sin(angle) * length);
            ctx.stroke();
            ctx.restore();
        }

        function checkAnswer() {
            const selectedHour = parseInt(hourSelect.value, 10);
            const selectedMinute = parseInt(minuteSelect.value, 10);
            const resultDiv = document.getElementById('result');
            
            if (selectedHour === currentHour && selectedMinute === currentMinute) {
                resultDiv.textContent = '🎉 PERFECT！即将刷新...';
                resultDiv.style.color = '#4CAF50';
                resultDiv.classList.add('correct-animation');
                correctSound.play();
                setTimeout(() => {
                    resultDiv.classList.remove('correct-animation');
                    generateNewQuestion();
                }, 1000);
            } else {
                resultDiv.textContent = '❌ 错误，请再试一次';
                resultDiv.style.color = '#ff4444';
                errorSound.play();
                resultDiv.style.animation = 'shake 0.5s';
                setTimeout(() => {
                    resultDiv.style.animation = '';
                }, 500);
            }
        }

        document.getElementById('practiceBtn').addEventListener('click', generateNewQuestion);
        document.getElementById('submitBtn').addEventListener('click', checkAnswer);

        // 初始生成题目
        generateNewQuestion();
    </script>
</body>
</html># Hana
