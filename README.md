<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>BPM 측정기</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            color: white;
        }

        .container {
            text-align: center;
            background: rgba(255, 255, 255, 0.1);
            backdrop-filter: blur(10px);
            border-radius: 20px;
            padding: 40px;
            box-shadow: 0 25px 45px rgba(0, 0, 0, 0.1);
            border: 1px solid rgba(255, 255, 255, 0.2);
            max-width: 500px;
            width: 90%;
        }

        h1 {
            font-size: 2.5em;
            margin-bottom: 10px;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
        }

        .subtitle {
            font-size: 1.1em;
            opacity: 0.8;
            margin-bottom: 30px;
        }

        .bpm-display {
            font-size: 4em;
            font-weight: bold;
            margin: 30px 0;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
            transition: all 0.3s ease;
        }

        .tap-button {
            width: 200px;
            height: 200px;
            border-radius: 50%;
            background: linear-gradient(45deg, #ff6b6b, #ee5a24);
            border: none;
            color: white;
            font-size: 1.5em;
            font-weight: bold;
            cursor: pointer;
            margin: 20px auto;
            display: block;
            transition: all 0.2s ease;
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
            outline: none; /* 키보드 포커스 테두리 제거 */
        }

        .tap-button:hover {
            transform: translateY(-5px);
            box-shadow: 0 15px 40px rgba(0, 0, 0, 0.4);
            background: linear-gradient(45deg, #ff5252, #d63031);
        }

        .tap-button:active, .tap-button.active {
            transform: scale(0.95) translateY(0);
            box-shadow: 0 5px 20px rgba(0, 0, 0, 0.3);
            background: linear-gradient(45deg, #ff3838, #c0392b);
        }

        .info {
            margin: 20px 0;
            font-size: 1.1em;
            opacity: 0.9;
        }

        .stats {
            display: flex;
            justify-content: space-around;
            margin: 20px 0;
            font-size: 0.9em;
        }

        .stat-item {
            text-align: center;
        }

        .stat-value {
            font-size: 1.5em;
            font-weight: bold;
            display: block;
        }

        .reset-button {
            background: rgba(255, 255, 255, 0.2);
            border: 1px solid rgba(255, 255, 255, 0.3);
            color: white;
            padding: 12px 24px;
            border-radius: 25px;
            cursor: pointer;
            font-size: 1em;
            margin-top: 20px;
            transition: all 0.3s ease;
        }

        .reset-button:hover {
            background: rgba(255, 255, 255, 0.3);
            transform: translateY(-2px);
        }

        .instructions {
            margin-top: 20px;
            font-size: 0.9em;
            opacity: 0.7;
            line-height: 1.5;
        }

        @keyframes pulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.05); }
            100% { transform: scale(1); }
        }

        .pulse {
            animation: pulse 0.2s ease-in-out;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>BPM 측정기</h1>
        <p class="subtitle">음악에 맞춰 버튼을 누르거나 키보드를 쳐보세요</p>
        
        <div class="bpm-display" id="bpmDisplay">--</div>
        
        <button class="tap-button" id="tapBtn">TAP</button>
        
        <div class="stats">
            <div class="stat-item">
                <span class="stat-value" id="tapCount">0</span>
                <span>탭 횟수</span>
            </div>
        </div>

        <button class="reset-button" id="resetBtn">리셋</button>
        
        <p class="instructions">
            아무 키보드 키나 마우스 클릭으로 간격을 측정합니다.<br>
            새로 시작하려면 '리셋' 버튼을 누르세요.
        </p>
    </div>

    <script>
        const bpmDisplay = document.getElementById('bpmDisplay');
        const tapBtn = document.getElementById('tapBtn');
        const tapCountDisplay = document.getElementById('tapCount');
        const resetBtn = document.getElementById('resetBtn');

        let timestamps = [];

        // 탭 입력 시 작동하는 핵심 함수
        function handleTap() {
            const now = Date.now();
            timestamps.push(now);

            // 최근 30개의 입력만 유지 (오래된 데이터는 성능 및 정확도를 위해 삭제)
            if (timestamps.length > 30) {
                timestamps.shift();
            }

            // 시각적 피드백 효과 (숫자 커졌다 작아지기)
            bpmDisplay.classList.remove('pulse');
            void bpmDisplay.offsetWidth; // 리플로우 강제 발생으로 애니메이션 초기화
            bpmDisplay.classList.add('pulse');

            // BPM 계산 (최소 2번 이상 눌려야 계산 가능)
            if (timestamps.length > 1) {
                let intervals = [];
                for (let i = 1; i < timestamps.length; i++) {
                    intervals.push(timestamps[i] - timestamps[i - 1]);
                }
                
                // 간격들의 평균 구하기 (밀리초 단위)
                const averageInterval = intervals.reduce((sum, val) => sum + val, 0) / intervals.length;
                
                // 1분(60,000ms)을 평균 간격으로 나누어 BPM 산출
                const bpm = Math.round(60000 / averageInterval);
                bpmDisplay.textContent = bpm;
            }

            tapCountDisplay.textContent = timestamps.length;
        }

        // 리셋 함수
        function resetCalculator() {
            timestamps = [];
            bpmDisplay.textContent = '--';
            tapCountDisplay.textContent = '0';
        }

        // 1. 마우스 클릭 / 터치 이벤트
        tapBtn.addEventListener('click', handleTap);

        // 2. 키보드 다운 이벤트 (아무 키나 눌렀을 때)
        window.addEventListener('keydown', (e) => {
            // 포커스가 리셋 버튼에 가있을 때 엔터/스페이스를 누르면 중복 작동하는 현상 방지
            if (e.target.tagName === 'BUTTON' && (e.key === ' ' || e.key === 'Enter')) {
                e.preventDefault();
            }
            
            // 키를 꾹 누르고 있을 때 연속으로 찍히는 것 방지
            if (e.repeat) return; 

            tapBtn.classList.add('active'); // 버튼이 눌린 스타일 적용
            handleTap();
        });

        // 3. 키보드 업 이벤트 (키에서 손을 뗐을 때 버튼 모양 복구)
        window.addEventListener('keyup', () => {
            tapBtn.classList.remove('active');
        });

        // 리셋 버튼 이벤트
        resetBtn.addEventListener('click', (e) => {
            resetCalculator();
            e.blur(); // 리셋 후 포커스를 해제하여 키보드 오작동 방지
        });
    </script>
</body>
</html>
