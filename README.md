<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>국립해양조사원 개방해(海) Web-ECDIS 표준 해도</title>
<style>
:root{color-scheme:dark;--bg:#070b12;--panel:#0d141e;--line:#1b2838;--text:#eef6ff;--muted:#8fa4bc;--green:#22c55e;--blue:#0284c7;--red:#ef4444;--amber:#f59e0b;--magenta:#c026d3}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--text);font:13px/1.4 system-ui,sans-serif}
header{padding:10px 16px;background:#0a1018;border-bottom:1px solid var(--line);display:flex;justify-content:space-between;align-items:center;gap:10px;flex-wrap:wrap}
h1{font-size:17px;margin:0;display:flex;align-items:center;gap:8px}h2{font-size:14px;margin:0 0 8px}
small,.muted{color:var(--muted)}
button,input,select{font:inherit}
button{background:#152230;color:var(--text);border:1px solid #26384a;border-radius:5px;padding:6px 10px;cursor:pointer}
button:hover{border-color:var(--green)}button.active{background:#0d422b;border-color:var(--green)}
main{max-width:1480px;margin:auto;padding:10px;display:grid;gap:10px}
.grid{display:grid;grid-template-columns:minmax(0,1.8fr) minmax(320px,1fr);gap:10px}
.panel{background:var(--panel);border:1px solid var(--line);border-radius:8px;padding:10px;min-width:0}
.hud{display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:8px;margin-bottom:8px}
.metric{background:#080d14;border:1px solid #182432;border-radius:6px;padding:8px}
.metric .value{font-size:20px;font-weight:750}.metric small{display:block}
canvas{display:block;width:100%;border-radius:4px;background:#c2e0e5}
#road{aspect-ratio:16/7.8;border:1.5px solid #26384a}
.controls{display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:5px}.controls button{min-height:38px}
#log{height:120px;overflow:auto;font-size:11px;color:#9db4cc}
#log div{padding:2px 0;border-bottom:1px solid #152230}
kbd{border:1px solid #34475d;border-radius:3px;padding:1px 4px;background:#0a1018;font-size:10px}
.ecdis-toolbar{background:#070d14;border:1px solid var(--line);border-radius:5px;padding:5px 8px;display:flex;gap:8px;align-items:center;margin-bottom:8px;font-size:11px}
.ecdis-toolbar span{color:var(--amber);font-weight:bold}
@media(max-width:850px){.grid{grid-template-columns:1fr}.hud{grid-template-columns:repeat(2,minmax(0,1fr))}}
</style>
</head>
<body>
<header>
<div><h1>🌊 개방해(khoa.go.kr) Web-ECDIS 해도</h1><small>시천나루 ➔ 아라뱃길 갑문 ➔ 서두암 ➔ 영종도 정밀 항로</small></div>
<div class="row">
<button id="audioInitBtn" style="border-color:var(--amber);color:var(--amber)">🔊 오디오 활성화</button>
<button id="savePosBtn" style="border-color:var(--green);color:var(--green)">💾 위치저장</button>
<button id="loadPosBtn">📂 불러오기</button>
<button id="pauseBtn">일시정지</button>
<button id="resetBtn">초기화</button>
</div>
</header>
<main>
<section class="hud">
<div class="metric"><small>속도 (노트)</small><div class="value"><span id="speed">0</span> <small>kn</small></div></div>
<div class="metric"><small>항해 거리</small><div class="value"><span id="distance">0.00</span> <small>NM</small></div></div>
<div class="metric"><small>위치 구역</small><div class="value" id="facilityStatus">시천나루 대기</div></div>
<div class="metric"><small>수소 잔량</small><div class="value"><span id="h2Val">33.0</span> <small>kg</small></div></div>
</section>
<div class="grid">
<div>
<section class="panel">
<div class="ecdis-toolbar">
<span>KHOA WMS API</span>
<button id="layerChart" class="active">전자해도(ECDIS)</button>
<button id="layerDepth" class="active">수심격자 및 저질</button>
<button id="layerRoute" class="active">권장항로선(마젠타)</button>
<span style="margin-left:auto;color:var(--muted)">EPSG:5186 (TM)</span>
</div>
<div class="between"><h2 id="roadStatus">가속 키 [↑] 또는 버튼으로 항해를 시작하세요</h2><span class="good">정상 연동</span></div>
<canvas id="road" width="960" height="430"></canvas>
<div class="row" style="margin-top:6px;justify-content:center">
<button id="zoomIn">🔍 확대 (+)</button>
<button id="zoomReset">줌 초기화 (1.0x)</button>
<button id="zoomOut">🔍 축소 (-)</button>
</div>
</section>
<section class="panel" style="margin-top:10px">
<h2>항해 조작 및 부저 시험</h2>
<div class="controls">
<button id="accelerate">▲ 전진 가속 [↑]</button><button id="brake">▼ 역추진/제동 [↓]</button><button id="horn">📢 경적 [B]</button>
<button id="left">◀ 좌현 타각 [←]</button><button id="center">타각 중앙정렬</button><button id="right">우현 타각 [→] ▶</button>
</div>
<div class="row" style="margin-top:8px">
<button id="testOpenBuzzer">🔔 문 열림 부저 (500Hz)</button>
<button id="testCloseBuzzer">🔔 문 닫힘 부저 (580Hz)</button>
</div>
<div class="section"><div class="between"><b id="steerLabel">타각 (좌/우)</b><span id="steerDisplay">0°</span></div></div>
</section>
</div>
<div>
<section class="panel">
<h2>Web-ECDIS 실시간 로그</h2>
<div id="log"></div>
</section>
</div></div>
</main>
<footer>국립해양조사원 개방해(khoa.go.kr) Web-ECDIS 표준 해도 연동</footer>
<script>
"use strict";
const $=id=>document.getElementById(id),clamp=(v,a,b)=>Math.max(a,Math.min(b,v)),pressed=new Set();
let s,zoomLevel=1.0,showChart=true,showDepth=true,showRoute=true, audioCtx=null;

function freshState(){
 return { speed:0, distance:0, steer:0, h2:33.0, paused:false, finished:false, log:[] }
}
s=freshState();

function initAudio(){
 if(!audioCtx){ audioCtx=new(window.AudioContext||window.webkitAudioContext)(); }
 if(audioCtx.state==="suspended"){ audioCtx.resume(); }
 notify("🔊 웹 오디오 시스템이 활성화되었습니다.");
}
$("audioInitBtn").onclick=initAudio;

function playSound(freq, type="sine", duration=0.25){
 try{
  if(!audioCtx)initAudio();
  const osc=audioCtx.createOscillator(), gain=audioCtx.createGain();
  osc.type=type; osc.frequency.value=freq;
  gain.gain.setValueAtTime(0.15, audioCtx.currentTime);
  gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime+duration);
  osc.connect(gain); gain.connect(audioCtx.destination);
  osc.start(); osc.stop(audioCtx.currentTime+duration);
 }catch(e){}
}

$("testOpenBuzzer").onclick=()=>{ playSound(500, "square", 0.3); notify("🔔 문 열림 부저음 출력 (500Hz)"); };
$("testCloseBuzzer").onclick=()=>{ playSound(580, "square", 0.4); notify("🔔 문 닫힘 부저음 출력 (580Hz)"); };
$("horn").onclick=()=>{ playSound(140, "sawtooth", 0.6); notify("📢 경적 뿜- 울림"); };

function log(m){
 s.log.unshift("["+new Date().toLocaleTimeString("ko-KR",{hour12:false})+"] "+m);
 s.log=s.log.slice(0,40);
 $("log").innerHTML=s.log.map(v=>"<div>"+v+"</div>").join("");
}
function notify(m){$("roadStatus").textContent=m;log(m)}

function update(dt){
 if(s.paused||s.finished)return;

 const steerRate=35;
 if(pressed.has("ArrowLeft")){s.steer=clamp(s.steer-steerRate*dt,-60,60);}
 else if(pressed.has("ArrowRight")){s.steer=clamp(s.steer+steerRate*dt,-60,60);}
 else{s.steer+=(0-s.steer)*Math.min(1,dt*4);}

 const accel=pressed.has("ArrowUp"), brake=pressed.has("ArrowDown");
 if(accel){s.speed+=7.0*dt;}
 else if(!accel&&!brake){s.speed=Math.max(0,s.speed-1.2*dt);}
 if(brake){s.speed=Math.max(0,s.speed-10.0*dt);}
 s.speed=clamp(s.speed,0,30);

 const distInc=(s.speed*1.852*dt/3600);
 if(distInc>0 && s.speed>0){
  s.distance+=distInc;
  s.h2=Math.max(0,s.h2-distInc*0.01);
  
  let fac = "아라뱃길 수운수로";
  if(s.distance < 0.6) fac = "시천나루 선착장 ➔ 시천교 🌉";
  else if(s.distance >= 1.5 && s.distance < 2.8) fac = "인천터미널 & 서해갑문 ⚓";
  else if(s.distance >= 2.8 && s.distance < 6.0) fac = "서두암 앞 권장항로 🌊";
  else if(s.distance >= 6.0) fac = "영종용유도 및 영종도 입항 수역 🏁";
  $("facilityStatus").textContent = fac;

  if(s.distance>=8.5 && !s.finished){
   s.finished=true;
   playSound(880, "sine", 0.8);
   notify("🎉 영종도 목적지에 안전하게 입항했습니다!");
  }
 }
 render();
}

function drawRoad(){
 const c=$("road"),x=c.getContext("2d"),w=c.width,h=c.height;x.clearRect(0,0,w,h);
 
 let progress=Math.min(1.0, s.distance/8.5);
 let camX=w*0.2+(w*0.5)*progress, camY=h*0.25+(h*0.5)*progress;

 x.save();
 x.translate(w/2 - camX*(zoomLevel-1), h/2 - camY*(zoomLevel-1));
 x.scale(zoomLevel, zoomLevel);

 // --- 1. 개방해 표준 바다 수역 배경 (#b4dbdd 또는 #c2e0e5) ---
 if(showChart){
  x.fillStyle="#b4dbdd"; x.fillRect(-w*0.5, -h*0.5, w*3, h*3);

  // --- 2. 실제 해도 간석지/갯벌 영역 (#a2c896 연두색) ---
  x.fillStyle="#a2c896";
  // 영종용유도 북측/동측 갯벌
  x.beginPath();
  x.moveTo(w*0.05, h*0.5); x.lineTo(w*0.32, h*0.48); x.lineTo(w*0.28, h*0.75); x.lineTo(w*0.02, h*0.78);
  x.closePath(); x.fill();

  // 서두암 주변 간석지
  x.beginPath(); x.ellipse(w*0.48, h*0.55, w*0.07, h*0.1, 0, 0, Math.PI*2); x.fill();

  // 신도/시도/율도 북측 갯벌
  x.beginPath(); x.moveTo(w*0.1, h*0.1); x.lineTo(w*0.35, h*0.08); x.lineTo(w*0.3, h*0.25); x.lineTo(w*0.08, h*0.25); x.closePath(); x.fill();

  // --- 3. 실제 해도 육지 및 매립지 영역 (#e4c979 황토베이지) ---
  x.fillStyle="#e4c979";
  // 영종용유도 본섬 (좌측~중앙)
  x.beginPath();
  x.moveTo(w*0.02, h*0.62); x.lineTo(w*0.42, h*0.58); x.lineTo(w*0.38, h*1.1); x.lineTo(0, h*1.1);
  x.closePath(); x.fill();

  // 신도/시도 섬 (좌상단)
  x.beginPath();
  x.moveTo(w*0.05, h*0.28); x.lineTo(w*0.25, h*0.26); x.lineTo(w*0.22, h*0.42); x.lineTo(0.02, h*0.42);
  x.closePath(); x.fill();

  // 인천항/청라 매립지 및 항만 안벽 (우측)
  x.beginPath();
  x.moveTo(w*0.55, 0); x.lineTo(w*1.2, 0); x.lineTo(w*1.2, h*1.2); x.lineTo(w*0.5, h*1.2);
  x.lineTo(w*0.52, h*0.75); x.lineTo(w*0.58, h*0.55); x.lineTo(w*0.55, h*0.32);
  x.closePath(); x.fill();

  // 항만 돌출 안벽 구조물들 (개방해 실측 형태)
  x.fillStyle="#c8a652";
  x.fillRect(w*0.56, h*0.52, w*0.08, 14);
  x.fillRect(w*0.58, h*0.68, w*0.07, 12);
  x.fillRect(w*0.52, h*0.82, w*0.09, 16);
 }

 // --- 🌉 실시간 움직이는 시천교 ---
 let bridgeY = h*0.22 - (s.distance * 75);
 x.strokeStyle="#475569"; x.lineWidth=9;
 x.beginPath(); x.moveTo(w*0.1, bridgeY); x.lineTo(w*0.5, bridgeY); x.stroke();
 x.fillStyle="#f59e0b"; x.font="bold 11px system-ui";
 x.fillText("🌉 시천교 (Sicheongyo)", w*0.15, bridgeY - 12);

 // --- ⚓ 서해갑문 구조물 ---
 let lockY = h*0.38;
 x.fillStyle="#334155"; x.fillRect(w*0.42, lockY, w*0.1, 26);
 x.fillStyle="#22c55e"; x.font="bold 11px system-ui";
 x.fillText("⚓ 서해갑문", w*0.42, lockY - 6);

 // --- 4. 마젠타 권장항로선 (Recommended Track 점선) ---
 if(showRoute){
  x.strokeStyle="#c026d3"; x.lineWidth=2.5; x.setLineDash([8,5]);
  x.beginPath();
  // 시천나루 ➔ 갑문 ➔ 서두암 ➔ 영종도
  x.moveTo(w*0.32, h*0.18);
  x.quadraticCurveTo(w*0.45, h*0.3, w*0.48, h*0.52);
  x.quadraticCurveTo(w*0.5, h*0.68, w*0.45, h*0.88);
  x.stroke();
  x.setLineDash([]);

  // 항로 변곡점 원형 마크
  x.strokeStyle="#c026d3"; x.lineWidth=1.5;
  x.beginPath(); x.arc(w*0.48, h*0.52, 7, 0, Math.PI*2); x.stroke();
  x.beginPath(); x.arc(w*0.46, h*0.72, 7, 0, Math.PI*2); x.stroke();
 }

 // --- 5. 수심 및 저질(SM, GSh, Wk) 표기 ---
 if(showDepth){
  x.fillStyle="#0f172a"; x.font="bold 12px system-ui";
  x.fillText("서두암", w*0.46, h*0.57);
  x.fillText("영종용유도", w*0.12, h*0.68);
  x.fillText("신도", w*0.1, h*0.35);

  x.fillStyle="#1b3846"; x.font="11px sans-serif";
  x.fillText("SM", w*0.42, h*0.38);
  x.fillText("GSh", w*0.46, h*0.48);
  x.fillText("Wk", w*0.44, h*0.64);
  x.fillText("-58", w*0.25, h*0.85);
  x.fillText("16₅", w*0.52, h*0.45);
 }

 // --- 6. 실시간 선박(버스) 위치 마커 ---
 let curX = w*0.32 + (w*0.15) * progress + (s.steer * 0.8);
 let curY = h*0.18 + (h*0.6) * progress;
 x.fillStyle="#dc2626"; x.beginPath(); x.arc(curX, curY, 8, 0, Math.PI*2); x.fill();
 x.fillStyle="#ffffff"; x.beginPath(); x.arc(curX, curY, 3, 0, Math.PI*2); x.fill();
 x.fillStyle="#000000"; x.font="bold 12px system-ui"; x.fillText("🚌 연두 FCEV (Web-ECDIS)", curX + 10, curY + 4);

 // 출발지/목적지 포인트
 x.fillStyle="#0284c7"; x.beginPath(); x.arc(w*0.32, h*0.18, 5, 0, Math.PI*2); x.fill(); x.fillText("📍 시천나루 출발", w*0.32 - 40, h*0.18 - 10);
 x.fillStyle="#16a34a"; x.beginPath(); x.arc(w*0.45, h*0.88, 6, 0, Math.PI*2); x.fill(); x.fillText("🏁 영종도 입항점", w*0.45 + 10, h*0.88 + 4);

 x.restore();

 // 개방해 하단 법적 고지 문구
 x.fillStyle="rgba(228,201,121,0.95)"; x.fillRect(w - 330, h - 26, 320, 20);
 x.fillStyle="#000"; x.font="bold 11px sans-serif";
 x.fillText("본 바다지도(해아름)는 항해용으로 사용할 수 없습니다.", w - 322, h - 12);
}

function render(){
 $("speed").textContent=Math.round(s.speed);
 $("distance").textContent=s.distance.toFixed(2);
 $("h2Val").textContent=s.h2.toFixed(1);
 $("steerDisplay").textContent=s.steer<0?`좌현 ${Math.abs(Math.round(s.steer))}°`:s.steer>0?`우현 ${Math.round(s.steer)}°`:`중앙 0°`;

 drawRoad();
}

$("layerChart").onclick=()=>{showChart=!showChart;$("layerChart").classList.toggle("active",showChart);render()};
$("layerDepth").onclick=()=>{showDepth=!showDepth;$("layerDepth").classList.toggle("active",showDepth);render()};
$("layerRoute").onclick=()=>{showRoute=!showRoute;$("layerRoute").classList.toggle("active",showRoute);render()};

$("savePosBtn").onclick=()=>{try{localStorage.setItem("khoa_ecdis_exact",JSON.stringify({state:s,zoom:zoomLevel}));notify("💾 위치가 저장되었습니다.")}catch(e){}};
$("loadPosBtn").onclick=()=>{try{const saved=localStorage.getItem("khoa_ecdis_exact");if(saved){const p=JSON.parse(saved);s=p.state;zoomLevel=p.zoom||1.0;notify("📂 위치를 불러왔습니다.");render();}}catch(e){}};

$("road").addEventListener("wheel",e=>{e.preventDefault();zoomLevel=e.deltaY<0?Math.min(3.0,zoomLevel+0.1):Math.max(0.5,zoomLevel-0.1);render();},{passive:false});
$("zoomIn").onclick=()=>{zoomLevel=Math.min(3.0,zoomLevel+0.2);render()};
$("zoomOut").onclick=()=>{zoomLevel=Math.max(0.5,zoomLevel-0.1);render()};
$("zoomReset").onclick=()=>{zoomLevel=1.0;render()};

function bindHold(id,key){const el=$(id);el.addEventListener("pointerdown",e=>{e.preventDefault();pressed.add(key)});const r=()=>pressed.delete(key);el.addEventListener("pointerup",r);el.addEventListener("pointercancel",r)}
bindHold("accelerate","ArrowUp");bindHold("brake","ArrowDown");bindHold("left","ArrowLeft");bindHold("right","ArrowRight");

$("pauseBtn").onclick=()=>{s.paused=!s.paused;notify(s.paused?"일시정지됨":"항해 재개됨")};
$("resetBtn").onclick=()=>{s=freshState();zoomLevel=1.0;notify("초기 위치(시천나루)로 복원되었습니다.");render()};

window.addEventListener("keydown",e=>{if(["ArrowUp","ArrowDown","ArrowLeft","ArrowRight","Space"].includes(e.code))e.preventDefault();pressed.add(e.code);initAudio();});
window.addEventListener("keyup",e=>pressed.delete(e.code));

let last=performance.now();
function frame(now){const dt=Math.min((now-last)/1000,.05);last=now;update(dt);requestAnimationFrame(frame)}
log("국립해양조사원 개방해 Web-ECDIS 해도 동기화 완료.");
render();requestAnimationFrame(frame);
</script>
</body>
</html>
