<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Roi Alva — Gaming HUD Animation</title>
  <style>
    :root{
      --bg:#08060b; --neon:#00f7ff; --accent:#ff4d6d; --muted:#9aa4b2;
      --glass: rgba(255,255,255,0.04);
    }
    html,body{height:100%;margin:0;font-family:Inter,ui-sans-serif,system-ui,Segoe UI,Roboto,'Helvetica Neue',Arial}
    body{background:linear-gradient(180deg,#03030a 0%, #071025 60%);overflow:hidden;color:#cfeff6}
    /* full canvas background */
    canvas#bg{position:fixed;inset:0;z-index:0}

    /* center container */
    .wrap{position:relative;z-index:1;max-width:1100px;margin:40px auto;padding:28px;border-radius:18px;backdrop-filter:blur(6px);background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));box-shadow:0 8px 40px rgba(2,6,23,0.7);border:1px solid rgba(0,247,255,0.06)}

    .grid{display:grid;grid-template-columns:380px 1fr;gap:22px;align-items:start}

    /* left column — avatar + mini-hud */
    .panel{background:var(--glass);padding:18px;border-radius:14px;border:1px solid rgba(255,255,255,0.03)}
    .avatar{width:100%;height:260px;border-radius:12px;overflow:hidden;position:relative;background:linear-gradient(135deg,#001b2b 0%, #001018 100%);display:flex;align-items:center;justify-content:center}
    .avatar svg{width:80%;height:80%;filter:drop-shadow(0 8px 28px rgba(0,0,0,0.6))}

    .tagline{display:flex;justify-content:space-between;align-items:center;margin-top:12px}
    .tag{font-family: 'Press Start 2P', monospace;font-size:11px;color:var(--muted);letter-spacing:1px}
    .level{font-weight:700;color:var(--neon);font-family:Inter,system-ui}

    .bar{height:10px;background:rgba(255,255,255,0.04);border-radius:999px;overflow:hidden;margin-top:10px}
    .bar > i{display:block;height:100%;background:linear-gradient(90deg,var(--neon),var(--accent));width:0%;border-radius:999px;box-shadow:0 6px 20px rgba(0,247,255,0.12),0 0 30px rgba(255,77,109,0.04)}

    .loadout{display:flex;gap:8px;flex-wrap:wrap;margin-top:12px}
    .chip{padding:6px 10px;border-radius:10px;background:rgba(255,255,255,0.02);font-size:12px;border:1px solid rgba(255,255,255,0.02)}

    /* right column — HUD and stats */
    .hud{display:flex;flex-direction:column;gap:14px}
    .row{display:flex;gap:12px}
    .card{flex:1;padding:14px;border-radius:12px;background:linear-gradient(180deg, rgba(255,255,255,0.015), rgba(255,255,255,0.01));border:1px solid rgba(255,255,255,0.03);min-height:110px}

    .title{font-size:12px;color:var(--muted);letter-spacing:1px}
    .value{font-size:20px;font-weight:700;color:#fff;margin-top:6px}

    /* small blinking dot */
    .dot{width:10px;height:10px;border-radius:50%;background:linear-gradient(135deg,var(--neon),#8df0ff);box-shadow:0 6px 18px rgba(0,247,255,0.14),0 0 8px rgba(0,247,255,0.25);animation:blink 1.6s infinite}
    @keyframes blink{0%{opacity:1}50%{opacity:0.35}100%{opacity:1}}

    /* animated orbit icons */
    .orbit-wrap{position:relative;height:110px}
    .orbit{position:absolute;inset:0;margin:auto;width:110px;height:110px;border-radius:50%;display:flex;align-items:center;justify-content:center}
    .icon{position:absolute;width:36px;height:36px;border-radius:8px;background:rgba(255,255,255,0.03);display:grid;place-items:center;border:1px solid rgba(255,255,255,0.03);box-shadow:0 6px 16px rgba(2,6,23,0.6)}

    /* footer wave */
    .footer{display:flex;justify-content:space-between;align-items:center;margin-top:18px}
    .badge{padding:8px 12px;border-radius:12px;background:linear-gradient(90deg,rgba(0,0,0,0.18),rgba(255,255,255,0.01));border:1px solid rgba(255,255,255,0.03)}

    /* neon glow titles */
    h1{font-family: 'Press Start 2P', monospace;font-size:13px;margin:0;color:var(--neon);text-shadow:0 6px 24px rgba(0,247,255,0.06)}

    /* small responsive tweak */
    @media (max-width:900px){.grid{grid-template-columns:1fr;}.avatar{height:220px}}
  </style>
</head>
<body>
  <canvas id="bg"></canvas>

  <div class="wrap">
    <div class="grid">
      <div>
        <div class="panel">
          <div class="avatar" id="avatar">
            <!-- Animated SVG avatar (pixel-cyber) -->
            <svg viewBox="0 0 200 200" xmlns="http://www.w3.org/2000/svg">
              <defs>
                <linearGradient id="g" x1="0" x2="1">
                  <stop offset="0" stop-color="#00f7ff"/>
                  <stop offset="1" stop-color="#ff4d6d"/>
                </linearGradient>
                <filter id="f1" x="-50%" y="-50%" width="200%" height="200%">
                  <feGaussianBlur stdDeviation="2" result="b"/>
                  <feMerge><feMergeNode in="b"/><feMergeNode in="SourceGraphic"/></feMerge>
                </filter>
              </defs>
              <rect width="200" height="200" fill="#00121a" rx="12"/>

              <g transform="translate(28,28)">
                <rect x="0" y="0" width="144" height="144" rx="10" fill="#01151b"/>
                <g id="helmet" transform="translate(10,12)">
                  <circle cx="62" cy="46" r="36" fill="url(#g)" opacity="0.06"/>
                  <rect x="10" y="24" width="104" height="72" rx="12" fill="#00202a" stroke="url(#g)" stroke-width="1.8"/>
                  <rect x="24" y="36" width="76" height="36" rx="6" fill="#00121a" stroke="#05212a"/>
                  <rect x="36" y="46" width="44" height="12" rx="4" fill="url(#g)"/>
                </g>
                <g id="visor" transform="translate(14,12)">
                  <rect x="22" y="32" width="88" height="16" rx="8" fill="#001e27"/>
                  <rect id="glow" x="24" y="34" width="84" height="12" rx="6" fill="url(#g)" opacity="0.9" filter="url(#f1)"/>
                </g>
                <g id="pixels" transform="translate(0,100)">
                  <!-- animated pixels -->
                  <rect class="p" x="6" y="0" width="6" height="6" fill="#00f7ff" opacity="0.4"/>
                  <rect class="p" x="18" y="0" width="6" height="6" fill="#ff4d6d" opacity="0.35"/>
                  <rect class="p" x="30" y="0" width="6" height="6" fill="#00f7ff" opacity="0.3"/>
                  <rect class="p" x="42" y="0" width="6" height="6" fill="#00f7ff" opacity="0.25"/>
                </g>
              </g>
            </svg>
          </div>

          <div class="tagline">
            <div>
              <div class="tag">Roi Alva</div>
              <div class="level">LEVEL 42</div>
            </div>
            <div class="dot" title="Online"></div>
          </div>

          <div class="bar" style="margin-top:12px"><i id="xp"></i></div>

          <div class="loadout" style="margin-top:10px">
            <div class="chip">React</div>
            <div class="chip">Next.js</div>
            <div class="chip">Tailwind</div>
            <div class="chip">Node</div>
            <div class="chip">Unity</div>
          </div>
        </div>

        <div class="panel" style="margin-top:14px">
          <div style="display:flex;justify-content:space-between;align-items:center">
            <div>
              <div class="title">CURRENT SEASON</div>
              <div class="value">Cyber Clash • Rank: Diamon</div>
            </div>
            <div style="text-align:right">
              <div class="title">K/D</div>
              <div class="value">2.14</div>
            </div>
          </div>

          <div style="margin-top:12px" class="bar"><i id="seasonBar"></i></div>
        </div>
      </div>

      <div class="hud">
        <div class="row">
          <div class="card">
            <h1>LIVE HUD</h1>
            <div style="display:flex;justify-content:space-between;align-items:center;margin-top:12px">
              <div>
                <div class="title">CPU LOAD</div>
                <div class="value"><span id="cpuVal">12%</span></div>
              </div>
              <div class="orbit-wrap" aria-hidden>
                <div class="orbit" id="orbit"></div>
              </div>
            </div>

            <div style="display:flex;gap:10px;margin-top:14px">
              <div style="flex:1">
                <div class="title">RAM</div>
                <div class="value"><span id="ramVal">3.2GB</span></div>
              </div>
              <div style="flex:1">
                <div class="title">FPS</div>
                <div class="value"><span id="fpsVal">144</span></div>
              </div>
            </div>
          </div>

          <div class="card">
            <div class="title">ACHIEVEMENTS</div>
            <div style="display:flex;gap:8px;margin-top:10px;flex-wrap:wrap">
              <div class="badge">100+ Repos</div>
              <div class="badge">Open Source Contributor</div>
              <div class="badge">Winner: Ludum Dare 2023</div>
            </div>
          </div>
        </div>

        <div class="row">
          <div class="card" style="min-height:140px">
            <div class="title">CONTRIBUTION WAVE</div>
            <canvas id="spark" style="width:100%;height:110px;margin-top:10px;border-radius:8px"></canvas>
          </div>

          <div class="card" style="min-width:220px">
            <div class="title">SOCIAL</div>
            <div style="display:flex;flex-direction:column;gap:8px;margin-top:8px">
              <a class="badge" href="#">Twitch / roi_alva</a>
              <a class="badge" href="#">YouTube / roi_alva</a>
              <a class="badge" href="#">Discord / roi#1234</a>
            </div>
          </div>
        </div>

        <div class="footer">
          <div class="tag">© Roi Alva</div>
          <div class="badge">Press F to Pay Respects • 2025</div>
        </div>
      </div>
    </div>
  </div>

<script>
// Background particle field
const bg = document.getElementById('bg');
const ctx = bg.getContext('2d');
function resize(){bg.width=innerWidth;bg.height=innerHeight}
addEventListener('resize',resize);resize();

// create particles
const P = [];
const colors = ['#002b33','#003d4a','#00f7ff','#ff4d6d'];
for(let i=0;i<140;i++){
  P.push({x:Math.random()*bg.width,y:Math.random()*bg.height,r:Math.random()*1.6+0.3, vx:(Math.random()-0.5)*0.2, vy:(Math.random()-0.5)*0.2, c:colors[Math.floor(Math.random()*colors.length)], o:Math.random()*0.6+0.2})
}

function drawBG(t){
  ctx.clearRect(0,0,bg.width,bg.height);
  // moving gradient scanline
  const g = ctx.createLinearGradient(0,0,bg.width,bg.height);
  g.addColorStop(0,'rgba(0,0,0,0.2)'); g.addColorStop(1,'rgba(1,6,12,0.2)');
  ctx.fillStyle = g; ctx.fillRect(0,0,bg.width,bg.height);

  // grid subtle
  ctx.strokeStyle='rgba(255,255,255,0.02)'; ctx.lineWidth=1;
  const gap=120; for(let x=-gap*2;x<bg.width+gap*2;x+=gap){ctx.beginPath();ctx.moveTo(x+((t*0.02)%gap),0);ctx.lineTo(x+((t*0.02)%gap),bg.height);ctx.stroke()}

  // particles
  for(let p of P){
    p.x+=p.vx; p.y+=p.vy;
    if(p.x<0)p.x=bg.width; if(p.x>bg.width)p.x=0; if(p.y<0)p.y=bg.height; if(p.y>bg.height)p.y=0;
    ctx.beginPath(); ctx.fillStyle = p.c; ctx.globalAlpha=p.o; ctx.arc(p.x,p.y,p.r,0,Math.PI*2); ctx.fill(); ctx.globalAlpha=1;
  }
}

// avatar glow & pixel dance
const glow = document.getElementById('glow');
const pixels = document.querySelectorAll('.p');
let glowPhase=0;

// xp bar
const xp = document.getElementById('xp');
const season = document.getElementById('seasonBar');
let xpPct=0;

// live HUD numbers
const cpuVal = document.getElementById('cpuVal');
const ramVal = document.getElementById('ramVal');
const fpsVal = document.getElementById('fpsVal');

// orbit icons
const orbit = document.getElementById('orbit');
const icons = [];
const iconCount = 5;
for(let i=0;i<iconCount;i++){
  const el = document.createElement('div'); el.className='icon'; el.style.left='50%'; el.style.top='50%'; el.innerHTML = ['⚡','💻','🎮','🌐','♻️'][i%5];
  orbit.appendChild(el); icons.push(el);
}

// sparkline canvas
const spark = document.getElementById('spark');
const sctx = spark.getContext('2d'); function resizeSpark(){spark.width = spark.clientWidth; spark.height = spark.clientHeight;} resizeSpark(); addEventListener('resize',resizeSpark);
let spikes = new Array(40).fill(0).map(()=>Math.random()*0.6+0.2);

let last = performance.now(), frame=0, fps=60;
function loop(t){
  frame++; const dt = (t-last)/1000; if(dt>=0.5){last=t;}
  // bg
  drawBG(t);
  // avatar glow
  glowPhase += 0.02; glow.setAttribute('opacity', 0.85 + Math.sin(glowPhase)*0.12);
  // pixels twitch
  pixels.forEach((p,i)=>{p.setAttribute('y', Math.abs(Math.sin(t*0.002 + i))*6); p.setAttribute('opacity', 0.25 + Math.abs(Math.cos(t*0.004+i))*0.35)})

  // xp animate
  xpPct = (50 + Math.sin(t*0.0018)*40) ; xp.style.width = xpPct+'%';
  season.querySelector ? season.style.width = (30 + Math.abs(Math.sin(t*0.0008))*60)+'%' : season.style.width = '40%';

  // hud dynamic numbers
  const cpu = Math.round(10 + Math.abs(Math.sin(t*0.0015))*60);
  const ram = (2 + Math.abs(Math.sin(t*0.0009))*6).toFixed(2);
  const fpsEst = Math.round(120 + Math.sin(t*0.0035)*30);
  cpuVal.textContent = cpu + '%'; ramVal.textContent = ram + 'GB'; fpsVal.textContent = fpsEst;

  // orbit animation
  const R = 36; for(let i=0;i<icons.length;i++){
    const a = (t*0.0012 + i*(Math.PI*2/icons.length));
    const x = Math.cos(a)*R; const y = Math.sin(a)*R;
    icons[i].style.transform = `translate(${x}px,${y}px) rotate(${t*0.02}deg)`;
  }

  // sparkline
  for(let i=0;i<spikes.length;i++){spikes[i] = 0.1 + Math.abs(Math.sin((t*0.002) + i*0.6))*0.9}
  sctx.clearRect(0,0,spark.width,spark.height);
  sctx.beginPath(); sctx.lineWidth=2; sctx.strokeStyle='rgba(0,247,255,0.95)';
  for(let i=0;i<spikes.length;i++){const x = i*(spark.width/spikes.length); const y = spark.height - spikes[i]*spark.height; if(i===0) sctx.moveTo(x,y); else sctx.lineTo(x,y)} sctx.stroke();

  last = t; requestAnimationFrame(loop);
}
requestAnimationFrame(loop);

// small UI polished transitions
document.querySelectorAll('.bar > i').forEach(el=>{el.style.transition='width 1.2s cubic-bezier(.2,.8,.2,1)'; el.style.width='40%'});

// progress initial animation
setTimeout(()=>{document.getElementById('xp').style.width='68%';document.getElementById('seasonBar').style.width='62%';},600);

// micro interactions: hover avatar -> burst
const avatar = document.getElementById('avatar');
avatar.addEventListener('mousemove', (e)=>{
  const rect = avatar.getBoundingClientRect(); const nx = (e.clientX-rect.left)/rect.width - 0.5; const ny = (e.clientY-rect.top)/rect.height - 0.5;
  avatar.style.transform = `translateZ(0) perspective(600px) rotateX(${(-ny*6)}deg) rotateY(${(nx*8)}deg)`;
});
avatar.addEventListener('mouseleave', ()=>{avatar.style.transform='none'});

</script>
</body>
</html>
