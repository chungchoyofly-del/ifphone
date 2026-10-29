# ifphone<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>힐링 스파 & 스킨케어 숍</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Malgun Gothic', sans-serif;
        }
        body {
            background-color: #f7f3ef;
            color: #5a524c;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }
        .game-container {
            width: 420px;
            background: #ffffff;
            border-radius: 24px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.05);
            padding: 24px;
            text-align: center;
        }
        h1 {
            font-size: 20px;
            color: #8c7a6b;
            margin-bottom: 8px;
            font-weight: 600;
        }
        .status-bar {
            display: flex;
            justify-content: space-around;
            background: #fdfbf7;
            padding: 12px;
            border-radius: 12px;
            margin-bottom: 20px;
            font-size: 14px;
            color: #7a6e65;
        }
        .customer-card {
            background: #faf6f0;
            border-radius: 20px;
            padding: 20px;
            margin-bottom: 20px;
            position: relative;
        }
        .face-avatar {
            width: 120px;
            height: 120px;
            background: #ffe5d9;
            border-radius: 50%;
            margin: 0 auto 15px;
            position: relative;
            display: flex;
            justify-content: center;
            align-items: center;
            transition: all 0.5s ease;
        }
        .eyes {
            width: 60px;
            display: flex;
            justify-content: space-between;
            position: absolute;
            top: 45px;
        }
        .eye {
            width: 10px;
            height: 10px;
            background: #5a524c;
            border-radius: 50%;
        }
        .smile {
            width: 30px;
            height: 12px;
            border-bottom: 3px solid #5a524c;
            border-radius: 0 0 15px 15px;
            position: absolute;
            top: 65px;
            transition: all 0.3s ease;
        }
        /* 피부 상태 오버레이 */
        .skin-layer {
            position: absolute;
            width: 100%;
            height: 100%;
            border-radius: 50%;
            top: 0;
            left: 0;
            pointer-events: none;
            transition: opacity 0.5s ease;
        }
        .dirt { background: rgba(100, 70, 40, 0.25); border-radius: 50%; }
        .mask { background: rgba(168, 218, 220, 0.7); border-radius: 50%; }
        .glow { box-shadow: 0 0 25px #ffd1dc; }

        .dialogue {
            font-size: 14px;
            color: #6c5ce7;
            min-height: 40px;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .tools-grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 10px;
            margin-bottom: 15px;
        }
        .tool-btn {
            background: #ffffff;
            border: 2px solid #f0e6df;
            border-radius: 16px;
            padding: 12px 8px;
            cursor: pointer;
            transition: all 0.2s ease;
            font-size: 13px;
            color: #5a524c;
        }
        .tool-btn:hover {
            background: #f7ede2;
            border-color: #f5abe2;
            transform: translateY(-2px);
        }
        .tool-btn:disabled {
            opacity: 0.4;
            cursor: not-allowed;
            transform: none;
        }
        .tool-icon {
            font-size: 22px;
            display: block;
            margin-bottom: 4px;
        }
        .action-btn {
            width: 100%;
            padding: 14px;
            background: #e8aea1;
            color: white;
            border: none;
            border-radius: 16px;
            font-size: 15px;
            font-weight: bold;
            cursor: pointer;
            transition: background 0.2s;
        }
        .action-btn:hover {
            background: #d99b8e;
        }
    </style>
</head>
<body>

<div class="game-container">
    <h1>🌿 마인드 힐링 스파</h1>
    
    <div class="status-bar">
        <span>만족한 손님: <strong id="guest-count">0</strong>명</span>
        <span>보유 골드: <strong id="gold">0</strong> G</span>
    </div>

    <div class="customer-card">
        <div class="face-avatar" id="face">
            <div class="eyes">
                <div class="eye"></div>
                <div class="eye"></div>
            </div>
            <div class="smile" id="smile"></div>
            <div class="skin-layer" id="skin-layer"></div>
        </div>
        <div class="dialogue" id="dialogue">손님이 방문했습니다!</div>
    </div>

    <div class="tools-grid">
        <button class="tool-btn" onclick="applyStep(1)" id="btn-step1">
            <span class="tool-icon">🧼</span>
            클렌징 폼
        </button>
        <button class="tool-btn" onclick="applyStep(2)" id="btn-step2" disabled>
            <span class="tool-icon">🥒</span>
            진정 팩
        </button>
        <button class="tool-btn" onclick="applyStep(3)" id="btn-step3" disabled>
            <span class="tool-icon">✨</span>
            보습 에센스
        </button>
    </div>

    <button class="action-btn" id="next-btn" onclick="nextCustomer()" style="display: none;">다음 손님 맞이하기</button>
</div>

<script>
    let gold = 0;
    let guests = 0;
    let currentStep = 0; // 0: 시작 전, 1: 세안완료, 2: 팩완료, 3: 에센스완료

    const dialogueText = document.getElementById('dialogue');
    const skinLayer = document.getElementById('skin-layer');
    const smile = document.getElementById('smile');
    const nextBtn = document.getElementById('next-btn');
    
    const btn1 = document.getElementById('btn-step1');
    const btn2 = document.getElementById('btn-step2');
    const btn3 = document.getElementById('btn-step3');

    function initCustomer() {
        currentStep = 0;
        skinLayer.className = 'skin-layer dirt';
        smile.style.borderRadius = '15px 15px 0 0'; // 시무룩한 표정
        smile.style.borderBottom = 'none';
        smile.style.borderTop = '3px solid #5a524c';
        
        dialogueText.innerText = "피부가 지치고 건조해요... 케어가 필요합니다.";
        
        btn1.disabled = false;
        btn2.disabled = true;
        btn3.disabled = true;
        nextBtn.style.display = 'none';
    }

    function applyStep(step) {
        if (step === 1 && currentStep === 0) {
            currentStep = 1;
            skinLayer.className = 'skin-layer'; // 노폐물 제거
            dialogueText.innerText = "부드럽게 클렌징하여 먼지를 닦아냈습니다.";
            btn1.disabled = true;
            btn2.disabled = false;
        } else if (step === 2 && currentStep === 1) {
            currentStep = 2;
            skinLayer.className = 'skin-layer mask'; // 팩 적용
            dialogueText.innerText = "시원한 수분 팩으로 피부를 진정시키는 중...";
            btn2.disabled = true;
            btn3.disabled = false;
        } else if (step === 3 && currentStep === 2) {
            currentStep = 3;
            skinLayer.className = 'skin-layer glow'; // 미백/광채 효과
            
            // 미소 짓는 표정으로 변경
            smile.style.borderRadius = '0 0 15px 15px';
            smile.style.borderTop = 'none';
            smile.style.borderBottom = '3px solid #5a524c';

            dialogueText.innerText = "피부가 촉촉하게 빛나요! 최고의 스파였습니다 🌸";
            btn3.disabled = true;

            // 보상 지급
            gold += 50;
            guests += 1;
            document.getElementById('gold').innerText = gold;
            document.getElementById('guest-count').innerText = guests;

            nextBtn.style.display = 'block';
        }
    }

    function nextCustomer() {
        initCustomer();
    }

    // 첫 손님 실행
    initCustomer();
</script>

</body>
</html>
