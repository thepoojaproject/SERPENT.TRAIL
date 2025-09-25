
<html lang="en">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>SERPENT'S TRAIL</title>
<style>
:root{
  --bg:#071029;
  --card:#0c1a2b;
  --accent1:#22c1c3;
  --accent2:#f7797d;
  --muted:#9fb0c8;
  --snake-grad1:#0f0;
  --snake-grad2:#00ff88;
  --food-color:#ff2a68;
  font-family: Inter, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial;
}
html,body{height:100%;margin:0;background:linear-gradient(180deg,var(--bg), #04121a);color:#e6f3f7;overflow:hidden;}
.wrap{min-height:100%;display:flex;align-items:center;justify-content:center;padding:28px;box-sizing:border-box;transition:background 1s;}
.card{width:100%;max-width:960px;background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));border-radius:20px;padding:20px;box-shadow:0 15px 40px rgba(0,0,0,0.7);backdrop-filter:blur(6px);}
header{display:flex;align-items:center;justify-content:space-between;gap:12px;margin-bottom:12px}
h1{font-size:22px;margin:0;text-shadow:0 0 10px #0ff;}
.controls{display:flex;gap:8px;align-items:center;flex-wrap:wrap}
button, select{background:transparent;border:1px solid rgba(255,255,255,0.08);color:inherit;padding:8px 12px;border-radius:12px;cursor:pointer;backdrop-filter: blur(4px);transition:0.3s;}
button:hover, select:hover{border-color:rgba(255,255,255,0.2);box-shadow:0 0 8px var(--accent1);}
.game-row{display:flex;gap:16px;align-items:flex-start}
.canvas-wrap{background:linear-gradient(135deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));padding:12px;border-radius:16px;display:flex;align-items:center;justify-content:center;box-shadow:0 0 20px rgba(0,255,200,0.1);}
canvas{background:linear-gradient(180deg, rgba(2,10,20,0.8), rgba(4,18,30,0.8));border-radius:12px;display:block;max-width:100%;box-shadow:0 0 15px rgba(0,255,0,0.2);}
.sidebar{width:300px;max-width:34%;display:flex;flex-direction:column;gap:12px;}
.panel{background:rgba(255,255,255,0.02);padding:14px;border-radius:12px;border:1px solid rgba(255,255,255,0.04);backdrop-filter:blur(6px);box-shadow:0 5px 15px rgba(0,0,0,0.3);}
.stat{display:flex;justify-content:space-between;font-weight:700;font-size:16px;}
.muted{color:var(--muted);font-size:13px}
.footer{display:flex;justify-content:center;align-items:center;margin-top:10px;color:var(--muted);font-size:13px;text-align:center;}
@media(max-width:900px){.game-row{flex-direction:column-reverse}.sidebar{width:100%;max-width:none}}
</style>
</head>
<body>
<div class="wrap">
  <div class="card">
    <header>
      <h1>🐍 SERPENT’S TRAIL</h1>
      <div class="controls">
        <button id="btnStart">Start</button>
        <button id="btnPause">Pause</button>
        <button id="btnRestart">Restart</button>
        <label class="muted" style="padding-left:6px">Speed:</label>
        <select id="speedSel">
          <option value="6">Relax (Slow)</option>
          <option value="9" selected>Normal</option>
          <option value="13">Fast</option>
          <option value="18">Extreme</option>
        </select>
      </div>
    </header>

    <div class="game-row">
      <div class="canvas-wrap" style="flex:1">
        <canvas id="gameCanvas" width="720" height="720"></canvas>
      </div>

      <aside class="sidebar">
        <div class="panel">
          <div class="stat"><span>Score</span><span id="score">0</span></div>
          <div class="stat" style="margin-top:6px"><span>High Score</span><span id="highscore">0</span></div>
          <div style="margin-top:8px" class="muted">Arrow keys / WASD or swipe. Space = pause.</div>
        </div>
        <div class="panel">
          <div style="font-weight:700;margin-bottom:6px">Leaderboard (Top 5)</div>
          <ol id="leaderboard" style="margin:8px 0 0 18px;padding:0;color:#dff6ff"></ol>
        </div>
      </aside>
    </div>

    <div class="footer">Made with ❤ by Armeen</div>
  </div>
</div>

<script>
const canvas=document.getElementById('gameCanvas');
const ctx=canvas.getContext('2d');
const scoreEl=document.getElementById('score');
const highEl=document.getElementById('highscore');
const lbEl=document.getElementById('leaderboard');
const btnStart=document.getElementById('btnStart');
const btnPause=document.getElementById('btnPause');
const btnRestart=document.getElementById('btnRestart');
const speedSel=document.getElementById('speedSel');

let grid=24,cols=canvas.width/grid,rows=canvas.height/grid;
let speed=Number(speedSel.value),running=false,paused=false,last=0,acc=0;
let snake,dir,nextDir,food,score=0;
let highScore=Number(localStorage.getItem('serpent_high')||0);
let leader=JSON.parse(localStorage.getItem('serpent_lead')||'[]');

function init(){
  snake=[{x:6,y:8},{x:5,y:8},{x:4,y:8}];
  dir={x:1,y:0};nextDir={x:1,y:0};score=0;
  placeFood();draw();scoreEl.textContent=0;
}
function placeFood(){
  food={x:Math.floor(Math.random()*cols),y:Math.floor(Math.random()*rows)};
}
function loop(t){
  if(!running||paused)return;
  const dt=(t-last)/1000;last=t;acc+=dt;
  while(acc>1/speed){tick();acc-=1/speed;}
  draw();requestAnimationFrame(loop);
}
function tick(){
  if(!(nextDir.x===-dir.x&&nextDir.y===-dir.y))dir={...nextDir};
  const head={x:(snake[0].x+dir.x+cols)%cols,y:(snake[0].y+dir.y+rows)%rows};
  if(snake.some(s=>s.x===head.x&&s.y===head.y)){gameOver();return;}
  snake.unshift(head);
  if(head.x===food.x&&head.y===food.y){score+=10;scoreEl.textContent=score;placeFood();}
  else snake.pop();
}
function draw(){
  ctx.fillStyle="#031";
  ctx.fillRect(0,0,canvas.width,canvas.height);

  // Draw food with glow
  ctx.fillStyle=foodColor();
  ctx.beginPath();
  ctx.arc(food.x*grid+grid/2,food.y*grid+grid/2,grid/2-2,0,2*Math.PI);
  ctx.fill();

  // Draw snake with gradient
  snake.forEach((s,i)=>{
    let grad=ctx.createLinearGradient(s.x*grid, s.y*grid, (s.x+1)*grid, (s.y+1)*grid);
    grad.addColorStop(0,'#0f0'); grad.addColorStop(1,'#0ff');
    ctx.fillStyle=grad;
    ctx.fillRect(s.x*grid,s.y*grid,grid-2,grid-2);
  });
}
function foodColor(){
  const time=Date.now()*0.005;
  const r=Math.floor(255+(Math.sin(time)*50));
  return `rgb(${r%255},42,104)`;
}
function gameOver(){
  running=false;
  if(score>highScore){highScore=score;localStorage.setItem('serpent_high',highScore);}
  leader.unshift(score);leader=leader.slice(0,5);
  localStorage.setItem('serpent_lead',JSON.stringify(leader));
  renderLB();
  alert("Game Over! Score: "+score);
}
function renderLB(){highEl.textContent=highScore;lbEl.innerHTML=leader.map(s=>`<li>${s}</li>`).join('');}
function start(){running=true;paused=false;last=performance.now();acc=0;init();requestAnimationFrame(loop);}
btnStart.onclick=()=>{if(!running)start();else paused=false;};
btnPause.onclick=()=>{if(running)paused=!paused;if(!paused)last=performance.now();requestAnimationFrame(loop);};
btnRestart.onclick=()=>start();
speedSel.onchange=()=>{speed=Number(speedSel.value);};
window.onkeydown=e=>{
  const k=e.key;
  if(k==='ArrowUp'||k==='w')nextDir={x:0,y:-1};
  if(k==='ArrowDown'||k==='s')nextDir={x:0,y:1};
  if(k==='ArrowLeft'||k==='a')nextDir={x:-1,y:0};
  if(k==='ArrowRight'||k==='d')nextDir={x:1,y:0};
  if(k===' ')paused=!paused;
};
renderLB();init();
</script>
</body>
</html>
# SERPENT.TRAIL
