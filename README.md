# sphere-dash
juegi retro 
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="theme-color" content="#0a0420">
<title>Sphere Dash</title>
<style>
  :root{
    --cian:#21e6ff;
    --magenta:#ff2bd6;
    --violeta:#8a4dff;
    --noche:#0a0420;
  }
  *{box-sizing:border-box;-webkit-tap-highlight-color:transparent}
  html,body{margin:0;height:100%;overflow:hidden;background:var(--noche);
    font-family:"Trebuchet MS","Segoe UI",Arial,sans-serif;color:#eaf6ff;
    -webkit-user-select:none;user-select:none}
  canvas{display:block;width:100vw;height:100vh;touch-action:none}

  /* HUD */
  #hud{position:fixed;top:14px;left:0;right:0;display:flex;justify-content:space-between;
    padding:0 18px;font-weight:700;letter-spacing:1px;pointer-events:none;z-index:5}
  #hud .tag{font-size:12px;opacity:.65;text-transform:uppercase;letter-spacing:3px}
  #hud .val{font-size:26px;text-shadow:0 0 14px var(--cian)}
  #best .val{color:var(--magenta);text-shadow:0 0 14px var(--magenta)}
  #hud .right{text-align:right}

  /* botón de sonido */
  #sound{position:fixed;top:14px;left:50%;transform:translateX(-50%);z-index:6;
    width:42px;height:42px;border-radius:12px;border:1px solid #2bd4ff55;
    background:#0e0a2e88;color:var(--cian);font-size:18px;cursor:pointer;
    display:flex;align-items:center;justify-content:center;backdrop-filter:blur(4px)}
  #sound:active{transform:translateX(-50%) scale(.92)}

  /* menús */
  .overlay{position:fixed;inset:0;z-index:10;display:flex;align-items:center;justify-content:center;
    text-align:center;padding:24px;background:radial-gradient(ellipse at 50% 35%, #1a0b4099, #050218ee)}
  .panel{position:relative;max-width:380px;width:100%;padding:34px 28px 30px;border-radius:24px;
    background:linear-gradient(160deg,#160a3a,#0a0628);
    border:1px solid #2bd4ff44;
    box-shadow:0 0 60px #21e6ff22, inset 0 0 40px #ff2bd611}
  h1{margin:0 0 4px;font-size:42px;letter-spacing:3px;line-height:1;
    background:linear-gradient(90deg,var(--cian),var(--magenta));
    -webkit-background-clip:text;background-clip:text;color:transparent;
    filter:drop-shadow(0 0 18px #ff2bd655)}
  .sub{margin:0 0 22px;font-size:13px;letter-spacing:4px;text-transform:uppercase;color:#9fb6ff;opacity:.8}
  .msg{font-size:15px;line-height:1.5;color:#cfe2ff;opacity:.9;margin:0 0 22px}
  .scoreRow{display:flex;gap:14px;justify-content:center;margin:0 0 24px}
  .card{flex:1;padding:12px 8px;border-radius:14px;background:#0c0730;border:1px solid #ffffff14}
  .card .k{font-size:10px;letter-spacing:2px;opacity:.55;text-transform:uppercase}
  .card .v{font-size:30px;font-weight:800}
  .card.now .v{color:var(--cian);text-shadow:0 0 12px var(--cian)}
  .card.high .v{color:var(--magenta);text-shadow:0 0 12px var(--magenta)}
  button.play{width:100%;padding:16px;border:0;border-radius:16px;font-size:20px;font-weight:800;
    letter-spacing:3px;cursor:pointer;color:#05021a;
    background:linear-gradient(90deg,var(--cian),var(--violeta),var(--magenta));
    box-shadow:0 8px 30px #ff2bd644, 0 0 22px #21e6ff55;text-transform:uppercase}
  button.play:active{transform:translateY(2px)}
  .hint{margin-top:16px;font-size:12px;letter-spacing:1px;color:#8ea7d6;opacity:.7}
  .hidden{display:none !important}
</style>
</head>
<body>

<div id="hud">
  <div id="cur"><div class="tag">Puntos</div><div class="val" id="score">0</div></div>
  <div id="best" class="right"><div class="tag">Récord</div><div class="val" id="bscore">0</div></div>
</div>

<button id="sound" aria-label="Sonido">♪</button>

<canvas id="game"></canvas>

<!-- Menú inicial -->
<div class="overlay" id="menu">
  <div class="panel">
    <h1>SPHERE DASH</h1>
    <p class="sub">Synthwave Runner</p>
    <p class="msg">Salta los pinchos, bloques y sierras.<br>Esquiva los peligros voladores… ¡no saltes esos!</p>
    <button class="play" onclick="startGame()">Jugar</button>
    <p class="hint">Espacio · Clic · Toca la pantalla para saltar</p>
  </div>
</div>

<!-- Menú game over -->
<div class="overlay hidden" id="over">
  <div class="panel">
    <h1 style="font-size:38px">GAME OVER</h1>
    <p class="sub" id="overSub">Has chocado</p>
    <div class="scoreRow">
      <div class="card now"><div class="k">Puntos</div><div class="v" id="finalScore">0</div></div>
      <div class="card high"><div class="k">Récord</div><div class="v" id="finalBest">0</div></div>
    </div>
    <button class="play" onclick="startGame()">Reintentar</button>
    <p class="hint">Espacio · Clic · Toca para volver a jugar</p>
  </div>
</div>

<script>
/* =========================================================
   SPHERE DASH — runner synthwave de un solo archivo
   ========================================================= */
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");
const menu = document.getElementById("menu");
const over = document.getElementById("over");
const scoreText = document.getElementById("score");
const bestText  = document.getElementById("bscore");
const soundBtn  = document.getElementById("sound");

let W, H, ground, DPR;
let player, obstacles, particles, stars;
let score, bestScore = 0, speed, running, started = false, holding = false;
let spawnDist, distSinceSpawn, lastType = "";
let bgScroll = 0, gameTime = 0;

/* ---------- escalado responsive con densidad de píxel ---------- */
function resize(){
  DPR = Math.min(window.devicePixelRatio || 1, 2);
  W = window.innerWidth;
  H = window.innerHeight;
  canvas.width  = W * DPR;
  canvas.height = H * DPR;
  ctx.setTransform(DPR, 0, 0, DPR, 0, 0);
  ground = H * 0.80;
  if (player) player.baseY = ground;
}
window.addEventListener("resize", resize);

/* ---------- estrellas de fondo ---------- */
function makeStars(){
  stars = [];
  const n = 70;
  for (let i = 0; i < n; i++){
    stars.push({
      x: Math.random() * W,
      y: Math.random() * (H * 0.7),
      r: Math.random() * 1.6 + 0.4,
      tw: Math.random() * Math.PI * 2,
      sp: Math.random() * 0.4 + 0.1
    });
  }
}

/* =========================================================
   AUDIO — música synthwave + efectos (Web Audio API)
   ========================================================= */
const Audio = (() => {
  let ac = null, master = null, musicGain = null, sfxGain = null;
  let enabled = true, playing = false;
  let nextNoteTime = 0, step = 0, schedTimer = null;
  const BPM = 112;
  const stepDur = (60 / BPM) / 4;          // semicorcheas
  const lookahead = 0.1;

  // Progresión Am – F – C – G (vibe retro). Notas en Hz.
  const N = {
    A2:110.0, C3:130.8, E3:164.8, F2:87.31, A3:220.0,
    C4:261.6, E4:329.6, F3:174.6, G2:98.0, G3:196.0,
    D4:293.7, B3:246.9
  };
  // raíces del bajo por compás (4 compases)
  const bassRoots = [N.A2, N.F2, N.C3, N.G2];
  // patrón de arpegio lead (índices de acorde) por paso (16 pasos x 4 compases)
  const chords = [
    [N.A3, N.C4, N.E4],   // Am
    [N.A3, N.C4, N.F3],   // F
    [N.G3, N.C4, N.E4],   // C
    [N.G3, N.B3, N.D4]    // G
  ];
  const arpPattern = [0,1,2,1, 0,2,1,2, 0,1,2,1, 0,2,1,0];
  const kickSteps  = [0,4,8,12];
  const snareSteps = [4,12];
  const hatSteps   = [2,6,10,14];

  function init(){
    if (ac) return;
    const AC = window.AudioContext || window.webkitAudioContext;
    ac = new AC();
    master = ac.createGain(); master.gain.value = 0.9; master.connect(ac.destination);
    musicGain = ac.createGain(); musicGain.gain.value = 0.5; musicGain.connect(master);
    sfxGain = ac.createGain(); sfxGain.gain.value = 0.6; sfxGain.connect(master);
  }

  function noise(dur){
    const n = Math.floor(ac.sampleRate * dur);
    const buf = ac.createBuffer(1, n, ac.sampleRate);
    const d = buf.getChannelData(0);
    for (let i = 0; i < n; i++) d[i] = Math.random() * 2 - 1;
    const src = ac.createBufferSource(); src.buffer = buf; return src;
  }

  /* --- instrumentos --- */
  function bass(freq, t){
    const o = ac.createOscillator(), g = ac.createGain(), f = ac.createBiquadFilter();
    o.type = "sawtooth"; o.frequency.value = freq;
    f.type = "lowpass"; f.frequency.value = 600; f.Q.value = 6;
    g.gain.setValueAtTime(0.0001, t);
    g.gain.exponentialRampToValueAtTime(0.5, t + 0.02);
    g.gain.exponentialRampToValueAtTime(0.0001, t + stepDur * 2.2);
    o.connect(f); f.connect(g); g.connect(musicGain);
    o.start(t); o.stop(t + stepDur * 2.4);
  }
  function lead(freq, t){
    const o = ac.createOscillator(), g = ac.createGain(), f = ac.createBiquadFilter();
    o.type = "square"; o.frequency.value = freq;
    f.type = "lowpass"; f.frequency.setValueAtTime(2600, t);
    f.frequency.exponentialRampToValueAtTime(900, t + 0.25);
    g.gain.setValueAtTime(0.0001, t);
    g.gain.exponentialRampToValueAtTime(0.22, t + 0.01);
    g.gain.exponentialRampToValueAtTime(0.0001, t + stepDur * 1.8);
    o.connect(f); f.connect(g); g.connect(musicGain);
    o.start(t); o.stop(t + stepDur * 2);
  }
  function kick(t){
    const o = ac.createOscillator(), g = ac.createGain();
    o.frequency.setValueAtTime(140, t);
    o.frequency.exponentialRampToValueAtTime(45, t + 0.12);
    g.gain.setValueAtTime(0.7, t);
    g.gain.exponentialRampToValueAtTime(0.0001, t + 0.16);
    o.connect(g); g.connect(musicGain);
    o.start(t); o.stop(t + 0.18);
  }
  function snare(t){
    const s = noise(0.2), g = ac.createGain(), f = ac.createBiquadFilter();
    f.type = "highpass"; f.frequency.value = 1500;
    g.gain.setValueAtTime(0.4, t);
    g.gain.exponentialRampToValueAtTime(0.0001, t + 0.18);
    s.connect(f); f.connect(g); g.connect(musicGain);
    s.start(t); s.stop(t + 0.2);
  }
  function hat(t){
    const s = noise(0.05), g = ac.createGain(), f = ac.createBiquadFilter();
    f.type = "highpass"; f.frequency.value = 7000;
    g.gain.setValueAtTime(0.18, t);
    g.gain.exponentialRampToValueAtTime(0.0001, t + 0.04);
    s.connect(f); f.connect(g); g.connect(musicGain);
    s.start(t); s.stop(t + 0.05);
  }

  function scheduleStep(s, t){
    const bar = Math.floor(s / 16) % 4;
    const i = s % 16;
    if (i === 0) bass(bassRoots[bar], t);
    if (i === 8) bass(bassRoots[bar] * 1.5, t);     // quinta a mitad de compás
    const ch = chords[bar];
    lead(ch[arpPattern[i]], t);
    if (kickSteps.includes(i)) kick(t);
    if (snareSteps.includes(i)) snare(t);
    if (hatSteps.includes(i)) hat(t);
  }

  function scheduler(){
    while (nextNoteTime < ac.currentTime + lookahead){
      scheduleStep(step, nextNoteTime);
      nextNoteTime += stepDur;
      step = (step + 1) % 64;
    }
  }

  function startMusic(){
    if (!ac || playing || !enabled) return;
    playing = true; step = 0; nextNoteTime = ac.currentTime + 0.05;
    schedTimer = setInterval(scheduler, 25);
  }
  function stopMusic(){
    playing = false;
    if (schedTimer){ clearInterval(schedTimer); schedTimer = null; }
  }

  /* --- efectos --- */
  function sfxJump(){
    if (!ac || !enabled) return;
    const t = ac.currentTime, o = ac.createOscillator(), g = ac.createGain();
    o.type = "triangle";
    o.frequency.setValueAtTime(420, t);
    o.frequency.exponentialRampToValueAtTime(880, t + 0.12);
    g.gain.setValueAtTime(0.3, t);
    g.gain.exponentialRampToValueAtTime(0.0001, t + 0.16);
    o.connect(g); g.connect(sfxGain); o.start(t); o.stop(t + 0.18);
  }
  function sfxDie(){
    if (!ac || !enabled) return;
    const t = ac.currentTime, o = ac.createOscillator(), g = ac.createGain();
    o.type = "sawtooth";
    o.frequency.setValueAtTime(440, t);
    o.frequency.exponentialRampToValueAtTime(70, t + 0.5);
    g.gain.setValueAtTime(0.4, t);
    g.gain.exponentialRampToValueAtTime(0.0001, t + 0.6);
    o.connect(g); g.connect(sfxGain); o.start(t); o.stop(t + 0.62);
    const s = noise(0.3), ng = ac.createGain();
    ng.gain.setValueAtTime(0.25, t);
    ng.gain.exponentialRampToValueAtTime(0.0001, t + 0.3);
    s.connect(ng); ng.connect(sfxGain); s.start(t); s.stop(t + 0.3);
  }
  function sfxPoint(){
    if (!ac || !enabled) return;
    const t = ac.currentTime, o = ac.createOscillator(), g = ac.createGain();
    o.type = "square"; o.frequency.setValueAtTime(880, t);
    o.frequency.setValueAtTime(1320, t + 0.06);
    g.gain.setValueAtTime(0.2, t);
    g.gain.exponentialRampToValueAtTime(0.0001, t + 0.14);
    o.connect(g); g.connect(sfxGain); o.start(t); o.stop(t + 0.15);
  }

  function unlock(){ init(); if (ac.state === "suspended") ac.resume(); }
  function toggle(){
    enabled = !enabled;
    if (!enabled){ stopMusic(); if (master) master.gain.value = 0; }
    else { if (master) master.gain.value = 0.9; if (running) startMusic(); }
    return enabled;
  }

  return { unlock, startMusic, stopMusic, sfxJump, sfxDie, sfxPoint, toggle,
           get enabled(){ return enabled; } };
})();

/* =========================================================
   OBSTÁCULOS — definiciones variadas
   Cada tipo describe cómo se dibuja y sus cajas de colisión.
   ========================================================= */
const TYPES = {
  spike:      { w: 46,  airborne:false },  // pincho simple
  doubleSpike:{ w: 92,  airborne:false },  // dos pinchos
  tripleSpike:{ w: 138, airborne:false },  // tres pinchos
  block:      { w: 50,  airborne:false },  // bloque medio
  tallBlock:  { w: 50,  airborne:false },  // bloque alto
  saw:        { w: 64,  airborne:false },  // sierra giratoria
  flySpike:   { w: 70,  airborne:true  },  // pincho volador (NO saltar)
  flyBar:     { w: 90,  airborne:true  }   // barra voladora (NO saltar)
};

function spawnObstacle(){
  // elige un tipo distinto al anterior para variar el ritmo
  const keys = Object.keys(TYPES);
  let type;
  do { type = keys[(Math.random() * keys.length) | 0]; }
  while (type === lastType && Math.random() < 0.7);
  lastType = type;

  obstacles.push({ type, x: W + 60, rot: 0, scored: false });
}

/* --- cajas de colisión de cada obstáculo (rectángulos / círculo) --- */
function hitboxes(o){
  const g = ground, x = o.x;
  switch (o.type){
    case "spike":       return [{x:x+8,  y:g-38, w:30, h:38}];
    case "doubleSpike": return [{x:x+8,  y:g-38, w:30, h:38},{x:x+54, y:g-38, w:30, h:38}];
    case "tripleSpike": return [{x:x+8,  y:g-38, w:30, h:38},{x:x+54, y:g-38, w:30, h:38},{x:x+100,y:g-38, w:30, h:38}];
    case "block":       return [{x:x+5,  y:g-58, w:40, h:58}];
    case "tallBlock":   return [{x:x+5,  y:g-104,w:40, h:104}];
    case "saw":         return [{circle:true, cx:x+32, cy:g-26, r:30}];
    case "flySpike":    return [{x:x+12, y:g-150,w:46, h:62}]; // banda alta: choca si saltas
    case "flyBar":      return [{x:x+5,  y:g-148,w:80, h:34}];
  }
  return [];
}

function circleRect(c, r){
  const px = Math.max(r.x, Math.min(c.x, r.x + r.w));
  const py = Math.max(r.y, Math.min(c.y, r.y + r.h));
  return Math.hypot(c.x - px, c.y - py) < c.r;
}
function circleCircle(c, o){
  return Math.hypot(c.x - o.cx, c.y - o.cy) < c.r + o.r;
}
function collides(o){
  const c = { x: player.x, y: player.y, r: player.r - 4 };
  for (const box of hitboxes(o)){
    if (box.circle){ if (circleCircle(c, box)) return true; }
    else if (circleRect(c, box)) return true;
  }
  return false;
}

/* =========================================================
   PARTÍCULAS
   ========================================================= */
function burst(x, y, color, n, power){
  for (let i = 0; i < n; i++){
    const a = Math.random() * Math.PI * 2;
    const s = Math.random() * power + 1;
    particles.push({
      x, y, vx: Math.cos(a) * s, vy: Math.sin(a) * s - 1,
      life: 1, r: Math.random() * 3 + 1.5, color
    });
  }
}

/* =========================================================
   CONTROL DE JUEGO
   ========================================================= */
function startGame(){
  Audio.unlock();
  menu.classList.add("hidden");
  over.classList.add("hidden");
  player = { x: W * 0.18, y: ground - 30, r: 28, vy: 0, onGround: true,
             baseY: ground, rot: 0, squash: 1 };
  obstacles = []; particles = [];
  score = 0; speed = 6.2; running = true; started = true; holding = false;
  spawnDist = 360; distSinceSpawn = 0; lastType = ""; gameTime = 0;
  makeStars();
  scoreText.textContent = "0";
  Audio.startMusic();
  requestAnimationFrame(loop);
}

function jump(){
  if (!running){ return; }
  if (player && player.onGround){
    player.vy = -15.5;
    player.onGround = false;
    player.squash = 0.7;
    Audio.sfxJump();
    burst(player.x, player.y + player.r, "#21e6ff", 8, 3);
  }
}

function gameOver(){
  running = false;
  Audio.stopMusic();
  Audio.sfxDie();
  burst(player.x, player.y, "#ff2bd6", 36, 8);
  burst(player.x, player.y, "#21e6ff", 24, 6);
  bestScore = Math.max(bestScore, score);
  bestText.textContent = bestScore;
  document.getElementById("finalScore").textContent = score;
  document.getElementById("finalBest").textContent = bestScore;
  document.getElementById("overSub").textContent =
    (score >= bestScore && score > 0) ? "¡Nuevo récord!" : "Has chocado";
  // pequeño retardo para que se vea la explosión
  setTimeout(() => over.classList.remove("hidden"), 420);
}

/* =========================================================
   ACTUALIZACIÓN
   ========================================================= */
function update(dt){
  gameTime += dt;
  score++;
  scoreText.textContent = score;
  if (score % 500 === 0) Audio.sfxPoint();

  speed = 6.2 + score / 420;       // acelera progresivamente
  bgScroll += speed;

  // física de la esfera
  player.vy += 0.78;
  player.y  += player.vy;
  if (player.y + player.r >= ground){
    if (!player.onGround) burst(player.x, ground, "#8a4dff", 6, 2.4);
    player.y = ground - player.r;
    player.vy = 0;
    player.onGround = true;
  }
  // salto continuo: si mantienes pulsado, vuelve a saltar al tocar el suelo
  if (holding && player.onGround) jump();
  // rotación al rodar y squash/stretch elástico
  player.rot += speed / player.r;
  player.squash += (1 - player.squash) * 0.18;
  if (!player.onGround) player.squash = 1 + Math.min(0.25, Math.abs(player.vy) * 0.012);

  // spawn por distancia (más justo que por temporizador fijo)
  distSinceSpawn += speed;
  if (distSinceSpawn >= spawnDist){
    spawnObstacle();
    distSinceSpawn = 0;
    // separación que se ajusta a la velocidad para que siempre sea jugable
    spawnDist = 300 + Math.random() * 220 + speed * 14;
  }

  // mover y comprobar obstáculos
  for (const o of obstacles){
    o.x -= speed;
    if (o.type === "saw") o.rot += 0.25;
    if (collides(o)){ gameOver(); return; }
  }
  obstacles = obstacles.filter(o => o.x > -160);

  // estrellas
  for (const s of stars){
    s.x -= s.sp * speed * 0.25;
    if (s.x < -2){ s.x = W + 2; s.y = Math.random() * (H * 0.7); }
    s.tw += 0.05;
  }

  // partículas
  for (const p of particles){
    p.x += p.vx; p.y += p.vy; p.vy += 0.25; p.life -= 0.02;
  }
  particles = particles.filter(p => p.life > 0);
}

/* =========================================================
   DIBUJO
   ========================================================= */
function drawBackground(){
  // cielo degradado
  const sky = ctx.createLinearGradient(0, 0, 0, ground);
  sky.addColorStop(0, "#1a0838");
  sky.addColorStop(0.6, "#2a0b4a");
  sky.addColorStop(1, "#3a0f4f");
  ctx.fillStyle = sky;
  ctx.fillRect(0, 0, W, ground);

  // sol retrowave con franjas
  const cx = W / 2, sunY = ground - 150, sunR = Math.min(W, H) * 0.16;
  const sun = ctx.createLinearGradient(0, sunY - sunR, 0, sunY + sunR);
  sun.addColorStop(0, "#ffd35e");
  sun.addColorStop(0.5, "#ff7ad9");
  sun.addColorStop(1, "#ff2bd6");
  ctx.save();
  ctx.beginPath(); ctx.arc(cx, sunY, sunR, 0, Math.PI * 2); ctx.clip();
  ctx.fillStyle = sun; ctx.fillRect(cx - sunR, sunY - sunR, sunR * 2, sunR * 2);
  ctx.fillStyle = "#2a0b4a";
  for (let i = 0; i < 7; i++){
    const yy = sunY + 4 + i * 7;
    ctx.fillRect(cx - sunR, yy, sunR * 2, 3 + i);
  }
  ctx.restore();

  // estrellas
  for (const s of stars){
    ctx.globalAlpha = 0.4 + Math.sin(s.tw) * 0.4;
    ctx.fillStyle = "#cfe8ff";
    ctx.beginPath(); ctx.arc(s.x, s.y, s.r, 0, Math.PI * 2); ctx.fill();
  }
  ctx.globalAlpha = 1;

  // suelo synthwave
  ctx.fillStyle = "#0a0420";
  ctx.fillRect(0, ground, W, H - ground);

  // rejilla en perspectiva
  ctx.strokeStyle = "#21e6ff55";
  ctx.lineWidth = 1;
  const vanish = W / 2;
  // líneas horizontales que se alejan
  for (let i = 1; i <= 12; i++){
    const t = i / 12;
    const y = ground + t * t * (H - ground);
    ctx.globalAlpha = 0.5 * (1 - t) + 0.1;
    ctx.beginPath(); ctx.moveTo(0, y); ctx.lineTo(W, y); ctx.stroke();
  }
  // líneas verticales hacia el punto de fuga (con scroll)
  ctx.globalAlpha = 0.35;
  const spacing = 70;
  const off = bgScroll % spacing;
  for (let gx = -off; gx < W + spacing; gx += spacing){
    ctx.beginPath();
    ctx.moveTo(gx, ground);
    ctx.lineTo(vanish + (gx - vanish) * 5.5, H);
    ctx.stroke();
  }
  ctx.globalAlpha = 1;

  // línea de horizonte brillante
  ctx.strokeStyle = "#21e6ff";
  ctx.lineWidth = 3;
  ctx.shadowColor = "#21e6ff"; ctx.shadowBlur = 18;
  ctx.beginPath(); ctx.moveTo(0, ground); ctx.lineTo(W, ground); ctx.stroke();
  ctx.shadowBlur = 0;
}

function drawPlayer(){
  const p = player;
  ctx.save();
  ctx.translate(p.x, p.y);
  ctx.rotate(p.rot);
  const sx = 1 / p.squash, sy = p.squash;
  ctx.scale(sx, sy);

  // resplandor
  ctx.shadowColor = "#21e6ff"; ctx.shadowBlur = 22;

  // esfera con degradado para dar volumen
  const grad = ctx.createRadialGradient(-p.r*0.35, -p.r*0.35, p.r*0.2, 0, 0, p.r);
  grad.addColorStop(0, "#d6fbff");
  grad.addColorStop(0.4, "#33e9ff");
  grad.addColorStop(1, "#0a83c9");
  ctx.fillStyle = grad;
  ctx.beginPath(); ctx.arc(0, 0, p.r, 0, Math.PI * 2); ctx.fill();
  ctx.shadowBlur = 0;

  // banda ecuatorial para que se note el giro
  ctx.strokeStyle = "#ffffffaa"; ctx.lineWidth = 3;
  ctx.beginPath(); ctx.arc(0, 0, p.r * 0.62, 0, Math.PI * 2); ctx.stroke();
  ctx.fillStyle = "#ff2bd6";
  ctx.beginPath(); ctx.arc(p.r * 0.5, 0, p.r * 0.18, 0, Math.PI * 2); ctx.fill();

  ctx.restore();

  // brillo de contacto en el suelo
  if (p.onGround){
    ctx.globalAlpha = 0.3;
    ctx.fillStyle = "#21e6ff";
    ctx.beginPath();
    ctx.ellipse(p.x, ground, p.r * 0.9, 5, 0, 0, Math.PI * 2);
    ctx.fill();
    ctx.globalAlpha = 1;
  }
}

function drawObstacle(o){
  const g = ground, x = o.x;
  ctx.save();
  ctx.lineJoin = "round";

  const drawSpike = (sx) => {
    ctx.fillStyle = "#ff2bd6";
    ctx.shadowColor = "#ff2bd6"; ctx.shadowBlur = 14;
    ctx.beginPath();
    ctx.moveTo(sx, g);
    ctx.lineTo(sx + 23, g - 44);
    ctx.lineTo(sx + 46, g);
    ctx.closePath(); ctx.fill();
    ctx.shadowBlur = 0;
    ctx.strokeStyle = "#ffd1f5"; ctx.lineWidth = 2; ctx.stroke();
  };
  const drawBlock = (h) => {
    const grad = ctx.createLinearGradient(x, g - h, x, g);
    grad.addColorStop(0, "#8a4dff"); grad.addColorStop(1, "#5a1fb0");
    ctx.fillStyle = grad;
    ctx.shadowColor = "#8a4dff"; ctx.shadowBlur = 14;
    ctx.fillRect(x, g - h, 50, h);
    ctx.shadowBlur = 0;
    ctx.strokeStyle = "#c9a7ff"; ctx.lineWidth = 2; ctx.strokeRect(x, g - h, 50, h);
  };

  switch (o.type){
    case "spike":       drawSpike(x); break;
    case "doubleSpike": drawSpike(x); drawSpike(x + 46); break;
    case "tripleSpike": drawSpike(x); drawSpike(x + 46); drawSpike(x + 92); break;
    case "block":       drawBlock(58); break;
    case "tallBlock":   drawBlock(104); break;
    case "saw": {
      const cx = x + 32, cy = g - 26, r = 30;
      ctx.translate(cx, cy); ctx.rotate(o.rot);
      ctx.fillStyle = "#21e6ff";
      ctx.shadowColor = "#21e6ff"; ctx.shadowBlur = 16;
      ctx.beginPath();
      for (let i = 0; i < 12; i++){
        const a = (i / 12) * Math.PI * 2;
        const rr = (i % 2 === 0) ? r : r * 0.72;
        ctx.lineTo(Math.cos(a) * rr, Math.sin(a) * rr);
      }
      ctx.closePath(); ctx.fill();
      ctx.shadowBlur = 0;
      ctx.fillStyle = "#0a0420";
      ctx.beginPath(); ctx.arc(0, 0, r * 0.34, 0, Math.PI * 2); ctx.fill();
      ctx.strokeStyle = "#d6fbff"; ctx.lineWidth = 2;
      ctx.beginPath(); ctx.arc(0, 0, r * 0.34, 0, Math.PI * 2); ctx.stroke();
      break;
    }
    case "flySpike": {
      // pincho volador apuntando hacia abajo + aura de aviso
      const topY = g - 150;
      ctx.fillStyle = "#ffb347";
      ctx.shadowColor = "#ffb347"; ctx.shadowBlur = 16;
      ctx.beginPath();
      ctx.moveTo(x + 12, topY);
      ctx.lineTo(x + 58, topY);
      ctx.lineTo(x + 35, topY + 62);
      ctx.closePath(); ctx.fill();
      ctx.shadowBlur = 0;
      ctx.strokeStyle = "#fff0d0"; ctx.lineWidth = 2; ctx.stroke();
      drawWarn(x + 35, topY - 14);
      break;
    }
    case "flyBar": {
      const topY = g - 148;
      const grad = ctx.createLinearGradient(x, topY, x, topY + 34);
      grad.addColorStop(0, "#ffb347"); grad.addColorStop(1, "#ff7a1f");
      ctx.fillStyle = grad;
      ctx.shadowColor = "#ffb347"; ctx.shadowBlur = 16;
      ctx.fillRect(x + 5, topY, 80, 34);
      ctx.shadowBlur = 0;
      ctx.strokeStyle = "#fff0d0"; ctx.lineWidth = 2; ctx.strokeRect(x + 5, topY, 80, 34);
      // dientes
      ctx.fillStyle = "#ff7a1f";
      for (let i = 0; i < 5; i++){
        const bx = x + 9 + i * 16;
        ctx.beginPath();
        ctx.moveTo(bx, topY + 34);
        ctx.lineTo(bx + 8, topY + 34);
        ctx.lineTo(bx + 4, topY + 46);
        ctx.closePath(); ctx.fill();
      }
      drawWarn(x + 45, topY - 14);
      break;
    }
  }
  ctx.restore();
}

// flechita de aviso "no saltes" sobre los obstáculos voladores
function drawWarn(cx, cy){
  ctx.fillStyle = "#ffe08a";
  ctx.globalAlpha = 0.85;
  ctx.beginPath();
  ctx.moveTo(cx, cy + 8);
  ctx.lineTo(cx - 6, cy);
  ctx.lineTo(cx + 6, cy);
  ctx.closePath(); ctx.fill();
  ctx.globalAlpha = 1;
}

function drawParticles(){
  for (const p of particles){
    ctx.globalAlpha = Math.max(0, p.life);
    ctx.fillStyle = p.color;
    ctx.beginPath(); ctx.arc(p.x, p.y, p.r, 0, Math.PI * 2); ctx.fill();
  }
  ctx.globalAlpha = 1;
}

function draw(){
  ctx.clearRect(0, 0, W, H);
  drawBackground();
  for (const o of obstacles) drawObstacle(o);
  drawPlayer();
  drawParticles();
}

/* =========================================================
   BUCLE PRINCIPAL
   ========================================================= */
let lastTime = 0;
function loop(now){
  if (!running){ draw(); return; }
  const dt = lastTime ? Math.min((now - lastTime) / 16.67, 2) : 1;
  lastTime = now;
  update(dt);
  draw();
  requestAnimationFrame(loop);
}

/* =========================================================
   ENTRADAS
   ========================================================= */
function press(){ holding = true; jump(); }   // jump() ya se ignora si no se está jugando
function release(){ holding = false; }

window.addEventListener("keydown", e => {
  if (e.code === "Space" || e.code === "ArrowUp"){ e.preventDefault(); if (!e.repeat) press(); else holding = true; }
});
window.addEventListener("keyup", e => {
  if (e.code === "Space" || e.code === "ArrowUp") release();
});
window.addEventListener("mousedown", e => { press(); });
window.addEventListener("mouseup",   e => { release(); });
window.addEventListener("touchstart", e => {
  if (running) e.preventDefault();
  press();
}, { passive: false });
window.addEventListener("touchend",   e => { release(); }, { passive: false });
window.addEventListener("touchcancel", e => { release(); });

soundBtn.addEventListener("click", e => {
  e.stopPropagation();
  Audio.unlock();
  const on = Audio.toggle();
  soundBtn.textContent = on ? "♪" : "♪̸";
  soundBtn.style.opacity = on ? "1" : "0.5";
});

/* arranque */
resize();
makeStars();
// pantalla de fondo animada del menú
(function idle(){
  if (!started){ drawBackground(); requestAnimationFrame(idle); }
})();
</script>
</body>
</html>
