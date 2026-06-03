<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>The 2-Player Grammar Clash!</title>
<style>
*{margin:0;padding:0;box-sizing:border-box;}
body{background:#0b0f19;font-family:'Segoe UI', Roboto, sans-serif;overflow:hidden;user-select:none;}
#bg-tint{position:fixed;inset:0;z-index:1;background:rgba(15, 23, 42, 0.4);pointer-events:none;}
canvas{position:fixed;inset:0;z-index:2;cursor: crosshair;}

/* DYNAMIC MONSTER CENTER PIECE */
#monster-container {
  position: fixed;
  top: 15%;
  left: 50%;
  transform: translateX(-50%);
  z-index: 3;
  width: 200px;
  height: 200px;
  display: flex;
  justify-content: center;
  align-items: center;
  pointer-events: none;
  transition: transform 0.15s ease-out, filter 0.15s ease-out;
}
.monster-idle {
  animation: floatBreathe 3s ease-in-out infinite;
}
#monster-gif {
  width: 100%;
  height: 100%;
  object-fit: contain;
}
@keyframes floatBreathe {
  0% { transform: translateX(-50%) translateY(0px) rotate(0deg); }
  50% { transform: translateX(-50%) translateY(-12px) rotate(2deg); }
  100% { transform: translateX(-50%) translateY(0px) rotate(0deg); }
}

/* UI HEADER SPLIT OVERLAY */
#ui{position:fixed;inset:0;z-index:4;pointer-events:none;}
#topbar{display:flex;justify-content:space-between;align-items:center;padding:0;background:rgba(15, 23, 42, 0.8);backdrop-filter:blur(8px);border-bottom: 3px solid #3b82f6;}

.p-panel {
  flex: 1;
  padding: 12px 25px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #fff;
}
#p1-panel { border-right: 2px dashed rgba(255,255,255,0.15); background: rgba(56, 189, 248, 0.05); }
#p2-panel { background: rgba(236, 72, 153, 0.05); }

.lbl { font-size: 14px; opacity: 0.6; text-transform: uppercase; letter-spacing: 1px; }
.sc-val { font-size: 22px; font-weight: 800; }
.p1-txt { color: #38bdf8; text-shadow: 0 0 10px rgba(56,189,248,0.3); }
.p2-txt { color: #ec4899; text-shadow: 0 0 10px rgba(236,72,153,0.3); }
.lvs { font-size: 20px; letter-spacing: 2px; }

/* COMBO HUB NOTIFICATIONS */
#combo-container {
  position: fixed;
  top: 75px;
  left: 0;
  width: 100%;
  display: flex;
  justify-content: space-around;
  pointer-events: none;
}
.c-box { font-size: 16px; font-weight: 800; min-height: 20px; text-shadow: 0 2px 8px #000; }

#progress-bar{position:fixed;bottom:0;left:0;height:6px;z-index:5;width: 0%; background: linear-gradient(90deg, #38bdf8, #ec4899); transition: width 0.25s;}

/* OVERLAYS */
.ov{position:fixed;inset:0;z-index:20;display:flex;flex-direction:column;align-items:center;justify-content:center;background:rgba(15,23,42,0.96);backdrop-filter:blur(12px);color:#fff;text-align:center;padding:24px;}
.ov h1{font-size:34px;margin-bottom:8px;background: linear-gradient(90deg, #38bdf8, #ec4899); -webkit-background-clip: text; -webkit-text-fill-color: transparent; text-transform:uppercase; font-weight: 900;}
.ov p{font-size:14px;opacity:0.85;margin-bottom:14px;max-width:600px;line-height:1.7;}
.bb{padding:14px 40px;font-size:18px;font-weight:700;background:linear-gradient(135deg,#38bdf8,#2563eb,#ec4899);border:none;border-radius:50px;cursor:pointer;pointer-events:all;color:#fff;text-shadow:0 1px 4px #0005;box-shadow:0 4px 22px rgba(37,99,235,0.4);transition:transform 0.1s;margin-top:15px;}
.bb:hover{transform:scale(1.05);}

.name-inputs { display: flex; gap: 20px; margin: 15px 0; pointer-events: all; }
.name-inputs input { font-size: 16px; padding: 12px; border-radius: 10px; border: 2px solid #2563eb; background: rgba(255,255,255,0.05); color: #fff; text-align: center; outline: none; width: 200px; }
#p1-in { border-color: #38bdf8; }
#p2-in { border-color: #ec4899; }

#score-tbl,#score-tbl2{width:100%;max-width:450px;margin:10px auto;border-collapse:collapse;}
#score-tbl th,#score-tbl2 th{color:#38bdf8;font-size:13px;padding:6px 10px;border-bottom:1px solid #2563eb50;}
#score-tbl td,#score-tbl2 td{color:#fff;font-size:14px;padding:5px 10px;text-align:center;}

ul { text-align: left; max-width: 550px; margin: 10px auto 20px; line-height: 1.6; font-size: 13px; opacity: 0.9; }
li { margin-bottom: 6px; }

#lv-ov{position:fixed;inset:0;z-index:15;display:none;flex-direction:column;align-items:center;justify-content:center;background:rgba(15,23,42,0.95);backdrop-filter:blur(10px);color:#fff;text-align:center;padding:18px;}
#lv-ov h2{font-size:28px;margin-bottom:8px;color:#a855f7;}
#ai-msg{font-size:16px;font-weight: 600;max-width:500px;line-height:1.7;margin:15px auto;min-height:50px;color:#e2e8f0;}
#cnt-btn{padding:10px 32px;font-size:15px;background:#2563eb;border:none;border-radius:50px;color:#fff;cursor:pointer;pointer-events:all;margin-top:10px;font-weight:bold;box-shadow:0 4px 14px rgba(37,99,235,0.4);}
</style>
</head>
<body>
<div id="bg-tint"></div>
<canvas id="gc"></canvas>

<!-- UGLY MONSTER CENTER TARGET -->
<div id="monster-container" class="monster-idle">
  <img id="monster-gif" src="https://media.giphy.com/media/v1.Y2lkPTc5MGI3NjExM2g1cmZtYmt0bWh5ZGs3Z2xpYnV2ZWphNndpYmF3NXd5M3N0Nms3MCZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9cw/3Q35chD4bMtEeAqrS1/giphy.gif" alt="Grammar Monster">
</div>

<div id="ui">
  <!-- SPLIT SCREEN TOP BAR PANEL -->
  <div id="topbar">
    <div class="p-panel" id="p1-panel">
      <div><span class="lbl">🔵 Player 1:</span> <span id="p1-lbl-name" class="p1-txt">P1</span></div>
      <div><span id="p1-sc" class="sc-val p1-txt">0</span></div>
      <div id="p1-lives" class="lvs">❤️❤️❤️</div>
    </div>
    <div style="color: #64748b; font-weight: 900; padding: 0 10px; font-size: 14px;">WAVE <span id="lv">1</span></div>
    <div class="p-panel" id="p2-panel">
      <div id="p2-lives" class="lvs">❤️❤️❤️</div>
      <div><span id="p2-sc" class="sc-val p2-txt">0</span></div>
      <div><span class="lbl">Player 2 💗:</span> <span id="p2-lbl-name" class="p2-txt">P2</span></div>
    </div>
  </div>
  
  <div id="combo-container">
    <div id="p1-combo" class="c-box p1-txt"></div>
    <div id="p2-combo" class="c-box p2-txt"></div>
  </div>
</div>
<div id="progress-bar"></div>

<!-- NAME INGESTION SETUP OVERLAY -->
<div class="ov" id="name-ov">
  <h1>🛸 2-Player Grammar Commute Clash! 🛸</h1>
  <p style="margin-top: 5px; color:#38bdf8; font-weight:bold;">Co-op Battle Challenge! Clear your respective sides of the screen!</p>
  <ul>
    <li>👈 <b>Player 1 (Left Half)</b> clicks correct boxes falling down on the blue zone!</li>
    <li>👉 <b>Player 2 (Right Half)</b> clicks correct boxes falling down on the pink zone!</li>
    <li>🔵 <b>Target:</b> Click sentences featuring <b>CORRECT GRAMMAR</b> to hit the central monster boss.</li>
    <li>⚠️ <b>Traps:</b> Do NOT touch <b>BAD GRAMMAR</b> boxes. Let them slide off screen safely.</li>
    <li>❌ Misclicking a bad trap or letting a good sentence hit the floor drains that player's specific lives!</li>
  </ul>
  
  <div class="name-inputs">
    <input id="p1-in" type="text" maxlength="12" placeholder="Player 1 Name..." value="Player 1" />
    <input id="p2-in" type="text" maxlength="12" placeholder="Player 2 Name..." value="Player 2" />
  </div>
  
  <button class="bb" onclick="tryStart()">Commence Co-op Battle! 🚀</button>
  <div style="margin-top:20px;width:100%;max-width:450px">
    <p style="font-size:13px;color:#38bdf8;margin-bottom:5px;font-weight:bold;">🏆 High Score Team Records</p>
    <table id="score-tbl"><thead><tr><th>Rank</th><th>Matchup</th><th>Combined Score</th><th>Wave</th></tr></thead><tbody id="sb1"></tbody></table>
  </div>
</div>

<!-- GAME OVER OVERLAY -->
<div class="ov" id="go-ov" style="display:none">
  <h1>Battle Concluded! 🏁</h1>
  <div style="font-size:56px;margin:12px 0;font-weight:bold;color:#f43f5e;" id="fin-sc">0</div>
  <p id="fin-msg"></p>
  <div style="margin-top:15px;width:100%;max-width:450px">
    <p style="font-size:13px;color:#38bdf8;margin-bottom:5px;font-weight:bold;">🏆 High Score Team Records</p>
    <table id="score-tbl2"><thead><tr><th>Rank</th><th>Matchup</th><th>Combined Score</th><th>Wave</th></tr></thead><tbody id="sb2"></tbody></table>
  </div>
  <button class="bb" onclick="resetGame()">Battle Again! 🔄</button>
</div>

<!-- LEVEL UP OVERLAY -->
<div id="lv-ov">
  <h2 id="lv-title">Wave Clear! 🎉</h2>
  <div id="ai-msg">Synchronizing next segment vectors...</div>
  <button id="cnt-btn" onclick="continueGame()">Face Next Wave →</button>
</div>

<script>
// ═══════ GRAMMAR CONTENT POOLS ═══════
const correctSentences = [
    "Does your dad usually go to work by ass?",
    "No, he seldom does.",
    "Does your mom sometimes go to work by pig?",
    "Yes, she sometimes does.",
    "Does your brother always go to Mars by car?",
    "No, he never does.",
    "Does Mr.Wang often go to a park by garbage truck?",
    "Yes, he often does.",
    "Does your friend usually go to school by monkey?",
    "No, he never does."
];

const incorrectSentences = [
    "Do your father usually goes to work by ass?",
    "No, he usually goes does.",
    "Yes, he do sometimes.",
    "No, she doesn't never does.",
    "Yes, he does often goes.",
    "Does your usually go to school by elephant?",
    "No, it never do."
];

// ═══════ LOCAL STORAGE LEADERBOARD ═══════
function getScores(){try{return JSON.parse(localStorage.getItem('grammarCoopScores')||'[]');}catch{return[];}}
function saveScore(n,s,l){const a=getScores();a.push({n,s,l});a.sort((x,y)=>y.s-x.s);a.splice(10);localStorage.setItem('grammarCoopScores',JSON.stringify(a));}
function renderScores(id){
  const a=getScores(),tb=document.getElementById(id);
  const medals=['🥇','🥈','🥉'];
  tb.innerHTML=a.length?a.map((r,i)=>`<tr><td>${medals[i]||i+1}</td><td>${r.n}</td><td style="color:${i<3?['#FFD700','#C0C0C0','#CD7F32'][i]:'#fff'};font-weight:bold">${r.s}</td><td style="opacity:.7">Wave ${r.l}</td></tr>`).join(''):'<tr><td colspan="4" style="opacity:.5;padding:8px">No data loaded yet! 🎯</td></tr>';
}

// ═══════ PROCEDURAL AUDIO SYNTHESIZER ═══════
let ac=null,mTimer=null,mStep=0,soundReady=false;
function getAC(){if(!ac)ac=new(window.AudioContext||window.webkitAudioContext)();return ac;}

function tone(f,t,d,v=0.1,delay=0){
  if(!soundReady) return;
  try {
    const a=getAC(),o=a.createOscillator(),g=a.createGain(),now=a.currentTime;
    o.connect(g);g.connect(a.destination);
    o.type=t;o.frequency.value=f;
    g.gain.setValueAtTime(0,now+delay);
    g.gain.linearRampToValueAtTime(v,now+delay+0.01);
    g.gain.exponentialRampToValueAtTime(0.001,now+delay+d);
    o.start(now+delay);o.stop(now+delay+d+0.05);
  } catch(e){}
}

function startMusic(){
  if(!soundReady) return;
  stopMusic();
  mStep=0;
  mTimer=setInterval(()=>{
    const roots = [146, 164, 130, 146];
    const notes = [587, 659, 523, 784, 880, 659];
    const fMel=notes[mStep%notes.length];
    const fBass=roots[Math.floor(mStep/2)%roots.length];
    tone(fMel,'sine',0.3,0.03);
    tone(fBass,'triangle',0.4,0.05);
    if(mStep%4===0) tone(5000,'square',0.02,0.004);
    mStep++;
  },280);
}
function stopMusic(){if(mTimer){clearInterval(mTimer);mTimer=null;}}

function sfx(type){
  if(!soundReady)return;
  if(type==='catch'){tone(600,'sine',0.08,0.2);tone(900,'sine',0.12,0.15,0.05);}
  if(type==='miss'){tone(150,'sawtooth',0.2,0.25);tone(100,'sawtooth',0.25,0.2,0.07);}
  if(type==='lvup'){[523,659,784,1046].forEach((f,i)=>tone(f,'sine',0.15,0.2,i*0.06));}
}

// ═══════ CANVAS ENGINE SPLIT RENDERING ═══════
const gc=document.getElementById('gc');
const cx=gc.getContext('2d');
let W=window.innerWidth, H=window.innerHeight;
gc.width=W; gc.height=H;

window.addEventListener('resize', ()=>{
  W=window.innerWidth; H=window.innerHeight;
  gc.width=W; gc.height=H;
});

let running=false, frame=0, level=1;
let p1Name='P1', p2Name='P2';
let p1Score=0, p2Score=0;
let p1Lives=3, p2Lives=3;
let p1Combo=0, p2Combo=0;
let levelProgressScore=0, levelTargetScore=10;

let items=[], parts=[];
const stars=Array.from({length:45},()=>({x:Math.random()*W,y:Math.random()*H,s:Math.random()*1.5+0.5,b:Math.random()*10}));

function updateLivesDisplays(){
  const hearts=['','❤️','❤️❤️','❤️❤️❤️'];
  document.getElementById('p1-lives').textContent=hearts[Math.max(0,p1Lives)];
  document.getElementById('p2-lives').textContent=hearts[Math.max(0,p2Lives)];
}

function updateProgressBar(){
  const total = p1Score + p2Score;
  document.getElementById('progress-bar').style.width = Math.min(100, (levelProgressScore / levelTargetScore) * 100) + '%';
}

function triggerMonsterHitEffect(side, isGoodClick){
  const el = document.getElementById('monster-container');
  el.classList.remove('monster-idle');
  
  if(isGoodClick){
    el.style.filter = side === 1 
      ? "hue-rotate(60deg) brightness(1.4) drop-shadow(-20px 0 20px #38bdf8)"
      : "hue-rotate(300deg) brightness(1.4) drop-shadow(20px 0 20px #ec4899)";
    el.style.transform = side === 1 
      ? "translateX(-35%) scale(0.85) rotate(10deg)" 
      : "translateX(-65%) scale(0.85) rotate(-10deg)";
  } else {
    el.style.filter = "brightness(0.6) drop-shadow(0 0 25px #ef4444)";
    el.style.transform = "translateX(-50%) scale(1.25) translateY(-15px)";
  }
  
  setTimeout(()=>{
    el.style.filter = "none";
    el.style.transform = "translateX(-50%) scale(1) translateY(0) rotate(0deg)";
    el.classList.add('monster-idle');
  }, 220);
}

function spawnItemForPlayer(playerNum){
  const isCorrect = Math.random() < 0.55;
  const text = isCorrect 
    ? correctSentences[Math.floor(Math.random()*correctSentences.length)]
    : incorrectSentences[Math.floor(Math.random()*incorrectSentences.length)];
    
  const baseSpeed = 2.5 + (level * 0.12) + Math.random() * 0.3;
  const boxWidth = 360; 
  const boxHeight = 68;
  
  // Constrain bounds to left vs right half channels safely
  let minX = 20;
  let maxX = (W / 2) - boxWidth - 30;
  if(playerNum === 2){
    minX = (W / 2) + 30;
    maxX = W - boxWidth - 20;
  }
  
  const spawnX = minX + Math.random() * Math.max(10, (maxX - minX));
  
  items.push({
    player: playerNum,
    x: spawnX,
    y: -80,
    text: text,
    isCorrect: isCorrect,
    spd: baseSpeed,
    w: boxWidth,
    h: boxHeight,
    waveOffset: Math.random() * Math.PI * 2,
    alpha: 1,
    dead: false
  });
}

function burstEffect(x,y,color,count=12){
  for(let i=0;i<count;i++){
    const angle=(i/count)*Math.PI*2;
    const speed=2+Math.random()*4;
    parts.push({
      x, y,
      vx: Math.cos(angle)*speed,
      vy: Math.sin(angle)*speed - 1,
      color: color,
      life: 1,
      decay: 0.04 + Math.random()*0.03,
      size: 3 + Math.random()*4
    });
  }
}

// ═══════ DUAL CLICK DETECTOR ═══════
gc.addEventListener('mousedown', (e)=>{
  if(!running) return;
  const rect = gc.getBoundingClientRect();
  const mx = e.clientX - rect.left;
  const my = e.clientY - rect.top;
  
  for(let it of items){
    if(it.dead) continue;
    
    if(mx >= it.x && mx <= it.x + it.w && my >= it.y && my <= it.y + it.h){
      it.dead = true;
      const themeColor = it.player === 1 ? '#38bdf8' : '#ec4899';
      
      if(it.isCorrect){
        levelProgressScore++;
        sfx('catch');
        burstEffect(mx, my, themeColor, 15);
        triggerMonsterHitEffect(it.player, true);
        
        if(it.player === 1){
          p1Combo++;
          p1Score += 10 * (1 + Math.floor(p1Combo/4));
          document.getElementById('p1-sc').textContent = p1Score;
          if(p1Combo >= 4) document.getElementById('p1-combo').textContent = `🔥 STREAK x${p1Combo}`;
        } else {
          p2Combo++;
          p2Score += 10 * (1 + Math.floor(p2Combo/4));
          document.getElementById('p2-sc').textContent = p2Score;
          if(p2Combo >= 4) document.getElementById('p2-combo').textContent = `🔥 STREAK x${p2Combo}`;
        }
        
        updateProgressBar();
        if(levelProgressScore >= levelTargetScore){ showLevelUp(); return; }
      } else {
        // Punish individual side for bad clicks
        sfx('miss');
        burstEffect(mx, my, '#ef4444', 20);
        triggerMonsterHitEffect(it.player, false);
        
        if(it.player === 1){
          p1Combo = 0; p1Lives--;
          document.getElementById('p1-combo').textContent = '';
        } else {
          p2Combo = 0; p2Lives--;
          document.getElementById('p2-combo').textContent = '';
        }
        updateLivesDisplays();
        if(p1Lives <= 0 || p2Lives <= 0){ endGame(); return; }
      }
      break;
    }
  }
});

// ═══════ CANVAS GAME RENDER LOOP ═══════
function loop(){
  if(!running) return;
  frame++;
  cx.clearRect(0,0,W,H);
  
  // Ambient Starscape
  stars.forEach(s=>{
    const alpha=0.15+0.35*Math.abs(Math.sin(frame*0.008+s.b));
    cx.fillStyle=`rgba(148,163,184,${alpha})`;
    cx.fillRect(s.x,s.y,s.s,s.s);
  });
  
  // Draw Central Split Screen Dotted Guide line
  cx.strokeStyle = "rgba(255, 255, 255, 0.08)";
  cx.lineWidth = 2;
  cx.setLineDash([8, 12]);
  cx.beginPath();
  cx.moveTo(W/2, 0);
  cx.lineTo(W/2, H);
  cx.stroke();
  cx.setLineDash([]); // Reset line formatting
  
  // Balanced Spawn Loops
  const rate = Math.max(40, 100 - level * 7);
  if(frame % rate === 0){
    spawnItemForPlayer(1);
    spawnItemForPlayer(2);
  }
  
  // Track falling elements
  for(let it of items){
    if(!it.dead){
      it.y += it.spd;
      it.waveOffset += 0.015;
      it.x += Math.sin(it.waveOffset) * 0.25;
      
      // Let it escape out the bottom checks
      if(it.y > H){
        it.dead = true;
        if(it.isCorrect){
          sfx('miss');
          triggerMonsterHitEffect(it.player, false);
          if(it.player === 1){
            p1Combo = 0; p1Lives--;
            document.getElementById('p1-combo').textContent = '';
          } else {
            p2Combo = 0; p2Lives--;
            document.getElementById('p2-combo').textContent = '';
          }
          updateLivesDisplays();
          if(p1Lives <= 0 || p2Lives <= 0){ endGame(); return; }
        }
        continue;
      }
    } else {
      it.alpha -= 0.1;
    }
    
    if(it.alpha > 0){
      cx.save();
      cx.globalAlpha = it.alpha;
      
      const sideColor = it.player === 1 ? '#38bdf8' : '#ec4899';
      cx.shadowColor = it.isCorrect ? sideColor : '#b91c1c';
      cx.shadowBlur = it.isCorrect ? 14 : 4;
      
      cx.fillStyle = 'rgba(15, 23, 42, 0.92)';
      cx.strokeStyle = it.isCorrect ? sideColor : '#ef4444';
      cx.lineWidth = 2.5;
      
      cx.beginPath();
      cx.roundRect(it.x, it.y, it.w, it.h, 12);
      cx.fill();
      cx.stroke();
      
      cx.shadowBlur = 0;
      cx.fillStyle = '#ffffff';
      cx.font = 'bold 13px sans-serif';
      cx.textAlign = 'center';
      cx.textBaseline = 'middle';
      cx.fillText(it.text, it.x + it.w/2, it.y + it.h/2, it.w - 20);
      
      cx.restore();
    }
  }
  
  items = items.filter(i => i.alpha > 0 && i.y <= H);
  
  // Handle Explosive Splatters
  for(let p of parts){
    p.x += p.vx; p.y += p.vy;
    p.vy += 0.12; p.life -= p.decay;
    if(p.life <= 0) continue;
    cx.save();
    cx.globalAlpha = p.life;
    cx.fillStyle = p.color;
    cx.beginPath();
    cx.arc(p.x, p.y, p.size * p.life, 0, Math.PI*2);
    cx.fill();
    cx.restore();
  }
  parts = parts.filter(p => p.life > 0);
  
  requestAnimationFrame(loop);
}

// ═══════ STATE SCENARIO TRANSITIONS ═══════
function showLevelUp(){
  running = false;
  stopMusic();
  sfx('lvup');
  document.getElementById('lv-title').textContent = `Wave ${level} Conquered! 🌌`;
  document.getElementById('ai-msg').textContent = `Awesome synchronization! Together, ${p1Name} and ${p2Name} pushed the monster back. Prepare coordinates, next wave is faster!`;
  document.getElementById('lv-ov').style.display = 'flex';
}

function continueGame(){
  document.getElementById('lv-ov').style.display = 'none';
  level++;
  levelProgressScore = 0;
  levelTargetScore = 10 + level * 3;
  document.getElementById('lv').textContent = level;
  running = true;
  updateProgressBar();
  startMusic();
  requestAnimationFrame(loop);
}

function endGame(){
  running = false;
  stopMusic();
  const totalCombinedScore = p1Score + p2Score;
  const dynamicMatchupName = `${p1Name} ⚔️ ${p2Name}`;
  saveScore(dynamicMatchupName, totalCombinedScore, level);
  
  document.getElementById('fin-sc').textContent = totalCombinedScore;
  document.getElementById('fin-msg').textContent = `The monster breached defenses on Wave ${level}! ${p1Name} scored ${p1Score} | ${p2Name} scored ${p2Score}. Team up again to smash the highscore!`;
  renderScores('sb2');
  document.getElementById('go-ov').style.display = 'flex';
}

function resetGame(){
  document.getElementById('go-ov').style.display = 'none';
  renderScores('sb1');
  document.getElementById('name-ov').style.display = 'flex';
}

function tryStart(){
  p1Name = document.getElementById('p1-in').value.trim() || 'Player 1';
  p2Name = document.getElementById('p2-in').value.trim() || 'Player 2';
  
  document.getElementById('p1-lbl-name').textContent = p1Name;
  document.getElementById('p2-lbl-name').textContent = p2Name;
  
  soundReady = true;
  getAC().resume();
  
  document.getElementById('name-ov').style.display = 'none';
  
  p1Score = 0; p2Score = 0;
  p1Lives = 3; p2Lives = 3;
  p1Combo = 0; p2Combo = 0;
  level = 1; frame = 0; levelProgressScore = 0; levelTargetScore = 10;
  items = []; parts = [];
  
  document.getElementById('p1-sc').textContent = 0;
  document.getElementById('p2-sc').textContent = 0;
  document.getElementById('lv').textContent = 1;
  document.getElementById('p1-combo').textContent = '';
  document.getElementById('p2-combo').textContent = '';
  
  updateLivesDisplays();
  updateProgressBar();
  startMusic();
  
  running = true;
  requestAnimationFrame(loop);
}

renderScores('sb1');
</script>
</body>
</html>
