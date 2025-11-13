# Save-Nobita-Doraemon-Rescue-
Save Nobita — Doraemon Rescue is a fun and simple 2D browser-based demo game built using HTML5 Canvas and JavaScript. The goal is to help Nobita reach Doraemon while avoiding moving obstacles (enemies). The game includes multiple levels, animated characters, and a built-in save/load system using the browser’s localStorage.

<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Save Nobita — Doraemon Rescue (Demo)</title>
  <style>
    :root{font-family:system-ui,-apple-system,Segoe UI,Roboto,Ubuntu,'Helvetica Neue',Arial}
    body{display:flex;flex-direction:column;align-items:center;padding:18px;background:#f0f7ff}
    h1{margin:0 0 12px;font-size:20px}
    #gameCanvas{background:linear-gradient(#bfe6ff,#ffffff);border-radius:12px;box-shadow:0 6px 18px rgba(20,40,80,.12)}
    .controls{margin-top:12px;display:flex;gap:8px}
    button{padding:8px 12px;border-radius:8px;border:0;background:#0b76d1;color:#fff;cursor:pointer}
    button.secondary{background:#6c757d}
    .info{margin-top:8px}
    .small{font-size:13px;color:#333}
  </style>
</head>
<body>
  <h1>Save Nobita — Doraemon Rescue (Demo)</h1>
  <canvas id="gameCanvas" width="720" height="420"></canvas>
  <div class="controls">
    <button id="saveBtn">Save Game</button>
    <button id="loadBtn">Load Game</button>
    <button id="resetBtn" class="secondary">Reset</button>
  </div>
  <div class="info small" id="status">Use arrow keys (or WASD) to move Nobita. Reach Doraemon to win. Save/load uses localStorage.</div>

<script>
// Simple 2D canvas game with Save/Load using localStorage
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const W = canvas.width; const H = canvas.height;

// Game state
let state = {
  player: {x: 60, y: H/2, w: 28, h: 34, speed: 3.5},
  target: {x: W-80, y: H/2, r: 22}, // Doraemon location
  score: 0,
  level: 1,
  enemies: [],
  running: true,
  won: false,
  ticks: 0
};

// Create enemies (moving obstacles)
function initEnemies(level=1){
  state.enemies = [];
  const count = Math.min(6, 2 + level*1);
  for(let i=0;i<count;i++){
    state.enemies.push({
      x: 150 + i*80,
      y: 40 + (i*60 % (H-80)),
      w: 32, h: 20,
      vx: (1 + Math.random()*1.8) * (Math.random()<0.5?-1:1),
      vy: (Math.random()*1.2 - 0.6),
    })
  }
}
initEnemies();

// Input
const keys = {};
window.addEventListener('keydown', e=>{ keys[e.key.toLowerCase()]=true; if(e.key==='s' || e.key==='S'){ saveGame(); } });
window.addEventListener('keyup', e=>{ keys[e.key.toLowerCase()]=false; });

// Game loop
function update(){
  if(!state.running) return;
  state.ticks++;
  // Move player
  const p = state.player;
  if(keys['arrowleft']||keys['a']) p.x -= p.speed;
  if(keys['arrowright']||keys['d']) p.x += p.speed;
  if(keys['arrowup']||keys['w']) p.y -= p.speed;
  if(keys['arrowdown']||keys['s']) p.y += p.speed;
  // Clamp
  p.x = Math.max(8, Math.min(W - p.w - 8, p.x));
  p.y = Math.max(8, Math.min(H - p.h - 8, p.y));

  // Move enemies
  for(let e of state.enemies){
    e.x += e.vx;
    e.y += e.vy;
    if(e.x < 8 || e.x + e.w > W-8) e.vx *= -1;
    if(e.y < 8 || e.y + e.h > H-8) e.vy *= -1;
  }

  // collisions with enemies
  for(let e of state.enemies){
    if(rectsOverlap(p, e)){
      // penalty
      state.score = Math.max(0, state.score - 1);
      // push player back
      p.x = 60; p.y = H/2;
    }
  }

  // check reach Doraemon (target)
  const t = state.target;
  if(circleRectCollide(t, p)){
    state.won = true;
    state.score += 10 * state.level;
    state.level++;
    // move target and make game harder
    t.x = Math.max(120, Math.min(W-80, Math.random()*(W-160)+80));
    t.y = Math.max(60, Math.min(H-60, Math.random()*(H-120)+60));
    initEnemies(state.level);
  }
}

function rectsOverlap(a,b){
  return !(a.x + a.w < b.x || a.x > b.x + b.w || a.y + a.h < b.y || a.y > b.y + b.h);
}
function circleRectCollide(circle, rect){
  const cx = Math.max(rect.x, Math.min(circle.x, rect.x+rect.w));
  const cy = Math.max(rect.y, Math.min(circle.y, rect.y+rect.h));
  const dx = circle.x - cx; const dy = circle.y - cy;
  return (dx*dx + dy*dy) <= (circle.r*circle.r);
}

function draw(){
  // clear
  ctx.clearRect(0,0,W,H);

  // background grid
  ctx.fillStyle = 'rgba(255,255,255,0.6)';
  for(let x=0;x<W;x+=80){ ctx.fillRect(x,0,1,H); }
  for(let y=0;y<H;y+=80){ ctx.fillRect(0,y,W,1); }

  // Draw Doraemon (target) - big blue circle with a bell
  const t = state.target;
  ctx.save();
  ctx.beginPath(); ctx.arc(t.x, t.y, t.r, 0, Math.PI*2); ctx.fillStyle = '#1e90ff'; ctx.fill();
  // face white
  ctx.beginPath(); ctx.arc(t.x-6, t.y-2, t.r*0.6, 0, Math.PI*2); ctx.fillStyle='#fff'; ctx.fill();
  // bell
  ctx.beginPath(); ctx.arc(t.x, t.y+ t.r*0.5, 6, 0, Math.PI*2); ctx.fillStyle='#ffd700'; ctx.fill();
  ctx.restore();

  // Draw enemies (red moving bars)
  for(let e of state.enemies){
    ctx.fillStyle = '#d33';
    roundRect(ctx, e.x, e.y, e.w, e.h, 6, true, false);
  }

  // Draw Nobita (player) - yellow rectangle with simple face
  const p = state.player;
  ctx.save();
  roundRect(ctx, p.x, p.y, p.w, p.h, 6, true, false);
  ctx.fillStyle = '#f8e93b';
  ctx.fillRect(p.x, p.y, p.w, p.h);
  // eyes
  ctx.fillStyle = '#000';
  ctx.fillRect(p.x+6, p.y+6, 4,4);
  ctx.fillRect(p.x+18, p.y+6,4,4);
  ctx.restore();

  // HUD
  ctx.fillStyle = '#003b5c';
  ctx.font = '14px system-ui';
  ctx.fillText('Score: '+state.score, 12, 18);
  ctx.fillText('Level: '+state.level, 12, 36);
  ctx.fillText('Save: press the Save button or S', 12, H-12);

  if(state.won){
    ctx.fillStyle = 'rgba(0,0,0,0.6)';
    ctx.fillRect(W/2-180,H/2-50,360,100);
    ctx.fillStyle = '#fff'; ctx.font='20px system-ui';
    ctx.fillText('You rescued Doraemon! Level up!', W/2-150, H/2);
    // small timeout to clear the message
    setTimeout(()=>{ state.won=false; },700);
  }
}

function roundRect(ctx, x, y, w, h, r, fill, stroke){
  if (typeof r === 'undefined') r = 5;
  ctx.beginPath();
  ctx.moveTo(x + r, y);
  ctx.arcTo(x + w, y, x + w, y + h, r);
  ctx.arcTo(x + w, y + h, x, y + h, r);
  ctx.arcTo(x, y + h, x, y, r);
  ctx.arcTo(x, y, x + w, y, r);
  ctx.closePath();
  if(fill) ctx.fill();
  if(stroke) ctx.stroke();
}

// Save / Load
const SAVE_KEY = 'nobita-doraemon-save-v1';
function saveGame(){
  // Only save essential state
  const toSave = {
    player: state.player,
    target: state.target,
    score: state.score,
    level: state.level,
    enemies: state.enemies,
    ticks: state.ticks
  };
  try{
    localStorage.setItem(SAVE_KEY, JSON.stringify(toSave));
    showStatus('Game saved.');
  }catch(e){ showStatus('Save failed: '+e.message); }
}
function loadGame(){
  try{
    const raw = localStorage.getItem(SAVE_KEY);
    if(!raw) { showStatus('No save found.'); return; }
    const s = JSON.parse(raw);
    state.player = s.player;
    state.target = s.target;
    state.score = s.score;
    state.level = s.level;
    state.enemies = s.enemies;
    state.ticks = s.ticks || 0;
    state.won = false;
    showStatus('Game loaded.');
  }catch(e){ showStatus('Load failed: '+e.message); }
}

function resetGame(){
  state = {
    player: {x: 60, y: H/2, w: 28, h: 34, speed: 3.5},
    target: {x: W-80, y: H/2, r: 22},
    score: 0,
    level: 1,
    enemies: [],
    running: true,
    won: false,
    ticks: 0
  };
  initEnemies();
  showStatus('Game reset.');
}

// UI
document.getElementById('saveBtn').addEventListener('click', saveGame);
document.getElementById('loadBtn').addEventListener('click', loadGame);
document.getElementById('resetBtn').addEventListener('click', resetGame);
function showStatus(msg){ const el = document.getElementById('status'); el.textContent = msg; setTimeout(()=>{ el.textContent = 'Use arrow keys (or WASD) to move Nobita. Reach Doraemon to win. Save/load uses localStorage.'},1500); }

// Main loop
function loop(){ update(); draw(); requestAnimationFrame(loop); }
loop();

// Expose save/load to console for debugging
window.saveGame = saveGame; window.loadGame = loadGame; window.resetGame = resetGame;
</script>
</body>
</html>

