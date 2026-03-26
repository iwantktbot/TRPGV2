<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>ETERNAL LEGEND: REBORN</title>
    <style>
        :root {
            --bg: #0d1117; --pnl: #161b22; --brd: #30363d; --main: #58a6ff;
            --hp: #f85149; --mp: #388bfd; --exp: #d29922; --gold: #e3b341;
            --common: #8b949e; --rare: #1f6feb; --epic: #8957e5; --legend: #f1e05a;
        }
        * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Pretendard', sans-serif; -webkit-tap-highlight-color: transparent; }
        body { background: var(--bg); color: #c9d1d9; height: 100vh; overflow: hidden; display: flex; justify-content: center; }
        
        #app { width: 100%; max-width: 600px; height: 100%; display: flex; flex-direction: column; position: relative; border-left: 1px solid var(--brd); border-right: 1px solid var(--brd); }

        /* Auth Screen */
        #auth-screen { position: absolute; inset: 0; background: var(--bg); z-index: 1000; display: flex; flex-direction: column; padding: 40px 20px; gap: 12px; overflow-y: auto; }
        .auth-input { background: #090c10; border: 1px solid var(--brd); color: white; padding: 15px; border-radius: 8px; font-size: 16px; }
        .btn-primary { background: var(--main); color: white; border: none; padding: 15px; border-radius: 8px; font-weight: bold; cursor: pointer; transition: 0.2s; }
        .btn-primary:active { transform: scale(0.98); opacity: 0.8; }
        .btn-outline { background: transparent; border: 1px solid var(--main); color: var(--main); padding: 12px; border-radius: 8px; cursor: pointer; }

        /* HUD */
        .hud { padding: 15px; background: var(--pnl); border-bottom: 1px solid var(--brd); }
        .bar-wrap { position: relative; width: 100%; height: 22px; background: #21262d; border-radius: 6px; overflow: hidden; margin-bottom: 8px; border: 1px solid #444; }
        .bar-fill { height: 100%; transition: width 0.4s cubic-bezier(0.18, 0.89, 0.32, 1.28); }
        .bar-text { position: absolute; width: 100%; text-align: center; font-size: 12px; font-weight: bold; line-height: 20px; color: white; text-shadow: 1px 1px 2px black; z-index: 2; }
        
        #promo-banner { display: none; background: linear-gradient(90deg, #f1e05a, #d29922); color: black; text-align: center; padding: 10px; font-weight: 800; cursor: pointer; border-radius: 8px; margin-bottom: 10px; animation: pulse 1.5s infinite; }
        @keyframes pulse { 0% { transform: scale(1); } 50% { transform: scale(1.02); } 100% { transform: scale(1); } }

        /* Battle Area */
        #battle-field { flex: 1; display: flex; flex-direction: column; align-items: center; justify-content: center; position: relative; background: radial-gradient(circle, #1c2128 0%, #0d1117 100%); overflow: hidden; }
        .monster-box { text-align: center; width: 100%; transition: opacity 0.3s; }
        .monster-sprite { font-size: 80px; margin-bottom: 15px; display: block; filter: drop-shadow(0 0 10px rgba(0,0,0,0.5)); }
        .damage-pop { position: absolute; font-weight: 900; font-size: 32px; pointer-events: none; animation: popUp 0.8s ease-out forwards; z-index: 100; text-shadow: 2px 2px 0px black; }
        @keyframes popUp { 0% { transform: translateY(0) scale(0.5); opacity: 0; } 50% { opacity: 1; transform: translateY(-50px) scale(1.2); } 100% { transform: translateY(-120px) scale(1); opacity: 0; } }

        /* Controls */
        .controls { background: var(--pnl); padding: 12px; border-top: 1px solid var(--brd); }
        .btn-grid { display: grid; grid-template-columns: repeat(4, 1fr); gap: 6px; }
        .skill-btn { position: relative; background: #30363d; border: 1px solid var(--brd); border-radius: 10px; padding: 12px 4px; text-align: center; cursor: pointer; height: 85px; display: flex; flex-direction: column; justify-content: center; align-items: center; transition: 0.2s; }
        .skill-btn:active { background: #444; }
        .skill-btn.disabled { opacity: 0.4; filter: grayscale(1); }
        .skill-name { font-size: 11px; font-weight: bold; margin-bottom: 4px; line-height: 1.2; }
        .skill-cost { font-size: 10px; color: #8b949e; }
        .cd-overlay { position: absolute; inset: 0; background: rgba(0,0,0,0.75); border-radius: 10px; display: flex; align-items: center; justify-content: center; color: #ff7b72; font-size: 24px; font-weight: bold; }

        .pot-row { display: grid; grid-template-columns: repeat(3, 1fr); gap: 6px; margin-bottom: 10px; }
        .pot-btn { background: #21262d; border: 1px solid var(--brd); color: white; padding: 8px; border-radius: 6px; font-size: 11px; cursor: pointer; display: flex; flex-direction: column; align-items: center; }

        /* Log */
        #game-log { height: 90px; background: rgba(0,0,0,0.5); overflow-y: auto; padding: 8px; font-size: 12px; color: #8b949e; border-top: 1px solid var(--brd); scroll-behavior: smooth; }

        /* Navigation */
        .nav { display: grid; grid-template-columns: repeat(5, 1fr); background: var(--pnl); border-top: 1px solid var(--brd); }
        .nav-btn { padding: 16px 0; text-align: center; cursor: pointer; font-size: 12px; color: #8b949e; transition: 0.2s; }
        .nav-btn.active { color: var(--main); background: rgba(88, 166, 255, 0.1); }

        /* Modals */
        .modal { display: none; position: absolute; inset: 0; background: rgba(0,0,0,0.95); z-index: 500; flex-direction: column; animation: fadeIn 0.2s; }
        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
        .modal-header { padding: 20px; display: flex; justify-content: space-between; align-items: center; border-bottom: 1px solid var(--brd); }
        .modal-body { flex: 1; overflow-y: auto; padding: 15px; }
        .card { background: #161b22; border: 1px solid var(--brd); padding: 15px; border-radius: 12px; margin-bottom: 10px; display: flex; justify-content: space-between; align-items: center; }

        /* Responsive Fixes */
        @media (max-width: 400px) {
            .skill-name { font-size: 10px; }
            .monster-sprite { font-size: 60px; }
        }

        #death-screen { display: none; position: absolute; inset: 0; background: rgba(139, 0, 0, 0.9); z-index: 2000; flex-direction: column; align-items: center; justify-content: center; text-align: center; }
    </style>
</head>
<body>

<div id="app">
    <div id="auth-screen">
        <h1 style="text-align:center; color:var(--main); font-size: 32px; margin-bottom: 10px;">ETERNAL LEGEND</h1>
        <p style="text-align:center; font-size: 14px; color: #8b949e; margin-bottom: 20px;">서버에 접속하여 모험을 시작하세요.</p>
        <input type="text" id="auth-id" class="auth-input" placeholder="계정 아이디">
        <input type="password" id="auth-pw" class="auth-input" placeholder="비밀번호">
        <input type="text" id="auth-nick" class="auth-input" placeholder="닉네임 (가입 시 필수)">
        <select id="auth-job" class="auth-input">
            <option value="전사">전사 (Warrior)</option>
            <option value="마법사">마법사 (Mage)</option>
            <option value="도적">도적 (Rogue)</option>
            <option value="궁수">궁수 (Archer)</option>
        </select>
        <button class="btn-primary" onclick="engine.handleAuth('login')">로그인</button>
        <button class="btn-outline" onclick="engine.handleAuth('join')">회원가입</button>
    </div>

    <div id="main-hud" class="hud" style="display:none;">
        <div id="promo-banner" onclick="engine.evolveJob()">🎉 전직 가능! 새로운 힘을 깨우세요! (클릭)</div>
        <div style="display:flex; justify-content:space-between; align-items: flex-end; margin-bottom:8px;">
            <div>
                <span id="ui-job" style="color:var(--main); font-weight:bold; font-size: 18px;">-</span>
                <span id="ui-lv" style="margin-left:8px; font-size: 14px; opacity: 0.8;">Lv.1</span>
            </div>
            <span id="ui-gold" style="color:var(--gold); font-weight:bold;">0 GOLD</span>
        </div>
        <div class="bar-wrap">
            <div id="ui-hp-bar" class="bar-fill" style="background:var(--hp); width:100%;"></div>
            <div id="ui-hp-txt" class="bar-text">0 / 0</div>
        </div>
        <div class="bar-wrap">
            <div id="ui-mp-bar" class="bar-fill" style="background:var(--mp); width:100%;"></div>
            <div id="ui-mp-txt" class="bar-text">0 / 0</div>
        </div>
        <div class="bar-wrap" style="height:6px; margin-bottom: 0;">
            <div id="ui-exp-bar" class="bar-fill" style="background:var(--exp); width:0%;"></div>
        </div>
    </div>

    <div id="battle-field">
        <div id="mon-ui" class="monster-box" style="display:none;">
            <h3 id="mon-name" style="margin-bottom: 10px;">몬스터</h3>
            <span id="mon-sprite" class="monster-sprite">👾</span>
            <div class="bar-wrap" style="width:85%; margin: 0 auto;">
                <div id="mon-hp-bar" class="bar-fill" style="background:var(--hp); width:100%;"></div>
                <div id="mon-hp-txt" class="bar-text">0 / 0</div>
            </div>
            <div id="mon-tags" style="margin-top: 8px;"></div>
        </div>
        <div id="area-info" style="position:absolute; top:10px; right:10px; font-size:12px; background:rgba(0,0,0,0.4); padding:4px 8px; border-radius:4px;">
            📍 <span id="ui-area-name">초심자 숲</span>
        </div>
    </div>

    <div id="game-log"></div>

    <div id="action-ui" class="controls" style="display:none;">
        <div class="pot-row">
            <button class="pot-btn" onclick="engine.usePotion('hp')">
                <span>❤️ 힐링포션</span>
                <b id="cnt-hp">0</b>
            </button>
            <button class="pot-btn" onclick="engine.usePotion('mp')">
                <span>💧 마나포션</span>
                <b id="cnt-mp">0</b>
            </button>
            <button class="pot-btn" onclick="engine.usePotion('luck')">
                <span>🍀 럭키포션</span>
                <b id="cnt-luck">0</b>
            </button>
        </div>
        <div class="btn-grid">
            <div class="skill-btn" onclick="engine.turn('atk')">
                <span class="skill-name">기본 공격</span>
                <span class="skill-cost">No Cost</span>
            </div>
            <div id="sk1" class="skill-btn" onclick="engine.turn('s1')">
                <span id="sk1-n" class="skill-name">스킬1</span>
                <span id="sk1-c" class="skill-cost">MP 0</span>
                <div id="sk1-cd" class="cd-overlay" style="display:none;"></div>
            </div>
            <div id="sk2" class="skill-btn" onclick="engine.turn('s2')">
                <span id="sk2-n" class="skill-name">스킬2</span>
                <span id="sk2-c" class="skill-cost">MP 0</span>
                <div id="sk2-cd" class="cd-overlay" style="display:none;"></div>
            </div>
            <div id="sk3" class="skill-btn" onclick="engine.turn('ult')" style="border-color:var(--legend);">
                <span id="sk3-n" class="skill-name">궁극기</span>
                <span id="sk3-c" class="skill-cost">MP 0</span>
                <div id="sk3-cd" class="cd-overlay" style="display:none;"></div>
            </div>
        </div>
    </div>

    <div id="nav-ui" class="nav" style="display:none;">
        <div class="nav-btn" onclick="engine.startBattle()">⚔️ 사냥</div>
        <div class="nav-btn" onclick="engine.openModal('inv')">🎒 가방</div>
        <div class="nav-btn" onclick="engine.openModal('shop')">🛒 상점</div>
        <div class="nav-btn" onclick="engine.openModal('map')">🗺️ 이동</div>
        <div class="nav-btn" onclick="engine.openModal('etc')">⚙️ 기타</div>
    </div>

    <div id="game-modal" class="modal">
        <div class="modal-header">
            <h2 id="modal-title">TITLE</h2>
            <button style="background:none; border:none; color:white; font-size:30px;" onclick="engine.closeModal()">×</button>
        </div>
        <div id="modal-body" class="modal-body"></div>
    </div>

    <div id="death-screen">
        <h1 style="color:white; font-size:48px; margin-bottom: 20px;">YOU DIED</h1>
        <p style="color:#ccc; margin-bottom: 30px;">경험치와 골드의 일부를 잃고 마을로 소환됩니다.</p>
        <button class="btn-primary" style="width: 200px;" onclick="engine.revive()">부활하기</button>
    </div>
</div>

<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/12.11.0/firebase-app.js";
import { getDatabase, ref, set, get, update, push, remove, onValue, query, orderByChild, limitToLast } from "https://www.gstatic.com/firebasejs/12.11.0/firebase-database.js";

const firebaseConfig = {
    apiKey: "AIzaSyCLvEEVCx0mE3aThF8J_5hvGarEPN_jM-s",
    authDomain: "trpg-55953.firebaseapp.com",
    databaseURL: "https://trpg-55953-default-rtdb.firebaseio.com",
    projectId: "trpg-55953",
    storageBucket: "trpg-55953.firebasestorage.app",
    messagingSenderId: "79536714309",
    appId: "1:79536714309:web:8b31e7147f5ba0356ee92c"
};

const app = initializeApp(firebaseConfig);
const db = getDatabase(app);

// --- DB 구조 및 데이터 상수 ---
const JOBS = {
    "전사": ["전사", "기사", "성기사", "드래곤나이트", "군주"],
    "마법사": ["마법사", "위자드", "대마법사", "현자", "대현자"],
    "도적": ["도적", "어쌔신", "섀도어나이트", "로드", "심판자"],
    "궁수": ["궁수", "헌터", "레인저", "보우마스터", "신궁"]
};

const SKILLS = {
    "전사": {
        names: [["강타", "정의의 일격", "성스러운 가르기", "드래곤 슬래시", "군주의 심판"], 
                ["도발", "철벽 방어", "신성한 가호", "용의 비늘", "제국의 권능"],
                ["분노의 소용돌이", "천상의 심판", "멸절의 칼날", "드래곤 메테오", "진·천하통일"]],
        cost: [30, 60, 150], pow: [2.5, 0.8, 8.0]
    },
    "마법사": {
        names: [["매직 애로우", "파이어볼", "익스플로전", "진리의 살무사", "어비스 게이트"],
                ["마나 실드", "프로스트 가드", "차원 왜곡", "마법의 성역", "절대 영역"],
                ["메테오", "절대영도 블리자드", "아포칼립스", "빅뱅", "우주의 종말"]],
        cost: [50, 100, 300], pow: [4.0, 1.2, 12.0]
    },
    "도적": {
        names: [["기습", "암습", "그림자 찌르기", "심장 파괴", "사신의 처형"],
                ["은신", "환영 복제", "살기 방출", "섬광 이동", "무극의 경지"],
                ["난무", "블러디 메스", "심연의 추격", "살신 성인", "멸절"]],
        cost: [25, 50, 180], pow: [3.2, 1.5, 10.0]
    },
    "궁수": {
        names: [["파워 샷", "조준 사격", "멀티 샷", "바람의 화살", "신의 관통"],
                ["속사", "집중", "독수리의 눈", "바람의 가호", "천리안"],
                ["화살비", "피닉스 포스", "스타 폴", "엘프의 성전", "은하수 저격"]],
        cost: [40, 80, 220], pow: [3.5, 1.3, 11.0]
    }
};

const AREAS = [
    { name: "초심자 숲", lv: 1, attr: "grass", mobs: ["🌳 슬라임", "🌳 고블린", "🌳 숲요정"], boss: "👑 대왕 슬라임", icon: "🌲" },
    { name: "황혼의 동굴", lv: 20, attr: "fire", mobs: ["🔥 불박쥐", "🔥 코볼트", "🔥 파이어웜"], boss: "👑 화염 리치", icon: "🌋" },
    { name: "얼음 협곡", lv: 40, attr: "water", mobs: ["❄️ 설인", "❄️ 빙결정령", "❄️ 서리거미"], boss: "👑 고대 프로스트", icon: "🏔️" },
    { name: "심해 유적", lv: 60, attr: "water", mobs: ["🌊 인어", "🌊 크라켄", "🌊 심해아귀"], boss: "👑 해신 네레이드", icon: "🌊" },
    { name: "검은 요새", lv: 85, attr: "fire", mobs: ["🔥 다크나이트", "🔥 가고일", "🔥 지옥견"], boss: "👑 데스나이트", icon: "🏰" },
    { name: "천공의 탑", lv: 110, attr: "grass", mobs: ["☁️ 하피", "☁️ 그리핀", "☁️ 폭풍조"], boss: "👑 스카이로드", icon: "☁️" },
    { name: "용의 안식처", lv: 140, attr: "fire", mobs: ["🐉 와변", "🐉 드레이크", "🐉 용혈병사"], boss: "👑 레드 드래곤", icon: "🐉" },
    { name: "차원 균열", lv: 165, attr: "water", mobs: ["🌀 공허충", "🌀 쉐도우", "🌀 뒤틀린자"], boss: "👑 차원 포식자", icon: "🌀" },
    { name: "신성한 성역", lv: 185, attr: "grass", mobs: ["👼 엔젤", "👼 심판관", "👼 세라핌"], boss: "👑 대천사 가브리엘", icon: "👼" },
    { name: "신들의 황혼", lv: 200, attr: "fire", mobs: ["⚡ 펜리르", "⚡ 이미르", "⚡ 로키"], boss: "👑 오딘", icon: "⚡" }
];

const PREFIXES = ["날카로운", "단단한", "빛나는", "무거운", "고대의", "타락한", "축복받은"];

// --- 메인 게임 엔진 ---
class GameEngine {
    constructor() {
        this.u = null; // 유저 데이터
        this.m = null; // 몬스터 데이터
        this.cd = { s1: 0, s2: 0, ult: 0 };
        this.luckTimer = 0;
        this.isActing = false;
        this.invFilter = '전체';
        this.init();
    }

    init() {
        // 자동 저장 타이머 (30초)
        setInterval(() => this.saveData(), 30000);
        // 럭키 포션 타이머
        setInterval(() => { if(this.luckTimer > 0) this.luckTimer--; }, 1000);
    }

    async handleAuth(mode) {
        const id = document.getElementById('auth-id').value.trim();
        const pw = document.getElementById('auth-pw').value.trim();
        const nick = document.getElementById('auth-nick').value.trim();
        const job = document.getElementById('auth-job').value;

        if (!id || !pw) return alert("아이디와 비밀번호를 입력하세요.");

        const userRef = ref(db, `users/${id}`);
        const snap = await get(userRef);

        if (mode === 'join') {
            if (snap.exists()) return alert("이미 존재하는 아이디입니다.");
            if (!nick) return alert("닉네임을 입력하세요.");
            const newUser = {
                id, pw, nick, jobBase: job, tier: 0, lv: 1, exp: 0, gold: 10000,
                hp: 500, mhp: 500, mp: 200, mmp: 200,
                str: 10, int: 10, dex: 10, sp: 0,
                hpot: 10, mpot: 10, luckp: 3, stone: 5,
                area: 0, inv: [], 
                eq: { weapon: null, armor: null, helm: null, chest: null, leggings: null, boots: null, ring: null }
            };
            await set(userRef, newUser);
            alert("회원가입이 완료되었습니다!");
        } else {
            if (!snap.exists() || snap.val().pw !== pw) return alert("정보가 일치하지 않습니다.");
            this.u = snap.val();
            // NaN 및 누락 데이터 보정
            this.u.inv = this.u.inv || [];
            this.u.eq = this.u.eq || {};
            ['hpot', 'mpot', 'luckp', 'stone', 'gold', 'hp', 'mp'].forEach(k => this.u[k] = Number(this.u[k]) || 0);

            document.getElementById('auth-screen').style.display = 'none';
            document.getElementById('main-hud').style.display = 'block';
            document.getElementById('action-ui').style.display = 'block';
            document.getElementById('nav-ui').style.display = 'grid';
            
            this.log(`${this.u.nick}님, 환영합니다!`);
            this.render();
        }
    }

    render() {
        const u = this.u;
        document.getElementById('ui-job').innerText = JOBS[u.jobBase][u.tier];
        document.getElementById('ui-lv').innerText = `Lv.${u.lv}`;
        document.getElementById('ui-gold').innerText = `${u.gold.toLocaleString()} GOLD`;
        document.getElementById('ui-area-name').innerText = AREAS[u.area].name;

        // Bars
        const hpPct = (u.hp / u.mhp) * 100;
        document.getElementById('ui-hp-bar').style.width = Math.max(0, hpPct) + "%";
        document.getElementById('ui-hp-txt').innerText = `${Math.floor(u.hp)} / ${u.mhp}`;

        const mpPct = (u.mp / u.mmp) * 100;
        document.getElementById('ui-mp-bar').style.width = Math.max(0, mpPct) + "%";
        document.getElementById('ui-mp-txt').innerText = `${Math.floor(u.mp)} / ${u.mmp}`;

        const expPct = (u.exp / (u.lv * 2000)) * 100;
        document.getElementById('ui-exp-bar').style.width = expPct + "%";

        // Potions
        document.getElementById('cnt-hp').innerText = u.hpot;
        document.getElementById('cnt-mp').innerText = u.mpot;
        document.getElementById('cnt-luck').innerText = u.luckp;

        // Skills
        const skData = SKILLS[u.jobBase];
        ['s1', 's2', 'ult'].forEach((k, i) => {
            document.getElementById(`sk${i+1}-n`).innerText = skData.names[i][u.tier];
            document.getElementById(`sk${i+1}-c`).innerText = `MP ${skData.cost[i]}`;
            const cdBox = document.getElementById(`sk${i+1}-cd`);
            if (this.cd[k] > 0) {
                cdBox.style.display = 'flex';
                cdBox.innerText = this.cd[k];
            } else {
                cdBox.style.display = 'none';
            }
        });

        // Evolve Alert
        const nextLvs = [20, 50, 100, 200];
        document.getElementById('promo-banner').style.display = (u.tier < 4 && u.lv >= nextLvs[u.tier]) ? 'block' : 'none';
    }

    calcAtk() {
        let base = this.u.str * 10 + this.u.lv * 5;
        if (this.u.eq.weapon) base += (this.u.eq.weapon.atk + this.u.eq.weapon.enh * 40);
        return Math.floor(base);
    }

    calcDef() {
        let base = this.u.dex * 5 + this.u.lv * 2;
        Object.values(this.u.eq).forEach(it => {
            if (it && it.type !== 'weapon') base += (it.def + it.enh * 20);
        });
        return Math.floor(base);
    }

    startBattle() {
        if (this.m || this.isActing) return;
        const area = AREAS[this.u.area];
        const isBoss = Math.random() < 0.12;
        
        this.m = {
            name: isBoss ? area.boss : area.mobs[Math.floor(Math.random()*3)],
            lv: area.lv + (isBoss ? 10 : Math.floor(Math.random()*5)),
            hp: area.lv * (isBoss ? 35000 : 2500),
            mhp: area.lv * (isBoss ? 35000 : 2500),
            attr: area.attr,
            isBoss: isBoss,
            sprite: isBoss ? "🐉" : area.icon
        };

        document.getElementById('mon-ui').style.display = 'block';
        document.getElementById('mon-name').innerText = `${this.m.name} (Lv.${this.m.lv})`;
        document.getElementById('mon-sprite').innerText = this.m.sprite;
        document.getElementById('mon-tags').innerHTML = `<span style="padding:2px 6px; background:#444; border-radius:4px; font-size:10px;">${this.m.attr.toUpperCase()}</span>`;
        if(isBoss) document.getElementById('mon-tags').innerHTML += ` <span style="padding:2px 6px; background:purple; border-radius:4px; font-size:10px;">BOSS</span>`;
        
        this.updateMonUI();
        this.log(`${this.m.name}이(가) 나타났습니다!`);
    }

    updateMonUI() {
        if (!this.m) return;
        const pct = (this.m.hp / this.m.mhp) * 100;
        document.getElementById('mon-hp-bar').style.width = Math.max(0, pct) + "%";
        document.getElementById('mon-hp-txt').innerText = `${Math.ceil(this.m.hp)} / ${this.m.mhp}`;
    }

    turn(type) {
        if (!this.m || this.m.hp <= 0 || this.isActing) return;
        
        const skData = SKILLS[this.u.jobBase];
        let dmg = 0;
        let cost = 0;

        if (type === 'atk') {
            dmg = this.calcAtk();
            // 기본 공격 시에만 전체 쿨타임 감소
            for (let k in this.cd) if (this.cd[k] > 0) this.cd[k]--;
        } else {
            const idx = type === 's1' ? 0 : (type === 's2' ? 1 : 2);
            cost = skData.cost[idx];
            if (this.cd[type] > 0) return this.log("아직 스킬을 사용할 수 없습니다.");
            if (this.u.mp < cost) return this.log("마나가 부족합니다!");
            
            this.u.mp -= cost;
            dmg = this.calcAtk() * skData.pow[idx];
            this.cd[type] = (type === 'ult' ? 5 : 2) + 1; // 사용 턴 포함 보정
        }

        this.isActing = true;
        
        // 속성 연산 (불>풀>물>불)
        let multi = 1.0;
        const pAttr = (this.u.eq.weapon && this.u.eq.weapon.attr) ? this.u.eq.weapon.attr : 'none';
        if ((pAttr === 'fire' && this.m.attr === 'grass') || 
            (pAttr === 'water' && this.m.attr === 'fire') || 
            (pAttr === 'grass' && this.m.attr === 'water')) multi = 2.0;
        else if ((this.m.attr === 'fire' && pAttr === 'grass') || 
                 (this.m.attr === 'water' && pAttr === 'fire') || 
                 (this.m.attr === 'grass' && pAttr === 'water')) multi = 0.5;

        const finalDmg = Math.floor(dmg * multi * (0.9 + Math.random()*0.2));
        this.m.hp -= finalDmg;
        this.showDmg(finalDmg, false);
        this.updateMonUI();

        if (this.m.hp <= 0) {
            this.m.hp = 0;
            this.updateMonUI();
            setTimeout(() => this.win(), 500);
            return;
        }

        // 적 반격
        setTimeout(() => {
            const mAtk = this.m.lv * 60;
            const mDmg = Math.max(1, Math.floor(mAtk - this.calcDef() * 0.4));
            this.u.hp -= mDmg;
            this.showDmg(mDmg, true);
            this.render();
            if (this.u.hp <= 0) this.die();
            this.isActing = false;
        }, 400);
    }

    win() {
        this.isActing = false;
        const u = this.u;
        const isBoss = this.m.isBoss;
        
        let xp = this.m.lv * 1200;
        let gold = isBoss ? (this.m.lv * 150) : (this.m.lv * 350); // 보스 골드 하향 밸런싱
        
        if (this.luckTimer > 0) { xp *= 2; gold *= 1.5; }
        
        u.exp += xp;
        u.gold += gold;
        this.log(`승리! ${xp} EXP와 ${gold} GOLD를 얻었습니다.`, "var(--gold)");

        // 아이템 드랍 로직
        const dropCount = isBoss ? (4 + Math.floor(Math.random()*4)) : 1;
        const dropChance = isBoss ? 1.0 : (this.luckTimer > 0 ? 0.7 : 0.35);

        for (let i = 0; i < dropCount; i++) {
            if (Math.random() < dropChance) this.generateItem();
        }

        this.m = null;
        document.getElementById('mon-ui').style.display = 'none';
        this.checkLv();
        this.render();
    }

    generateItem() {
        const types = ["weapon", "armor", "helm", "chest", "leggings", "boots", "ring"];
        const type = types[Math.floor(Math.random()*types.length)];
        const r = Math.random();
        let grade = "common";
        if (r < 0.03) grade = "legend";
        else if (r < 0.12) grade = "epic";
        else if (r < 0.35) grade = "rare";

        const prefix = PREFIXES[Math.floor(Math.random()*PREFIXES.length)];
        const areaName = AREAS[this.u.area].name;
        const power = (this.u.area + 1) * (grade === 'legend' ? 800 : (grade === 'epic' ? 400 : 150));

        const item = {
            id: Date.now() + Math.random(),
            type, grade,
            name: `[${grade.toUpperCase()}] ${prefix} ${areaName}의 ${type.toUpperCase()}`,
            atk: (type === 'weapon' ? power : 0),
            def: (type !== 'weapon' && type !== 'ring' ? Math.floor(power * 0.7) : 0),
            power: power,
            enh: 0,
            attr: "none",
            jobReq: (type === 'weapon' ? this.u.jobBase : "전체")
        };

        this.u.inv.push(item);
        this.log(`${item.name} 획득!`, "lime");

        // 마법서 드랍
        if (Math.random() < 0.1) {
            const attrs = ["fire", "water", "grass"];
            const attr = attrs[Math.floor(Math.random()*3)];
            this.u.inv.push({ type: 'book', name: `마법서: ${attr.toUpperCase()}`, attr: attr, grade: 'rare', power: 0 });
        }
    }

    usePotion(type) {
        const u = this.u;
        if (type === 'hp') {
            if (u.hpot <= 0) return alert("포션이 없습니다.");
            u.hpot--;
            u.hp = Math.min(u.mhp, u.hp + u.mhp * 0.4);
            this.log("HP를 40% 회복했습니다.");
        } else if (type === 'mp') {
            if (u.mpot <= 0) return alert("포션이 없습니다.");
            u.mpot--;
            u.mp = Math.min(u.mmp, u.mp + u.mmp * 0.6);
            this.log("MP를 60% 회복했습니다.");
        } else if (type === 'luck') {
            if (u.luckp <= 0) return alert("럭키포션이 없습니다.");
            u.luckp--;
            this.luckTimer += 180;
            this.log("3분간 행운이 깃듭니다! (드랍률/경험치 상승)");
        }
        this.render();
    }

    openModal(mode) {
        const m = document.getElementById('game-modal');
        const b = document.getElementById('modal-body');
        const t = document.getElementById('modal-title');
        m.style.display = 'flex';
        b.innerHTML = "";

        if (mode === 'inv') {
            t.innerText = "인벤토리 (가방)";
            this.drawInv();
        } else if (mode === 'shop') {
            t.innerText = "마을 상점";
            const items = [
                { n: "❤️ 힐링포션 x10", p: 4500, type: 'hp' },
                { n: "💧 마나포션 x10", p: 4500, type: 'mp' },
                { n: "🍀 럭키포션 x1", p: 7500, type: 'luck' },
                { n: "💎 강화석 x1", p: 15000, type: 'stone' }
            ];
            items.forEach(i => {
                b.innerHTML += `
                    <div class="card">
                        <div><b>${i.n}</b><br><small>${i.p} Gold</small></div>
                        <button class="btn-primary" style="padding:8px 15px" onclick="engine.buyShop('${i.type}', ${i.p})">구매</button>
                    </div>`;
            });
        } else if (mode === 'map') {
            t.innerText = "세계 지도 (지역 이동)";
            AREAS.forEach((area, i) => {
                const locked = this.u.lv < area.lv;
                b.innerHTML += `
                    <div class="card" style="opacity:${locked ? 0.5 : 1}">
                        <div>${area.icon} <b>${area.name}</b> (Lv.${area.lv}+)</div>
                        <button class="btn-primary" ${locked ? 'disabled' : ''} onclick="engine.travel(${i})">이동</button>
                    </div>`;
            });
        } else if (mode === 'etc') {
            t.innerText = "시스템 메뉴";
            b.innerHTML = `
                <button class="btn-primary" style="width:100%; margin-bottom:10px;" onclick="engine.openModal('stat')">📊 능력치 강화 (SP: ${this.u.sp})</button>
                <button class="btn-primary" style="width:100%; margin-bottom:10px;" onclick="engine.openModal('forge')">🔨 대장간 (강화/인챈트)</button>
                <button class="btn-primary" style="width:100%; margin-bottom:10px;" onclick="engine.openModal('auc')">⚖️ 실시간 경매장</button>
                <button class="btn-primary" style="width:100%; margin-bottom:10px;" onclick="engine.openModal('rank')">🏆 전서버 랭킹</button>
                <button class="btn-outline" style="width:100%;" onclick="engine.saveData(true)">💾 수동 저장</button>
            `;
        } else if (mode === 'stat') {
            t.innerText = `스탯 분배 (잔여 SP: ${this.u.sp})`;
            ['str', 'int', 'dex'].forEach(s => {
                b.innerHTML += `<div class="card"><span>${s.toUpperCase()}: ${this.u[s]}</span>
                <button class="btn-primary" style="padding:5px 12px" onclick="engine.addStat('${s}')">+</button></div>`;
            });
        } else if (mode === 'forge') {
            t.innerText = "대장간";
            const w = this.u.eq.weapon;
            b.innerHTML = `
                <div style="background:#000; padding:15px; border-radius:10px; margin-bottom:20px; text-align:center;">
                    <h3 style="color:var(--${w?w.grade:'common'})">${w ? w.name + " (+" + w.enh + ")" : "장착된 무기 없음"}</h3>
                    <p style="font-size:12px; margin:5px 0;">보유 강화석: ${this.u.stone}개</p>
                    <button class="btn-primary" style="width:100%" onclick="engine.doEnhance()">장착 무기 강화하기</button>
                    <small style="color:#ff7b72; display:${w && w.enh>=5?'block':'none'}">⚠️ +5강부터는 파괴 확률이 존재합니다!</small>
                </div>
                <h4>마법서 부여 (인챈트)</h4>
                <div id="book-list"></div>
            `;
            this.u.inv.forEach((item, idx) => {
                if (item.type === 'book') {
                    b.querySelector('#book-list').innerHTML += `<div class="card"><span>${item.name}</span><button class="btn-outline" onclick="engine.doEnchant(${idx})">부여</button></div>`;
                }
            });
        } else if (mode === 'auc') {
            t.innerText = "실시간 경매장";
            onValue(ref(db, 'auc'), snap => {
                b.innerHTML = "";
                snap.forEach(child => {
                    const data = child.val();
                    b.innerHTML += `
                        <div class="card">
                            <div><span style="color:var(--${data.item.grade})">${data.item.name}</span><br><small>판매자: ${data.seller}</small></div>
                            <button class="btn-primary" onclick="engine.buyAuc('${child.key}', ${data.price})">${data.price}G 구매</button>
                        </div>`;
                });
            });
        } else if (mode === 'rank') {
            t.innerText = "TOP 10 랭커";
            const q = query(ref(db, 'users'), orderByChild('lv'), limitToLast(10));
            get(q).then(snap => {
                let arr = [];
                snap.forEach(c => arr.push(c.val()));
                arr.reverse().forEach((user, i) => {
                    b.innerHTML += `<div class="card"><span>#${i+1} <b>${user.nick}</b></span><span>Lv.${user.lv}</span></div>`;
                });
            });
        }
    }

    drawInv() {
        const b = document.getElementById('modal-body');
        let html = `<div style="display:flex; gap:5px; overflow-x:auto; margin-bottom:15px; padding-bottom:5px;">`;
        ['전체', 'weapon', 'armor', 'helm', 'chest', 'leggings', 'boots', 'ring'].forEach(f => {
            html += `<button class="btn-outline ${this.invFilter===f?'active':''}" style="font-size:11px; white-space:nowrap; padding:6px 12px;" onclick="engine.setFilter('${f}')">${f.toUpperCase()}</button>`;
        });
        html += `</div><div style="display:grid; grid-template-columns:1fr 1fr; gap:6px; margin-bottom:15px;">`;
        
        // 장착 장비 현황
        const slots = ["weapon", "armor", "helm", "chest", "leggings", "boots", "ring"];
        slots.forEach(s => {
            const it = this.u.eq[s];
            html += `<div style="background:#090c10; border:1px solid #333; padding:10px; border-radius:8px; font-size:11px; text-align:center;">
                <div style="color:#555; font-size:9px; margin-bottom:4px;">${s.toUpperCase()}</div>
                <div style="color:var(--${it?it.grade:'common'}); font-weight:bold;">${it ? it.name : '비어있음'}</div>
                ${it ? `<button style="margin-top:6px; font-size:10px; border:none; background:#444; color:white; padding:2px 8px; border-radius:4px;" onclick="engine.unequip('${s}')">해제</button>` : ''}
            </div>`;
        });
        html += `</div>`;

        // 아이템 목록
        const list = this.u.inv.filter(v => this.invFilter === '전체' || v.type === this.invFilter);
        list.forEach((item, i) => {
            const realIdx = this.u.inv.indexOf(item);
            html += `<div class="card">
                <div><span style="color:var(--${item.grade})">${item.name}</span><br><small>전투력: ${item.power}</small></div>
                <div style="display:flex; flex-direction:column; gap:4px;">
                    <button class="btn-primary" style="padding:4px 8px; font-size:11px;" onclick="engine.equip(${realIdx})">장착</button>
                    <button class="btn-outline" style="padding:4px 8px; font-size:11px; border-color:#f85149; color:#f85149;" onclick="engine.sellItem(${realIdx})">판매</button>
                    <button class="btn-primary" style="padding:4px 8px; font-size:11px; background:var(--gold); color:black;" onclick="engine.listAuc(${realIdx})">경매</button>
                </div>
            </div>`;
        });
        b.innerHTML = html;
    }

    setFilter(f) { this.invFilter = f; this.drawInv(); }

    equip(idx) {
        const item = this.u.inv[idx];
        if (item.type === 'book') return;
        if (item.jobReq !== "전체" && item.jobReq !== this.u.jobBase) return alert("자신의 직업 무기가 아닙니다.");
        
        const old = this.u.eq[item.type];
        if (old) this.u.inv.push(old);
        
        this.u.eq[item.type] = item;
        this.u.inv.splice(idx, 1);
        this.render(); this.drawInv();
    }

    unequip(slot) {
        this.u.inv.push(this.u.eq[slot]);
        this.u.eq[slot] = null;
        this.render(); this.drawInv();
    }

    sellItem(idx) {
        const item = this.u.inv[idx];
        this.u.gold += Math.floor(item.power * 2.5);
        this.u.inv.splice(idx, 1);
        this.render(); this.drawInv();
    }

    buyShop(type, pr) {
        if (this.u.gold < pr) return alert("골드가 부족합니다.");
        this.u.gold -= pr;
        if (type === 'hp') this.u.hpot += 10;
        else if (type === 'mp') this.u.mpot += 10;
        else if (type === 'luck') this.u.luckp += 1;
        else if (type === 'stone') this.u.stone += 1;
        this.render(); this.openModal('shop');
    }

    doEnhance() {
        const w = this.u.eq.weapon;
        if (!w) return alert("무기를 먼저 장착하세요.");
        const cost = (w.enh + 1) * 8000;
        if (this.u.gold < cost || this.u.stone < 1) return alert("재료나 골드가 부족합니다.");
        
        this.u.gold -= cost;
        this.u.stone--;
        
        let succ = w.enh < 5 ? 75 : 35;
        if (Math.random()*100 < succ) {
            w.enh++;
            this.log(`강화 성공! (+${w.enh}) 무기가 더 강력해졌습니다.`);
        } else {
            if (w.enh >= 5 && Math.random() < 0.25) {
                this.u.eq.weapon = null;
                this.log("⚠️ 강화 실패로 인해 무기가 파괴되었습니다!", "red");
            } else {
                this.log("강화에 실패했습니다.");
            }
        }
        this.render(); this.openModal('forge');
    }

    doEnchant(idx) {
        const book = this.u.inv[idx];
        if (!this.u.eq.weapon) return alert("속성을 부여할 무기를 장착하세요.");
        this.u.eq.weapon.attr = book.attr;
        this.u.eq.weapon.name = `(${book.attr.toUpperCase()}) ` + this.u.eq.weapon.name;
        this.u.inv.splice(idx, 1);
        this.log(`무기에 ${book.attr} 속성이 깃들었습니다!`);
        this.openModal('forge');
    }

    travel(idx) {
        this.u.area = idx;
        this.m = null;
        document.getElementById('mon-ui').style.display = 'none';
        this.log(`${AREAS[idx].name}(으)로 이동했습니다.`);
        this.closeModal();
        this.render();
    }

    evolveJob() {
        const u = this.u;
        const lvs = [20, 50, 100, 200];
        if (u.tier >= 4 || u.lv < lvs[u.tier]) return;
        
        u.tier++;
        u.str += 100; u.int += 100; u.dex += 100;
        u.mhp += 5000; u.mmp += 2000;
        u.hp = u.mhp; u.mp = u.mmp;
        this.log(`전직에 성공했습니다! [${JOBS[u.jobBase][u.tier]}]의 길을 걷습니다.`, "gold");
        this.render();
    }

    addStat(s) {
        if (this.u.sp <= 0) return;
        this.u.sp--;
        this.u[s] += 10;
        if (s === 'str') this.u.mhp += 300;
        if (s === 'int') this.u.mmp += 150;
        this.render(); this.openModal('stat');
    }

    listAuc(idx) {
        const price = prompt("경매 판매 희망 가격(GOLD)을 숫자로 입력하세요.");
        if (!price || isNaN(price)) return;
        push(ref(db, 'auc'), {
            seller: this.u.nick,
            item: this.u.inv[idx],
            price: parseInt(price)
        });
        this.u.inv.splice(idx, 1);
        this.drawInv();
    }

    async buyAuc(key, pr) {
        if (this.u.gold < pr) return alert("골드가 부족합니다.");
        const snap = await get(ref(db, `auc/${key}`));
        if (!snap.exists()) return alert("이미 팔린 물건입니다.");
        
        this.u.gold -= pr;
        this.u.inv.push(snap.val().item);
        await remove(ref(db, `auc/${key}`));
        this.render(); this.openModal('auc');
    }

    checkLv() {
        const next = this.u.lv * 2000;
        if (this.u.exp >= next) {
            this.u.lv++;
            this.u.exp = 0;
            this.u.sp += 15;
            this.u.mhp += 500;
            this.u.mmp += 200;
            this.u.hp = this.u.mhp;
            this.u.mp = this.u.mmp;
            this.log(`LEVEL UP! (Lv.${this.u.lv})`, "cyan");
            this.checkLv();
        }
    }

    die() {
        this.u.hp = 0;
        document.getElementById('death-screen').style.display = 'flex';
    }

    revive() {
        this.u.hp = this.u.mhp;
        this.u.gold = Math.floor(this.u.gold * 0.9);
        this.u.exp = Math.floor(this.u.exp * 0.8);
        this.u.area = 0; // 시작 마을로 귀환
        document.getElementById('death-screen').style.display = 'none';
        this.render();
    }

    showDmg(val, isMob) {
        const b = document.getElementById('battle-field');
        const e = document.createElement('div');
        e.className = 'damage-pop';
        e.innerText = val;
        e.style.color = isMob ? 'var(--hp)' : 'var(--legend)';
        e.style.left = (40 + Math.random()*20) + "%";
        e.style.top = "40%";
        b.appendChild(e);
        setTimeout(() => e.remove(), 800);
    }

    log(msg, color) {
        const box = document.getElementById('game-log');
        const div = document.createElement('div');
        div.style.color = color || "#8b949e";
        div.style.marginBottom = "4px";
        div.innerText = `[LOG] ${msg}`;
        box.prepend(div);
        if (box.children.length > 30) box.lastChild.remove();
    }

    saveData(manual = false) {
        if (!this.u) return;
        set(ref(db, `users/${this.u.id}`), this.u);
        if (manual) this.log("서버에 데이터가 안전하게 보관되었습니다.", "lime");
    }

    closeModal() { document.getElementById('game-modal').style.display = 'none'; }
}

const engine = new GameEngine();
window.engine = engine; // 전역 바인딩
</script>
</body>
</html>
