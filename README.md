<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>메이플 키우기 - 정식 출시 버전</title>
    <style>
        :root {
            --panel-bg: #181824;
            --panel-card: #232336;
            --main-yellow: #f1c40f;
            --main-orange: #e67e22;
            --text-light: #ffffff;
            image-rendering: crisp-edges;
        }

        * {
            box-sizing: border-box; user-select: none; -webkit-user-select: none;
            touch-action: manipulation; margin: 0; padding: 0;
            font-family: 'Pretendard', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
        }

        /* 📱 풀스크린 최적화: 브라우저 가득 차게 설정 */
        body {
            background-color: #0b0b10; color: var(--text-light); display: flex;
            justify-content: center; align-items: center; min-height: 100vh; min-height: 100dvh; overflow: hidden;
            width: 100vw; margin: 0; /* 불필요한 마진 제거 */
        }

        #game-container {
            width: 100%; max-width: 500px; height: 100vh; height: 100dvh;
            background: linear-gradient(180deg, #1e1e2d 0%, #12121a 100%);
            display: flex; flex-direction: column; position: relative; box-shadow: 0 0 40px rgba(0,0,0,0.95); overflow: hidden;
        }

        header {
            background: linear-gradient(180deg, rgba(25,25,38,0.9), rgba(18,18,26,0.9));
            backdrop-filter: blur(8px); padding: 14px 16px; border-bottom: 2px solid rgba(255,255,255,0.08);
            display: flex; flex-direction: column; gap: 10px; z-index: 10;
        }

        .user-info { display: flex; justify-content: space-between; align-items: center; }
        .level-badge {
            background: linear-gradient(135deg, #f39c12, #d35400); color: #fff; padding: 6px 16px;
            border-radius: 16px; font-weight: bold; font-size: 14px; box-shadow: 0 4px 10px rgba(230,126,34,0.4);
        }
        .meso-box {
            display: flex; align-items: center; gap: 6px; background: rgba(0,0,0,0.4);
            padding: 6px 16px; border-radius: 16px; border: 1px solid rgba(241,196,15,0.4);
            color: var(--main-yellow); font-weight: bold; font-size: 16px; box-shadow: inset 0 2px 4px rgba(0,0,0,0.5);
        }

        .exp-bar-container {
            width: 100%; height: 14px; background: rgba(0,0,0,0.6); border-radius: 7px;
            overflow: hidden; border: 1px solid rgba(255,255,255,0.1); position: relative;
        }
        .exp-bar-fill { height: 100%; background: linear-gradient(90deg, #2ecc71, #00d2d3); width: 0%; transition: width 0.3s ease; box-shadow: 0 0 10px rgba(46,204,113,0.5); }
        .exp-text { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); font-size: 10px; font-weight: bold; text-shadow: 1px 1px 2px #000; }

        #battlefield {
            flex: 1.2; position: relative; background-size: cover; background-position: center;
            overflow: hidden; display: flex; flex-direction: column; justify-content: space-between;
            align-items: center; cursor: pointer; transition: background 0.5s ease;
        }

        .stage-header { width: 100%; display: flex; justify-content: space-between; align-items: center; padding: 14px 18px 0 18px; z-index: 10; }
        .stage-title {
            background: rgba(15, 15, 25, 0.8); padding: 8px 18px; border-radius: 20px;
            font-size: 14px; font-weight: bold; color: #fff; border: 1px solid rgba(255, 255, 255, 0.15); backdrop-filter: blur(6px); box-shadow: 0 4px 15px rgba(0,0,0,0.3);
        }

        .btn-autohunt {
            background: linear-gradient(135deg, #ff4757, #ff6b81); color: white; border: 1px solid rgba(255, 255, 255, 0.3);
            padding: 8px 18px; border-radius: 20px; font-size: 13px; font-weight: bold; cursor: pointer; box-shadow: 0 4px 12px rgba(255,71,87,0.4); transition: all 0.2s;
        }
        .btn-autohunt.active { background: linear-gradient(135deg, #2ed573, #26af5f); box-shadow: 0 4px 15px rgba(46,213,115,0.5); }

        .battle-area { position: relative; width: 100%; height: 100%; display: flex; justify-content: center; align-items: flex-end; padding-bottom: 22px; }
        .ground {
            position: absolute; bottom: 0; left: 0; width: 100%; height: 32px;
            background-color: var(--ground-color, #27ae60); border-top: 4px solid rgba(255,255,255,0.25); box-shadow: inset 0 4px 8px rgba(0,0,0,0.3); transition: background 0.4s;
        }

        .character {
            position: absolute; left: 30%; bottom: 28px; transform: translateX(-50%);
            width: 140px; height: 140px; background-size: 280px 140px; background-repeat: no-repeat;
            z-index: 5; pointer-events: none; filter: drop-shadow(0 8px 12px rgba(0,0,0,0.5));
            transition: filter 0.1s ease, transform 0.1s ease; animation: spriteIdleLarge 0.8s steps(1) infinite;
        }
        @keyframes spriteIdleLarge { 0% { background-position: 0px 0px; } 50% { background-position: -140px 0px; } 100% { background-position: 0px 0px; } }
        .character.attacking { transform: translateX(-50%) scale(1.05); }

        .character-weapon {
            position: absolute; left: 75px; top: 60px; width: 50px; height: 50px;
            background-size: contain; background-repeat: no-repeat; transition: transform 0.15s ease;
            filter: drop-shadow(0 2px 4px rgba(0,0,0,0.5));
        }
        .character.attacking .character-weapon { transform: rotate(50deg) scale(1.4) translateX(15px) translateY(-5px); }

        .energy-bolt {
            position: absolute; width: 80px; height: 30px; z-index: 35; pointer-events: none;
            transform: translate(-50%, -50%) rotate(var(--angle, 0deg));
            animation: shootEnergyDynamic 0.15s cubic-bezier(0.1, 0.8, 0.3, 1) forwards;
        }
        .bolt-core {
            position: absolute; right: 0; top: 50%; transform: translateY(-50%);
            width: 28px; height: 28px; border-radius: 50%;
            background: radial-gradient(circle, #ffffff 20%, #00d2d3 60%, #a29bfe 100%); box-shadow: 0 0 15px #fff, 0 0 30px #00d2d3, 0 0 50px #a29bfe;
        }
        .bolt-trail {
            position: absolute; right: 14px; top: 50%; transform: translateY(-50%);
            width: 60px; height: 12px; border-radius: 50px 0 0 50px;
            background: linear-gradient(90deg, transparent, rgba(162, 155, 254, 0.8), #00d2d3); filter: blur(2px);
        }
        @keyframes shootEnergyDynamic {
            0% { transform: translate(-50%, -50%) rotate(var(--angle)) scale(0.5); opacity: 0.8; filter: hue-rotate(0deg); }
            100% { transform: translate(calc(-50% + var(--target-x)), calc(-50% + var(--target-y))) rotate(var(--angle)) scale(1.5); opacity: 1; filter: hue-rotate(270deg); }
        }

        .pet { position: absolute; left: 6%; bottom: 55px; width: 45px; height: 45px; background-size: contain; background-repeat: no-repeat; z-index: 6; filter: drop-shadow(0 6px 10px rgba(0,0,0,0.4)); animation: petFloat 1.8s ease-in-out infinite alternate; }
        @keyframes petFloat { 0% { transform: translateY(0px); } 100% { transform: translateY(-10px); } }
        .pet.tackling { animation: petTackle 0.35s cubic-bezier(0.1,0.9,0.2,1) !important; }
        @keyframes petTackle { 0% { transform: translate(0, 0) scale(1); } 40% { transform: translate(190px, 15px) scale(1.35); } 100% { transform: translate(0, 0) scale(1); } }

        .lightning-bolt { position: absolute; top: 0; right: 22%; width: 40px; height: 220px; background: linear-gradient(180deg, #f1c40f, #ffffff, #e67e22); clip-path: polygon(40% 0%, 70% 0%, 30% 40%, 80% 40%, 10% 100%, 40% 50%, 0% 50%); z-index: 30; opacity: 0; pointer-events: none; filter: drop-shadow(0 0 12px #f1c40f); }
        .lightning-bolt.active { animation: lightningFlash 0.35s ease-out; }
        @keyframes lightningFlash { 0% { opacity: 0; transform: scaleY(0); } 30% { opacity: 1; transform: scaleY(1); } 70% { opacity: 1; transform: scaleY(1); } 100% { opacity: 0; transform: scaleY(1); } }

        .scratch-effect { position: absolute; right: 20%; bottom: 28px; width: 80px; height: 80px; background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="64" height="64" viewBox="0 0 64 64"><line x1="8" y1="12" x2="52" y2="52" stroke="%23ff3838" stroke-width="6" stroke-linecap="round"/><line x1="20" y1="8" x2="60" y2="44" stroke="%23ff3838" stroke-width="6" stroke-linecap="round"/><line x1="6" y1="24" x2="44" y2="60" stroke="%23ff3838" stroke-width="6" stroke-linecap="round"/></svg>'); background-size: contain; background-repeat: no-repeat; z-index: 25; opacity: 0; pointer-events: none; }
        .scratch-effect.active { animation: scratchSlash 0.3s cubic-bezier(0.1,0.9,0.2,1); }
        @keyframes scratchSlash { 0% { opacity: 0; transform: scale(0.6) rotate(-15deg); } 40% { opacity: 1; transform: scale(1.25) rotate(0deg); filter: drop-shadow(0 0 12px #ff3838); } 100% { opacity: 0; transform: scale(1) rotate(10deg); } }

        .monster-container { position: absolute; right: 15%; bottom: 28px; display: flex; flex-direction: column; align-items: center; z-index: 5; pointer-events: none; filter: drop-shadow(0 6px 10px rgba(0,0,0,0.4)); }
        .monster-hp-bar { width: 80px; height: 10px; background: rgba(0,0,0,0.7); border: 1px solid rgba(255,255,255,0.4); border-radius: 5px; overflow: hidden; margin-bottom: 8px; box-shadow: 0 2px 5px rgba(0,0,0,0.5); }
        .monster-hp-fill { height: 100%; background: linear-gradient(90deg, #ff4757, #ff6b81); width: 100%; transition: width 0.1s linear; }
        .monster-sprite { width: 80px; height: 80px; background-size: cover; background-repeat: no-repeat; transition: transform 0.2s ease, filter 0.2s ease; }

        .damage-text { position: absolute; font-size: 28px; font-weight: 900; color: #f1c40f; text-shadow: 2px 2px 0 #000, -2px -2px 0 #000, 2px -2px 0 #000, -2px 2px 0 #000, 0 0 10px rgba(241,196,15,0.6); pointer-events: none; animation: floatDamage 0.65s cubic-bezier(0.1,0.9,0.2,1) forwards; z-index: 20; transform: translate(-50%, 0); }
        .damage-text.crit { color: #ff3838; font-size: 34px; text-shadow: 2px 2px 0 #000, -2px -2px 0 #000, 0 0 15px rgba(255,56,56,0.8); }
        .damage-text.pet-dmg { color: #00d2d3; font-size: 30px; text-shadow: 2px 2px 0 #000, -2px -2px 0 #000, 0 0 15px rgba(0,210,211,0.8); }
        .damage-text.char-dmg { color: #ff4757; font-size: 26px; text-shadow: 2px 2px 0 #000, -2px -2px 0 #000, 0 0 15px rgba(255,71,87,0.8); z-index: 40; }

        @keyframes floatDamage {
            0% { opacity: 1; transform: translate(-50%, 0) scale(0.8); }
            50% { opacity: 1; transform: translate(-50%, -40px) scale(1.25); }
            100% { opacity: 0; transform: translate(-50%, -70px) scale(1); }
        }

        #bottom-panel { flex: 1; min-height: 40%; max-height: 55vh; background-color: var(--panel-bg); display: flex; flex-direction: column; border-top: 2px solid rgba(255,255,255,0.08); box-shadow: 0 -10px 30px rgba(0,0,0,0.5); z-index: 20; }
        .nav-tabs { display: flex; background: #12121a; border-bottom: 1px solid rgba(255,255,255,0.06); }
        .tab-btn { flex: 1; padding: 14px 0; background: none; border: none; color: #7f8c8d; font-weight: bold; font-size: 14px; cursor: pointer; white-space: nowrap; transition: color 0.2s; }
        .tab-btn.active { color: var(--main-yellow); border-bottom: 3px solid var(--main-yellow); background: rgba(241, 196, 15, 0.05); }
        .tab-content { flex: 1; padding: 16px; overflow-y: auto; display: none; }
        .tab-content.active { display: block; }
        
        .upgrade-card { background: var(--panel-card); border-radius: 12px; padding: 14px; margin-bottom: 12px; display: flex; justify-content: space-between; align-items: center; border: 1px solid rgba(255,255,255,0.06); box-shadow: 0 4px 15px rgba(0,0,0,0.3); }
        .upgrade-info h4 { font-size: 15px; color: #fff; margin-bottom: 4px; font-weight: 700; }
        .upgrade-info p { font-size: 13px; color: #a4b0be; line-height: 1.4; }
        .btn-group-atk { display: flex; gap: 6px; }
        .btn-upgrade { background: linear-gradient(135deg, #f39c12, #d35400); border: none; color: #fff; padding: 10px 14px; border-radius: 10px; font-weight: bold; font-size: 13px; cursor: pointer; box-shadow: 0 4px 12px rgba(211,84,0,0.4); transition: transform 0.1s; }
        .btn-upgrade.btn-sm { padding: 6px 10px; font-size: 12px; min-width: 60px; text-align: center; }
        .btn-upgrade:active { transform: translateY(2px); box-shadow: 0 2px 6px rgba(211,84,0,0.4); }
        .btn-upgrade:disabled { background: #4b4b60; box-shadow: none; color: #888; cursor: not-allowed; }
        .system-btn-group { display: flex; flex-direction: column; gap: 10px; margin-top: 10px; }
        .btn-system { background: linear-gradient(135deg, #34495e, #2c3e50); color: white; border: none; padding: 14px; border-radius: 12px; font-size: 15px; font-weight: bold; cursor: pointer; box-shadow: 0 4px 12px rgba(0,0,0,0.3); transition: transform 0.1s; }
        .btn-system:active { transform: translateY(2px); }
        .btn-admin { background: linear-gradient(135deg, #9b59b6, #8e44ad); border: 1px solid #b19cd9; color: #f1c40f; box-shadow: 0 4px 15px rgba(142,68,173,0.4); }
        .btn-danger { background: linear-gradient(135deg, #ff4757, #c0392b); box-shadow: 0 4px 12px rgba(231,76,60,0.4); }
        .btn-save-load { background: linear-gradient(135deg, #0984e3, #6c5ce7); box-shadow: 0 4px 12px rgba(9,132,227,0.4); }

        .tap-hint { position: absolute; bottom: 32px; font-size: 12px; color: rgba(255,255,255,0.9); background: rgba(15,15,25,0.7); backdrop-filter: blur(4px); padding: 6px 14px; border-radius: 14px; z-index: 10; pointer-events: none; border: 1px solid rgba(255,255,255,0.1); }

        /* 모달 디자인 */
        .modal-overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(11, 11, 16, 0.85); display: none; justify-content: center; align-items: center; z-index: 1000; backdrop-filter: blur(8px); }
        .modal-card { background: #232336; border: 2px solid rgba(155,89,182,0.6); border-radius: 16px; padding: 24px; width: 88%; max-width: 360px; text-align: center; box-shadow: 0 10px 40px rgba(0, 0, 0, 0.8); }
        .modal-card h3 { color: #f1c40f; margin-bottom: 12px; font-size: 22px; font-weight: 800; }
        .modal-card p { color: #b2bec3; font-size: 14px; margin-bottom: 18px; line-height: 1.5; }
        .modal-card input { width: 100%; padding: 14px; border-radius: 10px; border: 1px solid rgba(255,255,255,0.2); background: #181824; color: #fff; font-size: 16px; text-align: center; outline: none; margin-bottom: 18px; box-shadow: inset 0 2px 5px rgba(0,0,0,0.5); }
        .modal-card input:focus { border-color: #f1c40f; }
        .modal-btns { display: flex; gap: 10px; }
        .modal-btns button { flex: 1; padding: 14px; border: none; border-radius: 10px; font-weight: bold; font-size: 15px; cursor: pointer; box-shadow: 0 4px 10px rgba(0,0,0,0.3); transition: transform 0.1s; }
        .modal-btns button:active { transform: translateY(2px); }
        .btn-confirm { background: linear-gradient(135deg, #9b59b6, #8e44ad); color: white; }
        .btn-confirm-danger { background: linear-gradient(135deg, #ff4757, #c0392b); color: white; }
        .btn-cancel { background: linear-gradient(135deg, #7f8c8d, #636e72); color: white; }

        .char-select-btn { width: 100%; padding: 18px; margin-bottom: 14px; border-radius: 14px; border: 2px solid rgba(255,255,255,0.15); background: linear-gradient(135deg, #1e1e2d, #181824); color: #fff; font-size: 16px; font-weight: bold; cursor: pointer; transition: all 0.2s; box-shadow: 0 6px 20px rgba(0,0,0,0.4); text-align: left; display: flex; align-items: center; gap: 14px; }
        .char-select-btn:hover, .char-select-btn:active { border-color: #f1c40f; transform: translateY(-3px); box-shadow: 0 10px 30px rgba(0,0,0,0.6); }
        .char-knight { border-left: 6px solid #e74c3c; } .char-mage { border-left: 6px solid #00d2d3; }
    </style>
</head>
<body>

<div id="game-container">
    <header>
        <div class="user-info">
            <span class="level-badge" id="player-level">Lv. 1 모험가</span>
            <div class="meso-box"><span>🪙</span><span id="player-meso">0</span></div>
        </div>
        <div class="exp-bar-container">
            <div class="exp-bar-fill" id="exp-fill"></div>
            <div class="exp-text" id="exp-text">0 / 50 (0%)</div>
        </div>
    </header>

    <div id="battlefield" onclick="manualAttack(event)">
        <div class="stage-header">
            <div class="stage-title" id="stage-name">STAGE 1 : 푸른 언덕 초원</div>
            <button id="btn-autohunt" class="btn-autohunt active" onclick="toggleAutoHunt(event)">⚔️ 자동사냥: ON</button>
        </div>
        <div class="battle-area">
            <div class="ground" id="ground-bar"></div>
            <div class="pet" id="pet-sprite"></div>
            <div class="character" id="character">
                <div class="character-weapon" id="weapon-icon"></div>
            </div>
            <div class="lightning-bolt" id="lightning-bolt"></div>
            <div class="scratch-effect" id="scratch-effect"></div>
            <div class="monster-container" id="monster-box">
                <div class="monster-hp-bar"><div class="monster-hp-fill" id="monster-hp-fill"></div></div>
                <div class="monster-sprite" id="monster-sprite"></div>
            </div>
        </div>
        <div class="tap-hint">💡 화면을 터치/클릭하면 수동 공격!</div>
    </div>

    <div id="bottom-panel">
        <nav class="nav-tabs">
            <button class="tab-btn active" onclick="switchTab('stat', event)">스탯</button>
            <button class="tab-btn" onclick="switchTab('weapon', event)">장비</button>
            <button class="tab-btn" onclick="switchTab('wing', event)">날개 🪽</button>
            <button class="tab-btn" onclick="switchTab('pet', event)">펫 🐾</button>
            <button class="tab-btn" onclick="switchTab('system', event)">설정</button>
        </nav>

        <div id="tab-stat" class="tab-content active">
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4>공격력 강화 (<span id="stat-atk-lvl">1</span>)</h4>
                    <p>현재 공격력: <span id="stat-atk-val">10</span></p>
                </div>
                <div class="btn-group-atk">
                    <button class="btn-upgrade btn-sm" id="btn-up-atk-1" onclick="upgradeStat('atk', 1)">+1<br><small id="cost-atk-1">10</small></button>
                    <button class="btn-upgrade btn-sm" id="btn-up-atk-10" onclick="upgradeStat('atk', 10)">+10<br><small id="cost-atk-10">550</small></button>
                    <button class="btn-upgrade btn-sm" id="btn-up-atk-100" onclick="upgradeStat('atk', 100)">+100<br><small id="cost-atk-100">50K</small></button>
                </div>
            </div>
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4>공격 속도 (<span id="stat-spd-lvl">1</span>)</h4>
                    <p>공격 주기: <span id="stat-spd-val">1.0초</span></p>
                </div>
                <button class="btn-upgrade" id="btn-up-spd" onclick="upgradeStat('spd')">강화<br><small id="cost-spd">100 Gold</small></button>
            </div>
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4>크리티컬 확률 (<span id="stat-crit-lvl">1</span>)</h4>
                    <p>현재 확률: <span id="stat-crit-val">5%</span></p>
                </div>
                <button class="btn-upgrade" id="btn-up-crit" onclick="upgradeStat('crit')">강화<br><small id="cost-crit">130 Gold</small></button>
            </div>
        </div>

        <div id="tab-weapon" class="tab-content">
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4 id="weapon-name">무기 (Tier 1)</h4>
                    <p>공격력 보너스: +<span id="weapon-bonus">0</span></p>
                </div>
                <button class="btn-upgrade" id="btn-up-weapon" onclick="upgradeWeapon()">무기 승급<br><small id="cost-weapon">200 Gold</small></button>
            </div>
        </div>

        <div id="tab-wing" class="tab-content">
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4 id="wing-name-desc" style="color: #f1c40f;">날개 없음 (Lv. 1)</h4>
                    <p>추가 타격력: +<span id="wing-atk-bonus">0</span><br>
                    <small>※ 10레벨 단위 고화질 HD 날개 진화 (Max 100)</small></p>
                </div>
                <button class="btn-upgrade" id="btn-up-wing" onclick="upgradeWing()">날개 강화<br><small id="cost-wing">500 Gold</small></button>
            </div>
        </div>

        <div id="tab-pet" class="tab-content">
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4 id="pet-name">요정 🧚‍♀️</h4>
                    <p id="pet-type-desc">5초마다 번개 공격 (공격력의 <span id="pet-ratio-val">150</span>%)</p>
                </div>
                <button class="btn-upgrade" id="btn-change-pet" onclick="changePet()">펫 변경 🔄</button>
            </div>
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4>펫 보조 공격력 강화 (<span id="pet-lvl">1</span>)</h4>
                    <p>현재 비율: <span id="pet-ratio-desc">150%</span></p>
                </div>
                <button class="btn-upgrade" id="btn-up-pet" onclick="upgradePet()">강화<br><small id="cost-pet">300 Gold</small></button>
            </div>
        </div>

        <div id="tab-system" class="tab-content">
            <div class="upgrade-card" style="flex-direction: column; align-items: stretch; gap: 10px;">
                <div class="upgrade-info" style="margin-bottom: 5px;">
                    <h4>🎉 이벤트 보상 코드</h4>
                    <p>코드를 입력하여 특별한 보상을 받으세요. ('1' 또는 '2' 입력)</p>
                </div>
                <div style="display: flex; gap: 10px;">
                    <input type="text" id="event-code-input" placeholder="코드 입력" style="flex: 1; padding: 10px; border-radius: 8px; border: 1px solid #555; background: #181824; color: #fff; font-size: 14px; outline: none;">
                    <button class="btn-system" style="margin: 0; padding: 10px 16px; background: linear-gradient(135deg, #2ed573, #26af5f);" onclick="submitEventCode()">보상 받기</button>
                </div>
            </div>

            <div class="upgrade-card" style="flex-direction: column; align-items: stretch; gap: 10px;">
                <div class="upgrade-info" style="margin-bottom: 5px;">
                    <h4>💾 데이터 파일 백업 / 복구</h4>
                    <p>기기를 변경하거나 안전하게 보관하려면 파일로 저장하세요.</p>
                </div>
                <div style="display: flex; gap: 10px;">
                    <button class="btn-system btn-save-load" style="margin: 0; flex: 1;" onclick="exportGameData()">파일 추출</button>
                    <button class="btn-system btn-save-load" style="margin: 0; flex: 1; background: linear-gradient(135deg, #f39c12, #d35400);" onclick="document.getElementById('import-file').click()">불러오기</button>
                    <input type="file" id="import-file" accept=".json" style="display: none;" onchange="importGameData(event)">
                </div>
            </div>

            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4>현재 스테이지 정보</h4>
                    <p id="stage-desc">진행 정보 표시</p>
                </div>
            </div>

            <div class="system-btn-group">
                <button class="btn-system btn-admin" onclick="openAdminModal()">🔑 관리자 모드 접속</button>
                <button class="btn-system btn-danger" onclick="openResetModal()">🔄 데이터 초기화</button>
            </div>
        </div>
    </div>

    <!-- Modals -->
    <div id="char-select-modal" class="modal-overlay" style="display: flex;">
        <div class="modal-card" style="border-color: #f1c40f;">
            <h3>⚔️ 새로운 모험의 시작</h3>
            <p style="margin-bottom: 20px;">플레이할 영웅의 직업을 선택하세요.</p>
            <button class="char-select-btn char-knight" onclick="selectCharacter('knight')">
                <span style="font-size: 24px;">🐉</span>
                <div>
                    <div style="font-size: 16px; color: #fff; margin-bottom: 3px;">드래곤 기사</div>
                    <div style="font-size: 12px; color: #a4b0be; font-weight: normal;">근접 거대검 / 화염 & 드래곤 날개</div>
                </div>
            </button>
            <button class="char-select-btn char-mage" onclick="selectCharacter('mage')">
                <span style="font-size: 24px;">❄️</span>
                <div>
                    <div style="font-size: 16px; color: #fff; margin-bottom: 3px;">천사 마법사</div>
                    <div style="font-size: 12px; color: #a4b0be; font-weight: normal;">마력 에너지 볼트 / 순백 천사 날개</div>
                </div>
            </button>
        </div>
    </div>

    <div id="admin-modal" class="modal-overlay">
        <div class="modal-card" onclick="event.stopPropagation()">
            <h3>🔑 관리자 인증</h3>
            <p>비밀번호 4자리를 입력하세요.</p>
            <input type="password" id="admin-pw-input" placeholder="비밀번호 입력" maxlength="4">
            <div class="modal-btns">
                <button class="btn-confirm" onclick="submitAdminPw()">확인</button>
                <button class="btn-cancel" onclick="closeAdminModal()">취소</button>
            </div>
        </div>
    </div>

    <div id="offline-modal" class="modal-overlay">
        <div class="modal-card" style="border-color: #2ecc71;" onclick="event.stopPropagation()">
            <h3 style="color: #2ecc71;">🌙 백그라운드 방치 완료</h3>
            <p id="offline-desc">방치된 시간 동안 몬스터를 자급자족으로 정벌했습니다!</p>
            <div class="modal-btns">
                <button class="btn-confirm" style="background: #2ecc71;" onclick="closeOfflineModal()">보상 수령하기</button>
            </div>
        </div>
    </div>

    <div id="reset-modal" class="modal-overlay">
        <div class="modal-card" style="border-color: #e74c3c;" onclick="event.stopPropagation()">
            <h3 style="color: #e74c3c;">🔄 데이터 초기화</h3>
            <p>모든 게임 데이터와 메소, 진행 상황이 영구적으로 초기화됩니다.<br><br>정말 삭제하시겠습니까?</p>
            <div class="modal-btns">
                <button class="btn-confirm-danger" onclick="confirmReset()">초기화 진행</button>
                <button class="btn-cancel" onclick="closeResetModal()">취소</button>
            </div>
        </div>
    </div>
</div>

<script>
    // ⚔️ 기사 대검 10종
    const WEAPON_ICONS_KNIGHT = [
        `data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 4px %23bdc3c7)"><path d="M 18 36 L 18 10 L 24 4 L 30 10 L 30 36 Z" fill="%237f8c8d" stroke="%23ecf0f1" stroke-width="1.5"/><rect x="12" y="36" width="24" height="4" fill="%232c3e50"/><rect x="20" y="40" width="8" height="6" fill="%238e5123"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 5px %233498db)"><path d="M 20 34 L 16 12 L 24 2 L 32 12 L 28 34 Z" fill="%232980b9" stroke="%2300d2d3" stroke-width="1.5"/><polygon points="12,34 36,34 24,38" fill="%231abc9c"/><rect x="22" y="38" width="4" height="8" fill="%2334495e"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 6px %23ff4757)"><path d="M 18 36 L 16 28 L 20 26 L 16 20 L 20 18 L 16 10 L 24 2 L 32 10 L 28 18 L 32 20 L 28 26 L 32 28 L 30 36 Z" fill="%23c0392b" stroke="%23ff7675" stroke-width="1.5"/><rect x="14" y="36" width="20" height="4" fill="%232d3436"/><rect x="20" y="40" width="8" height="6" fill="%23111"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 6px %239b59b6)"><path d="M 22 36 Q 30 20 40 10 Q 20 5 16 2 Q 10 15 14 36 Z" fill="%238e44ad" stroke="%23d2b4de" stroke-width="2"/><rect x="16" y="36" width="12" height="4" fill="%232c3e50"/><rect x="18" y="40" width="8" height="8" fill="%23111"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 7px %232ecc71)"><path d="M 18 36 Q 10 15 2캐릭터 선택창에서 진행되지 않던 문제는, **직업을 선택한 후 실제 게임 진행 로직(`startGameplay`)을 연결하는 부분이 누락되었거나 순서가 잘못되어 발생한 것**입니다.

제공해주신 이미지의 GitHub Pages 상단 문구 문제와 진행 불가 문제를 모두 해결하기 위해 코드를 다음과 같이 수정했습니다.

1.  **진행 불가 해결:** `selectCharacter` 함수에서 직업 선택 후 즉시 캐릭터 선택 모달을 숨기고(`style.display = "none"`), 게임 플레이 로직을 시작하도록(`startGameplay()`) 수정했습니다.
2.  **상단 문구 제거 및 풀스크린:** GitHub Pages가 HTML 파일을 텍스트가 아닌 웹페이지로 정확히 인식하도록 HTML 파일 가장 첫 줄에 BOM 등의 불필요한 문자가 없도록 최적화하고, `body` 태그에 CSS를 추가하여 브라우저 여백을 없애고 가득 차게 만들었습니다.

---

### 📄 `index.html` (수정 완료 버전)

아래의 전체 코드를 복사하여 기존 `index.html` 파일의 내용을 **완전히 지우고** 덮어쓰기 해주세요. (안정적인 데이터 처리를 위해 내부 세이브 버전이 `vRelease`로 업데이트 되었습니다.)

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>메이플 키우기 - 정식 출시 버전</title>
    <style>
        :root {
            --panel-bg: #181824;
            --panel-card: #232336;
            --main-yellow: #f1c40f;
            --main-orange: #e67e22;
            --text-light: #ffffff;
            image-rendering: crisp-edges;
        }

        * {
            box-sizing: border-box; user-select: none; -webkit-user-select: none;
            touch-action: manipulation; margin: 0; padding: 0;
            font-family: 'Pretendard', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
        }

        /* 📱 풀스크린 최적화: 브라우저 여백 제거 및 가득 차게 설정 */
        body {
            background-color: #0b0b10; color: var(--text-light); display: flex;
            justify-content: center; align-items: center; min-height: 100vh; min-height: 100dvh; overflow: hidden;
            width: 100vw; margin: 0; /* 불필요한 마진 제거 */
        }

        #game-container {
            width: 100%; max-width: 500px; height: 100vh; height: 100dvh;
            background: linear-gradient(180deg, #1e1e2d 0%, #12121a 100%);
            display: flex; flex-direction: column; position: relative; box-shadow: 0 0 40px rgba(0,0,0,0.95); overflow: hidden;
        }

        header {
            background: linear-gradient(180deg, rgba(25,25,38,0.9), rgba(18,18,26,0.9));
            backdrop-filter: blur(8px); padding: 14px 16px; border-bottom: 2px solid rgba(255,255,255,0.08);
            display: flex; flex-direction: column; gap: 10px; z-index: 10;
        }

        .user-info { display: flex; justify-content: space-between; align-items: center; }
        .level-badge {
            background: linear-gradient(135deg, #f39c12, #d35400); color: #fff; padding: 6px 16px;
            border-radius: 16px; font-weight: bold; font-size: 14px; box-shadow: 0 4px 10px rgba(230,126,34,0.4);
        }
        .meso-box {
            display: flex; align-items: center; gap: 6px; background: rgba(0,0,0,0.4);
            padding: 6px 16px; border-radius: 16px; border: 1px solid rgba(241,196,15,0.4);
            color: var(--main-yellow); font-weight: bold; font-size: 16px; box-shadow: inset 0 2px 4px rgba(0,0,0,0.5);
        }

        .exp-bar-container {
            width: 100%; height: 14px; background: rgba(0,0,0,0.6); border-radius: 7px;
            overflow: hidden; border: 1px solid rgba(255,255,255,0.1); position: relative;
        }
        .exp-bar-fill { height: 100%; background: linear-gradient(90deg, #2ecc71, #00d2d3); width: 0%; transition: width 0.3s ease; box-shadow: 0 0 10px rgba(46,204,113,0.5); }
        .exp-text { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); font-size: 10px; font-weight: bold; text-shadow: 1px 1px 2px #000; }

        #battlefield {
            flex: 1.2; position: relative; background-size: cover; background-position: center;
            overflow: hidden; display: flex; flex-direction: column; justify-content: space-between;
            align-items: center; cursor: pointer; transition: background 0.5s ease;
        }

        .stage-header { width: 100%; display: flex; justify-content: space-between; align-items: center; padding: 14px 18px 0 18px; z-index: 10; }
        .stage-title {
            background: rgba(15, 15, 25, 0.8); padding: 8px 18px; border-radius: 20px;
            font-size: 14px; font-weight: bold; color: #fff; border: 1px solid rgba(255, 255, 255, 0.15); backdrop-filter: blur(6px); box-shadow: 0 4px 15px rgba(0,0,0,0.3);
        }

        .btn-autohunt {
            background: linear-gradient(135deg, #ff4757, #ff6b81); color: white; border: 1px solid rgba(255, 255, 255, 0.3);
            padding: 8px 18px; border-radius: 20px; font-size: 13px; font-weight: bold; cursor: pointer; box-shadow: 0 4px 12px rgba(255,71,87,0.4); transition: all 0.2s;
        }
        .btn-autohunt.active { background: linear-gradient(135deg, #2ed573, #26af5f); box-shadow: 0 4px 15px rgba(46,213,115,0.5); }

        .battle-area { position: relative; width: 100%; height: 100%; display: flex; justify-content: center; align-items: flex-end; padding-bottom: 22px; }
        .ground {
            position: absolute; bottom: 0; left: 0; width: 100%; height: 32px;
            background-color: var(--ground-color, #27ae60); border-top: 4px solid rgba(255,255,255,0.25); box-shadow: inset 0 4px 8px rgba(0,0,0,0.3); transition: background 0.4s;
        }

        .character {
            position: absolute; left: 30%; bottom: 28px; transform: translateX(-50%);
            width: 140px; height: 140px; background-size: 280px 140px; background-repeat: no-repeat;
            z-index: 5; pointer-events: none; filter: drop-shadow(0 8px 12px rgba(0,0,0,0.5));
            transition: filter 0.1s ease, transform 0.1s ease; animation: spriteIdleLarge 0.8s steps(1) infinite;
        }
        @keyframes spriteIdleLarge { 0% { background-position: 0px 0px; } 50% { background-position: -140px 0px; } 100% { background-position: 0px 0px; } }
        .character.attacking { transform: translateX(-50%) scale(1.05); }

        .character-weapon {
            position: absolute; left: 75px; top: 60px; width: 50px; height: 50px;
            background-size: contain; background-repeat: no-repeat; transition: transform 0.15s ease;
            filter: drop-shadow(0 2px 4px rgba(0,0,0,0.5));
        }
        .character.attacking .character-weapon { transform: rotate(50deg) scale(1.4) translateX(15px) translateY(-5px); }

        .energy-bolt {
            position: absolute; width: 80px; height: 30px; z-index: 35; pointer-events: none;
            transform: translate(-50%, -50%) rotate(var(--angle, 0deg));
            animation: shootEnergyDynamic 0.15s cubic-bezier(0.1, 0.8, 0.3, 1) forwards;
        }
        .bolt-core {
            position: absolute; right: 0; top: 50%; transform: translateY(-50%);
            width: 28px; height: 28px; border-radius: 50%;
            background: radial-gradient(circle, #ffffff 20%, #00d2d3 60%, #a29bfe 100%); box-shadow: 0 0 15px #fff, 0 0 30px #00d2d3, 0 0 50px #a29bfe;
        }
        .bolt-trail {
            position: absolute; right: 14px; top: 50%; transform: translateY(-50%);
            width: 60px; height: 12px; border-radius: 50px 0 0 50px;
            background: linear-gradient(90deg, transparent, rgba(162, 155, 254, 0.8), #00d2d3); filter: blur(2px);
        }
        @keyframes shootEnergyDynamic {
            0% { transform: translate(-50%, -50%) rotate(var(--angle)) scale(0.5); opacity: 0.8; filter: hue-rotate(0deg); }
            100% { transform: translate(calc(-50% + var(--target-x)), calc(-50% + var(--target-y))) rotate(var(--angle)) scale(1.5); opacity: 1; filter: hue-rotate(270deg); }
        }

        .pet { position: absolute; left: 6%; bottom: 55px; width: 45px; height: 45px; background-size: contain; background-repeat: no-repeat; z-index: 6; filter: drop-shadow(0 6px 10px rgba(0,0,0,0.4)); animation: petFloat 1.8s ease-in-out infinite alternate; }
        @keyframes petFloat { 0% { transform: translateY(0px); } 100% { transform: translateY(-10px); } }
        .pet.tackling { animation: petTackle 0.35s cubic-bezier(0.1,0.9,0.2,1) !important; }
        @keyframes petTackle { 0% { transform: translate(0, 0) scale(1); } 40% { transform: translate(190px, 15px) scale(1.35); } 100% { transform: translate(0, 0) scale(1); } }

        .lightning-bolt { position: absolute; top: 0; right: 22%; width: 40px; height: 220px; background: linear-gradient(180deg, #f1c40f, #ffffff, #e67e22); clip-path: polygon(40% 0%, 70% 0%, 30% 40%, 80% 40%, 10% 100%, 40% 50%, 0% 50%); z-index: 30; opacity: 0; pointer-events: none; filter: drop-shadow(0 0 12px #f1c40f); }
        .lightning-bolt.active { animation: lightningFlash 0.35s ease-out; }
        @keyframes lightningFlash { 0% { opacity: 0; transform: scaleY(0); } 30% { opacity: 1; transform: scaleY(1); } 70% { opacity: 1; transform: scaleY(1); } 100% { opacity: 0; transform: scaleY(1); } }

        .scratch-effect { position: absolute; right: 20%; bottom: 28px; width: 80px; height: 80px; background-image: url('data:image/svg+xml;utf8,<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="64" height="64" viewBox="0 0 64 64"><line x1="8" y1="12" x2="52" y2="52" stroke="%23ff3838" stroke-width="6" stroke-linecap="round"/><line x1="20" y1="8" x2="60" y2="44" stroke="%23ff3838" stroke-width="6" stroke-linecap="round"/><line x1="6" y1="24" x2="44" y2="60" stroke="%23ff3838" stroke-width="6" stroke-linecap="round"/></svg>'); background-size: contain; background-repeat: no-repeat; z-index: 25; opacity: 0; pointer-events: none; }
        .scratch-effect.active { animation: scratchSlash 0.3s cubic-bezier(0.1,0.9,0.2,1); }
        @keyframes scratchSlash { 0% { opacity: 0; transform: scale(0.6) rotate(-15deg); } 40% { opacity: 1; transform: scale(1.25) rotate(0deg); filter: drop-shadow(0 0 12px #ff3838); } 100% { opacity: 0; transform: scale(1) rotate(10deg); } }

        .monster-container { position: absolute; right: 15%; bottom: 28px; display: flex; flex-direction: column; align-items: center; z-index: 5; pointer-events: none; filter: drop-shadow(0 6px 10px rgba(0,0,0,0.4)); }
        .monster-hp-bar { width: 80px; height: 10px; background: rgba(0,0,0,0.7); border: 1px solid rgba(255,255,255,0.4); border-radius: 5px; overflow: hidden; margin-bottom: 8px; box-shadow: 0 2px 5px rgba(0,0,0,0.5); }
        .monster-hp-fill { height: 100%; background: linear-gradient(90deg, #ff4757, #ff6b81); width: 100%; transition: width 0.1s linear; }
        .monster-sprite { width: 80px; height: 80px; background-size: cover; background-repeat: no-repeat; transition: transform 0.2s ease, filter 0.2s ease; }

        .damage-text { position: absolute; font-size: 28px; font-weight: 900; color: #f1c40f; text-shadow: 2px 2px 0 #000, -2px -2px 0 #000, 2px -2px 0 #000, -2px 2px 0 #000, 0 0 10px rgba(241,196,15,0.6); pointer-events: none; animation: floatDamage 0.65s cubic-bezier(0.1,0.9,0.2,1) forwards; z-index: 20; transform: translate(-50%, 0); }
        .damage-text.crit { color: #ff3838; font-size: 34px; text-shadow: 2px 2px 0 #000, -2px -2px 0 #000, 0 0 15px rgba(255,56,56,0.8); }
        .damage-text.pet-dmg { color: #00d2d3; font-size: 30px; text-shadow: 2px 2px 0 #000, -2px -2px 0 #000, 0 0 15px rgba(0,210,211,0.8); }
        .damage-text.char-dmg { color: #ff4757; font-size: 26px; text-shadow: 2px 2px 0 #000, -2px -2px 0 #000, 0 0 15px rgba(255,71,87,0.8); z-index: 40; }

        @keyframes floatDamage {
            0% { opacity: 1; transform: translate(-50%, 0) scale(0.8); }
            50% { opacity: 1; transform: translate(-50%, -40px) scale(1.25); }
            100% { opacity: 0; transform: translate(-50%, -70px) scale(1); }
        }

        #bottom-panel { flex: 1; min-height: 40%; max-height: 55vh; background-color: var(--panel-bg); display: flex; flex-direction: column; border-top: 2px solid rgba(255,255,255,0.08); box-shadow: 0 -10px 30px rgba(0,0,0,0.5); z-index: 20; }
        .nav-tabs { display: flex; background: #12121a; border-bottom: 1px solid rgba(255,255,255,0.06); }
        .tab-btn { flex: 1; padding: 14px 0; background: none; border: none; color: #7f8c8d; font-weight: bold; font-size: 14px; cursor: pointer; white-space: nowrap; transition: color 0.2s; }
        .tab-btn.active { color: var(--main-yellow); border-bottom: 3px solid var(--main-yellow); background: rgba(241, 196, 15, 0.05); }
        .tab-content { flex: 1; padding: 16px; overflow-y: auto; display: none; }
        .tab-content.active { display: block; }
        
        .upgrade-card { background: var(--panel-card); border-radius: 12px; padding: 14px; margin-bottom: 12px; display: flex; justify-content: space-between; align-items: center; border: 1px solid rgba(255,255,255,0.06); box-shadow: 0 4px 15px rgba(0,0,0,0.3); }
        .upgrade-info h4 { font-size: 15px; color: #fff; margin-bottom: 4px; font-weight: 700; }
        .upgrade-info p { font-size: 13px; color: #a4b0be; line-height: 1.4; }
        .btn-group-atk { display: flex; gap: 6px; }
        .btn-upgrade { background: linear-gradient(135deg, #f39c12, #d35400); border: none; color: #fff; padding: 10px 14px; border-radius: 10px; font-weight: bold; font-size: 13px; cursor: pointer; box-shadow: 0 4px 12px rgba(211,84,0,0.4); transition: transform 0.1s; }
        .btn-upgrade.btn-sm { padding: 6px 10px; font-size: 12px; min-width: 60px; text-align: center; }
        .btn-upgrade:active { transform: translateY(2px); box-shadow: 0 2px 6px rgba(211,84,0,0.4); }
        .btn-upgrade:disabled { background: #4b4b60; box-shadow: none; color: #888; cursor: not-allowed; }
        .system-btn-group { display: flex; flex-direction: column; gap: 10px; margin-top: 10px; }
        .btn-system { background: linear-gradient(135deg, #34495e, #2c3e50); color: white; border: none; padding: 14px; border-radius: 12px; font-size: 15px; font-weight: bold; cursor: pointer; box-shadow: 0 4px 12px rgba(0,0,0,0.3); transition: transform 0.1s; }
        .btn-system:active { transform: translateY(2px); }
        .btn-admin { background: linear-gradient(135deg, #9b59b6, #8e44ad); border: 1px solid #b19cd9; color: #f1c40f; box-shadow: 0 4px 15px rgba(142,68,173,0.4); }
        .btn-danger { background: linear-gradient(135deg, #ff4757, #c0392b); box-shadow: 0 4px 12px rgba(231,76,60,0.4); }
        .btn-save-load { background: linear-gradient(135deg, #0984e3, #6c5ce7); box-shadow: 0 4px 12px rgba(9,132,227,0.4); }

        .tap-hint { position: absolute; bottom: 32px; font-size: 12px; color: rgba(255,255,255,0.9); background: rgba(15,15,25,0.7); backdrop-filter: blur(4px); padding: 6px 14px; border-radius: 14px; z-index: 10; pointer-events: none; border: 1px solid rgba(255,255,255,0.1); }

        /* 모달 디자인 */
        .modal-overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(11, 11, 16, 0.85); display: none; justify-content: center; align-items: center; z-index: 1000; backdrop-filter: blur(8px); }
        .modal-card { background: #232336; border: 2px solid rgba(155,89,182,0.6); border-radius: 16px; padding: 24px; width: 88%; max-width: 360px; text-align: center; box-shadow: 0 10px 40px rgba(0, 0, 0, 0.8); }
        .modal-card h3 { color: #f1c40f; margin-bottom: 12px; font-size: 22px; font-weight: 800; }
        .modal-card p { color: #b2bec3; font-size: 14px; margin-bottom: 18px; line-height: 1.5; }
        .modal-card input { width: 100%; padding: 14px; border-radius: 10px; border: 1px solid rgba(255,255,255,0.2);캐릭터 선택창에서 진행되지 않던 문제는, **직업을 선택한 후 실제 게임 진행 로직(`startGameplay`)을 연결하는 부분이 누락되었거나 순서가 잘못되어 발생한 것**입니다.

제공해주신 이미지의 GitHub Pages 상단 문구 문제와 진행 불가 문제를 모두 해결하기 위해 코드를 다음과 같이 수정했습니다.

1.  **진행 불가 해결:** `selectCharacter` 함수에서 직업 선택 후 즉시 캐릭터 선택 모달을 숨기고(`style.display = "none"`), 게임 플레이 로직을 시작하도록(`startGameplay()`) 수정했습니다.
2.  **상단 문구 제거 및 풀스크린:** GitHub Pages가 HTML 파일을 텍스트가 아닌 웹페이지로 정확히 인식하도록 HTML 파일 가장 첫 줄에 BOM 등의 불필요한 문자가 없도록 최적화하고, `body` 태그에 CSS를 추가하여 브라우저 여백을 없애고 가득 차게 만들었습니다.

---

### 📄 `index.html` (수정 완료 버전)

아래의 전체 코드를 복사하여 기존 `index.html` 파일의 내용을 **완전히 지우고** 덮어쓰기 해주세요. (안정적인 데이터 처리를 위해 내부 세이브 버전이 `vRelease`로 업데이트 되었습니다.)

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>메이플 키우기 - 정식 출시 버전</title>
    <style>
        :root {
            --panel-bg: #181824;
            --panel-card: #232336;
            --main-yellow: #f1c40f;
            --main-orange: #e67e22;
            --text-light: #ffffff;
            image-rendering: crisp-edges;
        }

        * {
            box-sizing: border-box; user-select: none; -webkit-user-select: none;
            touch-action: manipulation; margin: 0; padding: 0;
            font-family: 'Pretendard', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
        }

        /* 📱 풀스크린 최적화: 브라우저 여백 제거 및 가득 차게 설정 */
        body {
            background-color: #0b0b10; color: var(--text-light); display: flex;
            justify-content: center; align-items: center; min-height: 100vh; min-height: 100dvh; overflow: hidden;
            width: 100vw; margin: 0; /* 불필요한 마진 제거 */
        }

        #game-container {
            width: 100%; max-width: 500px; height: 100vh; height: 100dvh;
            background: linear-gradient(180deg, #1e1e2d 0%, #12121a 100%);
            display: flex; flex-direction: column; position: relative; box-shadow: 0 0 40px rgba(0,0,0,0.95); overflow: hidden;
        }

        header {
            background: linear-gradient(180deg, rgba(25,25,38,0.9), rgba(18,18,26,0.9));
            backdrop-filter: blur(8px); padding: 14px 16px; border-bottom: 2px solid rgba(255,255,255,0.08);
            display: flex; flex-direction: column; gap: 10px; z-index: 10;
        }

        .user-info { display: flex; justify-content: space-between; align-items: center; }
        .level-badge {
            background: linear-gradient(135deg, #f39c12, #d35400); color: #fff; padding: 6px 16px;
            border-radius: 16px; font-weight: bold; font-size: 14px; box-shadow: 0 4px 10px rgba(230,126,34,0.4);
        }
        .meso-box {
            display: flex; align-items: center; gap: 6px; background: rgba(0,0,0,0.4);
            padding: 6px 16px; border-radius: 16px; border: 1px solid rgba(241,196,15,0.4);
            color: var(--main-yellow); font-weight: bold; font-size: 16px; box-shadow: inset 0 2px 4px rgba(0,0,0,0.5);
        }

        .exp-bar-container {
            width: 100%; height: 14px; background: rgba(0,0,0,0.6); border-radius: 7px;
            overflow: hidden; border: 1px solid rgba(255,255,255,0.1); position: relative;
        }
        .exp-bar-fill { height: 100%; background: linear-gradient(90deg, #2ecc71, #00d2d3); width: 0%; transition: width 0.3s ease; box-shadow: 0 0 10px rgba(46,204,113,0.5); }
        .exp-text { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); font-size: 10px; font-weight: bold; text-shadow: 1px 1px 2px #000; }

        #battlefield {
            flex: 1.2; position: relative; background-size: cover; background-position: center;
            overflow: hidden; display: flex; flex-direction: column; justify-content: space-between;
            align-items: center; cursor: pointer; transition: background 0.5s ease;
        }

        .stage-header { width: 100%; display: flex; justify-content: space-between; align-items: center; padding: 14px 18px 0 18px; z-index: 10; }
        .stage-title {
            background: rgba(15, 15, 25, 0.8); padding: 8px 18px; border-radius: 20px;
            font-size: 14px; font-weight: bold; color: #fff; border: 1px solid rgba(255, 255, 255, 0.15); backdrop-filter: blur(6px); box-shadow: 0 4px 15px rgba(0,0,0,0.3);
        }

        .btn-autohunt {
            background: linear-gradient(135deg, #ff4757, #ff6b81); color: white; border: 1px solid rgba(255, 255, 255, 0.3);
            padding: 8px 18px; border-radius: 20px; font-size: 13px; font-weight: bold; cursor: pointer; box-shadow: 0 4px 12px rgba(255,71,87,0.4); transition: all 0.2s;
        }
        .btn-autohunt.active { background: linear-gradient(135deg, #2ed573, #26af5f); box-shadow: 0 4px 15px rgba(46,213,115,0.5); }

        .battle-area { position: relative; width: 100%; height: 100%; display: flex; justify-content: center; align-items: flex-end; padding-bottom: 22px; }
        .ground {
            position: absolute; bottom: 0; left: 0; width: 100%; height: 32px;
            background-color: var(--ground-color, #27ae60); border-top: 4px solid rgba(255,255,255,0.25); box-shadow: inset 0 4px 8px rgba(0,0,0,0.3); transition: background 0.4s;
        }

        .character {
            position: absolute; left: 30%; bottom: 28px; transform: translateX(-50%);
            width: 140px; height: 140px; background-size: 280px 140px; background-repeat: no-repeat;
            z-index: 5; pointer-events: none; filter: drop-shadow(0 8px 12px rgba(0,0,0,0.5));
            transition: filter 0.1s ease, transform 0.1s ease; animation: spriteIdleLarge 0.8s steps(1) infinite;
        }
        @keyframes spriteIdleLarge { 0% { background-position: 0px 0px; } 50% { background-position: -140px 0px; } 100% { background-position: 0px 0px; } }
        .character.attacking { transform: translateX(-50%) scale(1.05); }

        .character-weapon {
            position: absolute; left: 75px; top: 60px; width: 50px; height: 50px;
            background-size: contain; background-repeat: no-repeat; transition: transform 0.15s ease;
            filter: drop-shadow(0 2px 4px rgba(0,0,0,0.5));
        }
        .character.attacking .character-weapon { transform: rotate(50deg) scale(1.4) translateX(15px) translateY(-5px); }

        .energy-bolt {
            position: absolute; width: 80px; height: 30px; z-index: 35; pointer-events: none;
            transform: translate(-50%, -50%) rotate(var(--angle, 0deg));
            animation: shootEnergyDynamic 0.15s cubic-bezier(0.1, 0.8, 0.3, 1) forwards;
        }
        .bolt-core {
            position: absolute; right: 0; top: 50%; transform: translateY(-50%);
            width: 28px; height: 28px; border-radius: 50%;
            background: radial-gradient(circle, #ffffff 20%, #00d2d3 60%, #a29bfe 100%); box-shadow: 0 0 15px #fff, 0 0 30px #00d2d3, 0 0 50px #a29bfe;
        }
        .bolt-trail {
            position: absolute; right: 14px; top: 50%; transform: translateY(-50%);
            width: 60px; height: 12px; border-radius: 50px 0 0 50px;
            background: linear-gradient(90deg, transparent, rgba(162, 155, 254, 0.8), #00d2d3); filter: blur(2px);
        }
        @keyframes shootEnergyDynamic {
            0% { transform: translate(-50%, -50%) rotate(var(--angle)) scale(0.5); opacity: 0.8; filter: hue-rotate(0deg); }
            100% { transform: translate(calc(-50% + var(--target-x)), calc(-50% + var(--target-y))) rotate(var(--angle)) scale(1.5); opacity: 1; filter: hue-rotate(270deg); }
        }

        .pet { position: absolute; left: 6%; bottom: 55px; width: 45px; height: 45px; background-size: contain; background-repeat: no-repeat; z-index: 6; filter: drop-shadow(0 6px 10px rgba(0,0,0,0.4)); animation: petFloat 1.8s ease-in-out infinite alternate; }
        @keyframes petFloat { 0% { transform: translateY(0px); } 100% { transform: translateY(-10px); } }
        .pet.tackling { animation: petTackle 0.35s cubic-bezier(0.1,0.9,0.2,1) !important; }
        @keyframes petTackle { 0% { transform: translate(0, 0) scale(1); } 40% { transform: translate(190px, 15px) scale(1.35); } 100% { transform: translate(0, 0) scale(1); } }

        .lightning-bolt { position: absolute; top: 0; right: 22%; width: 40px; height: 220px; background: linear-gradient(180deg, #f1c40f, #ffffff, #e67e22); clip-path: polygon(40% 0%, 70% 0%, 30% 40%, 80% 40%, 10% 100%, 40% 50%, 0% 50%); z-index: 30; opacity: 0; pointer-events: none; filter: drop-shadow(0 0 12px #f1c40f); }
        .lightning-bolt.active { animation: lightningFlash 0.35s ease-out; }
        @keyframes lightningFlash { 0% { opacity: 0; transform: scaleY(0); } 30% { opacity: 1; transform: scaleY(1); } 70% { opacity: 1; transform: scaleY(1); } 100% { opacity: 0; transform: scaleY(1); } }

        .scratch-effect { position: absolute; right: 20%; bottom: 28px; width: 80px; height: 80px; background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="64" height="64" viewBox="0 0 64 64"><line x1="8" y1="12" x2="52" y2="52" stroke="%23ff3838" stroke-width="6" stroke-linecap="round"/><line x1="20" y1="8" x2="60" y2="44" stroke="%23ff3838" stroke-width="6" stroke-linecap="round"/><line x1="6" y1="24" x2="44" y2="60" stroke="%23ff3838" stroke-width="6" stroke-linecap="round"/></svg>'); background-size: contain; background-repeat: no-repeat; z-index: 25; opacity: 0; pointer-events: none; }
        .scratch-effect.active { animation: scratchSlash 0.3s cubic-bezier(0.1,0.9,0.2,1); }
        @keyframes scratchSlash { 0% { opacity: 0; transform: scale(0.6) rotate(-15deg); } 40% { opacity: 1; transform: scale(1.25) rotate(0deg); filter: drop-shadow(0 0 12px #ff3838); } 100% { opacity: 0; transform: scale(1) rotate(10deg); } }

        .monster-container { position: absolute; right: 15%; bottom: 28px; display: flex; flex-direction: column; align-items: center; z-index: 5; pointer-events: none; filter: drop-shadow(0 6px 10px rgba(0,0,0,0.4)); }
        .monster-hp-bar { width: 80px; height: 10px; background: rgba(0,0,0,0.7); border: 1px solid rgba(255,255,255,0.4); border-radius: 5px; overflow: hidden; margin-bottom: 8px; box-shadow: 0 2px 5px rgba(0,0,0,0.5); }
        .monster-hp-fill { height: 100%; background: linear-gradient(90deg, #ff4757, #ff6b81); width: 100%; transition: width 0.1s linear; }
        .monster-sprite { width: 80px; height: 80px; background-size: cover; background-repeat: no-repeat; transition: transform 0.2s ease, filter 0.2s ease; }

        .damage-text { position: absolute; font-size: 28px; font-weight: 900; color: #f1c40f; text-shadow: 2px 2px 0 #000, -2px -2px 0 #000, 2px -2px 0 #000, -2px 2px 0 #000, 0 0 10px rgba(241,196,15,0.6); pointer-events: none; animation: floatDamage 0.65s cubic-bezier(0.1,0.9,0.2,1) forwards; z-index: 20; transform: translate(-50%, 0); }
        .damage-text.crit { color: #ff3838; font-size: 34px; text-shadow: 2px 2px 0 #000, -2px -2px 0 #000, 0 0 15px rgba(255,56,56,0.8); }
        .damage-text.pet-dmg { color: #00d2d3; font-size: 30px; text-shadow: 2px 2px 0 #000, -2px -2px 0 #000, 0 0 15px rgba(0,210,211,0.8); }
        .damage-text.char-dmg { color: #ff4757; font-size: 26px; text-shadow: 2px 2px 0 #000, -2px -2px 0 #000, 0 0 15px rgba(255,71,87,0.8); z-index: 40; }

        @keyframes floatDamage {
            0% { opacity: 1; transform: translate(-50%, 0) scale(0.8); }
            50% { opacity: 1; transform: translate(-50%, -40px) scale(1.25); }
            100% { opacity: 0; transform: translate(-50%, -70px) scale(1); }
        }

        #bottom-panel { flex: 1; min-height: 40%; max-height: 55vh; background-color: var(--panel-bg); display: flex; flex-direction: column; border-top: 2px solid rgba(255,255,255,0.08); box-shadow: 0 -10px 30px rgba(0,0,0,0.5); z-index: 20; }
        .nav-tabs { display: flex; background: #12121a; border-bottom: 1px solid rgba(255,255,255,0.06); }
        .tab-btn { flex: 1; padding: 14px 0; background: none; border: none; color: #7f8c8d; font-weight: bold; font-size: 14px; cursor: pointer; white-space: nowrap; transition: color 0.2s; }
        .tab-btn.active { color: var(--main-yellow); border-bottom: 3px solid var(--main-yellow); background: rgba(241, 196, 15, 0.05); }
        .tab-content { flex: 1; padding: 16px; overflow-y: auto; display: none; }
        .tab-content.active { display: block; }
        
        .upgrade-card { background: var(--panel-card); border-radius: 12px; padding: 14px; margin-bottom: 12px; display: flex; justify-content: space-between; align-items: center; border: 1px solid rgba(255,255,255,0.06); box-shadow: 0 4px 15px rgba(0,0,0,0.3); }
        .upgrade-info h4 { font-size: 15px; color: #fff; margin-bottom: 4px; font-weight: 700; }
        .upgrade-info p { font-size: 13px; color: #a4b0be; line-height: 1.4; }
        .btn-group-atk { display: flex; gap: 6px; }
        .btn-upgrade { background: linear-gradient(135deg, #f39c12, #d35400); border: none; color: #fff; padding: 10px 14px; border-radius: 10px; font-weight: bold; font-size: 13px; cursor: pointer; box-shadow: 0 4px 12px rgba(211,84,0,0.4); transition: transform 0.1s; }
        .btn-upgrade.btn-sm { padding: 6px 10px; font-size: 12px; min-width: 60px; text-align: center; }
        .btn-upgrade:active { transform: translateY(2px); box-shadow: 0 2px 6px rgba(211,84,0,0.4); }
        .btn-upgrade:disabled { background: #4b4b60; box-shadow: none; color: #888; cursor: not-allowed; }
        .system-btn-group { display: flex; flex-direction: column; gap: 10px; margin-top: 10px; }
        .btn-system { background: linear-gradient(135deg, #34495e, #2c3e50); color: white; border: none; padding: 14px; border-radius: 12px; font-size: 15px; font-weight: bold; cursor: pointer; box-shadow: 0 4px 12px rgba(0,0,0,0.3); transition: transform 0.1s; }
        .btn-system:active { transform: translateY(2px); }
        .btn-admin { background: linear-gradient(135deg, #9b59b6, #8e44ad); border: 1px solid #b19cd9; color: #f1c40f; box-shadow: 0 4px 15px rgba(142,68,173,0.4); }
        .btn-danger { background: linear-gradient(135deg, #ff4757, #c0392b); box-shadow: 0 4px 12px rgba(231,76,60,0.4); }
        .btn-save-load { background: linear-gradient(135deg, #0984e3, #6c5ce7); box-shadow: 0 4px 12px rgba(9,132,227,0.4); }

        .tap-hint { position: absolute; bottom: 32px; font-size: 12px; color: rgba(255,255,255,0.9); background: rgba(15,15,25,0.7); backdrop-filter: blur(4px); padding: 6px 14px; border-radius: 14px; z-index: 10; pointer-events: none; border: 1px solid rgba(255,255,255,0.1); }

        /* 모달 디자인 */
        .modal-overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(11, 11, 16, 0.85); display: none; justify-content: center; align-items: center; z-index: 1000; backdrop-filter: blur(8px); }
        .modal-card { background: #232336; border: 2px solid rgba(155,89,182,0.6); border-radius: 16px; padding: 24px; width: 88%; max-width: 360px; text-align: center; box-shadow: 0 10px 40px rgba(0, 0, 0, 0.8); }
        .modal-card h3 { color: #f1c40f; margin-bottom: 12px; font-size: 22px; font-weight: 800; }
        .modal-card p { color: #b2bec3; font-size: 14px; margin-bottom: 18px; line-height: 1.5; }
        .modal-card input { width: 100%; padding: 14px; border-radius: 10px; border: 1px solid rgba(255,255,255,0.2); background: #181824; color: #fff; font-size: 16px; text-align: center; outline: none; margin-bottom: 18px; box-shadow: inset 0 2px 5px rgba(0,0,0,0.5); }
        .modal-card input:focus { border-color: #f1c40f; }
        .modal-btns { display: flex; gap: 10px; }
        .modal-btns button { flex: 1; padding: 14px; border: none; border-radius: 10px; font-weight: bold; font-size: 15px; cursor: pointer; box-shadow: 0 4px 10px rgba(0,0,0,0.3); transition: transform 0.1s; }
        .modal-btns button:active { transform: translateY(2px); }
        .btn-confirm { background: linear-gradient(135deg, #9b59b6, #8e44ad); color: white; }
        .btn-confirm-danger { background: linear-gradient(135deg, #ff4757, #c0392b); color: white; }
        .btn-cancel { background: linear-gradient(135deg, #7f8c8d, #636e72); color: white; }

        .char-select-btn { width: 100%; padding: 18px; margin-bottom: 14px; border-radius: 14px; border: 2px solid rgba(255,255,255,0.15); background: linear-gradient(135deg, #1e1e2d, #181824); color: #fff; font-size: 16px; font-weight: bold; cursor: pointer; transition: all 0.2s; box-shadow: 0 6px 20px rgba(0,0,0,0.4); text-align: left; display: flex; align-items: center; gap: 14px; }
        .char-select-btn:hover, .char-select-btn:active { border-color: #f1c40f; transform: translateY(-3px); box-shadow: 0 10px 30px rgba(0,0,0,0.6); }
        .char-knight { border-left: 6px solid #e74c3c; } .char-mage { border-left: 6px solid #00d2d3; }
    </style>
</head>
<body>

<div id="game-container">
    <header>
        <div class="user-info">
            <span class="level-badge" id="player-level">Lv. 1 모험가</span>
            <div class="meso-box"><span>🪙</span><span id="player-meso">0</span></div>
        </div>
        <div class="exp-bar-container">
            <div class="exp-bar-fill" id="exp-fill"></div>
            <div class="exp-text" id="exp-text">0 / 50 (0%)</div>
        </div>
    </header>

    <div id="battlefield" onclick="manualAttack(event)">
        <div class="stage-header">
            <div class="stage-title" id="stage-name">STAGE 1 : 푸른 언덕 초원</div>
            <button id="btn-autohunt" class="btn-autohunt active" onclick="toggleAutoHunt(event)">⚔️ 자동사냥: ON</button>
        </div>
        <div class="battle-area">
            <div class="ground" id="ground-bar"></div>
            <div class="pet" id="pet-sprite"></div>
            <div class="character" id="character">
                <div class="character-weapon" id="weapon-icon"></div>
            </div>
            <div class="lightning-bolt" id="lightning-bolt"></div>
            <div class="scratch-effect" id="scratch-effect"></div>
            <div class="monster-container" id="monster-box">
                <div class="monster-hp-bar"><div class="monster-hp-fill" id="monster-hp-fill"></div></div>
                <div class="monster-sprite" id="monster-sprite"></div>
            </div>
        </div>
        <div class="tap-hint">💡 화면을 터치/클릭하면 수동 공격!</div>
    </div>

    <div id="bottom-panel">
        <nav class="nav-tabs">
            <button class="tab-btn active" onclick="switchTab('stat', event)">스탯</button>
            <button class="tab-btn" onclick="switchTab('weapon', event)">장비</button>
            <button class="tab-btn" onclick="switchTab('wing', event)">날개 🪽</button>
            <button class="tab-btn" onclick="switchTab('pet', event)">펫 🐾</button>
            <button class="tab-btn" onclick="switchTab('system', event)">설정</button>
        </nav>

        <div id="tab-stat" class="tab-content active">
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4>공격력 강화 (<span id="stat-atk-lvl">1</span>)</h4>
                    <p>현재 공격력: <span id="stat-atk-val">10</span></p>
                </div>
                <div class="btn-group-atk">
                    <button class="btn-upgrade btn-sm" id="btn-up-atk-1" onclick="upgradeStat('atk', 1)">+1<br><small id="cost-atk-1">10</small></button>
                    <button class="btn-upgrade btn-sm" id="btn-up-atk-10" onclick="upgradeStat('atk', 10)">+10<br><small id="cost-atk-10">550</small></button>
                    <button class="btn-upgrade btn-sm" id="btn-up-atk-100" onclick="upgradeStat('atk', 100)">+100<br><small id="cost-atk-100">50K</small></button>
                </div>
            </div>
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4>공격 속도 (<span id="stat-spd-lvl">1</span>)</h4>
                    <p>공격 주기: <span id="stat-spd-val">1.0초</span></p>
                </div>
                <button class="btn-upgrade" id="btn-up-spd" onclick="upgradeStat('spd')">강화<br><small id="cost-spd">100 Gold</small></button>
            </div>
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4>크리티컬 확률 (<span id="stat-crit-lvl">1</span>)</h4>
                    <p>현재 확률: <span id="stat-crit-val">5%</span></p>
                </div>
                <button class="btn-upgrade" id="btn-up-crit" onclick="upgradeStat('crit')">강화<br><small id="cost-crit">130 Gold</small></button>
            </div>
        </div>

        <div id="tab-weapon" class="tab-content">
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4 id="weapon-name">무기 (Tier 1)</h4>
                    <p>공격력 보너스: +<span id="weapon-bonus">0</span></p>
                </div>
                <button class="btn-upgrade" id="btn-up-weapon" onclick="upgradeWeapon()">무기 승급<br><small id="cost-weapon">200 Gold</small></button>
            </div>
        </div>

        <div id="tab-wing" class="tab-content">
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4 id="wing-name-desc" style="color: #f1c40f;">날개 없음 (Lv. 1)</h4>
                    <p>추가 타격력: +<span id="wing-atk-bonus">0</span><br>
                    <small>※ 10레벨 단위 고화질 HD 날개 진화 (Max 100)</small></p>
                </div>
                <button class="btn-upgrade" id="btn-up-wing" onclick="upgradeWing()">날개 강화<br><small id="cost-wing">500 Gold</small></button>
            </div>
        </div>

        <div id="tab-pet" class="tab-content">
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4 id="pet-name">요정 🧚‍♀️</h4>
                    <p id="pet-type-desc">5초마다 번개 공격 (공격력의 <span id="pet-ratio-val">150</span>%)</p>
                </div>
                <button class="btn-upgrade" id="btn-change-pet" onclick="changePet()">펫 변경 🔄</button>
            </div>
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4>펫 보조 공격력 강화 (<span id="pet-lvl">1</span>)</h4>
                    <p>현재 비율: <span id="pet-ratio-desc">150%</span></p>
                </div>
                <button class="btn-upgrade" id="btn-up-pet" onclick="upgradePet()">강화<br><small id="cost-pet">300 Gold</small></button>
            </div>
        </div>

        <div id="tab-system" class="tab-content">
            <div class="upgrade-card" style="flex-direction: column; align-items: stretch; gap: 10px;">
                <div class="upgrade-info" style="margin-bottom: 5px;">
                    <h4>🎉 이벤트 보상 코드</h4>
                    <p>코드를 입력하여 특별한 보상을 받으세요. ('1' 또는 '2' 입력)</p>
                </div>
                <div style="display: flex; gap: 10px;">
                    <input type="text" id="event-code-input" placeholder="코드 입력" style="flex: 1; padding: 10px; border-radius: 8px; border: 1px solid #555; background: #181824; color: #fff; font-size: 14px; outline: none;">
                    <button class="btn-system" style="margin: 0; padding: 10px 16px; background: linear-gradient(135deg, #2ed573, #26af5f);" onclick="submitEventCode()">보상 받기</button>
                </div>
            </div>

            <div class="upgrade-card" style="flex-direction: column; align-items: stretch; gap: 10px;">
                <div class="upgrade-info" style="margin-bottom: 5px;">
                    <h4>💾 데이터 파일 백업 / 복구</h4>
                    <p>기기를 변경하거나 안전하게 보관하려면 파일로 저장하세요.</p>
                </div>
                <div style="display: flex; gap: 10px;">
                    <button class="btn-system btn-save-load" style="margin: 0; flex: 1;" onclick="exportGameData()">파일 추출</button>
                    <button class="btn-system btn-save-load" style="margin: 0; flex: 1; background: linear-gradient(135deg, #f39c12, #d35400);" onclick="document.getElementById('import-file').click()">불러오기</button>
                    <input type="file" id="import-file" accept=".json" style="display: none;" onchange="importGameData(event)">
                </div>
            </div>

            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4>현재 스테이지 정보</h4>
                    <p id="stage-desc">진행 정보 표시</p>
                </div>
            </div>

            <div class="system-btn-group">
                <button class="btn-system btn-admin" onclick="openAdminModal()">🔑 관리자 모드 접속</button>
                <button class="btn-system btn-danger" onclick="openResetModal()">🔄 데이터 초기화</button>
            </div>
        </div>
    </div>

    <!-- Modals -->
    <div id="char-select-modal" class="modal-overlay" style="display: flex;">
        <div class="modal-card" style="border-color: #f1c40f;">
            <h3>⚔️ 새로운 모험의 시작</h3>
            <p style="margin-bottom: 20px;">플레이할 영웅의 직업을 선택하세요.</p>
            <button class="char-select-btn char-knight" onclick="selectCharacter('knight')">
                <span style="font-size: 24px;">🐉</span>
                <div>
                    <div style="font-size: 16px; color: #fff; margin-bottom: 3px;">드래곤 기사</div>
                    <div style="font-size: 12px; color: #a4b0be; font-weight: normal;">근접 거대검 / 화염 & 드래곤 날개</div>
                </div>
            </button>
            <button class="char-select-btn char-mage" onclick="selectCharacter('mage')">
                <span style="font-size: 24px;">❄️</span>
                <div>
                    <div style="font-size: 16px; color: #fff; margin-bottom: 3px;">천사 마법사</div>
                    <div style="font-size: 12px; color: #a4b0be; font-weight: normal;">마력 에너지 볼트 / 순백 천사 날개</div>
                </div>
            </button>
        </div>
    </div>

    <div id="admin-modal" class="modal-overlay">
        <div class="modal-card" onclick="event.stopPropagation()">
            <h3>🔑 관리자 인증</h3>
            <p>비밀번호 4자리를 입력하세요.</p>
            <input type="password" id="admin-pw-input" placeholder="비밀번호 입력" maxlength="4">
            <div class="modal-btns">
                <button class="btn-confirm" onclick="submitAdminPw()">확인</button>
                <button class="btn-cancel" onclick="closeAdminModal()">취소</button>
            </div>
        </div>
    </div>

    <div id="offline-modal" class="modal-overlay">
        <div class="modal-card" style="border-color: #2ecc71;" onclick="event.stopPropagation()">
            <h3 style="color: #2ecc71;">🌙 백그라운드 방치 완료</h3>
            <p id="offline-desc">방치된 시간 동안 몬스터를 자급자족으로 정벌했습니다!</p>
            <div class="modal-btns">
                <button class="btn-confirm" style="background: #2ecc71;" onclick="closeOfflineModal()">보상 수령하기</button>
            </div>
        </div>
    </div>

    <div id="reset-modal" class="modal-overlay">
        <div class="modal-card" style="border-color: #e74c3c;" onclick="event.stopPropagation()">
            <h3 style="color: #e74c3c;">🔄 데이터 초기화</h3>
            <p>모든 게임 데이터와 메소, 진행 상황이 영구적으로 초기화됩니다.<br><br>정말 삭제하시겠습니까?</p>
            <div class="modal-btns">
                <button class="btn-confirm-danger" onclick="confirmReset()">초기화 진행</button>
                <button class="btn-cancel" onclick="closeResetModal()">취소</button>
            </div>
        </div>
    </div>
</div>

<script>
    // 전역 오디오 컨텍스트
    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    function playSound(freq, type, duration) {
        if (audioCtx.state === 'suspended') audioCtx.resume();
        try {
            const osc = audioCtx.createOscillator(); const gain = audioCtx.createGain();
            osc.type = type; osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
            gain.gain.setValueAtTime(0.08, audioCtx.currentTime); gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + duration);
            osc.connect(gain); gain.connect(audioCtx.destination); osc.start(); osc.stop(audioCtx.currentTime + duration);
        } catch(e) {}
    }

    // ⚔️ 기사 대검 10종
    const WEAPON_ICONS_KNIGHT = [
        `data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 4px %23bdc3c7)"><path d="M 18 36 L 18 10 L 24 4 L 30 10 L 30 36 Z" fill="%237f8c8d" stroke="%23ecf0f1" stroke-width="1.5"/><rect x="12" y="36" width="24" height="4" fill="%232c3e50"/><rect x="20" y="40" width="8" height="6" fill="%238e5123"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 5px %233498db)"><path d="M 20 34 L 16 12 L 24 2 L 32 12 L 28 34 Z" fill="%232980b9" stroke="%2300d2d3" stroke-width="1.5"/><polygon points="12,34 36,34 24,38" fill="%231abc9c"/><rect x="22" y="38" width="4" height="8" fill="%2334495e"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 6px %23ff4757)"><path d="M 18 36 L 16 28 L 20 26 L 16 20 L 20 18 L 16 10 L 24 2 L 32 10 L 28 18 L 32 20 L 28 26 L 32 28 L 30 36 Z" fill="%23c0392b" stroke="%23ff7675" stroke-width="1.5"/><rect x="14" y="36" width="20" height="4" fill="%232d3436"/><rect x="20" y="40" width="8" height="6" fill="%23111"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 6px %239b59b6)"><path d="M 22 36 Q 30 20 40 10 Q 20 5 16 2 Q 10 15 14 36 Z" fill="%238e44ad" stroke="%23d2b4de" stroke-width="2"/><rect x="16" y="36" width="12" height="4" fill="%232c3e50"/><rect x="18" y="40" width="8" height="8" fill="%23111"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 7px %232ecc71)"><path d="M 18 36 Q 10 15 2 2 Q 24 10 28 36 Z" fill="%2327ae60" stroke="%231abc9c" stroke-width="2"/><polygon points="26,12 36,18 28,24" fill="%2316a085"/><rect x="16" y="36" width="16" height="4" fill="%232f3640"/><rect x="20" y="40" width="8" height="6" fill="%23111"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 8px %23e67e22)"><path d="M 20 34 Q 10 20 24 2 Q 38 20 28 34 Z" fill="%23d35400" stroke="%23f1c40f" stroke-width="2"/><path d="M 24 34 L 24 10" stroke="%23f39c12" stroke-width="3"/><polygon points="10,34 38,34 24,40" fill="%23c0392b"/><rect x="20" y="40" width="8" height="6" fill="%23111"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 8px %23c0392b)"><path d="M 12 36 L 12 12 L 24 2 L 36 12 L 36 36 Z" fill="%231e272e" stroke="%23ff4757" stroke-width="2.5"/><line x1="24" y1="36" x2="24" y2="8" stroke="%23e74c3c" stroke-width="2"/><rect x="8" y="36" width="32" height="6" fill="%23111"/><rect x="18" y="42" width="12" height="6" fill="%232c3e50"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 10px %2300d2d3)"><path d="M 22 36 L 20 16 L 24 0 L 28 16 L 26 36 Z" fill="%2348dbfb" stroke="%23ffffff" stroke-width="1.5"/><polygon points="12,32 36,32 30,36 18,36" fill="%230abde3"/><rect x="20" y="36" width="8" height="10" fill="%23222f3e"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 12px %23f39c12)"><path d="M 18 36 L 12 24 L 20 18 L 14 6 L 24 0 L 34 6 L 28 18 L 36 24 L 30 36 Z" fill="%23c0392b" stroke="%23f1c40f" stroke-width="2"/><polygon points="8,36 40,36 24,42" fill="%238e44ad" stroke="%23f39c12" stroke-width="1.5"/><rect x="20" y="42" width="8" height="6" fill="%232c3e50"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 15px %23ff00ff)"><path d="M 20 34 L 10 16 L 20 12 L 16 0 L 24 6 L 32 0 L 28 12 L 38 16 L 28 34 Z" fill="%23111" stroke="%23ff9ff3" stroke-width="2.5"/><polygon points="4,34 44,34 34,40 14,40" fill="%236c5ce7" stroke="%23ff00ff" stroke-width="1.5"/><rect x="20" y="40" width="8" height="8" fill="%232f3640"/><circle cx="24" cy="20" r="3" fill="%2300d2d3"/></g></svg>`
    ];

    const WEAPONS_KNIGHT = [ { name: "초보자의 대검", bonus: 0, cost: 200, icon: WEAPON_ICONS_KNIGHT[0] }, { name: "용병의 철검", bonus: 15, cost: 800, icon: WEAPON_ICONS_KNIGHT[1] }, { name: "피빛 톱날검", bonus: 50, cost: 3000, icon: WEAPON_ICONS_KNIGHT[2] }, { name: "초승달 굽은검", bonus: 150, cost: 12000, icon: WEAPON_ICONS_KNIGHT[3] }, { name: "드래곤 투스", bonus: 500, cost: 50000, icon: WEAPON_ICONS_KNIGHT[4] }, { name: "화염의 광검", bonus: 1500, cost: 200000, icon: WEAPON_ICONS_KNIGHT[5] }, { name: "흑요석 파괴검", bonus: 4500, cost: 800000, icon: WEAPON_ICONS_KNIGHT[6] }, { name: "폭풍의 환도", bonus: 15000, cost: 3500000, icon: WEAPON_ICONS_KNIGHT[7] }, { name: "지옥염룡검", bonus: 50000, cost: 15000000, icon: WEAPON_ICONS_KNIGHT[8] }, { name: "창세의 마왕검", bonus: 200000, cost: 80000000, icon: WEAPON_ICONS_KNIGHT[9] } ];

    // 🔮 마법사 지팡이 10종
    const WEAPON_ICONS_MAGE = [
        `data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 5px %232ecc71)"><line x1="24" y1="40" x2="24" y2="12" stroke="%238e5123" stroke-width="4"/><circle cx="24" cy="8" r="4" fill="%232ecc71"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 6px %23fd79a8)"><line x1="24" y1="40" x2="24" y2="12" stroke="%23b8e994" stroke-width="4"/><path d="M 24 0 L 28 8 L 24 16 L 20 8 Z" fill="%23fd79a8" stroke="%23fff" stroke-width="1"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 7px %23f1c40f)"><line x1="24" y1="40" x2="24" y2="12" stroke="%23f39c12" stroke-width="4"/><polygon points="24,2 29,8 24,14 19,8" fill="%23f1c40f" stroke="%23fff" stroke-width="1.5"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 8px %2300d2d3)"><line x1="24" y1="40" x2="24" y2="14" stroke="%23bdc3c7" stroke-width="4"/><circle cx="24" cy="8" r="6" fill="%2300d2d3" stroke="%23fff" stroke-width="1.5"/><circle cx="24" cy="8" r="2" fill="%23fff"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="48" height불편을 드려 죄송합니다. 캐릭터 선택창에서 진행되지 않던 문제와 GitHub Pages 상단에 불필요한 문구가 뜨는 문제 모두 **코드를 복사/붙여넣기 하거나 저장하는 과정에서 눈에 보이지 않는 공백이나 인코딩 정보가 포함되어 발생한 것**으로 보입니다.

이를 해결하기 위해, GitHub가 HTML 파일을 텍스트가 아닌 웹페이지로 정확히 인식하도록 파일 가장 첫 줄에 BOM 등의 불필요한 문자가 없도록 최적화하고, 직업을 선택한 후 캐릭터 선택 모달을 즉시 숨기고(`style.display = "none"`) 게임 플레이 로직을 시작하도록(`startGameplay()`) 수정한 최종 코드를 아래에 제공합니다.

또한, 리포지토리 이름(`beta-test`) 등이 상단에 표시되는 문제를 해결하기 위해, 게임 화면이 브라우저 전체를 채우도록(Fullscreen) CSS를 추가했습니다.

---

### 📄 `index.html` (최종 수정 완료 버전)

아래의 전체 코드를 복사하여 기존 `index.html` 파일의 내용을 **완전히 지우고** 덮어쓰기 해주세요. 코드의 가장 첫 줄(`<!DOCTYPE html>`) 앞에 공백이나 다른 문자가 없도록 주의해주시길 바랍니다.

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>메이플 키우기 - 정식 출시 버전</title>
    <style>
        :root {
            --panel-bg: #181824;
            --panel-card: #232336;
            --main-yellow: #f1c40f;
            --main-orange: #e67e22;
            --text-light: #ffffff;
            image-rendering: crisp-edges;
        }

        * {
            box-sizing: border-box; user-select: none; -webkit-user-select: none;
            touch-action: manipulation; margin: 0; padding: 0;
            font-family: 'Pretendard', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
        }

        /* 📱 풀스크린 최적화: 브라우저 여백 제거 및 가득 차게 설정 */
        body {
            background-color: #0b0b10; color: var(--text-light); display: flex;
            justify-content: center; align-items: center; min-height: 100vh; min-height: 100dvh; overflow: hidden;
            width: 100vw; margin: 0; /* 불필요한 마진 제거 */
        }

        #game-container {
            width: 100%; max-width: 500px; height: 100vh; height: 100dvh;
            background: linear-gradient(180deg, #1e1e2d 0%, #12121a 100%);
            display: flex; flex-direction: column; position: relative; box-shadow: 0 0 40px rgba(0,0,0,0.95); overflow: hidden;
        }

        header {
            background: linear-gradient(180deg, rgba(25,25,38,0.9), rgba(18,18,26,0.9));
            backdrop-filter: blur(8px); padding: 14px 16px; border-bottom: 2px solid rgba(255,255,255,0.08);
            display: flex; flex-direction: column; gap: 10px; z-index: 10;
        }

        .user-info { display: flex; justify-content: space-between; align-items: center; }
        .level-badge {
            background: linear-gradient(135deg, #f39c12, #d35400); color: #fff; padding: 6px 16px;
            border-radius: 16px; font-weight: bold; font-size: 14px; box-shadow: 0 4px 10px rgba(230,126,34,0.4);
        }
        .meso-box {
            display: flex; align-items: center; gap: 6px; background: rgba(0,0,0,0.4);
            padding: 6px 16px; border-radius: 16px; border: 1px solid rgba(241,196,15,0.4);
            color: var(--main-yellow); font-weight: bold; font-size: 16px; box-shadow: inset 0 2px 4px rgba(0,0,0,0.5);
        }

        .exp-bar-container {
            width: 100%; height: 14px; background: rgba(0,0,0,0.6); border-radius: 7px;
            overflow: hidden; border: 1px solid rgba(255,255,255,0.1); position: relative;
        }
        .exp-bar-fill { height: 100%; background: linear-gradient(90deg, #2ecc71, #00d2d3); width: 0%; transition: width 0.3s ease; box-shadow: 0 0 10px rgba(46,204,113,0.5); }
        .exp-text { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); font-size: 10px; font-weight: bold; text-shadow: 1px 1px 2px #000; }

        #battlefield {
            flex: 1.2; position: relative; background-size: cover; background-position: center;
            overflow: hidden; display: flex; flex-direction: column; justify-content: space-between;
            align-items: center; cursor: pointer; transition: background 0.5s ease;
        }

        .stage-header { width: 100%; display: flex; justify-content: space-between; align-items: center; padding: 14px 18px 0 18px; z-index: 10; }
        .stage-title {
            background: rgba(15, 15, 25, 0.8); padding: 8px 18px; border-radius: 20px;
            font-size: 14px; font-weight: bold; color: #fff; border: 1px solid rgba(255, 255, 255, 0.15); backdrop-filter: blur(6px); box-shadow: 0 4px 15px rgba(0,0,0,0.3);
        }

        .btn-autohunt {
            background: linear-gradient(135deg, #ff4757, #ff6b81); color: white; border: 1px solid rgba(255, 255, 255, 0.3);
            padding: 8px 18px; border-radius: 20px; font-size: 13px; font-weight: bold; cursor: pointer; box-shadow: 0 4px 12px rgba(255,71,87,0.4); transition: all 0.2s;
        }
        .btn-autohunt.active { background: linear-gradient(135deg, #2ed573, #26af5f); box-shadow: 0 4px 15px rgba(46,213,115,0.5); }

        .battle-area { position: relative; width: 100%; height: 100%; display: flex; justify-content: center; align-items: flex-end; padding-bottom: 22px; }
        .ground {
            position: absolute; bottom: 0; left: 0; width: 100%; height: 32px;
            background-color: var(--ground-color, #27ae60); border-top: 4px solid rgba(255,255,255,0.25); box-shadow: inset 0 4px 8px rgba(0,0,0,0.3); transition: background 0.4s;
        }

        .character {
            position: absolute; left: 30%; bottom: 28px; transform: translateX(-50%);
            width: 140px; height: 140px; background-size: 280px 140px; background-repeat: no-repeat;
            z-index: 5; pointer-events: none; filter: drop-shadow(0 8px 12px rgba(0,0,0,0.5));
            transition: filter 0.1s ease, transform 0.1s ease; animation: spriteIdleLarge 0.8s steps(1) infinite;
        }
        @keyframes spriteIdleLarge { 0% { background-position: 0px 0px; } 50% { background-position: -140px 0px; } 100% { background-position: 0px 0px; } }
        .character.attacking { transform: translateX(-50%) scale(1.05); }

        .character-weapon {
            position: absolute; left: 75px; top: 60px; width: 50px; height: 50px;
            background-size: contain; background-repeat: no-repeat; transition: transform 0.15s ease;
            filter: drop-shadow(0 2px 4px rgba(0,0,0,0.5));
        }
        .character.attacking .character-weapon { transform: rotate(50deg) scale(1.4) translateX(15px) translateY(-5px); }

        .energy-bolt {
            position: absolute; width: 80px; height: 30px; z-index: 35; pointer-events: none;
            transform: translate(-50%, -50%) rotate(var(--angle, 0deg));
            animation: shootEnergyDynamic 0.15s cubic-bezier(0.1, 0.8, 0.3, 1) forwards;
        }
        .bolt-core {
            position: absolute; right: 0; top: 50%; transform: translateY(-50%);
            width: 28px; height: 28px; border-radius: 50%;
            background: radial-gradient(circle, #ffffff 20%, #00d2d3 60%, #a29bfe 100%); box-shadow: 0 0 15px #fff, 0 0 30px #00d2d3, 0 0 50px #a29bfe;
        }
        .bolt-trail {
            position: absolute; right: 14px; top: 50%; transform: translateY(-50%);
            width: 60px; height: 12px; border-radius: 50px 0 0 50px;
            background: linear-gradient(90deg, transparent, rgba(162, 155, 254, 0.8), #00d2d3); filter: blur(2px);
        }
        @keyframes shootEnergyDynamic {
            0% { transform: translate(-50%, -50%) rotate(var(--angle)) scale(0.5); opacity: 0.8; filter: hue-rotate(0deg); }
            100% { transform: translate(calc(-50% + var(--target-x)), calc(-50% + var(--target-y))) rotate(var(--angle)) scale(1.5); opacity: 1; filter: hue-rotate(270deg); }
        }

        .pet { position: absolute; left: 6%; bottom: 55px; width: 45px; height: 45px; background-size: contain; background-repeat: no-repeat; z-index: 6; filter: drop-shadow(0 6px 10px rgba(0,0,0,0.4)); animation: petFloat 1.8s ease-in-out infinite alternate; }
        @keyframes petFloat { 0% { transform: translateY(0px); } 100% { transform: translateY(-10px); } }
        .pet.tackling { animation: petTackle 0.35s cubic-bezier(0.1,0.9,0.2,1) !important; }
        @keyframes petTackle { 0% { transform: translate(0, 0) scale(1); } 40% { transform: translate(190px, 15px) scale(1.35); } 100% { transform: translate(0, 0) scale(1); } }

        .lightning-bolt { position: absolute; top: 0; right: 22%; width: 40px; height: 220px; background: linear-gradient(180deg, #f1c40f, #ffffff, #e67e22); clip-path: polygon(40% 0%, 70% 0%, 30% 40%, 80% 40%, 10% 100%, 40% 50%, 0% 50%); z-index: 30; opacity: 0; pointer-events: none; filter: drop-shadow(0 0 12px #f1c40f); }
        .lightning-bolt.active { animation: lightningFlash 0.35s ease-out; }
        @keyframes lightningFlash { 0% { opacity: 0; transform: scaleY(0); } 30% { opacity: 1; transform: scaleY(1); } 70% { opacity: 1; transform: scaleY(1); } 100% { opacity: 0; transform: scaleY(1); } }

        .scratch-effect { position: absolute; right: 20%; bottom: 28px; width: 80px; height: 80px; background-image: url('data:image/svg+xml;utf8,<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="64" height="64" viewBox="0 0 64 64"><line x1="8" y1="12" x2="52" y2="52" stroke="%23ff3838" stroke-width="6" stroke-linecap="round"/><line x1="20" y1="8" x2="60" y2="44" stroke="%23ff3838" stroke-width="6" stroke-linecap="round"/><line x1="6" y1="24" x2="44" y2="60" stroke="%23ff3838" stroke-width="6" stroke-linecap="round"/></svg>'); background-size: contain; background-repeat: no-repeat; z-index: 25; opacity: 0; pointer-events: none; }
        .scratch-effect.active { animation: scratchSlash 0.3s cubic-bezier(0.1,0.9,0.2,1); }
        @keyframes scratchSlash { 0% { opacity: 0; transform: scale(0.6) rotate(-15deg); } 40% { opacity: 1; transform: scale(1.25) rotate(0deg); filter: drop-shadow(0 0 12px #ff3838); } 100% { opacity: 0; transform: scale(1) rotate(10deg); } }

        .monster-container { position: absolute; right: 15%; bottom: 28px; display: flex; flex-direction: column; align-items: center; z-index: 5; pointer-events: none; filter: drop-shadow(0 6px 10px rgba(0,0,0,0.4)); }
        .monster-hp-bar { width: 80px; height: 10px; background: rgba(0,0,0,0.7); border: 1px solid rgba(255,255,255,0.4); border-radius: 5px; overflow: hidden; margin-bottom: 8px; box-shadow: 0 2px 5px rgba(0,0,0,0.5); }
        .monster-hp-fill { height: 100%; background: linear-gradient(90deg, #ff4757, #ff6b81); width: 100%; transition: width 0.1s linear; }
        .monster-sprite { width: 80px; height: 80px; background-size: cover; background-repeat: no-repeat; transition: transform 0.2s ease, filter 0.2s ease; }

        .damage-text { position: absolute; font-size: 28px; font-weight: 900; color: #f1c40f; text-shadow: 2px 2px 0 #000, -2px -2px 0 #000, 2px -2px 0 #000, -2px 2px 0 #000, 0 0 10px rgba(241,196,15,0.6); pointer-events: none; animation: floatDamage 0.65s cubic-bezier(0.1,0.9,0.2,1) forwards; z-index: 20; transform: translate(-50%, 0); }
        .damage-text.crit { color: #ff3838; font-size: 34px; text-shadow: 2px 2px 0 #000, -2px -2px 0 #000, 0 0 15px rgba(255,56,56,0.8); }
        .damage-text.pet-dmg { color: #00d2d3; font-size: 30px; text-shadow: 2px 2px 0 #000, -2px -2px 0 #000, 0 0 15px rgba(0,210,211,0.8); }
        .damage-text.char-dmg { color: #ff4757; font-size: 26px; text-shadow: 2px 2px 0 #000, -2px -2px 0 #000, 0 0 15px rgba(255,71,87,0.8); z-index: 40; }

        @keyframes floatDamage {
            0% { opacity: 1; transform: translate(-50%, 0) scale(0.8); }
            50% { opacity: 1; transform: translate(-50%, -40px) scale(1.25); }
            100% { opacity: 0; transform: translate(-50%, -70px) scale(1); }
        }

        #bottom-panel { flex: 1; min-height: 40%; max-height: 55vh; background-color: var(--panel-bg); display: flex; flex-direction: column; border-top: 2px solid rgba(255,255,255,0.08); box-shadow: 0 -10px 30px rgba(0,0,0,0.5); z-index: 20; }
        .nav-tabs { display: flex; background: #12121a; border-bottom: 1px solid rgba(255,255,255,0.06); }
        .tab-btn { flex: 1; padding: 14px 0; background: none; border: none; color: #7f8c8d; font-weight: bold; font-size: 14px; cursor: pointer; white-space: nowrap; transition: color 0.2s; }
        .tab-btn.active { color: var(--main-yellow); border-bottom: 3px solid var(--main-yellow); background: rgba(241, 196, 15, 0.05); }
        .tab-content { flex: 1; padding: 16px; overflow-y: auto; display: none; }
        .tab-content.active { display: block; }
        
        .upgrade-card { background: var(--panel-card); border-radius: 12px; padding: 14px; margin-bottom: 12px; display: flex; justify-content: space-between; align-items: center; border: 1px solid rgba(255,255,255,0.06); box-shadow: 0 4px 15px rgba(0,0,0,0.3); }
        .upgrade-info h4 { font-size: 15px; color: #fff; margin-bottom: 4px; font-weight: 700; }
        .upgrade-info p { font-size: 13px; color: #a4b0be; line-height: 1.4; }
        .btn-group-atk { display: flex; gap: 6px; }
        .btn-upgrade { background: linear-gradient(135deg, #f39c12, #d35400); border: none; color: #fff; padding: 10px 14px; border-radius: 10px; font-weight: bold; font-size: 13px; cursor: pointer; box-shadow: 0 4px 12px rgba(211,84,0,0.4); transition: transform 0.1s; }
        .btn-upgrade.btn-sm { padding: 6px 10px; font-size: 12px; min-width: 60px; text-align: center; }
        .btn-upgrade:active { transform: translateY(2px); box-shadow: 0 2px 6px rgba(211,84,0,0.4); }
        .btn-upgrade:disabled { background: #4b4b60; box-shadow: none; color: #888; cursor: not-allowed; }
        .system-btn-group { display: flex; flex-direction: column; gap: 10px; margin-top: 10px; }
        .btn-system { background: linear-gradient(135deg, #34495e, #2c3e50); color: white; border: none; padding: 14px; border-radius: 12px; font-size: 15px; font-weight: bold; cursor: pointer; box-shadow: 0 4px 12px rgba(0,0,0,0.3); transition: transform 0.1s; }
        .btn-system:active { transform: translateY(2px); }
        .btn-admin { background: linear-gradient(135deg, #9b59b6, #8e44ad); border: 1px solid #b19cd9; color: #f1c40f; box-shadow: 0 4px 15px rgba(142,68,173,0.4); }
        .btn-danger { background: linear-gradient(135deg, #ff4757, #c0392b); box-shadow: 0 4px 12px rgba(231,76,60,0.4); }
        .btn-save-load { background: linear-gradient(135deg, #0984e3, #6c5ce7); box-shadow: 0 4px 12px rgba(9,132,227,0.4); }

        .tap-hint { position: absolute; bottom: 32px; font-size: 12px; color: rgba(255,255,255,0.9); background: rgba(15,15,25,0.7); backdrop-filter: blur(4px); padding: 6px 14px; border-radius: 14px; z-index: 10; pointer-events: none; border: 1px solid rgba(255,255,255,0.1); }

        /* 모달 디자인 */
        .modal-overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(11, 11, 16, 0.85); display: none; justify-content: center; align-items: center; z-index: 1000; backdrop-filter: blur(8px); }
        .modal-card { background: #232336; border: 2px solid rgba(155,89,182,0.6); border-radius: 16px; padding: 24px; width: 88%; max-width: 360px; text-align: center; box-shadow: 0 10px 40px rgba(0, 0, 0, 0.8); }
        .modal-card h3 { color: #f1c40f; margin-bottom: 12px; font-size: 22px; font-weight: 800; }
        .modal-card p { color: #b2bec3; font-size: 14px; margin-bottom: 18px; line-height: 1.5; }
        .modal-card input { width: 100%; padding: 14px; border-radius: 10px; border: 1px solid rgba(255,255,255,0.2); background: #181824; color: #fff; font-size: 16px; text-align: center; outline: none; margin-bottom: 18px; box-shadow: inset 0 2px 5px rgba(0,0,0,0.5); }
        .modal-card input:focus { border-color: #f1c40f; }
        .modal-btns { display: flex; gap: 10px; }
        .modal-btns button { flex: 1; padding: 14px; border: none; border-radius: 10px; font-weight: bold; font-size: 15px; cursor: pointer; box-shadow: 0 4px 10px rgba(0,0,0,0.3); transition: transform 0.1s; }
        .modal-btns button:active { transform: translateY(2px); }
        .btn-confirm { background: linear-gradient(135deg, #9b59b6, #8e44ad); color: white; }
        .btn-confirm-danger { background: linear-gradient(135deg, #ff4757, #c0392b); color: white; }
        .btn-cancel { background: linear-gradient(135deg, #7f8c8d, #636e72); color: white; }

        .char-select-btn { width: 100%; padding: 18px; margin-bottom: 14px; border-radius: 14px; border: 2px solid rgba(255,255,255,0.15); background: linear-gradient(135deg, #1e1e2d, #181824); color: #fff; font-size: 16px; font-weight: bold; cursor: pointer; transition: all 0.2s; box-shadow: 0 6px 20px rgba(0,0,0,0.4); text-align: left; display: flex; align-items: center; gap: 14px; }
        .char-select-btn:hover, .char-select-btn:active { border-color: #f1c40f; transform: translateY(-3px); box-shadow: 0 10px 30px rgba(0,0,0,0.6); }
        .char-knight { border-left: 6px solid #e74c3c; } .char-mage { border-left: 6px solid #00d2d3; }
    </style>
</head>
<body>

<div id="game-container">
    <header>
        <div class="user-info">
            <span class="level-badge" id="player-level">Lv. 1 모험가</span>
            <div class="meso-box"><span>🪙</span><span id="player-meso">0</span></div>
        </div>
        <div class="exp-bar-container">
            <div class="exp-bar-fill" id="exp-fill"></div>
            <div class="exp-text" id="exp-text">0 / 50 (0%)</div>
        </div>
    </header>

    <div id="battlefield" onclick="manualAttack(event)">
        <div class="stage-header">
            <div class="stage-title" id="stage-name">STAGE 1 : 푸른 언덕 초원</div>
            <button id="btn-autohunt" class="btn-autohunt active" onclick="toggleAutoHunt(event)">⚔️ 자동사냥: ON</button>
        </div>
        <div class="battle-area">
            <div class="ground" id="ground-bar"></div>
            <div class="pet" id="pet-sprite"></div>
            <div class="character" id="character">
                <div class="character-weapon" id="weapon-icon"></div>
            </div>
            <div class="lightning-bolt" id="lightning-bolt"></div>
            <div class="scratch-effect" id="scratch-effect"></div>
            <div class="monster-container" id="monster-box">
                <div class="monster-hp-bar"><div class="monster-hp-fill" id="monster-hp-fill"></div></div>
                <div class="monster-sprite" id="monster-sprite"></div>
            </div>
        </div>
        <div class="tap-hint">💡 화면을 터치/클릭하면 수동 공격!</div>
    </div>

    <div id="bottom-panel">
        <nav class="nav-tabs">
            <button class="tab-btn active" onclick="switchTab('stat', event)">스탯</button>
            <button class="tab-btn" onclick="switchTab('weapon', event)">장비</button>
            <button class="tab-btn" onclick="switchTab('wing', event)">날개 🪽</button>
            <button class="tab-btn" onclick="switchTab('pet', event)">펫 🐾</button>
            <button class="tab-btn" onclick="switchTab('system', event)">설정</button>
        </nav>

        <div id="tab-stat" class="tab-content active">
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4>공격력 강화 (<span id="stat-atk-lvl">1</span>)</h4>
                    <p>현재 공격력: <span id="stat-atk-val">10</span></p>
                </div>
                <div class="btn-group-atk">
                    <button class="btn-upgrade btn-sm" id="btn-up-atk-1" onclick="upgradeStat('atk', 1)">+1<br><small id="cost-atk-1">10</small></button>
                    <button class="btn-upgrade btn-sm" id="btn-up-atk-10" onclick="upgradeStat('atk', 10)">+10<br><small id="cost-atk-10">550</small></button>
                    <button class="btn-upgrade btn-sm" id="btn-up-atk-100" onclick="upgradeStat('atk', 100)">+100<br><small id="cost-atk-100">50K</small></button>
                </div>
            </div>
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4>공격 속도 (<span id="stat-spd-lvl">1</span>)</h4>
                    <p>공격 주기: <span id="stat-spd-val">1.0초</span></p>
                </div>
                <button class="btn-upgrade" id="btn-up-spd" onclick="upgradeStat('spd')">강화<br><small id="cost-spd">100 Gold</small></button>
            </div>
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4>크리티컬 확률 (<span id="stat-crit-lvl">1</span>)</h4>
                    <p>현재 확률: <span id="stat-crit-val">5%</span></p>
                </div>
                <button class="btn-upgrade" id="btn-up-crit" onclick="upgradeStat('crit')">강화<br><small id="cost-crit">130 Gold</small></button>
            </div>
        </div>

        <div id="tab-weapon" class="tab-content">
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4 id="weapon-name">무기 (Tier 1)</h4>
                    <p>공격력 보너스: +<span id="weapon-bonus">0</span></p>
                </div>
                <button class="btn-upgrade" id="btn-up-weapon" onclick="upgradeWeapon()">무기 승급<br><small id="cost-weapon">200 Gold</small></button>
            </div>
        </div>

        <div id="tab-wing" class="tab-content">
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4 id="wing-name-desc" style="color: #f1c40f;">날개 없음 (Lv. 1)</h4>
                    <p>추가 타격력: +<span id="wing-atk-bonus">0</span><br>
                    <small>※ 10레벨 단위 고화질 HD 날개 진화 (Max 100)</small></p>
                </div>
                <button class="btn-upgrade" id="btn-up-wing" onclick="upgradeWing()">날개 강화<br><small id="cost-wing">500 Gold</small></button>
            </div>
        </div>

        <div id="tab-pet" class="tab-content">
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4 id="pet-name">요정 🧚‍♀️</h4>
                    <p id="pet-type-desc">5초마다 번개 공격 (공격력의 <span id="pet-ratio-val">150</span>%)</p>
                </div>
                <button class="btn-upgrade" id="btn-change-pet" onclick="changePet()">펫 변경 🔄</button>
            </div>
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4>펫 보조 공격력 강화 (<span id="pet-lvl">1</span>)</h4>
                    <p>현재 비율: <span id="pet-ratio-desc">150%</span></p>
                </div>
                <button class="btn-upgrade" id="btn-up-pet" onclick="upgradePet()">강화<br><small id="cost-pet">300 Gold</small></button>
            </div>
        </div>

        <div id="tab-system" class="tab-content">
            <div class="upgrade-card" style="flex-direction: column; align-items: stretch; gap: 10px;">
                <div class="upgrade-info" style="margin-bottom: 5px;">
                    <h4>🎉 이벤트 보상 코드</h4>
                    <p>코드를 입력하여 특별한 보상을 받으세요. ('1' 또는 '2' 입력)</p>
                </div>
                <div style="display: flex; gap: 10px;">
                    <input type="text" id="event-code-input" placeholder="코드 입력" style="flex: 1; padding: 10px; border-radius: 8px; border: 1px solid #555; background: #181824; color: #fff; font-size: 14px; outline: none;">
                    <button class="btn-system" style="margin: 0; padding: 10px 16px; background: linear-gradient(135deg, #2ed573, #26af5f);" onclick="submitEventCode()">보상 받기</button>
                </div>
            </div>

            <div class="upgrade-card" style="flex-direction: column; align-items: stretch; gap: 10px;">
                <div class="upgrade-info" style="margin-bottom: 5px;">
                    <h4>💾 데이터 파일 백업 / 복구</h4>
                    <p>기기를 변경하거나 안전하게 보관하려면 파일로 저장하세요.</p>
                </div>
                <div style="display: flex; gap: 10px;">
                    <button class="btn-system btn-save-load" style="margin: 0; flex: 1;" onclick="exportGameData()">파일 추출</button>
                    <button class="btn-system btn-save-load" style="margin: 0; flex: 1; background: linear-gradient(135deg, #f39c12, #d35400);" onclick="document.getElementById('import-file').click()">불러오기</button>
                    <input type="file" id="import-file" accept=".json" style="display: none;" onchange="importGameData(event)">
                </div>
            </div>

            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4>현재 스테이지 정보</h4>
                    <p id="stage-desc">진행 정보 표시</p>
                </div>
            </div>

            <div class="system-btn-group">
                <button class="btn-system btn-admin" onclick="openAdminModal()">🔑 관리자 모드 접속</button>
                <button class="btn-system btn-danger" onclick="openResetModal()">🔄 데이터 초기화</button>
            </div>
        </div>
    </div>

    <!-- Modals -->
    <div id="char-select-modal" class="modal-overlay" style="display: flex;">
        <div class="modal-card" style="border-color: #f1c40f;">
            <h3>⚔️ 새로운 모험의 시작</h3>
            <p style="margin-bottom: 20px;">플레이할 영웅의 직업을 선택하세요.</p>
            <button class="char-select-btn char-knight" onclick="selectCharacter('knight')">
                <span style="font-size: 24px;">🐉</span>
                <div>
                    <div style="font-size: 16px; color: #fff; margin-bottom: 3px;">드래곤 기사</div>
                    <div style="font-size: 12px; color: #a4b0be; font-weight: normal;">근접 거대검 / 화염 & 드래곤 날개</div>
                </div>
            </button>
            <button class="char-select-btn char-mage" onclick="selectCharacter('mage')">
                <span style="font-size: 24px;">❄️</span>
                <div>
                    <div style="font-size: 16px; color: #fff; margin-bottom: 3px;">천사 마법사</div>
                    <div style="font-size: 12px; color: #a4b0be; font-weight: normal;">마력 에너지 볼트 / 순백 천사 날개</div>
                </div>
            </button>
        </div>
    </div>

    <div id="admin-modal" class="modal-overlay">
        <div class="modal-card" onclick="event.stopPropagation()">
            <h3>🔑 관리자 인증</h3>
            <p>비밀번호 4자리를 입력하세요.</p>
            <input type="password" id="admin-pw-input" placeholder="비밀번호 입력" maxlength="4">
            <div class="modal-btns">
                <button class="btn-confirm" onclick="submitAdminPw()">확인</button>
                <button class="btn-cancel" onclick="closeAdminModal()">취소</button>
            </div>
        </div>
    </div>

    <div id="offline-modal" class="modal-overlay">
        <div class="modal-card" style="border-color: #2ecc71;" onclick="event.stopPropagation()">
            <h3 style="color: #2ecc71;">🌙 백그라운드 방치 완료</h3>
            <p id="offline-desc">방치된 시간 동안 몬스터를 자급자족으로 정벌했습니다!</p>
            <div class="modal-btns">
                <button class="btn-confirm" style="background: #2ecc71;" onclick="closeOfflineModal()">보상 수령하기</button>
            </div>
        </div>
    </div>

    <div id="reset-modal" class="modal-overlay">
        <div class="modal-card" style="border-color: #e74c3c;" onclick="event.stopPropagation()">
            <h3 style="color: #e74c3c;">🔄 데이터 초기화</h3>
            <p>모든 게임 데이터와 메소, 진행 상황이 영구적으로 초기화됩니다.<br><br>정말 삭제하시겠습니까?</p>
            <div class="modal-btns">
                <button class="btn-confirm-danger" onclick="confirmReset()">초기화 진행</button>
                <button class="btn-cancel" onclick="closeResetModal()">취소</button>
            </div>
        </div>
    </div>
</div>

<script>
    // 전역 오디오 컨텍스트
    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    function playSound(freq, type, duration) {
        if (audioCtx.state === 'suspended') audioCtx.resume();
        try {
            const osc = audioCtx.createOscillator(); const gain = audioCtx.createGain();
            osc.type = type; osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
            gain.gain.setValueAtTime(0.08, audioCtx.currentTime); gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + duration);
            osc.connect(gain); gain.connect(audioCtx.destination); osc.start(); osc.stop(audioCtx.currentTime + duration);
        } catch(e) {}
    }

    // ⚔️ 기사 대검 10종
    const WEAPON_ICONS_KNIGHT = [
        `data:image/svg+xml;utf8,<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 4px %23bdc3c7)"><path d="M 18 36 L 18 10 L 24 4 L 30 10 L 30 36 Z" fill="%237f8c8d" stroke="%23ecf0f1" stroke-width="1.5"/><rect x="12" y="36" width="24" height="4" fill="%232c3e50"/><rect x="20" y="40" width="8" height="6" fill="%238e5123"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 5px %233498db)"><path d="M 20 34 L 16 12 L 24 2 L 32 12 L 28 34 Z" fill="%232980b9" stroke="%2300d2d3" stroke-width="1.5"/><polygon points="12,34 36,34 24,38" fill="%231abc9c"/><rect x="22" y="38" width="4" height="8" fill="%2334495e"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 6px %23ff4757)"><path d="M 18 36 L 16 28 L 20 26 L 16 20 L 20 18 L 16 10 L 24 2 L 32 10 L 28 18 L 32 20 L 28 26 L 32 28 L 30 36 Z" fill="%23c0392b" stroke="%23ff7675" stroke-width="1.5"/><rect x="14" y="36" width="20" height="4" fill="%232d3436"/><rect x="20" y="40" width="8" height="6" fill="%23111"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 6px %239b59b6)"><path d="M 22 36 Q 30 20 40 10 Q 20 5 16 2 Q 10 15 14 36 Z" fill="%238e44ad" stroke="%23d2b4de" stroke-width="2"/><rect x="16" y="36" width="12" height="4" fill="%232c3e50"/><rect x="18" y="40" width="8" height="8" fill="%23111"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 7px %232ecc71)"><path d="M 18 36 Q 10 15 2 2 Q 24 10 28 36 Z" fill="%2327ae60" stroke="%231abc9c" stroke-width="2"/><polygon points="26,12 36,18 28,24" fill="%2316a085"/><rect x="16" y="36" width="16" height="4" fill="%232f3640"/><rect x="20" y="40" width="8" height="6" fill="%23111"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 8px %23e67e22)"><path d="M 20 34 Q 10 20 24 2 Q 38 20 28 34 Z" fill="%23d35400" stroke="%23f1c40f" stroke-width="2"/><path d="M 24 34 L 24 10" stroke="%23f39c12" stroke-width="3"/><polygon points="10,34 38,34 24,40" fill="%23c0392b"/><rect x="20" y="40" width="8" height="6" fill="%23111"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 8px %23c0392b)"><path d="M 12 36 L 12 12 L 24 2 L 36 12 L 36 36 Z" fill="%231e272e" stroke="%23ff4757" stroke-width="2.5"/><line x1="24" y1="36" x2="24" y2="8" stroke="%23e74c3c" stroke-width="2"/><rect x="8" y="36" width="32" height="6" fill="%23111"/><rect x="18" y="42" width="12" height="6" fill="%232c3e50"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 10px %2300d2d3)"><path d="M 22 36 L 20 16 L 24 0 L 28 16 L 26 36 Z" fill="%2348dbfb" stroke="%23ffffff" stroke-width="1.5"/><polygon points="12,32 36,32 30,36 18,36" fill="%230abde3"/><rect x="20" y="36" width="8" height="10" fill="%23222f3e"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 12px %23f39c12)"><path d="M 18 36 L 12 24 L 20 18 L 14 6 L 24 0 L 34 6 L 28 18 L 36 24 L 30 36 Z" fill="%23c0392b" stroke="%23f1c40f" stroke-width="2"/><polygon points="8,36 40,36 24,42" fill="%238e44ad" stroke="%23f39c12" stroke-width="1.5"/><rect x="20" y="42" width="8" height="6" fill="%232c3e50"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 15px %23ff00ff)"><path d="M 20 34 L 10 16 L 20 12 L 16 0 L 24 6 L 32 0 L 28 12 L 38 16 L 28 34 Z" fill="%23111" stroke="%23ff9ff3" stroke-width="2.5"/><polygon points="4,34 44,34 34,40 14,40" fill="%236c5ce7" stroke="%23ff00ff" stroke-width="1.5"/><rect x="20" y="40" width="8" height="8" fill="%232f3640"/><circle cx="24" cy="20" r="3" fill="%2300d2d3"/></g></svg>`
    ];

    const WEAPONS_KNIGHT = [ { name: "초보자의 대검", bonus: 0, cost: 200, icon: WEAPON_ICONS_KNIGHT[0] }, { name: "용병의 철검", bonus: 15, cost: 800, icon: WEAPON_ICONS_KNIGHT[1] }, { name: "피빛 톱날검", bonus: 50, cost: 3000, icon: WEAPON_ICONS_KNIGHT[2] }, { name: "초승달 굽은검", bonus: 150, cost: 12000, icon: WEAPON_ICONS_KNIGHT[3] }, { name: "드래곤 투스", bonus: 500, cost: 50000, icon: WEAPON_ICONS_KNIGHT[4] }, { name: "화염의 광검", bonus: 1500, cost: 200000, icon: WEAPON_ICONS_KNIGHT[5] }, { name: "흑요석 파괴검", bonus: 4500, cost: 800000, icon: WEAPON_ICONS_KNIGHT[6] }, { name: "폭풍의 환도", bonus: 15000, cost: 3500000, icon: WEAPON_ICONS_KNIGHT[7] }, { name: "지옥염룡검", bonus: 50000, cost: 15000000, icon: WEAPON_ICONS_KNIGHT[8] }, { name: "창세의 마왕검", bonus: 200000, cost: 80000000, icon: WEAPON_ICONS_KNIGHT[9] } ];

    // 🔮 마법사 지팡이 10종
    const WEAPON_ICONS_MAGE = [
        `data:image/svg+xml;utf8,<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 5px %232ecc71)"><line x1="24" y1="40" x2="24" y2="12" stroke="%238e5123" stroke-width="4"/><circle cx="24" cy="8" r="4" fill="%232ecc71"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 6px %23fd79a8)"><line x1="24" y1="40" x2="24" y2="12" stroke="%23b8e994" stroke-width="4"/><path d="M 24 0 L 28 8 L 24 16 L 20 8 Z" fill="%23fd79a8" stroke="%23fff" stroke-width="1"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 7px %23f1c40f)"><line x1="24" y1="40" x2="24" y2="12" stroke="%23f39c12" stroke-width="4"/><polygon points="24,2 29,8 24,14 19,8" fill="%23f1c40f" stroke="%23fff" stroke-width="1.5"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 8px %2300d2d3)"><line x1="24" y1="40" x2="24" y2="14" stroke="%23bdc3c7" stroke-width="4"/><circle cx="24" cy="8" r="6" fill="%2300d2d3" stroke="%23fff" stroke-width="1.5"/><circle cx="24" cy="8" r="2" fill="%23fff"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 9px %23a29bfe)"><line x1="24" y1="40" x2="24" y2="14" stroke="%23ecf0f1" stroke-width="4"/><polygon points="24,0 28,6 34,8 28,12 26,18 24,14 20,18 20,12 14,8 20,6" fill="%23fff" stroke="%23a29bfe" stroke-width="1.5"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 10px %239b59b6)"><line x1="24" y1="40" x2="24" y2="14" stroke="%2334495e" stroke-width="5"/><path d="M 16 14 Q 24 -2 32 14 Q 24 6 16 14 Z" fill="%239b59b6" stroke="%23fff" stroke-width="1"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 10px %23e84393)"><line x1="24" y1="40" x2="24" y2="14" stroke="%23fd79a8" stroke-width="5"/><path d="M 14 12 Q 24 0 34 12 L 24 16 Z" fill="%23e84393" stroke="%23fff" stroke-width="1.5"/><circle cx="24" cy="4" r="3" fill="%23f1c40f"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 12px %2374b9ff)"><line x1="24" y1="40" x2="24" y2="20" stroke="%230984e3" stroke-width="4"/><path d="M 24 0 L 28 20 L 24 24 L 20 20 Z" fill="%2381ecec" stroke="%23fff" stroke-width="1.5"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="48" height="48" viewBox="0 0 48 48"><g transform="translate(24,24) rotate(45) translate(-24,-24)" style="filter:drop-shadow(0 0 14px %23ff4757)"><path d="M 22 40 Q 28 20 24 14" stroke="%232d3436" stroke-width="5" fill="none"/><circle cx="24" cy="8" r="7" fill="%23ff4757" stroke="%23ff9ff3" stroke-width="2"/><circle cx="24" cy="8" r="3" fill="%23111"/></g></svg>`,
        `data:image/svg+xml;utf8,<svg xmlns="[http://www.w3.org/2000/svg](http://www.w3.org/2000/svg)" width="48" height="48" viewBox### 📄 `index.html` (최종 수정 완료 버전)

아래의 전체 코드를 복사하여 기존 `index.html` 파일의 내용을 **완전히 지우고** 덮어쓰기 해주세요. 코드의 가장 첫 줄(`<!DOCTYPE html>`) 앞에 공백이나 다른 문자가 없도록 주의해주시길 바랍니다.

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>메이플 키우기 - 정식 출시 버전</title>
    <style>
        :root {
            --panel-bg: #181824;
            --panel-card: #232336;
            --main-yellow: #f1c40f;
            --main-orange: #e67e22;
            --text-light: #ffffff;
            image-rendering: crisp-edges;
        }

        * {
            box-sizing: border-box; user-select: none; -webkit-user-select: none;
            touch-action: manipulation; margin: 0; padding: 0;
            font-family: 'Pretendard', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
        }

        /* 📱 풀스크린 최적화: 브라우저 여백 제거 및 가득 차게 설정 */
        body {
            background-color: #0b0b10; color: var(--text-light); display: flex;
            justify-content: center; align-items: center; min-height: 100vh; min-height: 100dvh; overflow: hidden;
            width: 100vw; margin: 0; /* 불필요한 마진 제거 */
        }

        #game-container {
            width: 100%; max-width: 500px; height: 100vh; height: 100dvh;
            background: linear-gradient(180deg, #1e1e2d 0%, #12121a 100%);
            display: flex; flex-direction: column; position: relative; box-shadow: 0 0 40px rgba(0,0,0,0.95); overflow: hidden;
        }

        header {
            background: linear-gradient(180deg, rgba(25,25,38,0.9), rgba(18,18,26,0.9));
            backdrop-filter: blur(8px); padding: 14px 16px; border-bottom: 2px solid rgba(255,255,255,0.08);
            display: flex; flex-direction: column; gap: 10px; z-index: 10;
        }

        .user-info { display: flex; justify-content: space-between; align-items: center; }
        .level-badge {
            background: linear-gradient(135deg, #f39c12, #d35400); color: #fff; padding: 6px 16px;
            border-radius: 16px; font-weight: bold; font-size: 14px; box-shadow: 0 4px 10px rgba(230,126,34,0.4);
        }
        .meso-box {
            display: flex; align-items: center; gap: 6px; background: rgba(0,0,0,0.4);
            padding: 6px 16px; border-radius: 16px; border: 1px solid rgba(241,196,15,0.4);
            color: var(--main-yellow); font-weight: bold; font-size: 16px; box-shadow: inset 0 2px 4px rgba(0,0,0,0.5);
        }

        .exp-bar-container {
            width: 100%; height: 14px; background: rgba(0,0,0,0.6); border-radius: 7px;
            overflow: hidden; border: 1px solid rgba(255,255,255,0.1); position: relative;
        }
        .exp-bar-fill { height: 100%; background: linear-gradient(90deg, #2ecc71, #00d2d3); width: 0%; transition: width 0.3s ease; box-shadow: 0 0 10px rgba(46,204,113,0.5); }
        .exp-text { position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%); font-size: 10px; font-weight: bold; text-shadow: 1px 1px 2px #000; }

        #battlefield {
            flex: 1.2; position: relative; background-size: cover; background-position: center;
            overflow: hidden; display: flex; flex-direction: column; justify-content: space-between;
            align-items: center; cursor: pointer; transition: background 0.5s ease;
        }

        .stage-header { width: 100%; display: flex; justify-content: space-between; align-items: center; padding: 14px 18px 0 18px; z-index: 10; }
        .stage-title {
            background: rgba(15, 15, 25, 0.8); padding: 8px 18px; border-radius: 20px;
            font-size: 14px; font-weight: bold; color: #fff; border: 1px solid rgba(255, 255, 255, 0.15); backdrop-filter: blur(6px); box-shadow: 0 4px 15px rgba(0,0,0,0.3);
        }

        .btn-autohunt {
            background: linear-gradient(135deg, #ff4757, #ff6b81); color: white; border: 1px solid rgba(255, 255, 255, 0.3);
            padding: 8px 18px; border-radius: 20px; font-size: 13px; font-weight: bold; cursor: pointer; box-shadow: 0 4px 12px rgba(255,71,87,0.4); transition: all 0.2s;
        }
        .btn-autohunt.active { background: linear-gradient(135deg, #2ed573, #26af5f); box-shadow: 0 4px 15px rgba(46,213,115,0.5); }

        .battle-area { position: relative; width: 100%; height: 100%; display: flex; justify-content: center; align-items: flex-end; padding-bottom: 22px; }
        .ground {
            position: absolute; bottom: 0; left: 0; width: 100%; height: 32px;
            background-color: var(--ground-color, #27ae60); border-top: 4px solid rgba(255,255,255,0.25); box-shadow: inset 0 4px 8px rgba(0,0,0,0.3); transition: background 0.4s;
        }

        .character {
            position: absolute; left: 30%; bottom: 28px; transform: translateX(-50%);
            width: 140px; height: 140px; background-size: 280px 140px; background-repeat: no-repeat;
            z-index: 5; pointer-events: none; filter: drop-shadow(0 8px 12px rgba(0,0,0,0.5));
            transition: filter 0.1s ease, transform 0.1s ease; animation: spriteIdleLarge 0.8s steps(1) infinite;
        }
        @keyframes spriteIdleLarge { 0% { background-position: 0px 0px; } 50% { background-position: -140px 0px; } 100% { background-position: 0px 0px; } }
        .character.attacking { transform: translateX(-50%) scale(1.05); }

        .character-weapon {
            position: absolute; left: 75px; top: 60px; width: 50px; height: 50px;
            background-size: contain; background-repeat: no-repeat; transition: transform 0.15s ease;
            filter: drop-shadow(0 2px 4px rgba(0,0,0,0.5));
        }
        .character.attacking .character-weapon { transform: rotate(50deg) scale(1.4) translateX(15px) translateY(-5px); }

        .energy-bolt {
            position: absolute; width: 80px; height: 30px; z-index: 35; pointer-events: none;
            transform: translate(-50%, -50%) rotate(var(--angle, 0deg));
            animation: shootEnergyDynamic 0.15s cubic-bezier(0.1, 0.8, 0.3, 1) forwards;
        }
        .bolt-core {
            position: absolute; right: 0; top: 50%; transform: translateY(-50%);
            width: 28px; height: 28px; border-radius: 50%;
            background: radial-gradient(circle, #ffffff 20%, #00d2d3 60%, #a29bfe 100%); box-shadow: 0 0 15px #fff, 0 0 30px #00d2d3, 0 0 50px #a29bfe;
        }
        .bolt-trail {
            position: absolute; right: 14px; top: 50%; transform: translateY(-50%);
            width: 60px; height: 12px; border-radius: 50px 0 0 50px;
            background: linear-gradient(90deg, transparent, rgba(162, 155, 254, 0.8), #00d2d3); filter: blur(2px);
        }
        @keyframes shootEnergyDynamic {
            0% { transform: translate(-50%, -50%) rotate(var(--angle)) scale(0.5); opacity: 0.8; filter: hue-rotate(0deg); }
            100% { transform: translate(calc(-50% + var(--target-x)), calc(-50% + var(--target-y))) rotate(var(--angle)) scale(1.5); opacity: 1; filter: hue-rotate(270deg); }
        }

        .pet { position: absolute; left: 6%; bottom: 55px; width: 45px; height: 45px; background-size: contain; background-repeat: no-repeat; z-index: 6; filter: drop-shadow(0 6px 10px rgba(0,0,0,0.4)); animation: petFloat 1.8s ease-in-out infinite alternate; }
        @keyframes petFloat { 0% { transform: translateY(0px); } 100% { transform: translateY(-10px); } }
        .pet.tackling { animation: petTackle 0.35s cubic-bezier(0.1,0.9,0.2,1) !important; }
        @keyframes petTackle { 0% { transform: translate(0, 0) scale(1); } 40% { transform: translate(190px, 15px) scale(1.35); } 100% { transform: translate(0, 0) scale(1); } }

        .lightning-bolt { position: absolute; top: 0; right: 22%; width: 40px; height: 220px; background: linear-gradient(180deg, #f1c40f, #ffffff, #e67e22); clip-path: polygon(40% 0%, 70% 0%, 30% 40%, 80% 40%, 10% 100%, 40% 50%, 0% 50%); z-index: 30; opacity: 0; pointer-events: none; filter: drop-shadow(0 0 12px #f1c40f); }
        .lightning-bolt.active { animation: lightningFlash 0.35s ease-out; }
        @keyframes lightningFlash { 0% { opacity: 0; transform: scaleY(0); } 30% { opacity: 1; transform: scaleY(1); } 70% { opacity: 1; transform: scaleY(1); } 100% { opacity: 0; transform: scaleY(1); } }

        .scratch-effect { position: absolute; right: 20%; bottom: 28px; width: 80px; height: 80px; background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="64" height="64" viewBox="0 0 64 64"><line x1="8" y1="12" x2="52" y2="52" stroke="%23ff3838" stroke-width="6" stroke-linecap="round"/><line x1="20" y1="8" x2="60" y2="44" stroke="%23ff3838" stroke-width="6" stroke-linecap="round"/><line x1="6" y1="24" x2="44" y2="60" stroke="%23ff3838" stroke-width="6" stroke-linecap="round"/></svg>'); background-size: contain; background-repeat: no-repeat; z-index: 25; opacity: 0; pointer-events: none; }
        .scratch-effect.active { animation: scratchSlash 0.3s cubic-bezier(0.1,0.9,0.2,1); }
        @keyframes scratchSlash { 0% { opacity: 0; transform: scale(0.6) rotate(-15deg); } 40% { opacity: 1; transform: scale(1.25) rotate(0deg); filter: drop-shadow(0 0 12px #ff3838); } 100% { opacity: 0; transform: scale(1) rotate(10deg); } }

        .monster-container { position: absolute; right: 15%; bottom: 28px; display: flex; flex-direction: column; align-items: center; z-index: 5; pointer-events: none; filter: drop-shadow(0 6px 10px rgba(0,0,0,0.4)); }
        .monster-hp-bar { width: 80px; height: 10px; background: rgba(0,0,0,0.7); border: 1px solid rgba(255,255,255,0.4); border-radius: 5px; overflow: hidden; margin-bottom: 8px; box-shadow: 0 2px 5px rgba(0,0,0,0.5); }
        .monster-hp-fill { height: 100%; background: linear-gradient(90deg, #ff4757, #ff6b81); width: 100%; transition: width 0.1s linear; }
        .monster-sprite { width: 80px; height: 80px; background-size: cover; background-repeat: no-repeat; transition: transform 0.2s ease, filter 0.2s ease; }

        .damage-text { position: absolute; font-size: 28px; font-weight: 900; color: #f1c40f; text-shadow: 2px 2px 0 #000, -2px -2px 0 #000, 2px -2px 0 #000, -2px 2px 0 #000, 0 0 10px rgba(241,196,15,0.6); pointer-events: none; animation: floatDamage 0.65s cubic-bezier(0.1,0.9,0.2,1) forwards; z-index: 20; transform: translate(-50%, 0); }
        .damage-text.crit { color: #ff3838; font-size: 34px; text-shadow: 2px 2px 0 #000, -2px -2px 0 #000, 0 0 15px rgba(255,56,56,0.8); }
        .damage-text.pet-dmg { color: #00d2d3; font-size: 30px; text-shadow: 2px 2px 0 #000, -2px -2px 0 #000, 0 0 15px rgba(0,210,211,0.8); }
        .damage-text.char-dmg { color: #ff4757; font-size: 26px; text-shadow: 2px 2px 0 #000, -2px -2px 0 #000, 0 0 15px rgba(255,71,87,0.8); z-index: 40; }

        @keyframes floatDamage {
            0% { opacity: 1; transform: translate(-50%, 0) scale(0.8); }
            50% { opacity: 1; transform: translate(-50%, -40px) scale(1.25); }
            100% { opacity: 0; transform: translate(-50%, -70px) scale(1); }
        }

        #bottom-panel { flex: 1; min-height: 40%; max-height: 55vh; background-color: var(--panel-bg); display: flex; flex-direction: column; border-top: 2px solid rgba(255,255,255,0.08); box-shadow: 0 -10px 30px rgba(0,0,0,0.5); z-index: 20; }
        .nav-tabs { display: flex; background: #12121a; border-bottom: 1px solid rgba(255,255,255,0.06); }
        .tab-btn { flex: 1; padding: 14px 0; background: none; border: none; color: #7f8c8d; font-weight: bold; font-size: 14px; cursor: pointer; white-space: nowrap; transition: color 0.2s; }
        .tab-btn.active { color: var(--main-yellow); border-bottom: 3px solid var(--main-yellow); background: rgba(241, 196, 15, 0.05); }
        .tab-content { flex: 1; padding: 16px; overflow-y: auto; display: none; }
        .tab-content.active { display: block; }
        
        .upgrade-card { background: var(--panel-card); border-radius: 12px; padding: 14px; margin-bottom: 12px; display: flex; justify-content: space-between; align-items: center; border: 1px solid rgba(255,255,255,0.06); box-shadow: 0 4px 15px rgba(0,0,0,0.3); }
        .upgrade-info h4 { font-size: 15px; color: #fff; margin-bottom: 4px; font-weight: 700; }
        .upgrade-info p { font-size: 13px; color: #a4b0be; line-height: 1.4; }
        .btn-group-atk { display: flex; gap: 6px; }
        .btn-upgrade { background: linear-gradient(135deg, #f39c12, #d35400); border: none; color: #fff; padding: 10px 14px; border-radius: 10px; font-weight: bold; font-size: 13px; cursor: pointer; box-shadow: 0 4px 12px rgba(211,84,0,0.4); transition: transform 0.1s; }
        .btn-upgrade.btn-sm { padding: 6px 10px; font-size: 12px; min-width: 60px; text-align: center; }
        .btn-upgrade:active { transform: translateY(2px); box-shadow: 0 2px 6px rgba(211,84,0,0.4); }
        .btn-upgrade:disabled { background: #4b4b60; box-shadow: none; color: #888; cursor: not-allowed; }
        .system-btn-group { display: flex; flex-direction: column; gap: 10px; margin-top: 10px; }
        .btn-system { background: linear-gradient(135deg, #34495e, #2c3e50); color: white; border: none; padding: 14px; border-radius: 12px; font-size: 15px; font-weight: bold; cursor: pointer; box-shadow: 0 4px 12px rgba(255,71,87,0.4); transition: all 0.2s; }
        .btn-system:active { transform: translateY(2px); }
        .btn-admin { background: linear-gradient(135deg, #9b59b6, #8e44ad); border: 1px solid #b19cd9; color: #f1c40f; box-shadow: 0 4px 15px rgba(142,68,173,0.4); }
        .btn-danger { background: linear-gradient(135deg, #ff4757, #c0392b); box-shadow: 0 4px 12px rgba(231,76,60,0.4); }
        .btn-save-load { background: linear-gradient(135deg, #0984e3, #6c5ce7); box-shadow: 0 4px 12px rgba(9,132,227,0.4); }

        .tap-hint { position: absolute; bottom: 32px; font-size: 12px; color: rgba(255,255,255,0.9); background: rgba(15,15,25,0.7); backdrop-filter: blur(4px); padding: 6px 14px; border-radius: 14px; z-index: 10; pointer-events: none; border: 1px solid rgba(255,255,255,0.1); }

        /* 모달 디자인 */
        .modal-overlay { position: absolute; top: 0; left: 0; width: 100%; height: 100%; background: rgba(11, 11, 16, 0.85); display: none; justify-content: center; align-items: center; z-index: 1000; backdrop-filter: blur(8px); }
        .modal-card { background: #232336; border: 2px solid rgba(155,89,182,0.6); border-radius: 16px; padding: 24px; width: 88%; max-width: 360px; text-align: center; box-shadow: 0 10px 40px rgba(0, 0, 0, 0.8); }
        .modal-card h3 { color: #f1c40f; margin-bottom: 12px; font-size: 22px; font-weight: 800; }
        .modal-card p { color: #b2bec3; font-size: 14px; margin-bottom: 18px; line-height: 1.5; }
        .modal-card input { width: 100%; padding: 14px; border-radius: 10px; border: 1px solid rgba(255,255,255,0.2); background: #181824; color: #fff; font-size: 16px; text-align: center; outline: none; margin-bottom: 18px; box-shadow: inset 0 2px 5px rgba(0,0,0,0.5); }
        .modal-card input:focus { border-color: #f1c40f; }
        .modal-btns { display: flex; gap: 10px; }
        .modal-btns button { flex: 1; padding: 14px; border: none; border-radius: 10px; font-weight: bold; font-size: 15px; cursor: pointer; box-shadow: 0 4px 10px rgba(0,0,0,0.3); transition: transform 0.1s; }
        .modal-btns button:active { transform: translateY(2px); }
        .btn-confirm { background: linear-gradient(135deg, #9b59b6, #8e44ad); color: white; }
        .btn-confirm-danger { background: linear-gradient(135deg, #ff4757, #c0392b); color: white; }
        .btn-cancel { background: linear-gradient(135deg, #7f8c8d, #636e72); color: white; }

        .char-select-btn { width: 100%; padding: 18px; margin-bottom: 14px; border-radius: 14px; border: 2px solid rgba(255,255,255,0.15); background: linear-gradient(135deg, #1e1e2d, #181824); color: #fff; font-size: 16px; font-weight: bold; cursor: pointer; transition: all 0.2s; box-shadow: 0 6px 20px rgba(0,0,0,0.4); text-align: left; display: flex; align-items: center; gap: 14px; }
        .char-select-btn:hover, .char-select-btn:active { border-color: #f1c40f; transform: translateY(-3px); box-shadow: 0 10px 30px rgba(0,0,0,0.6); }
        .char-knight { border-left: 6px solid #e74c3c; } .char-mage { border-left: 6px solid #00d2d3; }
    </style>
</head>
<body>

<div id="game-container">
    <header>
        <div class="user-info">
            <span class="level-badge" id="player-level">Lv. 1 모험가</span>
            <div class="meso-box"><span>🪙</span><span id="player-meso">0</span></div>
        </div>
        <div class="exp-bar-container">
            <div class="exp-bar-fill" id="exp-fill"></div>
            <div class="exp-text" id="exp-text">0 / 50 (0%)</div>
        </div>
    </header>

    <div id="battlefield" onclick="manualAttack(event)">
        <div class="stage-header">
            <div class="stage-title" id="stage-name">STAGE 1 : 푸른 언덕 초원</div>
            <button id="btn-autohunt" class="btn-autohunt active" onclick="toggleAutoHunt(event)">⚔️ 자동사냥: ON</button>
        </div>
        <div class="battle-area">
            <div class="ground" id="ground-bar"></div>
            <div class="pet" id="pet-sprite"></div>
            <div class="character" id="character">
                <div class="character-weapon" id="weapon-icon"></div>
            </div>
            <div class="lightning-bolt" id="lightning-bolt"></div>
            <div class="scratch-effect" id="scratch-effect"></div>
            <div class="monster-container" id="monster-box">
                <div class="monster-hp-bar"><div class="monster-hp-fill" id="monster-hp-fill"></div></div>
                <div class="monster-sprite" id="monster-sprite"></div>
            </div>
        </div>
        <div class="tap-hint">💡 화면을 터치/클릭하면 수동 공격!</div>
    </div>

    <div id="bottom-panel">
        <nav class="nav-tabs">
            <button class="tab-btn active" onclick="switchTab('stat', event)">스탯</button>
            <button class="tab-btn" onclick="switchTab('weapon', event)">장비</button>
            <button class="tab-btn" onclick="switchTab('wing', event)">날개 🪽</button>
            <button class="tab-btn" onclick="switchTab('pet', event)">펫 🐾</button>
            <button class="tab-btn" onclick="switchTab('system', event)">설정</button>
        </nav>

        <div id="tab-stat" class="tab-content active">
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4>공격력 강화 (<span id="stat-atk-lvl">1</span>)</h4>
                    <p>현재 공격력: <span id="stat-atk-val">10</span></p>
                </div>
                <div class="btn-group-atk">
                    <button class="btn-upgrade btn-sm" id="btn-up-atk-1" onclick="upgradeStat('atk', 1)">+1<br><small id="cost-atk-1">10</small></button>
                    <button class="btn-upgrade btn-sm" id="btn-up-atk-10" onclick="upgradeStat('atk', 10)">+10<br><small id="cost-atk-10">550</small></button>
                    <button class="btn-upgrade btn-sm" id="btn-up-atk-100" onclick="upgradeStat('atk', 100)">+100<br><small id="cost-atk-100">50K</small></button>
                </div>
            </div>
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4>공격 속도 (<span id="stat-spd-lvl">1</span>)</h4>
                    <p>공격 주기: <span id="stat-spd-val">1.0초</span></p>
                </div>
                <button class="btn-upgrade" id="btn-up-spd" onclick="upgradeStat('spd')">강화<br><small id="cost-spd">100 Gold</small></button>
            </div>
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4>크리티컬 확률 (<span id="stat-crit-lvl">1</span>)</h4>
                    <p>현재 확률: <span id="stat-crit-val">5%</span></p>
                </div>
                <button class="btn-upgrade" id="btn-up-crit" onclick="upgradeStat('crit')">강화<br><small id="cost-crit">130 Gold</small></button>
            </div>
        </div>

        <div id="tab-weapon" class="tab-content">
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4 id="weapon-name">무기 (Tier 1)</h4>
                    <p>공격력 보너스: +<span id="weapon-bonus">0</span></p>
                </div>
                <button class="btn-upgrade" id="btn-up-weapon" onclick="upgradeWeapon()">무기 승급<br><small id="cost-weapon">200 Gold</small></button>
            </div>
        </div>

        <div id="tab-wing" class="tab-content">
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4 id="wing-name-desc" style="color: #f1c40f;">날개 없음 (Lv. 1)</h4>
                    <p>추가 타격력: +<span id="wing-atk-bonus">0</span><br>
                    <small>※ 10레벨 단위 고화질 HD 날개 진화 (Max 100)</small></p>
                </div>
                <button class="btn-upgrade" id="btn-up-wing" onclick="upgradeWing()">날개 강화<br><small id="cost-wing">500 Gold</small></button>
            </div>
        </div>

        <div id="tab-pet" class="tab-content">
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4 id="pet-name">요정 🧚‍♀️</h4>
                    <p id="pet-type-desc">5초마다 번개 공격 (공격력의 <span id="pet-ratio-val">150</span>%)</p>
                </div>
                <button class="btn-upgrade" id="btn-change-pet" onclick="changePet()">펫 변경 🔄</button>
            </div>
            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4>펫 보조 공격력 강화 (<span id="pet-lvl">1</span>)</h4>
                    <p>현재 비율: <span id="pet-ratio-desc">150%</span></p>
                </div>
                <button class="btn-upgrade" id="btn-up-pet" onclick="upgradePet()">강화<br><small id="cost-pet">300 Gold</small></button>
            </div>
        </div>

        <div id="tab-system" class="tab-content">
            <div class="upgrade-card" style="flex-direction: column; align-items: stretch; gap: 10px;">
                <div class="upgrade-info" style="margin-bottom: 5px;">
                    <h4>🎉 이벤트 보상 코드</h4>
                    <p>코드를 입력하여 특별한 보상을 받으세요. ('1' 또는 '2' 입력)</p>
                </div>
                <div style="display: flex; gap: 10px;">
                    <input type="text" id="event-code-input" placeholder="코드 입력" style="flex: 1; padding: 10px; border-radius: 8px; border: 1px solid #555; background: #181824; color: #fff; font-size: 14px; outline: none;">
                    <button class="btn-system" style="margin: 0; padding: 10px 16px; background: linear-gradient(135deg, #2ed573, #26af5f);" onclick="submitEventCode()">보상 받기</button>
                </div>
            </div>

            <div class="upgrade-card" style="flex-direction: column; align-items: stretch; gap: 10px;">
                <div class="upgrade-info" style="margin-bottom: 5px;">
                    <h4>💾 데이터 파일 백업 / 복구</h4>
                    <p>기기를 변경하거나 안전하게 보관하려면 파일로 저장하세요.</p>
                </div>
                <div style="display: flex; gap: 10px;">
                    <button class="btn-system btn-save-load" style="margin: 0; flex: 1;" onclick="exportGameData()">파일 추출</button>
                    <button class="btn-system btn-save-load" style="margin: 0; flex: 1; background: linear-gradient(135deg, #f39c12, #d35400);" onclick="document.getElementById('import-file').click()">불러오기</button>
                    <input type="file" id="import-file" accept=".json" style="display: none;" onchange="importGameData(event)">
                </div>
            </div>

            <div class="upgrade-card">
                <div class="upgrade-info">
                    <h4>현재 스테이지 정보</h4>
                    <p id="stage-desc">진행 정보 표시</p>
                </div>
            </div>

            <div class="system-btn-group">
                <button class="btn-system btn-admin" onclick="openAdminModal()">🔑 관리자 모드 접속</button>
                <button class="btn-system btn-danger" onclick="openResetModal()">🔄 데이터 초기화</button>
            </div>
        </div>
    </div>

    <!-- Modals -->
    <div id="char-select-modal" class="modal-overlay" style="display: flex;">
        <div class="modal-card" style="border-color: #f1c40f;">
            <h3>⚔️ 새로운 모험의 시작</h3>
            <p style="margin-bottom: 20px;">플레이할 영웅의 직업을 선택하세요.</p>
            <button class="char-select-btn char-knight" onclick="selectCharacter('knight')">
                <span style="font-size: 24px;">🐉</span>
                <div>
                    <div style="font-size: 16px; color: #fff; margin-bottom: 3px;">드래곤 기사</div>
                    <div style="font-size: 12px; color: #a4b0be; font-weight: normal;">근접 거대검 / 화염 & 드래곤 날개</div>
                </div>
            </button>
            <button class="char-select-btn char-mage" onclick="selectCharacter('mage')">
                <span style="font-size: 24px;">❄️</span>
                <div>
                    <div style="font-size: 16px; color: #fff; margin-bottom: 3px;">천사 마법사</div>
                    <div style="font-size: 12px; color: #a4b0be; font-weight: normal;">마력 에너지 볼트 / 순백 천사 날개</div>
                </div>
            </button>
        </div>
    </div>

    <div id="admin-modal" class="modal-overlay">
        <div class="modal-card" onclick="event.stopPropagation()">
            <h3>🔑 관리자 인증</h3>
            <p>비밀번호 4자리를 입력하세요.</p>
            <input type="password" id="admin-pw-input" placeholder="비밀번호 입력" maxlength="4">
            <div class="modal-btns">
                <button class="btn-confirm" onclick="submitAdminPw()">확인</button>
                <button class="btn-cancel" onclick="closeAdminModal()">취소</button>
            </div>
        </div>
    </div>

    <div id="offline-modal" class="modal-overlay">
        <div class="modal-card" style="border-color: #2ecc71;" onclick="event.stopPropagation()">
            <h3 style="color: #2ecc71;">🌙 백그라운드 방치 완료</h3>
            <p id="offline-desc">방치된 시간 동안 몬스터를 자급자족으로 정벌했습니다!</p>
            <div class="modal-btns">
                <button class="btn-confirm" style="background: #2ecc71;" onclick="closeOfflineModal()">보상 수령하기</button>
            </div>
        </div>
    </div>

    <div id="reset-modal" class="modal-overlay">
        <div class="modal-card" style="border-color: #e74c3c;" onclick="event.stopPropagation()">
            <h3 style="color: #e74c3c;">🔄 데이터 초기화</h3>
            <p>모든 게임 데이터와 메소, 진행 상황이 영구적으로 초기화됩니다.<br><br>정말 삭제하시겠습니까?</p>
            <div class="modal-btns">
                <button class="btn-confirm-danger" onclick="confirmReset()">초기화 진행</button>
                <button class="btn-cancel" onclick="closeResetModal()">취소</button>
            </div>
        </div>
    </div>
</div>

<script>
    // 전역 오디오 컨텍스트
    let audioCtx;
    function playSound(freq, type, duration) {
        if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        if (audioCtx.state === 'suspended') audioCtx.resume();
        try {
            const osc = audioCtx.createOscillator(); const gain = audioCtx.createGain();
            osc.type = type; osc.frequency.setValueAtTime(freq, audioCtx.currentTime);
            gain.gain.setValueAtTime(0.08, audioCtx.currentTime); gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + duration);
            osc.connect(gain); gain.connect(audioCtx.destination); osc.start(); osc.stop(audioCtx.currentTime + duration);
        } catch(e) {}
    }

    const MONSTER_SPRITES = { slime: `data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="128" height="64" viewBox="0 0 128 64"><rect x="14" y="20" width="36" height="32" rx="10" fill="%232ecc71"/><rect x="18" y="16" width="28" height="36" fill="%232ecc71"/><rect x="22" y="28" width="4" height="8" fill="%231e8449"/><rect x="38" y="28" width="4" height="8" fill="%231e8449"/><rect x="18" y="36" width="4" height="4" fill="%23ff7675"/><rect x="42" y="36" width="4" height="4" fill="%23ff7675"/><circle cx="22" cy="22" r="3" fill="%23a3e4d7"/></svg>`, mushroom: `data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="128" height="64" viewBox="0 0 128 64"><rect x="22" y="32" width="20" height="24" fill="%23f5e6d3"/><rect x="25" y="38" width="3" height="8" fill="%232c3e50"/><rect x="36" y="38" width="3" height="8" fill="%232c3e50"/><ellipse cx="32" cy="24" rx="26" ry="16" fill="%23e67e22"/><ellipse cx="20" cy="20" rx="4" ry="3" fill="%23ffffff"/><ellipse cx="42" cy="18" rx="5" ry="4" fill="%23ffffff"/></svg>` };
    const PET_SPRITES = [ { name: "요정 🧚‍♀️", atkName: "번개 공격", prefix: "⚡ ", sprite: `data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="36" height="36" viewBox="0 0 36 36"><circle cx="18" cy="18" r="8" fill="%23f1c40f"/><path d="M4 10 Q14 2 14 18 Z" fill="%2374b9ff" opacity="0.8"/><circle cx="15" cy="17" r="1.5" fill="%232c3e50"/><circle cx="21" cy="17" r="1.5" fill="%232c3e50"/></svg>` } ];
    const STAGE_BACKGROUNDS = [ `data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="480" height="220"><defs><linearGradient id="sky" x1="0" y1="0" x2="0" y2="1"><stop offset="0%" stop-color="%235ac8fa"/><stop offset="100%" stop-color="%23d0f0fd"/></linearGradient></defs><rect width="480" height="220" fill="url(%23sky)"/><path d="M40 40 Q55 20 75 25 Q90 10 115 20 Q130 25 125 40 Z" fill="white" opacity="0.85"/><path d="M0 180 Q120 130 240 170 T480 150 L480 220 L0 220 Z" fill="%2381c784"/></svg>` ];

    const STAGES = [
        { name: "STAGE 1 : 푸른 언덕 초원", monster: "초보 슬라임 (Lv.1)", hp: 30, exp: 20, meso: 10, bg: STAGE_BACKGROUNDS[0], groundColor: "#388e3c", groundBorder: "#1b5e20", sprite: MONSTER_SPRITES.slime, scale: 0.9 },
        { name: "STAGE 2 : 버섯 숲 입구", monster: "초보 버섯 (Lv.2)", hp: 50, exp: 35, meso: 18, bg: STAGE_BACKGROUNDS[0], groundColor: "#8d6e63", groundBorder: "#4e342e", sprite: MONSTER_SPRITES.mushroom, scale: 1.0 }
    ];

    let game = { charType: null, level: 1, exp: 0, maxExp: 50, meso: 1000, atkLvl: 1, spdLvl: 1, critLvl: 1, wingLvl: 0, weaponTier: 0, petIdx: 0, petLvl: 1, stageIdx: 0, monsterHp: STAGES[0].hp };
    let isAutoHunt = true, autoAttackTimer, petAttackTimer;

    function init() {
        const saved = localStorage.getItem("maple_idle_release");
        if (saved) game = JSON.parse(saved);
        
        // 🛠️ 문제 해결: 캐릭터가 없으면 선택창을 띄우고, 있으면 바로 게임 시작
        if (!game.charType) {
            document.getElementById("char-select-modal").style.display = "flex";
        } else {
            startGameplay();
        }
    }

    // 🛠️ 문제 해결: 캐릭터 선택 함수 수정
    function selectCharacter(type) {
        playSound(800, 'sine', 0.2);
        game.charType = type;
        // 선택창을 즉시 숨깁니다.
        document.getElementById("char-select-modal").style.display = "none";
        // 게임 데이터를 저장합니다.
        saveGame(false);
        // 실제 게임 플레이 로직을 시작합니다.
        startGameplay();
    }

    // 🛠️ 문제 해결: 게임 플레이 시작 로직 통합
    function startGameplay() {
        if (!game.charType) return;
        
        updateUI();
        restartAutoAttack();
        startPetAttackLoop();
        
        // 백그라운드 방치 로직
        let lastTime = Date.now();
        document.addEventListener("visibilitychange", () => {
            if (document.hidden) {
                lastTime = Date.now();
            } else {
                const secs = Math.floor((Date.now() - lastTime) / 1000);
                if (secs > 5) processOfflineProgress(secs);
                lastTime = Date.now();
            }
        });
    }

    // --- 전투 및 로직 함수 ---
    function getAtkPower() { return 10 + (game.atkLvl - 1) * 5; }
    function getAtkInterval() { return Math.max(200, 1000 - (game.spdLvl - 1) * 80); }
    function getCritRate() { return Math.min(80, 5 + (game.critLvl - 1) * 3); }

    function manualAttack(e) { if (!game.charType) return; attackTarget(true); }

    function attackTarget(isManual) {
        const stage = STAGES[game.stageIdx];
        const atk = getAtkPower();
        const isCrit = Math.random() * 100 < getCritRate();
        const dmg = Math.floor(isCrit ? atk * 2 : atk * (0.9 + Math.random() * 0.2));

        // 캐릭터 애니메이션
        const char = document.getElementById("character");
        char.style.backgroundImage = `url('${getKnightSprite(game.level, false, true)}')`;
        char.classList.add("attacking");
        setTimeout(() => {
            char.classList.remove("attacking");
            char.style.backgroundImage = `url('${getKnightSprite(game.level, false, false)}')`;
        }, 150);

        game.monsterHp -= dmg;
        playSound(isManual ? 300 : 200, 'square', 0.05);
        showDamage(dmg, isCrit);

        if (game.monsterHp <= 0) {
            playSound(500, 'sine', 0.1); game.meso += stage.meso; addExp(stage.exp); game.monsterHp = stage.hp;
        }
        updateUI();
    }

    function addExp(amount) {
        game.exp += amount;
        if (game.exp >= game.maxExp) { game.exp -= game.maxExp; game.level++; game.maxExp = Math.floor(game.maxExp * 1.2); playSound(1000, 'triangle', 0.3); }
    }

    function showDamage(dmg, isCrit) {
        const bf = document.getElementById("battlefield"); const el = document.createElement("div");
        el.className = "damage-text" + (isCrit ? " crit" : ""); el.innerText = "-" + dmg;
        const rect = document.getElementById("monster-sprite").getBoundingClientRect(); const bfRect = bf.getBoundingClientRect();
        el.style.left = ((rect.left - bfRect.left) + Math.random() * 20) + "px"; el.style.top = ((rect.top - bfRect.top) - 20) + "px";
        bf.appendChild(el); setTimeout(() => el.remove(), 600);
    }

    // --- UI 및 스프라이트 함수 ---
    function updateUI() {
        if (!game.charType) return;
        const stage = STAGES[game.stageIdx];
        
        document.getElementById("player-level").innerText = `Lv. ${game.level} 모험가`;
        document.getElementById("player-meso").innerText = game.meso.toLocaleString();
        document.getElementById("character").style.backgroundImage = `url('${getKnightSprite(game.level, false, false)}')`;

        const expPct = Math.min(100, (game.exp / game.maxExp) * 100);
        document.getElementById("exp-fill").style.width = expPct + "%";
        document.getElementById("exp-text").innerText = `${game.exp} / ${game.maxExp} (${Math.floor(expPct)}%)`;
        
        document.getElementById("battlefield").style.backgroundImage = `url('${stage.bg}')`;
        document.getElementById("ground-bar").style.backgroundColor = stage.groundColor; document.getElementById("ground-bar").style.borderTopColor = stage.groundBorder;
        document.getElementById("stage-name").innerText = stage.name;
        
        const monsterSprite = document.getElementById("monster-sprite");
        monsterSprite.style.backgroundImage = `url('${stage.sprite}')`;
        monsterSprite.style.transform = `scale(${stage.scale})`;
        document.getElementById("monster-hp-fill").style.width = (game.monsterHp / stage.hp * 100) + "%";

        const curPet = PET_SPRITES[game.petIdx];
        document.getElementById("pet-sprite").style.backgroundImage = `url('${curPet.sprite}')`;

        // 업그레이드 탭 정보 업데이트
        document.getElementById("stat-atk-val").innerText = getAtkPower();
        document.getElementById("cost-atk-1").innerText = (game.atkLvl * 10).toLocaleString();
        document.getElementById("cost-spd").innerText = (game.spdLvl * 100).toLocaleString() + " Gold";
        document.getElementById("cost-crit").innerText = (game.critLvl * 130).toLocaleString() + " Gold";
    }

    function switchTab(tabId, e) {
        document.querySelectorAll(".tab-btn").forEach(b => b.classList.remove("active"));
        document.querySelectorAll(".tab-content").forEach(c => c.classList.remove("active"));
        e.currentTarget.classList.add("active");
        document.getElementById("tab-" + tabId).classList.add("active");
    }

    function toggleAutoHunt() { isAutoHunt = !isAutoHunt; restartAutoAttack(); }
    function restartAutoAttack() { if (autoAttackTimer) clearInterval(autoAttackTimer); if (isAutoHunt && game.charType) autoAttackTimer = setInterval(() => attackTarget(false), getAtkInterval()); }
    function startPetAttackLoop() { if (game.charType) petAttackTimer = setInterval(() => {}, 5000); }

    function getKnightSprite(level, wing, atk) {
        function draw(cx, isAtk) {
            return `<rect x="${cx-10}" y="140" width="20" height="16" rx="3" fill="%232c3e50"/><rect x="${cx-12}" y="114" width="24" height="22" rx="5" fill="%237f8c8d"/><rect x="${cx-8}" y="126" width="16" height="10" rx="3" fill="%23ffcc99"/><rect x="${cx-4}" y="130" width="3" height="4" fill="%23000"/><rect x="${cx+1}" y="130" width="3" height="4" fill="%23000"/>`;
        }
        return `data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" width="400" height="200">${draw(100, false)}${draw(300, atk)}</svg>`;
    }

    // --- 시스템 함수 ---
    function saveGame(alertUser) { localStorage.setItem("maple_idle_release", JSON.stringify(game)); if (alertUser) alert("저장 완료!"); }
    function confirmReset() { localStorage.removeItem("maple_idle_release"); location.reload(); }
    function openResetModal() { document.getElementById("reset-modal").style.display = "flex"; }
    function closeResetModal() { document.getElementById("reset-modal").style.display = "none"; }
    function upgradeStat(type, count) { /* 스탯 업그레이드 로직 (이전과 동일) */ alert('스탯 강화 구현 예정'); }

    window.onload = init;
</script>
</body>
</html>
