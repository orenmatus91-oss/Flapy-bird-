<!DOCTYPE html>

<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Flappy — 10 Levels</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Silkscreen:wght@400;700&display=swap');
  * { margin:0; padding:0; box-sizing:border-box; }
  body {
    background:#060610;
    display:flex; flex-direction:column;
    align-items:center; justify-content:center;
    min-height:100vh;
    font-family:'Silkscreen',monospace;
    overflow:hidden;
  }
  #game-wrapper {
    position:relative;
    width:400px; height:600px;
  }
  canvas { display:block; image-rendering:pixelated; }
  #scanlines {
    position:absolute; top:0; left:0; width:100%; height:100%;
    background:repeating-linear-gradient(0deg,transparent,transparent 2px,rgba(0,0,0,0.07) 2px,rgba(0,0,0,0.07) 4px);
    pointer-events:none; z-index:10;
  }
  .overlay {
    position:absolute; top:0; left:0; width:100%; height:100%;
    display:flex; flex-direction:column; align-items:center; justify-content:center;
    background:rgba(0,0,0,0.72); backdrop-filter:blur(3px);
    pointer-events:all; cursor:pointer; z-index:20;
  }
  .hidden { display:none !important; }
  #start-screen .big-title {
    font-size:56px; font-weight:700; color:#00ffe0;
    text-shadow:0 0 30px #00ffe0, 0 0 60px rgba(0,255,224,0.4);
    letter-spacing:6px; margin-bottom:4px;
    animation:pulse 1.6s ease-in-out infinite;
  }
  #start-screen .sub { font-size:12px; color:#555; letter-spacing:3px; margin-bottom:44px; }
  #level-screen { gap:0; background:rgba(0,0,0,0.88); }
  #level-screen h2 { font-size:20px; color:#fff; letter-spacing:4px; margin-bottom:22px; }
  #level-grid { display:grid; grid-template-columns:repeat(5,1fr); gap:10px; width:320px; }
  .lvl-btn {
    width:54px; height:54px; border:2px solid #333;
    background:rgba(255,255,255,0.04); color:#fff;
    font-family:'Silkscreen',monospace; font-size:14px; cursor:pointer;
    transition:all 0.15s; display:flex; flex-direction:column;
    align-items:center; justify-content:center; gap:2px;
  }
  .lvl-btn:hover { transform:scale(1.08); }
  .lvl-btn .lnum { font-size:16px; font-weight:700; }
  .lvl-btn .lname { font-size:7px; letter-spacing:1px; opacity:0.7; }
  .lvl-btn.locked { opacity:0.25; cursor:not-allowed; }
  #level-screen .back-hint { font-size:10px; color:#555; letter-spacing:2px; margin-top:20px; }
  #hud {
    position:absolute; top:14px; left:0; width:100%;
    display:flex; align-items:center; justify-content:center;
    z-index:15; pointer-events:none;
  }
  #score-display { font-size:40px; font-weight:700; color:#fff; text-shadow:0 0 18px rgba(255,255,255,.5),2px 2px 0 rgba(0,0,0,.8); }
  #level-badge { font-size:11px; color:rgba(255,255,255,.5); letter-spacing:2px; margin-top:6px; text-align:center; }
  #levelup-screen { background:rgba(0,0,0,.78); }
  #levelup-screen .lu-title { font-size:38px; color:#FFD700; text-shadow:0 0 30px #FFD700; letter-spacing:4px; margin-bottom:6px; animation:pulse 1s ease-in-out infinite; }
  #levelup-screen .lu-next { font-size:13px; color:#aaa; letter-spacing:3px; margin-bottom:30px; }
  #levelup-screen .lu-hint { font-size:12px; color:#FFD700; letter-spacing:2px; animation:blink 1s step-end infinite; }
  #dead-screen .dead-title { font-size:48px; color:#ff4466; text-shadow:0 0 30px #ff4466; letter-spacing:4px; margin-bottom:4px; animation:shake .4s ease; }
  #dead-screen .score-lbl { font-size:11px; color:#666; letter-spacing:3px; margin-bottom:4px; }
  #dead-screen .score-big { font-size:60px; color:#fff; text-shadow:0 0 20px rgba(255,255,255,.4); margin-bottom:4px; }
  #dead-screen .best { font-size:11px; color:#aaa; letter-spacing:2px; margin-bottom:26px; }
  #dead-screen .hint { font-size:12px; letter-spacing:2px; animation:blink 1s step-end infinite; }
  #win-screen { background:rgba(0,0,0,.88); }
  #win-screen .win-title { font-size:44px; color:#FFD700; text-shadow:0 0 40px #FFD700,0 0 80px rgba(255,215,0,.4); letter-spacing:5px; margin-bottom:8px; animation:pulse 1.2s ease-in-out infinite; }
  #win-screen .win-sub { font-size:12px; color:#aaa; letter-spacing:3px; margin-bottom:8px; }
  #win-screen .win-score { font-size:48px; color:#fff; margin-bottom:22px; }
  #win-screen .hint { font-size:12px; color:#FFD700; letter-spacing:2px; animation:blink 1s step-end infinite; }
  #progress-bar-wrap {
    position:absolute; bottom:36px; left:20px; right:20px;
    pointer-events:none; z-index:15;
  }
  #progress-label { font-size:9px; color:rgba(255,255,255,.4); letter-spacing:2px; margin-bottom:4px; }
  #progress-track { width:100%; height:5px; background:rgba(255,255,255,.1); border-radius:3px; overflow:hidden; }
  #progress-fill { height:100%; width:0%; border-radius:3px; transition:width .3s; }
  @keyframes pulse { 0%,100%{opacity:1;transform:scale(1)} 50%{opacity:.85;transform:scale(.97)} }
  @keyframes blink { 0%,100%{opacity:1} 50%{opacity:0} }
  @keyframes shake { 0%,100%{transform:translateX(0)} 25%{transform:translateX(-8px)} 75%{transform:translateX(8px)} }
</style>
</head>
<body>
<div id="game-wrapper">
  <canvas id="canvas" width="400" height="600"></canvas>
  <div id="scanlines"></div>

  <div id="hud" class="hidden">
    <div>
      <div id="score-display">0</div>
      <div id="level-badge">LEVEL 1</div>
    </div>
  </div>

  <div id="progress-bar-wrap" class="hidden">
    <div id="progress-label">LEVEL PROGRESS</div>
    <div id="progress-track"><div id="progress-fill"></div></div>
  </div>

  <div id="start-screen" class="overlay">
    <div class="big-title">FLAPPY</div>
    <div class="sub">NEON EDITION · 10 LEVELS</div>
    <div style="font-size:13px;color:#00ffe0;letter-spacing:3px;animation:blink 1.2s step-end infinite">TAP TO START</div>
  </div>

  <div id="level-screen" class="overlay hidden">
    <h2>SELECT LEVEL</h2>
    <div id="level-grid"></div>
    <div class="back-hint" style="margin-top:18px;color:#444">TAP LEVEL TO PLAY · TAP BACKGROUND TO GO BACK</div>
  </div>

  <div id="levelup-screen" class="overlay hidden">
    <div class="lu-title">LEVEL UP!</div>
    <div class="lu-next" id="lu-next-text">ENTERING LEVEL 2</div>
    <div class="lu-hint">TAP TO CONTINUE</div>
  </div>

  <div id="dead-screen" class="overlay hidden">
    <div class="dead-title">DEAD</div>
    <div class="score-lbl">SCORE</div>
    <div class="score-big" id="dead-score">0</div>
    <div class="best" id="dead-best">BEST: 0</div>
    <div class="hint" id="dead-hint" style="color:#ff4466">TAP TO RETRY</div>
  </div>

  <div id="win-screen" class="overlay hidden">
    <div class="win-title">YOU WIN!</div>
    <div class="win-sub">ALL 10 LEVELS CLEARED</div>
    <div class="win-score" id="win-score">0</div>
    <div class="hint">TAP TO PLAY AGAIN</div>
  </div>
</div>

<script>
// ===================== LEVEL DEFINITIONS =====================
// gap: opening size, speed: pipe scroll speed, interval: frames between pipes
// pipes: how many to pass to advance, gravity/jump: physics
const LEVELS = [
  { name:'BREEZY',   pipes:6,  gap:175, speed:2.5,  interval:98, gravity:0.37, jump:-7.1, color:'#00ffe0', bg:'#05051a' },
  { name:'CRUISIN',  pipes:7,  gap:163, speed:2.78, interval:92, gravity:0.38, jump:-7.2, color:'#00e5ff', bg:'#030d15' },
  { name:'STEADY',   pipes:8,  gap:151, speed:3.05, interval:87, gravity:0.39, jump:-7.3, color:'#40c4ff', bg:'#050a18' },
  { name:'GUSTY',    pipes:8,  gap:140, speed:3.32, interval:83, gravity:0.40, jump:-7.3, color:'#7c4dff', bg:'#080512' },
  { name:'TRICKY',   pipes:9,  gap:129, speed:3.6,  interval:79, gravity:0.41, jump:-7.4, color:'#e040fb', bg:'#100512' },
  { name:'WINDY',    pipes:10, gap:119, speed:3.9,  interval:75, gravity:0.42, jump:-7.5, color:'#ff4081', bg:'#140410' },
  { name:'STORMY',   pipes:11, gap:110, speed:4.2,  interval:71, gravity:0.43, jump:-7.5, color:'#ff6d00', bg:'#150900' },
  { name:'BRUTAL',   pipes:12, gap:101, speed:4.6,  interval:67, gravity:0.44, jump:-7.6, color:'#ffab00', bg:'#141000' },
  { name:'INSANE',   pipes:13, gap:93,  speed:5.05, interval:62, gravity:0.45, jump:-7.7, color:'#ff3d00', bg:'#140400' },
  { name:'HELLMODE', pipes:14, gap:86,  speed:5.55, interval:57, gravity:0.46, jump:-7.8, color:'#ff1744', bg:'#180000' },
];

const PIPE_WIDTH = 58, W = 400, H = 600;
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');

const startScreen   = document.getElementById('start-screen');
const levelScreen   = document.getElementById('level-screen');
const levelupScreen = document.getElementById('levelup-screen');
const deadScreen    = document.getElementById('dead-screen');
const winScreen     = document.getElementById('win-screen');
const hud           = document.getElementById('hud');
const progressWrap  = document.getElementById('progress-bar-wrap');
const progressFill  = document.getElementById('progress-fill');
const scoreDisplay  = document.getElementById('score-display');
const levelBadge    = document.getElementById('level-badge');
const deadScoreEl   = document.getElementById('dead-score');
const deadBestEl    = document.getElementById('dead-best');
const deadHint      = document.getElementById('dead-hint');
const luNextText    = document.getElementById('lu-next-text');
const winScoreEl    = document.getElementById('win-score');

let gameState = 'start';
let currentLevel = 0, highestUnlocked = 0;
let bird, pipes, score, frameCount, levelScore;
let stars = [], particles = [];
let bestScores = new Array(10).fill(0);
let pendingTimeout = null;

function clearPending() { if(pendingTimeout!==null){clearTimeout(pendingTimeout);pendingTimeout=null;} }

try {
  const s = JSON.parse(localStorage.getItem('flappy10v2')||'{}');
  if (s.highestUnlocked != null) highestUnlocked = s.highestUnlocked;
  if (s.bestScores) bestScores = s.bestScores;
} catch(e){}

function saveProgress() {
  try { localStorage.setItem('flappy10v2', JSON.stringify({highestUnlocked, bestScores})); } catch(e){}
}

for (let i=0;i<90;i++) stars.push({
  x:Math.random()*W, y:Math.random()*H,
  r:Math.random()*1.5+0.3,
  a:Math.random()*0.8+0.2,
  speed:Math.random()*0.4+0.05
});

// ===================== LEVEL SELECT =====================
function buildLevelGrid() {
  const grid = document.getElementById('level-grid');
  grid.innerHTML='';
  LEVELS.forEach((lv,i)=>{
    const btn = document.createElement('div');
    btn.className='lvl-btn'+(i>highestUnlocked?' locked':'');
    btn.style.borderColor = i<=highestUnlocked ? lv.color : '#333';
    btn.style.color = i<=highestUnlocked ? lv.color : '#444';
    btn.style.boxShadow = i<=highestUnlocked ? `0 0 10px ${lv.color}44` : 'none';
    btn.innerHTML=`<span class="lnum">${i+1}</span><span class="lname">${lv.name}</span>`;
    if (i<=highestUnlocked) {
      btn.addEventListener('click', e=>{ e.stopPropagation(); startLevel(i); });
    }
    grid.appendChild(btn);
  });
}

// ===================== INIT =====================
function initGame(idx) {
  const lv = LEVELS[idx];
  canvas.style.borderLeft=`3px solid ${lv.color}`;
  canvas.style.borderRight=`3px solid ${lv.color}`;
  canvas.style.boxShadow=`0 0 40px ${lv.color}55,0 0 80px ${lv.color}22`;
  bird = {x:90, y:H/2, vy:0, rot:0, flap:0};
  pipes=[]; score=0; levelScore=0; frameCount=0; particles=[];
  scoreDisplay.textContent='0';
  levelBadge.textContent=`LEVEL ${idx+1} — ${lv.name}`;
  levelBadge.style.color=lv.color;
  progressFill.style.background=lv.color;
  progressFill.style.boxShadow=`0 0 8px ${lv.color}`;
  updateProgressBar(idx);
}

function updateProgressBar(idx) {
  const lv = LEVELS[idx];
  const pct = Math.min(100, (levelScore/lv.pipes)*100);
  progressFill.style.width = pct+'%';
}

function startLevel(idx) {
  currentLevel=idx;
  hideAll();
  hud.classList.remove('hidden');
  progressWrap.classList.remove('hidden');
  initGame(idx);
  gameState='playing';
  bird.vy=LEVELS[idx].jump;
  bird.flap=0;
}

// ===================== PIPE SPAWN =====================
function spawnPipe() {
  const lv=LEVELS[currentLevel];
  const minY=70, maxY=H-70-lv.gap;
  const topH=Math.floor(Math.random()*(maxY-minY)+minY);
  pipes.push({x:W+10, topH, scored:false});
}

// ===================== HELPERS =====================
function hexShift(hex, amt) {
  let r=parseInt(hex.slice(1,3),16), g=parseInt(hex.slice(3,5),16), b=parseInt(hex.slice(5,7),16);
  r=Math.min(255,Math.max(0,r+amt)); g=Math.min(255,Math.max(0,g+amt)); b=Math.min(255,Math.max(0,b+amt));
  return '#'+[r,g,b].map(v=>v.toString(16).padStart(2,'0')).join('');
}

function hideAll() {
  clearPending();
  [startScreen,levelScreen,levelupScreen,deadScreen,winScreen,hud,progressWrap].forEach(el=>el.classList.add('hidden'));
}

// ===================== DRAWING =====================
function drawBackground() {
  const lv=LEVELS[currentLevel];
  const sky=ctx.createLinearGradient(0,0,0,H);
  sky.addColorStop(0,lv.bg);
  sky.addColorStop(1,hexShift(lv.bg,18));
  ctx.fillStyle=sky;
  ctx.fillRect(0,0,W,H);

  for (const s of stars) {
    ctx.globalAlpha=s.a*(0.6+0.4*Math.sin(frameCount*0.02+s.x));
    ctx.fillStyle='#fff';
    ctx.beginPath(); ctx.arc(s.x,s.y,s.r,0,Math.PI*2); ctx.fill();
  }
  ctx.globalAlpha=1;

  ctx.fillStyle='#0d1410';
  ctx.fillRect(0,H-30,W,30);
  ctx.shadowColor=lv.color; ctx.shadowBlur=10;
  ctx.fillStyle=lv.color;
  ctx.fillRect(0,H-32,W,3);
  ctx.shadowBlur=0;

  ctx.strokeStyle=lv.color+'22'; ctx.lineWidth=1;
  for (let gx=(frameCount*lv.speed)%40; gx<W; gx+=40) {
    ctx.beginPath(); ctx.moveTo(gx,H-30); ctx.lineTo(gx,H); ctx.stroke();
  }
}

function drawBird(b) {
  ctx.save();
  ctx.translate(b.x,b.y);
  ctx.rotate(Math.max(-0.5,Math.min(1.2,b.rot)));
  const grd=ctx.createRadialGradient(0,0,2,0,0,18);
  grd.addColorStop(0,'rgba(255,220,60,1)');
  grd.addColorStop(0.6,'rgba(255,160,0,.9)');
  grd.addColorStop(1,'rgba(255,80,0,0)');
  ctx.fillStyle=grd;
  ctx.beginPath(); ctx.ellipse(0,0,17,13,0,0,Math.PI*2); ctx.fill();
  ctx.fillStyle='#FFD700';
  ctx.beginPath(); ctx.ellipse(0,0,14,11,0,0,Math.PI*2); ctx.fill();
  ctx.strokeStyle='#FF8C00'; ctx.lineWidth=1.5; ctx.stroke();
  const wy=Math.sin(b.flap*0.4)*5;
  ctx.fillStyle='#FFA500';
  ctx.beginPath(); ctx.ellipse(-3,wy,9,5,-0.4,0,Math.PI*2); ctx.fill();
  ctx.fillStyle='#fff'; ctx.beginPath(); ctx.arc(8,-3,4.5,0,Math.PI*2); ctx.fill();
  ctx.fillStyle='#111'; ctx.beginPath(); ctx.arc(9,-3,2.5,0,Math.PI*2); ctx.fill();
  ctx.fillStyle='#fff'; ctx.beginPath(); ctx.arc(10,-4,1,0,Math.PI*2); ctx.fill();
  ctx.fillStyle='#FF6600';
  ctx.beginPath(); ctx.moveTo(12,0); ctx.lineTo(20,-2); ctx.lineTo(20,3); ctx.closePath(); ctx.fill();
  ctx.shadowColor='#FFD700'; ctx.shadowBlur=12;
  ctx.strokeStyle='rgba(255,200,0,.4)'; ctx.lineWidth=1;
  ctx.beginPath(); ctx.ellipse(0,0,15,12,0,0,Math.PI*2); ctx.stroke();
  ctx.shadowBlur=0;
  ctx.restore();
}

function drawPipe(p) {
  const lv=LEVELS[currentLevel];
  const c=lv.color, dk=hexShift(lv.bg,30);
  const x=p.x;
  ctx.save();
  ctx.shadowColor=c; ctx.shadowBlur=16;
  const makeG=()=>{ const g=ctx.createLinearGradient(x,0,x+PIPE_WIDTH,0); g.addColorStop(0,dk); g.addColorStop(0.45,c+'cc'); g.addColorStop(1,dk); return g; };
  ctx.fillStyle=makeG(); ctx.fillRect(x,0,PIPE_WIDTH,p.topH);
  ctx.strokeStyle=c; ctx.lineWidth=2; ctx.strokeRect(x,0,PIPE_WIDTH,p.topH);
  ctx.fillStyle=c; ctx.fillRect(x-6,p.topH-20,PIPE_WIDTH+12,20); ctx.strokeRect(x-6,p.topH-20,PIPE_WIDTH+12,20);
  const botY=p.topH+lv.gap, botH=H-botY;
  ctx.fillStyle=makeG(); ctx.fillRect(x,botY,PIPE_WIDTH,botH);
  ctx.strokeStyle=c; ctx.lineWidth=2; ctx.strokeRect(x,botY,PIPE_WIDTH,botH);
  ctx.fillStyle=c; ctx.fillRect(x-6,botY,PIPE_WIDTH+12,20); ctx.strokeRect(x-6,botY,PIPE_WIDTH+12,20);
  ctx.shadowBlur=0; ctx.restore();
}

function drawParticles() {
  for (let i=particles.length-1;i>=0;i--) {
    const p=particles[i];
    p.x+=p.vx; p.y+=p.vy; p.vy+=0.15; p.life--;
    p.alpha=p.life/p.maxLife;
    if(p.life<=0){particles.splice(i,1);continue;}
    ctx.globalAlpha=p.alpha;
    ctx.fillStyle=p.color; ctx.shadowColor=p.color; ctx.shadowBlur=6;
    ctx.beginPath(); ctx.arc(p.x,p.y,p.r,0,Math.PI*2); ctx.fill();
    ctx.shadowBlur=0;
  }
  ctx.globalAlpha=1;
}

function spawnDeathP(bx,by) {
  const lv=LEVELS[currentLevel];
  const cols=['#FFD700','#FF8C00','#FFA500','#fff',lv.color,'#ff4466'];
  for(let i=0;i<30;i++){
    const a=Math.random()*Math.PI*2, s=Math.random()*6+1;
    particles.push({x:bx,y:by,vx:Math.cos(a)*s,vy:Math.sin(a)*s-2,r:Math.random()*4+1.5,
      color:cols[Math.floor(Math.random()*cols.length)],life:Math.floor(Math.random()*30+20),maxLife:50,alpha:1});
  }
}

function spawnScoreP(bx,by) {
  const c=LEVELS[currentLevel].color;
  for(let i=0;i<7;i++){
    particles.push({x:bx+20,y:by,vx:(Math.random()-.5)*3,vy:Math.random()*-3-1,
      r:Math.random()*3+1,color:c,life:25,maxLife:25,alpha:1});
  }
}

// ===================== COLLISION =====================
function checkCollision() {
  const lv=LEVELS[currentLevel];
  const bx=bird.x-10, by=bird.y-9, bw=22, bh=18;
  if(bird.y+13>=H-32||bird.y-13<=0) return true;
  for(const p of pipes){
    if(bx+bw>p.x+4&&bx<p.x+PIPE_WIDTH-4){
      if(by<p.topH||by+bh>p.topH+lv.gap) return true;
    }
  }
  return false;
}

// ===================== INPUT =====================
function handleTap() {
  if(gameState==='start'){
    buildLevelGrid();
    hideAll();
    levelScreen.classList.remove('hidden');
    gameState='levelselect';
  } else if(gameState==='levelselect'){
    hideAll();
    startScreen.classList.remove('hidden');
    gameState='start';
  } else if(gameState==='playing'){
    bird.vy=LEVELS[currentLevel].jump;
    bird.flap=0;
  } else if(gameState==='levelup'){
    hideAll();
    hud.classList.remove('hidden');
    progressWrap.classList.remove('hidden');
    startLevel(currentLevel);
  } else if(gameState==='dead'){
    hideAll();
    hud.classList.remove('hidden');
    progressWrap.classList.remove('hidden');
    startLevel(currentLevel);
  } else if(gameState==='win'){
    hideAll();
    buildLevelGrid();
    levelScreen.classList.remove('hidden');
    gameState='levelselect';
  }
}

document.addEventListener('keydown',e=>{
  if(e.code==='Space'||e.code==='ArrowUp'){e.preventDefault();handleTap();}
});
document.getElementById('game-wrapper').addEventListener('click',e=>{
  if(e.target.closest('.lvl-btn')&&!e.target.closest('.lvl-btn').classList.contains('locked')) return;
  handleTap();
});
document.getElementById('game-wrapper').addEventListener('touchstart',e=>{
  if(e.target.closest('.lvl-btn')&&!e.target.closest('.lvl-btn').classList.contains('locked')) return;
  e.preventDefault(); handleTap();
});

// ===================== LOOP =====================
frameCount=0; currentLevel=0;

function loop(){
  frameCount++;
  for(const s of stars){ s.x-=s.speed; if(s.x<0){s.x=W;s.y=Math.random()*H;} }

  drawBackground();

  if(gameState==='playing'){
    const lv=LEVELS[currentLevel];
    if(frameCount%lv.interval===0) spawnPipe();
    bird.vy+=lv.gravity;
    bird.y+=bird.vy;
    bird.rot=bird.vy*0.07;
    bird.flap++;

    for(let i=pipes.length-1;i>=0;i--){
      pipes[i].x-=lv.speed;
      if(!pipes[i].scored&&pipes[i].x+PIPE_WIDTH<bird.x){
        pipes[i].scored=true;
        score++; levelScore++;
        scoreDisplay.textContent=score;
        spawnScoreP(bird.x,bird.y);
        updateProgressBar(currentLevel);

        if(levelScore>=lv.pipes){
          if(currentLevel>=9){
            spawnDeathP(bird.x,bird.y);
            if(score>bestScores[currentLevel]){bestScores[currentLevel]=score;saveProgress();}
            gameState='win';
            hideAll();
            winScoreEl.textContent=score;
            pendingTimeout=setTimeout(()=>winScreen.classList.remove('hidden'),500);
          } else {
            if(score>bestScores[currentLevel]){bestScores[currentLevel]=score;saveProgress();}
            if(currentLevel+1>highestUnlocked){highestUnlocked=currentLevel+1;saveProgress();}
            currentLevel++;
            gameState='levelup';
            const nlv=LEVELS[currentLevel];
            luNextText.textContent=`ENTERING LEVEL ${currentLevel+1}: ${nlv.name}`;
            luNextText.style.color=nlv.color;
            hideAll();
            pendingTimeout=setTimeout(()=>levelupScreen.classList.remove('hidden'),300);
          }
        }
      }
      if(pipes[i].x+PIPE_WIDTH<-20) pipes.splice(i,1);
    }

    if(gameState==='playing'&&checkCollision()){
      spawnDeathP(bird.x,bird.y);
      if(score>bestScores[currentLevel]){bestScores[currentLevel]=score;saveProgress();}
      gameState='dead';
      hideAll();
      deadScoreEl.textContent=score;
      deadBestEl.textContent=`BEST LVL ${currentLevel+1}: ${bestScores[currentLevel]}`;
      deadHint.style.color=LEVELS[currentLevel].color;
      pendingTimeout=setTimeout(()=>deadScreen.classList.remove('hidden'),600);
    }
  }

  if(gameState==='playing'||gameState==='levelup'||gameState==='dead'){
    for(const p of pipes) drawPipe(p);
    if(gameState!=='dead') drawBird(bird);
  }

  if(gameState==='start'||gameState==='levelselect'){
    const t=frameCount*0.04;
    drawBird({x:200,y:H/2+Math.sin(t)*22,vy:0,rot:Math.sin(t)*0.2,flap:frameCount});
  }

  drawParticles();
  requestAnimationFrame(loop);
}

loop();
</script>

</body>
</html>
