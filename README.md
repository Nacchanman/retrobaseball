<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8" />
  <meta
    name="viewport"
    content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no, viewport-fit=cover"
  />
  <title>RETRO BASEBALL BOARD '86</title>
  <style>
    :root{
      --bg:#0f380f;
      --bg2:#306230;
      --bg3:#8bac0f;
      --bg4:#9bbc0f;
      --ink:#081820;
      --white:#e0f8d0;
      --danger:#ff6b6b;
      --accent:#ffd166;
      --dirt:#9c6b30;
      --chalk:#f4f6d2;
      --grass:#2e7d32;
      --slotHit:#7ee081;
      --slotOut:#ff8a80;
      --slotHr:#ffd166;
      --slot2:#9ad8ff;
      --slot3:#d7b6ff;
    }

    *{
      box-sizing:border-box;
      -webkit-tap-highlight-color:transparent;
      touch-action:manipulation;
    }

    html, body{
      margin:0;
      padding:0;
      background:var(--ink);
      color:var(--white);
      font-family:"Courier New", monospace;
      min-height:100%;
      overscroll-behavior:none;
    }

    body{
      display:flex;
      justify-content:center;
      align-items:flex-start;
      padding:10px;
    }

    .frame{
      width:min(100%, 980px);
      border:4px solid var(--white);
      background:var(--bg);
      box-shadow:0 0 0 4px var(--ink), 0 0 24px rgba(0,0,0,.45);
      padding:10px;
    }

    .title{
      text-align:center;
      font-size:clamp(20px, 5vw, 32px);
      letter-spacing:2px;
      margin-bottom:10px;
      color:var(--accent);
      text-shadow:2px 2px 0 var(--ink);
    }

    .hud{
      display:grid;
      grid-template-columns:repeat(2, 1fr);
      gap:8px;
      margin-bottom:10px;
    }

    .panel{
      border:3px solid var(--white);
      padding:8px;
      background:var(--bg2);
      min-height:58px;
    }

    .panel h3{
      margin:0 0 6px 0;
      font-size:12px;
      color:var(--bg4);
    }

    .panel .value{
      font-size:clamp(18px, 4.5vw, 22px);
      color:var(--white);
    }

    .game-area{
      display:grid;
      grid-template-columns:1fr;
      gap:10px;
    }

    .field-wrap{
      border:3px solid var(--white);
      background:linear-gradient(var(--bg2), var(--bg));
      padding:8px;
    }

    .canvas-box{
      width:100%;
      aspect-ratio:640 / 420;
      background:#1b5e20;
      border:3px solid var(--ink);
      overflow:hidden;
    }

    canvas{
      display:block;
      width:100%;
      height:100%;
      image-rendering:pixelated;
      touch-action:none;
    }

    .message{
      margin-top:10px;
      border:3px solid var(--white);
      background:var(--bg3);
      color:var(--ink);
      padding:10px;
      min-height:56px;
      font-size:clamp(14px, 3.8vw, 18px);
      font-weight:bold;
    }

    .mobile-controls{
      display:grid;
      grid-template-columns:1fr 1fr 1fr 1fr;
      gap:10px;
      margin-top:10px;
    }

    .btn{
      appearance:none;
      border:none;
      border:3px solid var(--white);
      background:var(--bg2);
      color:var(--white);
      font-family:inherit;
      font-weight:bold;
      font-size:16px;
      padding:16px 10px;
      min-height:64px;
      border-radius:0;
      box-shadow: inset -3px -3px 0 rgba(0,0,0,.25);
    }

    .btn:active{
      transform:translateY(1px);
      filter:brightness(1.08);
    }

    .btn-primary{
      background:var(--danger);
      color:#fff;
    }

    .btn-secondary{
      background:var(--accent);
      color:var(--ink);
    }

    .side{
      display:flex;
      flex-direction:column;
      gap:10px;
    }

    .controls, .log{
      border:3px solid var(--white);
      background:var(--bg2);
      padding:10px;
    }

    .controls{
      font-size:13px;
      line-height:1.7;
    }

    .log{
      max-height:220px;
      overflow:auto;
      font-size:13px;
      line-height:1.5;
    }

    .log p{
      margin:0 0 6px;
    }

    .footer{
      margin-top:10px;
      text-align:center;
      font-size:11px;
      color:var(--bg4);
    }

    .blink{
      animation:blink .8s steps(1) infinite;
    }

    @keyframes blink{
      50%{opacity:0;}
    }

    @media (min-width: 860px){
      .hud{
        grid-template-columns:repeat(6, 1fr);
      }
      .game-area{
        grid-template-columns:1fr 290px;
      }
    }
  </style>
</head>
<body>
  <div class="frame">
    <div class="title">RETRO BASEBALL BOARD '86</div>

    <div class="hud">
      <div class="panel">
        <h3>SCORE</h3>
        <div class="value" id="score">0</div>
      </div>
      <div class="panel">
        <h3>INNING</h3>
        <div class="value" id="inning">1</div>
      </div>
      <div class="panel">
        <h3>OUT</h3>
        <div class="value" id="outs">0</div>
      </div>
      <div class="panel">
        <h3>COUNT</h3>
        <div class="value"><span id="balls">0</span>B - <span id="strikes">0</span>S</div>
      </div>
      <div class="panel">
        <h3>MODE</h3>
        <div class="value" id="modeText">READY</div>
      </div>
      <div class="panel">
        <h3>SOUND</h3>
        <div class="value" id="soundText">OFF</div>
      </div>
    </div>

    <div class="game-area">
      <div class="field-wrap">
        <div class="canvas-box">
          <canvas id="gameCanvas" width="640" height="420"></canvas>
        </div>

        <div class="message" id="message">STARTで開始 / SWINGで打つ</div>

        <div class="mobile-controls">
          <button class="btn btn-secondary" id="startBtn">START</button>
          <button class="btn btn-primary" id="swingBtn">SWING</button>
          <button class="btn" id="soundBtn">SOUND</button>
          <button class="btn" id="resetBtn">RESET</button>
        </div>
      </div>

      <div class="side">
        <div class="controls">
          <div><strong>仕様</strong></div>
          <div>・小さいストライクゾーン</div>
          <div>・ゾーン内なら基本的にバットに当たる</div>
          <div>・打球速度は投球速度を継承</div>
          <div>・野球盤ポケットで結果判定</div>
          <div>・BGM / ヒット音 / HR専用音あり</div>
          <div>・SOUNDで音ON/OFF</div>
        </div>

        <div class="log" id="log"></div>
      </div>
    </div>

    <div class="footer">Retro board baseball with BGM and hit sounds</div>
  </div>

  <script>
    const canvas = document.getElementById("gameCanvas");
    const ctx = canvas.getContext("2d");

    const scoreEl = document.getElementById("score");
    const inningEl = document.getElementById("inning");
    const outsEl = document.getElementById("outs");
    const ballsEl = document.getElementById("balls");
    const strikesEl = document.getElementById("strikes");
    const modeTextEl = document.getElementById("modeText");
    const soundTextEl = document.getElementById("soundText");
    const messageEl = document.getElementById("message");
    const logEl = document.getElementById("log");

    const startBtn = document.getElementById("startBtn");
    const swingBtn = document.getElementById("swingBtn");
    const soundBtn = document.getElementById("soundBtn");
    const resetBtn = document.getElementById("resetBtn");

    const W = canvas.width;
    const H = canvas.height;

    const mound = { x: 320, y: 202 };
    const pitcher = { x: mound.x - 8, y: mound.y - 18 };

    const homePlate = { x: 320, y: 332 };

    const batterBoxRight = { x: 368, y: 286, w: 28, h: 52 };
    const batter = { x: 382, y: 286 };

    // 小さめのストライクゾーン
    const zone = { x: 296, y: 228, w: 48, h: 70 };

    const pitchRelease = { x: mound.x, y: mound.y - 8 };
    const batContact = { x: 346, y: 274 };

    const boardBounds = { left: 18, top: 18, right: W - 18, bottom: H - 22 };
    const pocketRadius = 23;

    const pockets = [
      { x: 110, y: 88,  r: pocketRadius, type: "OUT", label: "OUT", color: "#ff8a80" },
      { x: 212, y: 70,  r: pocketRadius, type: "2B",  label: "2B",  color: "#9ad8ff" },
      { x: 320, y: 58,  r: pocketRadius, type: "HR",  label: "HR",  color: "#ffd166" },
      { x: 430, y: 70,  r: pocketRadius, type: "3B",  label: "3B",  color: "#d7b6ff" },
      { x: 530, y: 88,  r: pocketRadius, type: "OUT", label: "OUT", color: "#ff8a80" },
      { x: 82,  y: 165, r: pocketRadius, type: "1B",  label: "1B",  color: "#7ee081" },
      { x: 558, y: 165, r: pocketRadius, type: "1B",  label: "1B",  color: "#7ee081" },
      { x: 130, y: 265, r: pocketRadius, type: "FOUL",label: "F",   color: "#cccccc" },
      { x: 510, y: 265, r: pocketRadius, type: "FOUL",label: "F",   color: "#cccccc" }
    ];

    let gameStarted = false;
    let score = 0;
    let inning = 1;
    let outs = 0;
    let balls = 0;
    let strikes = 0;
    let bases = [false, false, false];

    let pitchActive = false;
    let pitch = null;

    let batterAnim = 0;
    let gameOver = false;

    let boardBall = null;
    let hitPauseTimer = 0;

    let audioCtx = null;
    let bgmInterval = null;
    let soundEnabled = false;
    let bgmStep = 0;

    function log(text) {
      const p = document.createElement("p");
      p.textContent = "▶ " + text;
      logEl.prepend(p);
    }

    function setMessage(text, blink = false) {
      messageEl.textContent = text;
      if (blink) messageEl.classList.add("blink");
      else messageEl.classList.remove("blink");
    }

    function updateHUD() {
      scoreEl.textContent = score;
      inningEl.textContent = inning;
      outsEl.textContent = outs;
      ballsEl.textContent = balls;
      strikesEl.textContent = strikes;

      if (gameOver) modeTextEl.textContent = "END";
      else if (boardBall) modeTextEl.textContent = "BOARD";
      else if (pitchActive) modeTextEl.textContent = "PITCH";
      else modeTextEl.textContent = "READY";

      soundTextEl.textContent = soundEnabled ? "ON" : "OFF";
    }

    function resetCount() {
      balls = 0;
      strikes = 0;
    }

    function resetGame() {
      gameStarted = false;
      score = 0;
      inning = 1;
      outs = 0;
      balls = 0;
      strikes = 0;
      bases = [false, false, false];
      pitchActive = false;
      pitch = null;
      batterAnim = 0;
      gameOver = false;
      boardBall = null;
      hitPauseTimer = 0;
      updateHUD();
      setMessage("STARTで開始 / SWINGで打つ");
      logEl.innerHTML = "";
      log("ようこそ RETRO BASEBALL BOARD '86");
    }

    function ensureAudio() {
      if (!audioCtx) {
        audioCtx = new (window.AudioContext || window.webkitAudioContext)();
      }
      if (audioCtx.state === "suspended") {
        audioCtx.resume();
      }
    }

    function makeBeep(freq, duration, type = "square", volume = 0.03, startOffset = 0) {
      if (!soundEnabled || !audioCtx) return;
      const now = audioCtx.currentTime + startOffset;
      const osc = audioCtx.createOscillator();
      const gain = audioCtx.createGain();

      osc.type = type;
      osc.frequency.setValueAtTime(freq, now);

      gain.gain.setValueAtTime(0.0001, now);
      gain.gain.linearRampToValueAtTime(volume, now + 0.01);
      gain.gain.exponentialRampToValueAtTime(0.0001, now + duration);

      osc.connect(gain);
      gain.connect(audioCtx.destination);

      osc.start(now);
      osc.stop(now + duration + 0.02);
    }

    function playHitSound() {
      if (!soundEnabled || !audioCtx) return;

      const now = audioCtx.currentTime;

      const master = audioCtx.createGain();
      master.gain.setValueAtTime(0.0001, now);
      master.gain.linearRampToValueAtTime(0.18, now + 0.005);
      master.gain.exponentialRampToValueAtTime(0.0001, now + 0.32);
      master.connect(audioCtx.destination);

      // アタック音
      const attackOsc = audioCtx.createOscillator();
      const attackGain = audioCtx.createGain();
      const attackFilter = audioCtx.createBiquadFilter();

      attackOsc.type = "triangle";
      attackOsc.frequency.setValueAtTime(1800, now);
      attackOsc.frequency.exponentialRampToValueAtTime(520, now + 0.045);

      attackFilter.type = "bandpass";
      attackFilter.frequency.setValueAtTime(1400, now);
      attackFilter.Q.setValueAtTime(3, now);

      attackGain.gain.setValueAtTime(0.0001, now);
      attackGain.gain.linearRampToValueAtTime(0.22, now + 0.003);
      attackGain.gain.exponentialRampToValueAtTime(0.0001, now + 0.06);

      attackOsc.connect(attackFilter);
      attackFilter.connect(attackGain);
      attackGain.connect(master);

      attackOsc.start(now);
      attackOsc.stop(now + 0.07);

      // ボディ音
      const bodyOsc1 = audioCtx.createOscillator();
      const bodyOsc2 = audioCtx.createOscillator();
      const bodyGain = audioCtx.createGain();

      bodyOsc1.type = "square";
      bodyOsc2.type = "triangle";

      bodyOsc1.frequency.setValueAtTime(740, now + 0.01);
      bodyOsc2.frequency.setValueAtTime(1110, now + 0.01);

      bodyOsc1.frequency.exponentialRampToValueAtTime(330, now + 0.18);
      bodyOsc2.frequency.exponentialRampToValueAtTime(495, now + 0.16);

      bodyGain.gain.setValueAtTime(0.0001, now + 0.01);
      bodyGain.gain.linearRampToValueAtTime(0.14, now + 0.02);
      bodyGain.gain.exponentialRampToValueAtTime(0.0001, now + 0.22);

      bodyOsc1.connect(bodyGain);
      bodyOsc2.connect(bodyGain);
      bodyGain.connect(master);

      bodyOsc1.start(now + 0.01);
      bodyOsc2.start(now + 0.01);
      bodyOsc1.stop(now + 0.23);
      bodyOsc2.stop(now + 0.20);

      // 余韻
      const tailOsc = audioCtx.createOscillator();
      const tailGain = audioCtx.createGain();

      tailOsc.type = "sine";
      tailOsc.frequency.setValueAtTime(660, now + 0.03);
      tailOsc.frequency.exponentialRampToValueAtTime(440, now + 0.28);

      tailGain.gain.setValueAtTime(0.0001, now + 0.03);
      tailGain.gain.linearRampToValueAtTime(0.05, now + 0.05);
      tailGain.gain.exponentialRampToValueAtTime(0.0001, now + 0.30);

      tailOsc.connect(tailGain);
      tailGain.connect(master);

      tailOsc.start(now + 0.03);
      tailOsc.stop(now + 0.31);
    }

    function playHomeRunSound() {
      if (!soundEnabled || !audioCtx) return;

      const now = audioCtx.currentTime;
      const master = audioCtx.createGain();
      master.gain.setValueAtTime(0.0001, now);
      master.gain.linearRampToValueAtTime(0.22, now + 0.01);
      master.gain.exponentialRampToValueAtTime(0.0001, now + 0.95);
      master.connect(audioCtx.destination);

      // 最初に派手なキラッ
      const spark = audioCtx.createOscillator();
      const sparkGain = audioCtx.createGain();
      spark.type = "triangle";
      spark.frequency.setValueAtTime(1400, now);
      spark.frequency.exponentialRampToValueAtTime(2800, now + 0.06);
      sparkGain.gain.setValueAtTime(0.0001, now);
      sparkGain.gain.linearRampToValueAtTime(0.14, now + 0.01);
      sparkGain.gain.exponentialRampToValueAtTime(0.0001, now + 0.10);
      spark.connect(sparkGain);
      sparkGain.connect(master);
      spark.start(now);
      spark.stop(now + 0.11);

      // ファンファーレ
      const notes = [523.25, 659.25, 783.99, 1046.5];
      notes.forEach((freq, i) => {
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.type = i % 2 === 0 ? "square" : "triangle";
        osc.frequency.setValueAtTime(freq, now + i * 0.09);
        gain.gain.setValueAtTime(0.0001, now + i * 0.09);
        gain.gain.linearRampToValueAtTime(0.10, now + i * 0.09 + 0.01);
        gain.gain.exponentialRampToValueAtTime(0.0001, now + i * 0.09 + 0.22);
        osc.connect(gain);
        gain.connect(master);
        osc.start(now + i * 0.09);
        osc.stop(now + i * 0.09 + 0.24);
      });

      // 低音支え
      [130.81, 196.0, 261.63].forEach((freq, i) => {
        const osc = audioCtx.createOscillator();
        const gain = audioCtx.createGain();
        osc.type = "triangle";
        osc.frequency.setValueAtTime(freq, now + i * 0.18);
        gain.gain.setValueAtTime(0.0001, now + i * 0.18);
        gain.gain.linearRampToValueAtTime(0.055, now + i * 0.18 + 0.02);
        gain.gain.exponentialRampToValueAtTime(0.0001, now + i * 0.18 + 0.30);
        osc.connect(gain);
        gain.connect(master);
        osc.start(now + i * 0.18);
        osc.stop(now + i * 0.18 + 0.32);
      });
    }

    function startBGM() {
      if (!soundEnabled || !audioCtx || bgmInterval) return;

      const melody = [262, 330, 392, 330, 294, 349, 440, 349];
      const bass   = [131, 131, 147, 147, 165, 165, 147, 147];

      bgmInterval = setInterval(() => {
        if (!soundEnabled || !audioCtx) return;
        const step = bgmStep % melody.length;
        makeBeep(melody[step], 0.18, "square", 0.025, 0);
        makeBeep(bass[step], 0.22, "triangle", 0.018, 0);
        if (step % 2 === 0) {
          makeBeep(melody[step] * 2, 0.08, "square", 0.012, 0.09);
        }
        bgmStep++;
      }, 220);
    }

    function stopBGM() {
      if (bgmInterval) {
        clearInterval(bgmInterval);
        bgmInterval = null;
      }
    }

    function toggleSound() {
      ensureAudio();
      soundEnabled = !soundEnabled;
      if (soundEnabled) {
        startBGM();
        makeBeep(440, 0.08, "square", 0.03, 0);
        makeBeep(660, 0.08, "square", 0.02, 0.08);
      } else {
        stopBGM();
      }
      updateHUD();
    }

    function randomPitch() {
      const strikeLike = Math.random() < 0.78;

      let finalX, finalY;
      if (strikeLike) {
        finalX = zone.x + 8 + Math.random() * (zone.w - 16);
        finalY = zone.y + 8 + Math.random() * (zone.h - 16);
      } else {
        const side = Math.random() < 0.5 ? -1 : 1;
        finalX = zone.x + zone.w / 2 + side * (45 + Math.random() * 30);
        finalY = zone.y - 18 + Math.random() * (zone.h + 34);
      }

      const frames = 48;
      const vx = (finalX - pitchRelease.x) / frames;
      const vy = (finalY - pitchRelease.y) / frames;

      return {
        x: pitchRelease.x,
        y: pitchRelease.y,
        vx,
        vy,
        speed: Math.hypot(vx, vy),
        frame: 0,
        maxFrame: frames,
        finalX,
        finalY
      };
    }

    function startPitch() {
      if (!gameStarted || gameOver || pitchActive || boardBall) return;
      pitch = randomPitch();
      pitchActive = true;
      setMessage("投球！");
      updateHUD();
    }

    function scheduleNextPitch(delay = 700) {
      if (gameOver) return;
      setTimeout(() => {
        if (!pitchActive && !boardBall && gameStarted && !gameOver) {
          startPitch();
        }
      }, delay);
    }

    function walkBatter() {
      if (bases[0] && bases[1] && bases[2]) {
        score++;
        log("押し出し四球！1点追加！");
        setMessage("押し出し！");
      } else if (bases[0] && bases[1]) {
        bases[2] = true;
      } else if (bases[0]) {
        bases[1] = true;
      }
      bases[0] = true;
    }

    function advanceRunners(hitBases) {
      let runs = 0;
      const newBases = [false, false, false];

      for (let i = 2; i >= 0; i--) {
        if (bases[i]) {
          const next = i + hitBases;
          if (next >= 3) runs++;
          else newBases[next] = true;
        }
      }

      if (hitBases >= 4) {
        runs++;
      } else {
        newBases[hitBases - 1] = true;
      }

      bases = newBases;
      score += runs;

      if (runs > 0) {
        log(runs + "点入った！");
      }
    }

    function endInning() {
      outs = 0;
      resetCount();
      bases = [false, false, false];
      inning++;
      updateHUD();

      if (inning > 3) {
        gameOver = true;
        setMessage("GAME OVER! 最終得点: " + score, true);
        log("試合終了！スコア: " + score);
        updateHUD();
        return;
      }

      log(inning + "回表！");
      setMessage(inning + "回へ！");
      scheduleNextPitch(1000);
    }

    function addStrike(looking = false) {
      strikes++;
      if (strikes >= 3) {
        outs++;
        log(looking ? "見逃し三振！" : "空振り三振！");
        setMessage("STRIKE OUT!");
        resetCount();
        updateHUD();

        if (outs >= 3) endInning();
        else scheduleNextPitch(850);
      } else {
        log(looking ? "見逃しストライク" : "ストライク");
        setMessage("STRIKE!");
        updateHUD();
        scheduleNextPitch(700);
      }
    }

    function addBall() {
      balls++;
      if (balls >= 4) {
        log("フォアボール！");
        setMessage("WALK!");
        walkBatter();
        resetCount();
        updateHUD();
        scheduleNextPitch(900);
      } else {
        log("ボール");
        setMessage("BALL");
        updateHUD();
        scheduleNextPitch(700);
      }
    }

    function launchBoardBall(angleOffset, inheritedSpeed, centerHit) {
      const baseAngle = -Math.PI / 2;
      const angle = baseAngle + angleOffset;

      const outSpeed = inheritedSpeed;
      const vx = Math.cos(angle) * outSpeed;
      const vy = Math.sin(angle) * outSpeed;

      boardBall = {
        x: batContact.x,
        y: batContact.y,
        vx,
        vy,
        radius: 7,
        active: true,
        centerHit
      };

      playHitSound();
      hitPauseTimer = 3;
      pitchActive = false;
      pitch = null;
      updateHUD();
    }

    function judgeSwing() {
      if (!pitchActive || !pitch || boardBall) return;

      const zoneCenterX = zone.x + zone.w / 2;
      const zoneCenterY = zone.y + zone.h / 2;

      const dx = pitch.x - zoneCenterX;
      const dy = pitch.y - zoneCenterY;
      const dist = Math.hypot(dx, dy);

      const inZone =
        pitch.x >= zone.x &&
        pitch.x <= zone.x + zone.w &&
        pitch.y >= zone.y &&
        pitch.y <= zone.y + zone.h;

      batterAnim = 10;

      if (inZone) {
        const edgeX = Math.abs(dx) / (zone.w / 2);
        const edgeY = Math.abs(dy) / (zone.h / 2);
        const contactQuality = Math.max(0, 1 - ((edgeX + edgeY) * 0.5));
        const centerHit = contactQuality > 0.6;

        const angleOffset =
          (dx / (zone.w / 2)) * 0.55 +
          (Math.random() - 0.5) * (centerHit ? 0.14 : 0.28);

        log(centerHit ? "クリーンヒット！" : "バットに当たった！");
        setMessage(centerHit ? "NICE CONTACT!" : "CONTACT!");
        launchBoardBall(angleOffset, pitch.speed, centerHit);
        return;
      }

      if (dist < 68) {
        strikes++;
        if (strikes >= 3) {
          outs++;
          log("空振り三振！");
          setMessage("STRIKE OUT!");
          resetCount();
          pitchActive = false;
          pitch = null;
          updateHUD();
          if (outs >= 3) endInning();
          else scheduleNextPitch(900);
        } else {
          log("ファウルチップ！");
          setMessage("FOUL TIP!");
          if (strikes > 2) strikes = 2;
          pitchActive = false;
          pitch = null;
          updateHUD();
          scheduleNextPitch(750);
        }
      } else {
        pitchActive = false;
        pitch = null;
        addStrike(false);
      }
    }

    function resolveBoardResult(type) {
      boardBall = null;
      updateHUD();

      if (type === "1B") {
        log("シングルヒット！");
        setMessage("SINGLE!");
        advanceRunners(1);
        resetCount();
        updateHUD();
        scheduleNextPitch(1000);
        return;
      }

      if (type === "2B") {
        log("ツーベースヒット！");
        setMessage("DOUBLE!");
        advanceRunners(2);
        resetCount();
        updateHUD();
        scheduleNextPitch(1000);
        return;
      }

      if (type === "3B") {
        log("スリーベースヒット！");
        setMessage("TRIPLE!");
        advanceRunners(3);
        resetCount();
        updateHUD();
        scheduleNextPitch(1000);
        return;
      }

      if (type === "HR") {
        log("ホームラン！");
        setMessage("HOME RUN!!", true);
        playHomeRunSound();
        advanceRunners(4);
        resetCount();
        updateHUD();
        scheduleNextPitch(1200);
        return;
      }

      if (type === "FOUL") {
        log("ファウル！");
        setMessage("FOUL!");
        if (strikes < 2) strikes++;
        pitchActive = false;
        pitch = null;
        boardBall = null;
        updateHUD();
        scheduleNextPitch(850);
        return;
      }

      if (type === "OUT") {
        outs++;
        log("アウト！");
        setMessage("OUT!");
        resetCount();
        updateHUD();
        if (outs >= 3) endInning();
        else scheduleNextPitch(900);
      }
    }

    function updatePitch() {
      if (!pitchActive || !pitch) return;

      pitch.x += pitch.vx;
      pitch.y += pitch.vy;
      pitch.frame++;

      if (pitch.frame >= pitch.maxFrame) {
        const inZone =
          pitch.finalX >= zone.x &&
          pitch.finalX <= zone.x + zone.w &&
          pitch.finalY >= zone.y &&
          pitch.finalY <= zone.y + zone.h;

        pitchActive = false;
        pitch = null;

        if (inZone) addStrike(true);
        else addBall();
      }
    }

    function updateBoardBall() {
      if (!boardBall) return;

      if (hitPauseTimer > 0) {
        hitPauseTimer--;
        return;
      }

      boardBall.x += boardBall.vx;
      boardBall.y += boardBall.vy;

      // 投球速度を引き継いだ打球。減速は少しだけ
      boardBall.vx *= 0.996;
      boardBall.vy *= 0.996;

      if (boardBall.x < boardBounds.left || boardBall.x > boardBounds.right) {
        boardBall.vx *= -0.95;
        boardBall.x = Math.max(boardBounds.left, Math.min(boardBounds.right, boardBall.x));
      }

      if (boardBall.y < boardBounds.top) {
        boardBall.vy *= -0.95;
        boardBall.y = boardBounds.top;
      }

      if (boardBall.y > boardBounds.bottom) {
        boardBall.vy *= -0.90;
        boardBall.y = boardBounds.bottom;
      }

      for (const pocket of pockets) {
        const dx = boardBall.x - pocket.x;
        const dy = boardBall.y - pocket.y;
        const d = Math.hypot(dx, dy);
        if (d < pocket.r - 2) {
          resolveBoardResult(pocket.type);
          return;
        }
      }

      const speed = Math.hypot(boardBall.vx, boardBall.vy);

      if (speed < 0.45) {
        let nearest = null;
        let nearestD = Infinity;

        for (const pocket of pockets) {
          const dx = boardBall.x - pocket.x;
          const dy = boardBall.y - pocket.y;
          const d = Math.hypot(dx, dy);
          if (d < nearestD) {
            nearestD = d;
            nearest = pocket;
          }
        }

        if (nearest && nearestD < 90) resolveBoardResult(nearest.type);
        else resolveBoardResult("OUT");
      }
    }

    function updateAnimation() {
      if (batterAnim > 0) batterAnim--;
    }

    function drawPixelText(text, x, y, size = 20, color = "#e0f8d0", align = "left") {
      ctx.save();
      ctx.fillStyle = color;
      ctx.font = `bold ${size}px Courier New`;
      ctx.textAlign = align;
      ctx.fillText(text, x, y);
      ctx.restore();
    }

    function drawBase(x, y) {
      ctx.save();
      ctx.translate(x, y);
      ctx.rotate(Math.PI / 4);
      ctx.fillStyle = "#f5f5dc";
      ctx.fillRect(-8, -8, 16, 16);
      ctx.restore();
    }

    function drawMiniRunner(x, y) {
      ctx.fillStyle = "#0d1b2a";
      ctx.fillRect(x - 4, y - 10, 8, 10);
      ctx.fillStyle = "#ffd166";
      ctx.fillRect(x - 3, y - 16, 6, 6);
    }

    function drawBoardPockets() {
      for (const pocket of pockets) {
        ctx.beginPath();
        ctx.fillStyle = pocket.color;
        ctx.arc(pocket.x, pocket.y, pocket.r, 0, Math.PI * 2);
        ctx.fill();

        ctx.beginPath();
        ctx.strokeStyle = "#081820";
        ctx.lineWidth = 4;
        ctx.arc(pocket.x, pocket.y, pocket.r - 2, 0, Math.PI * 2);
        ctx.stroke();

        drawPixelText(pocket.label, pocket.x, pocket.y + 5, 14, "#081820", "center");
      }
    }

    function drawField() {
      ctx.fillStyle = "#2e7d32";
      ctx.fillRect(0, 0, W, H);

      drawBoardPockets();

      ctx.strokeStyle = "rgba(224,248,208,.45)";
      ctx.lineWidth = 2;

      ctx.beginPath();
      ctx.moveTo(homePlate.x, homePlate.y);
      ctx.lineTo(125, 100);
      ctx.stroke();

      ctx.beginPath();
      ctx.moveTo(homePlate.x, homePlate.y);
      ctx.lineTo(515, 100);
      ctx.stroke();

      ctx.beginPath();
      ctx.arc(homePlate.x, homePlate.y, 275, Math.PI * 1.18, Math.PI * 1.82);
      ctx.stroke();

      ctx.beginPath();
      ctx.moveTo(155, 150);
      ctx.quadraticCurveTo(320, 95, 485, 150);
      ctx.stroke();

      ctx.beginPath();
      ctx.moveTo(130, 230);
      ctx.quadraticCurveTo(320, 160, 510, 230);
      ctx.stroke();

      ctx.fillStyle = "#9c6b30";
      ctx.beginPath();
      ctx.moveTo(245, 260);
      ctx.lineTo(320, 185);
      ctx.lineTo(395, 260);
      ctx.lineTo(320, 335);
      ctx.closePath();
      ctx.fill();

      ctx.strokeStyle = "#f4f6d2";
      ctx.lineWidth = 2;
      ctx.beginPath();
      ctx.moveTo(homePlate.x, homePlate.y);
      ctx.lineTo(122, 110);
      ctx.moveTo(homePlate.x, homePlate.y);
      ctx.lineTo(518, 110);
      ctx.stroke();

      drawBase(320, 185);
      drawBase(395, 260);
      drawBase(320, 335);
      drawBase(245, 260);

      ctx.fillStyle = "#b58a50";
      ctx.beginPath();
      ctx.arc(mound.x, mound.y, 24, 0, Math.PI * 2);
      ctx.fill();

      ctx.strokeStyle = "#f4f6d2";
      ctx.lineWidth = 2;
      ctx.strokeRect(244, 286, 28, 52);
      ctx.strokeRect(batterBoxRight.x, batterBoxRight.y, batterBoxRight.w, batterBoxRight.h);

      ctx.fillStyle = "#f5f5dc";
      ctx.beginPath();
      ctx.moveTo(homePlate.x - 9, homePlate.y - 4);
      ctx.lineTo(homePlate.x + 9, homePlate.y - 4);
      ctx.lineTo(homePlate.x + 11, homePlate.y + 5);
      ctx.lineTo(homePlate.x, homePlate.y + 12);
      ctx.lineTo(homePlate.x - 11, homePlate.y + 5);
      ctx.closePath();
      ctx.fill();

      ctx.strokeStyle = "#e0f8d0";
      ctx.lineWidth = 3;
      ctx.strokeRect(zone.x, zone.y, zone.w, zone.h);
    }

    function drawRunners() {
      const positions = [
        [395, 250],
        [315, 180],
        [240, 250]
      ];

      bases.forEach((occupied, i) => {
        if (!occupied) return;
        drawMiniRunner(positions[i][0], positions[i][1]);
      });
    }

    function drawPitcher() {
      ctx.fillStyle = "#0d47a1";
      ctx.fillRect(pitcher.x, pitcher.y, 16, 28);
      ctx.fillStyle = "#ffcc80";
      ctx.fillRect(pitcher.x + 3, pitcher.y - 9, 10, 10);
      ctx.fillStyle = "#ffffff";
      ctx.fillRect(pitcher.x + 16, pitcher.y + 10, 6, 4);
    }

    function drawBatter() {
      const swingOffset = batterAnim > 0 ? Math.min(30, batterAnim * 3) : 0;

      ctx.fillStyle = "#c62828";
      ctx.fillRect(batter.x, batter.y, 16, 30);

      ctx.fillStyle = "#ffcc80";
      ctx.fillRect(batter.x + 3, batter.y - 10, 10, 10);

      ctx.save();
      ctx.translate(batter.x + 7, batter.y + 9);
      ctx.rotate((-135 + swingOffset) * Math.PI / 180);
      ctx.fillStyle = "#d7a86e";
      ctx.fillRect(-30, -2, 40, 5);
      ctx.restore();
    }

    function drawPitchBall() {
      if (!pitchActive || !pitch) return;
      ctx.fillStyle = "#ffffff";
      ctx.beginPath();
      ctx.arc(pitch.x, pitch.y, 6, 0, Math.PI * 2);
      ctx.fill();

      ctx.strokeStyle = "#d32f2f";
      ctx.lineWidth = 1;
      ctx.beginPath();
      ctx.arc(pitch.x, pitch.y, 4, 0.3, 1.4);
      ctx.stroke();
    }

    function drawBoardBall() {
      if (!boardBall) return;
      ctx.fillStyle = "#ffffff";
      ctx.beginPath();
      ctx.arc(boardBall.x, boardBall.y, boardBall.radius, 0, Math.PI * 2);
      ctx.fill();

      ctx.strokeStyle = "#d32f2f";
      ctx.lineWidth = 1;
      ctx.beginPath();
      ctx.arc(boardBall.x, boardBall.y, boardBall.radius - 2, 0.3, 1.4);
      ctx.stroke();
    }

    function drawScoreboardMini() {
      ctx.fillStyle = "#081820";
      ctx.fillRect(12, 12, 180, 78);
      ctx.strokeStyle = "#e0f8d0";
      ctx.lineWidth = 3;
      ctx.strokeRect(12, 12, 180, 78);

      drawPixelText("RETRO PARK", 102, 35, 18, "#ffd166", "center");
      drawPixelText(`SCORE ${score}`, 24, 58, 16, "#e0f8d0", "left");
      drawPixelText(`INN ${inning}`, 24, 78, 16, "#e0f8d0", "left");
    }

    function drawCountLights() {
      const x = 425;
      const y = 34;
      drawPixelText("B S O", x - 6, y, 16, "#e0f8d0");

      for (let i = 0; i < 4; i++) {
        ctx.fillStyle = i < balls ? "#ffd166" : "#3b4d1f";
        ctx.fillRect(x + i * 12, y + 10, 8, 8);
      }
      for (let i = 0; i < 3; i++) {
        ctx.fillStyle = i < strikes ? "#ff6b6b" : "#3b4d1f";
        ctx.fillRect(x + i * 12 + 48, y + 10, 8, 8);
      }
      for (let i = 0; i < 3; i++) {
        ctx.fillStyle = i < outs ? "#e0f8d0" : "#3b4d1f";
        ctx.fillRect(x + i * 12 + 88, y + 10, 8, 8);
      }
    }

    function renderOverlayText() {
      if (!gameStarted) {
        drawPixelText("TAP START TO PLAY", W / 2, H / 2 - 12, 24, "#e0f8d0", "center");
        drawPixelText("SMALL STRIKE ZONE!", W / 2, H / 2 + 20, 18, "#ffd166", "center");
      }

      if (boardBall) {
        drawPixelText("BOARD BALL!", W / 2, 395, 18, "#ffd166", "center");
      }

      if (gameOver) {
        ctx.fillStyle = "rgba(0,0,0,0.55)";
        ctx.fillRect(0, 0, W, H);
        drawPixelText("GAME OVER", W / 2, H / 2 - 20, 34, "#ffd166", "center");
        drawPixelText(`FINAL SCORE ${score}`, W / 2, H / 2 + 18, 22, "#e0f8d0", "center");
        drawPixelText("TAP RESET", W / 2, H / 2 + 52, 18, "#e0f8d0", "center");
      }
    }

    function render() {
      ctx.clearRect(0, 0, W, H);
      drawField();
      drawRunners();
      drawPitcher();
      drawBatter();
      drawPitchBall();
      drawBoardBall();
      drawScoreboardMini();
      drawCountLights();
      renderOverlayText();
    }

    function loop() {
      updatePitch();
      updateBoardBall();
      updateAnimation();
      updateHUD();
      render();
      requestAnimationFrame(loop);
    }

    function handleStart() {
      ensureAudio();

      if (soundEnabled && !bgmInterval) {
        startBGM();
      }

      if (!gameStarted) {
        gameStarted = true;
        log("プレイボール！");
        setMessage("試合開始！");
        updateHUD();
        startPitch();
      } else if (!pitchActive && !boardBall && !gameOver) {
        startPitch();
      }
    }

    function handleSwing() {
      ensureAudio();
      if (!gameStarted || gameOver || boardBall) return;
      judgeSwing();
    }

    function pressEffect(btn, handler) {
      btn.addEventListener("touchstart", (e) => {
        e.preventDefault();
        handler();
      }, { passive: false });

      btn.addEventListener("click", (e) => {
        e.preventDefault();
        handler();
      });
    }

    pressEffect(startBtn, handleStart);
    pressEffect(swingBtn, handleSwing);
    pressEffect(soundBtn, toggleSound);
    pressEffect(resetBtn, resetGame);

    document.addEventListener("keydown", (e) => {
      const key = e.key.toLowerCase();

      if (key === "r") {
        resetGame();
        return;
      }

      if (key === "m") {
        toggleSound();
        return;
      }

      if (key === " " || e.code === "Space") {
        e.preventDefault();
        handleStart();
        return;
      }

      if (key === "z") {
        handleSwing();
      }
    });

    canvas.addEventListener("touchstart", (e) => {
      e.preventDefault();
      ensureAudio();
      if (!gameStarted) handleStart();
      else if (!boardBall) handleSwing();
    }, { passive: false });

    resetGame();
    updateHUD();
    loop();
  </script>
</body>
</html>