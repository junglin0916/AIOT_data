
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>The True or False Grammar Clash!</title>
<style>
*{margin:0;padding:0;box-sizing:border-box;}
body{background:#0b0f19;font-family:'Segoe UI', Roboto, sans-serif;overflow:hidden;user-select:none;}
#bg-tint{position:fixed;inset:0;z-index:1;background:rgba(15, 23, 42, 0.4);pointer-events:none;}
canvas{position:fixed;inset:0;z-index:2;cursor: crosshair;}
#ui{position:fixed;inset:0;z-index:3;pointer-events:none;}
#topbar{display:flex;justify-content:space-between;align-items:center;padding:12px 20px;background:rgba(15, 23, 42, 0.75);backdrop-filter:blur(8px);border-bottom: 2px solid #2563eb;}
#score-box{color:#fff;font-size:18px;font-weight:700;text-shadow:0 2px 8px #000;}
#lives{font-size:24px;letter-spacing:3px;}
#combo{color:#fbbf24;font-size:15px;font-weight:700;min-height:18px;text-align:center;text-shadow:0 2px 8px #000;margin-top:6px;}

#turn-bar {
  display: flex;
  justify-content: center;
  padding: 8px;
}
#turn-indicator {
  padding: 6px 24px;
  font-size: 16px;
  font-weight: bold;
  border-radius: 20px;
  color: white;
  text-transform: uppercase;
  letter-spacing: 1px;
  box-shadow: 0 4px 12px rgba(0,0,0,0.3);
  transition: all 0.4s ease;
}
.p1-active { background: #38bdf8; color: #020617 !important; box-shadow: 0 0 15px #38bdf880 !important; }
.p2-active { background: #ec4899; color: #ffffff !important; box-shadow: 0 0 15px #ec489980 !important; }

#progress-bar{position:fixed;bottom:0;left:0;height:6px;z-index:4;transition:width 0.25s, background 0.5s;}

/* OVERLAYS */
.ov{position:fixed;inset:0;z-index:20;display:flex;flex-direction:column;align-items:center;justify-content:center;background:rgba(15,23,42,0.95);backdrop-filter:blur(12px);color:#fff;text-align:center;padding:24px;}
.ov h1{font-size:32px;margin-bottom:8px;color:#38bdf8;text-transform:uppercase;}
.ov p{font-size:14px;opacity:0.85;margin-bottom:14px;max-width:550px;line-height:1.7;}
.bb{padding:14px 36px;font-size:18px;font-weight:700;background:linear-gradient(135deg,#38bdf8,#2563eb,#ec4899);border:none;border-radius:50px;cursor:pointer;pointer-events:all;color:#fff;text-shadow:0 1px 4px #0005;box-shadow:0 4px 22px rgba(37,99,235,0.4);transition:transform 0.1s;margin-top:10px;}
.bb:hover{transform:scale(1.05);}

#name-in{font-size:20px;padding:10px 20px;border-radius:12px;border:2px solid #2563eb;background:rgba(255,255,255,0.05);color:#fff;text-align:center;outline:none;width:280px;margin:12px 0;pointer-events:all;}
#name-in::placeholder{color:rgba(255,255,255,0.3);}
#score-tbl,#score-tbl2{width:100%;max-width:400px;margin:10px auto;border-collapse:collapse;}
#score-tbl th,#score-tbl2 th{color:#38bdf8;font-size:13px;padding:6px 10px;border-bottom:1px solid #2563eb50;}
#score-tbl td,#score-tbl2 td{color:#fff;font-size:14px;padding:5px 10px;text-align:center;}
#score-tbl tr:nth-child(even) td,#score-tbl2 tr:nth-child(even) td{background:rgba(255,255,255,0.03);}

ul { text-align: left; max-width: 500px; margin: 10px auto 20px; line-height: 1.6; font-size: 13px; opacity: 0.9; }
li { margin-bottom: 6px; }

/* Level overlay */
#lv-ov{position:fixed;inset:0;z-index:15;display:none;flex-direction:column;align-items:center;justify-content:center;background:rgba(15,23,42,0.95);backdrop-filter:blur(10px);color:#fff;text-align:center;padding:18px;}
#lv-ov h2{font-size:28px;margin-bottom:8px;color:#a855f7;}
#ai-msg{font-size:16px;font-weight: 600;max-width:500px;line-height:1.7;margin:15px auto;min-height:50px;color:#e2e8f0;}
#cnt-btn{padding:10px 32px;font-size:15px;background:#2563eb;border:none;border-radius:50px;color:#fff;cursor:pointer;pointer-events:all;margin-top:10px;font-weight:bold;box-shadow:0 4px 14px rgba(37,99,235,0.4);}
#cnt-btn:hover{transform:scale(1.05);}
</style>
</head>
<body>
<div id="bg-tint"></div>
<canvas id="gc"></canvas>
<div id="ui">
  <div id="topbar">
    <div id="score-box"><span id="pname"></span> Score: <span id="sc">0</span> | Level: <span id="lv">1</span></div>
    <div id="lives">❤️❤️❤️</div>
  </div>
  <div id="turn-bar">
    <div id="turn-indicator" class="p1-active">PLAYER 1's TURN</div>
  </div>
  <div id="combo"></div>
</div>
<div id="progress-bar"></div>

<!-- TEAM NAME OVERLAY -->
<div class="ov" id="name-ov">
  <h1>🛸 The Grammar Commute Clash! 🛸</h1>
  <p style="margin-top: 5px; color:#38bdf8; font-weight:bold;">Click the CORRECT sentences! Avoid the BAD grammar traps.</p>
  <ul>
    <li>🔵 <b>Player 1 (Blue)</b> and 💗 <b>Player 2 (Pink)</b> automatically alternate turns every 9 seconds!</li>
    <li>👉 <b>Action:</b> Click falling items containing <b>GOOD GRAMMAR</b> (e.g., <i>"Does he go...?"</i>) for points.</li>
    <li>⚠️ <b>Warning:</b> DO NOT click <b>BAD GRAMMAR</b> boxes! Letting them pass safely is correct.</li>
    <li>❌ Clicking bad grammar or letting a correct sentence slip past the bottom costs 1 Life!</li>
  </ul>
  <p style="font-size:13px;color:#94a3b8;margin-bottom:2px">Enter a Team/Player Name for the Leaderboard:</p>
  <input id="name-in" type="text" maxlength="16" placeholder="Team Name..." />
  <button class="bb" onclick="tryStart()">Launch Battle! 🚀</button>
  <div style="margin-top:20px;width:100%;max-width:400px">
    <p style="font-size:13px;color:#38bdf8;margin-bottom:5px;font-weight:bold;">🏆 High Score Leaderboard</p>
    <table id="score-tbl"><thead><tr><th>Rank</th><th>Name</th><th>Score</th><th>Level</th></tr></thead><tbody id="sb1"></tbody></table>
  </div>
</div>

<!-- GAME OVER OVERLAY -->
<div class="ov" id="go-ov" style="display:none">
  <h1>Battle Concluded! 🏁</h1>
  <div style="font-size:56px;margin:12px 0;font-weight:bold;color:#f43f5e;" id="fin-sc">0</div>
  <p id="fin-msg"></p>
  <div style="margin-top:15px;width:100%;max-width:400px">
    <p style="font-size:13px;color:#38bdf8;margin-bottom:5px;font-weight:bold;">🏆 High Score Leaderboard</p>
    <table id="score-tbl2"><thead><tr><th>Rank</th><th>Name</th><th>Score</th><th>Level</th></tr></thead><tbody id="sb2"></tbody></table>
  </div>
  <button class="bb" onclick="resetGame()">Battle Again! 🔄</button>
</div>

<!-- LEVEL UP OVERLAY -->
<div id="lv-ov">
  <h2 id="lv-title">Level Complete! 🎉</h2>
  <div id="ai-msg">Loading next challenge parameters...</div>
  <button id="cnt-btn" onclick="continueGame()">Continue Next Wave →</button>
</div>

<script>
// ═══════ GRAMMAR CONTENT POOLS ═══════
const correctSentences = [
    "Does your father usually go to work by elephant?",
    "No, he seldom does.",
    "Does your mom sometimes go to work by pig?",
    "Yes, she sometimes does.",
    "Does your brother always go to Mars by car?",
    "No, he never does.",
    "Does the Principal often go to a post office by garbage truck?",
    "Yes, he often does.",
    "Does your friend usually go to school by monkey?",
    "No, he never does."
];

const incorrectSentences = [
    "Do your father usually goes to work by elephant?",
    "No, he usually goes does.",
    "Yes, he do sometimes.",
    "No, she doesn't rarely does.",
    "Yes, he does often goes.",
    "Does your usually go to school by elephant?",
    "No, it never do."
];

// ═══════ LOCAL STORAGE LEADERBOARD ═══════
function getScores(){try{return JSON.parse(localStorage.getItem('grammarDanceScores')||'[]');}catch{return[];}}
function saveScore(n,s,l){const a=getScores();a.push({n,s,l});a.sort((x,y)=>y.s-x.s);a.splice(10);localStorage.setItem('grammarDanceScores',JSON.stringify(a));}
function renderScores(id){
  const a=getScores(),tb=document.getElementById(id);
  const medals=['🥇','🥈','🥉'];
  tb.innerHTML=a.length?a.map((r,i)=>`<tr><td>${medals[i]||i+1}</td><td>${r.n}</td><td style="color:${i<3?['#FFD700','#C0C0C0','#CD7F32'][i]:'#fff'};font-weight:bold">${r.s}</td><td style="opacity:.7">Lv${r.l}</td></tr>`).join(''):'<tr><td colspan="4" style="opacity:.5;padding:8px">No scores uploaded yet! 🎯</td></tr>';
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

// Music loop configuration based on active player tracking
const PAT={
  p1:{bpm:110,st:[523, 587, 659, 784],bs:[130, 130, 146, 164]},
  p2:{bpm:115,st:[587, 698, 880, 987],bs:[146, 146, 174, 220]}
};

function startMusic(mode){
  if(!soundReady) return;
  stopMusic();
  mStep=0;
  const p=PAT[mode],ms=(60/p.bpm)*1000;
  mTimer=setInterval(()=>{
    const fMel=p.st[mStep%p.st.length];
    const fBass=p.bs[mStep%p.bs.length];
    tone(fMel,'sine',ms/1000,0.04);
    tone(fBass,'triangle',ms/800,0.06);
    if(mStep%4===0) tone(6000,'square',0.02,0.005);
    mStep++;
  },ms);
}
function stopMusic(){if(mTimer){clearInterval(mTimer);mTimer=null;}}

function sfx(type){
  if(!soundReady)return;
  if(type==='catch'){tone(587,'sine',0.08,0.25);tone(880,'sine',0.12,0.2,0.06);}
  else if(type==='miss'){tone(180,'sawtooth',0.2,0.25);tone(120,'sawtooth',0.25,0.2,0.08);}
  else if(type==='lvup'){[523,659,784,1046].forEach((f,i)=>tone(f,'sine',0.15,0.25,i*0.08));}
  else if(type==='combo'){tone(880,'sine',0.08,0.3);tone(1174,'sine',0.1,0.25,0.05);}
}

// ═══════ CANVAS GAME ENGINE ═══════
const gc=document.getElementById('gc');
const cx=gc.getContext('2d');
let W=window.innerWidth, H=window.innerHeight;
gc.width=W; gc.height=H;

window.addEventListener('resize', ()=>{
  W=window.innerWidth; H=window.innerHeight;
  gc.width=W; gc.height=H;
});

let score=0,lives=3,combo=0,level=1,running=false,frame=0;
let levelScore=0,levelTarget=6,items=[],parts=[];
let currentPlayer=1,playerName='Team',lastTurnSwap=0;

const stars=Array.from({length:40},()=>({x:Math.random()*W,y:Math.random()*H,s:Math.random()*1.5+0.5,b:Math.random()*10}));

function updateLivesDisplay(){const h=['','❤️','❤️❤️','❤️❤️❤️'];document.getElementById('lives').textContent=h[Math.max(0,lives)];}
function updateProgressBar(){const b=document.getElementById('progress-bar');b.style.width=((levelScore/levelTarget)*100)+'%';b.style.background=currentPlayer===1?'#38bdf8':'#ec4899';}

function switchPlayerTurn(){
  currentPlayer = currentPlayer === 1 ? 2 : 1;
  const indicator = document.getElementById('turn-indicator');
  if(currentPlayer === 1){
    indicator.textContent = "PLAYER 1's TURN";
    indicator.className = "p1-active";
    startMusic('p1');
  } else {
    indicator.textContent = "PLAYER 2's TURN";
    indicator.className = "p2-active";
    startMusic('p2');
  }
  updateProgressBar();
}

function spawnItem(){
  const isCorrect = Math.random() < 0.55;
  const text = isCorrect 
    ? correctSentences[Math.floor(Math.random()*correctSentences.length)]
    : incorrectSentences[Math.floor(Math.random()*incorrectSentences.length)];
    
  // Balanced slower downward sliding velocities
  const baseSpeed = 1.2 + (level * 0.15) + Math.random() * 0.4;
  
  // Create full metrics for dynamic clickable boxes on canvas
  items.push({
    x: 40 + Math.random() * (W - 360),
    y: -50,
    text: text,
    isCorrect: isCorrect,
    spd: baseSpeed,
    w: 290,
    h: 54,
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
      size: 3 + Math.random()*5
    });
  }
}

// ═══════ INTERACTIVE CLICK LOGIC ═══════
gc.addEventListener('mousedown', (e)=>{
  if(!running) return;
  
  const rect = gc.getBoundingClientRect();
  const mx = e.clientX - rect.left;
  const my = e.clientY - rect.top;
  
  let hitDetected = false;
  
  for(let it of items){
    if(it.dead) continue;
    
    // Check box alignment boundaries
    if(mx >= it.x && mx <= it.x + it.w && my >= it.y && my <= it.y + it.h){
      it.dead = true;
      hitDetected = true;
      
      if(it.isCorrect){
        // Successfully caught correct grammar item
        score += 10 * (1 + Math.floor(combo/4));
        combo++;
        levelScore++;
        sfx(combo % 4 === 0 ? 'combo' : 'catch');
        burstEffect(mx, my, currentPlayer === 1 ? '#38bdf8' : '#ec4899', 18);
        document.getElementById('sc').textContent = score;
        
        if(combo % 4 === 0){
          const cbEl = document.getElementById('combo');
          cbEl.textContent = `🔥 ${combo}x STREAK MULTIPLIER!`;
          setTimeout(()=>cbEl.textContent='', 1500);
        }
        updateProgressBar();
        if(levelScore >= levelTarget){ showLevelUp(); return; }
      } else {
        // Trigger penalty for clicking bad grammar items
        combo = 0;
        lives--;
        sfx('miss');
        burstEffect(mx, my, '#ef4444', 20);
        document.getElementById('combo').textContent = '';
        updateLivesDisplay();
        if(lives <= 0){ endGame(); return; }
      }
      break;
    }
  }
});

// ═══════ MAIN RENDER LOOP ═══════
function loop(){
  if(!running)return;
  frame++;
  cx.clearRect(0,0,W,H);
  
  // Stars Ambient Background
  stars.forEach(s=>{
    const alpha=0.15+0.35*Math.abs(Math.sin(frame*0.008+s.b));
    cx.fillStyle=`rgba(148,163,184,${alpha})`;
    cx.fillRect(s.x,s.y,s.s,s.s);
  });
  
  // Game Loop Tickers
  if(frame % Math.max(35, 110 - level * 8) === 0) spawnItem();
  
  // Automated player turn swap cycle every 9 seconds (540 frames @60Hz)
  if(frame - lastTurnSwap >= 540){
    switchPlayerTurn();
    lastTurnSwap = frame;
  }
  
  // Render Item Elements
  for(let it of items){
    if(!it.dead){
      it.y += it.spd;
      it.waveOffset += 0.02;
      it.x += Math.sin(it.waveOffset) * 0.4;
      
      // Check Floor Overflow Failures
      if(it.y > H){
        it.dead = true;
        if(it.isCorrect){
          // Dropped a good sentence without clicking it!
          combo = 0;
          lives--;
          sfx('miss');
          document.getElementById('combo').textContent = '';
          updateLivesDisplay();
          if(lives <= 0){ endGame(); return; }
        }
        continue;
      }
    } else {
      it.alpha -= 0.1;
    }
    
    if(it.alpha > 0){
      cx.save();
      cx.globalAlpha = it.alpha;
      
      // Dynamic Glowing Frame Setup
      const activeThemeColor = currentPlayer === 1 ? '#38bdf8' : '#ec4899';
      cx.shadowColor = it.isCorrect ? activeThemeColor : '#b91c1c';
      cx.shadowBlur = it.isCorrect ? 16 : 4;
      
      // Draw Box Background
      cx.fillStyle = 'rgba(15, 23, 42, 0.9)';
      cx.strokeStyle = it.isCorrect ? activeThemeColor : '#ef4444';
      cx.lineWidth = 2.5;
      
      // Rounded Rectangle Box Path
      cx.beginPath();
      cx.roundRect(it.x, it.y, it.w, it.h, 12);
      cx.fill();
      cx.stroke();
      
      // Text Drawing Settings
      cx.shadowBlur = 0;
      cx.fillStyle = '#ffffff';
      cx.font = 'bold 13px sans-serif';
      cx.textAlign = 'center';
      cx.textBaseline = 'middle';
      
      // Wrap Text strings gracefully inside the canvas boundary box
      cx.fillText(it.text, it.x + it.w/2, it.y + it.h/2, it.w - 20);
      cx.restore();
    }
  }
  
  // Filter out expired items
  items = items.filter(i => i.alpha > 0 && i.y <= H);
  
  // Particle Systems Render
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

// ═══════ LEVEL ADVANCEMENT ═══════
function showLevelUp(){
  running = false;
  stopMusic();
  sfx('lvup');
  document.getElementById('lv-title').textContent = `Wave ${level} Over! 🌌`;
  document.getElementById('ai-msg').textContent = `Excellent work, ${playerName}! Your grammar sensors handled that speed efficiently. Ready for the next wave?`;
  document.getElementById('lv-ov').style.display = 'flex';
}

function continueGame(){
  document.getElementById('lv-ov').style.display = 'none';
  level++;
  levelScore = 0;
  levelTarget = 6 + level * 2;
  document.getElementById('lv').textContent = level;
  running = true;
  lastTurnSwap = frame;
  updateProgressBar();
  startMusic(currentPlayer === 1 ? 'p1' : 'p2');
  requestAnimationFrame(loop);
}

// ═══════ GAME TERMINATION ═══════
function endGame(){
  running = false;
  stopMusic();
  saveScore(playerName, score, level);
  document.getElementById('fin-sc').textContent = score;
  document.getElementById('fin-msg').textContent = `Fantastic attempt, ${playerName}! You managed to navigate all the way to Level ${level}! Keep practicing those "Does" and "Do" question structures! 🛸`;
  renderScores('sb2');
  document.getElementById('go-ov').style.display = 'flex';
}

function resetGame(){
  document.getElementById('go-ov').style.display = 'none';
  renderScores('sb1');
  document.getElementById('name-ov').style.display = 'flex';
}

// ═══════ ENGAGEMENT INITIALIZATION ═══════
function tryStart(){
  const n = document.getElementById('name-in').value.trim();
  if(!n){
    document.getElementById('name-in').style.borderColor = '#ef4444';
    document.getElementById('name-in').placeholder = 'Enter a team name!';
    return;
  }
  playerName = n;
  document.getElementById('pname').textContent = n + ' |';
  
  // Safe user activation gesture to kick off internal synthesizers
  soundReady = true;
  getAC().resume();
  
  document.getElementById('name-ov').style.display = 'none';
  score = 0; lives = 3; combo = 0; level = 1; frame = 0; levelScore = 0; levelTarget = 6;
  items = []; parts = [];
  currentPlayer = 1;
  lastTurnSwap = 0;
  
  document.getElementById('sc').textContent = 0;
  document.getElementById('lv').textContent = 1;
  document.getElementById('combo').textContent = '';
  
  const indicator = document.getElementById('turn-indicator');
  indicator.textContent = "PLAYER 1's TURN";
  indicator.className = "p1-active";
  
  updateLivesDisplay();
  updateProgressBar();
  startMusic('p1');
  
  running = true;
  requestAnimationFrame(loop);
}

document.getElementById('name-in').addEventListener('keydown', (e)=>{ if(e.key==='Enter') tryStart(); });
renderScores('sb1');
</script>
</body>
</html>
