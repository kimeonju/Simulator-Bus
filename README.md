<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>연두 Yeondoo FCEV · 수륙양용 시뮬레이터</title>
<style>
:root{color-scheme:dark;--bg:#0b1119;--panel:#141f2d;--line:#2b3d50;--text:#eef6ff;--muted:#94a9bf;--green:#49e0a0;--blue:#258dff;--red:#ff6576;--amber:#ffc45c;--magenta:#c026d3}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--text);font:14px/1.5 system-ui,sans-serif}
header{padding:14px 18px;background:#111b28;border-bottom:1px solid var(--line);display:flex;justify-content:space-between;align-items:center;gap:12px;flex-wrap:wrap}
h1{font-size:20px;margin:0}
small,.muted{color:var(--muted)}
button,input,select{font:inherit}
button{background:#223348;color:var(--text);border:1px solid #3a5068;border-radius:8px;padding:8px 11px;cursor:pointer}
button:hover{border-color:var(--green)}button.active{background:#145c48;border-color:var(--green)}
main{max-width:1450px;margin:auto;padding:14px;display:grid;gap:14px}
.grid{display:grid;grid-template-columns:minmax(0,1.7fr) minmax(300px,1fr);gap:14px}
.panel{background:var(--panel);border:1px solid var(--line);border-radius:13px;padding:14px;min-width:0}
.hud{display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:9px;margin-bottom:12px}
.metric{background:#0b1520;border:1px solid #24374a;border-radius:9px;padding:10px}
.metric .value{font-size:22px;font-weight:750}.metric small{display:block}
canvas{display:block;width:100%;border-radius:8px;background:#101b26}
#road{aspect-ratio:16/7}#map{height:140px}
.mirrors{display:grid;grid-template-columns:1fr 1fr;gap:8px;margin-top:10px}.mirrors canvas{height:75px}
.section{margin-top:14px;padding-top:13px;border-top:1px solid var(--line)}
.row{display:flex;flex-wrap:wrap;gap:7px;align-items:center}
.between{display:flex;justify-content:space-between;align-items:center;gap:8px}
.status{padding:7px 10px;border-radius:7px;background:#0b1520;margin:8px 0}
.good{color:var(--green)}.bad{color:var(--red)}
.bar{height:9px;background:#263548;border-radius:20px;overflow:hidden;margin:7px 0 12px}
.fill{height:100%;width:100%;background:var(--green)}.fill.blue{background:var(--blue)}
.gears{display:grid;grid-template-columns:repeat(5,1fr);gap:5px}.gears button{padding:8px 3px}
.controls{display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:7px}.controls button{min-height:42px}
#log{height:150px;overflow:auto;font-size:12px;color:#bdcddd}
#log div{padding:4px 0;border-bottom:1px solid #233243}
kbd{border:1px solid #51647a;border-radius:4px;padding:1px 5px;background:#101923}
input[type=range]{width:100%;accent-color:var(--green)}
input[type=checkbox]{accent-color:var(--green)}
.freq{display:grid;grid-template-columns:1fr 1fr;gap:12px}
.ecdis-toolbar{background:#0b1520;border:1px solid var(--line);border-radius:8px;padding:6px 10px;display:flex;gap:8px;align-items:center;margin-bottom:8px;font-size:12px}
@media(max-width:850px){.grid{grid-template-columns:1fr}.hud{grid-template-columns:repeat(2,minmax(0,1fr))}}
</style>
</head>
<body>
<header>
<div><h1>🚌 연두 Yeondoo FCEV</h1><small>수륙양용 버스 시뮬레이터 · 국립해양조사원 Web-ECDIS 연동</small></div>
<div class="row">
<button id="audioInitBtn" style="border-color:var(--amber);color:var(--amber)">🔊 오디오 활성화</button>
<button id="savePosBtn" style="border-color:var(--green);color:var(--green)">💾 위치 저장</button>
<button id="loadPosBtn">📂 불러오기</button>
<button id="modeBtn">자동변속 전환</button>
<button id="pauseBtn">일시정지</button>
<button id="resetBtn">초기화</button>
</div>
</header>
<main>
<section class="hud">
<div class="metric"><small>속도</small><div class="value"><span id="speed">0</span> <small id="speedUnit">km/h</small></div></div>
<div class="metric"><small>주행/항해 거리</small><div class="value"><span id="distance">0.0</span> <small id="distUnit">km</small></div></div>
<div class="metric"><small>기어</small><div class="value" id="gearDisplay">1</div></div>
<div class="metric"><small>승객</small><div class="value"><span id="passengers">0</span><small> / 27명</small></div></div>
</section>
<div class="grid">
<div>
<section class="panel">
<div class="ecdis-toolbar" id="ecdisToolbar" style="display:none">
<span style="color:var(--amber);font-weight:bold">KHOA WMS API</span>
<button id="layerChart" class="active">전자해도</button>
<button id="layerDepth" class="active">수심/갯벌</button>
<button id="layerRoute" class="active">권장항로</button>
<span style="margin-left:auto;color:var(--muted)">EPSG:5186</span>
</div>
<div class="between"><h2>전방 화면</h2><span id="roadStatus" class="good">서울 반포</span></div>
<canvas id="road" width="960" height="420"></canvas>
<div class="row" id="zoomControls" style="display:none;margin-top:6px;justify-content:center">
<button id="zoomIn">🔍 확대 (+)</button>
<button id="zoomReset">줌 초기화 (1.0x)</button>
<button id="zoomOut">🔍 축소 (-)</button>
</div>
<div class="mirrors">
<div><small>좌측 사이드미러</small><canvas id="mirrorL" width="400" height="75"></canvas></div>
<div><small>우측 사이드미러</small><canvas id="mirrorR" width="400" height="75"></canvas></div>
</div>
<div class="section" id="highwaySection"><div class="between"><h2>경부고속도로 노선도</h2><small>파란색: 버스전용차로</small></div>
<canvas id="map" width="960" height="140"></canvas>
</div>
</section>
<section class="panel" style="margin-top:14px">
<h2>운전 조작</h2>
<div class="controls">
<button id="accelerate">▲ 가속 [↑]</button><button id="brake">▼ 브레이크 [↓]</button><button id="horn">📢 경적 [B]</button>
<button id="left">◀ 좌측/좌현 [←]</button><button id="center">핸들 중앙</button><button id="right">우측/우현 [→] ▶</button>
<button id="sigL">좌측 깜빡이 A</button><button id="hazard">비상등</button><button id="sigR">우측 깜빡이 F</button>
</div>
<div class="section"><div class="between"><b>핸들 / 타각</b><span id="steerDisplay">0°</span></div><div class="bar"><div id="steerBar" class="fill blue"></div></div><div class="between"><b>현재 위치</b><span id="laneDisplay">3차로</span></div></div>
</section>
<section class="panel" style="margin-top:14px">
<h2>수륙양용 주행 모드</h2>
<div class="status">주행 환경: <b id="terrainState">육상 · 경부고속도로</b></div>
<div class="row"><button id="landMode" class="active">🛣️ 육상 주행 모드</button><button id="waterMode">🌊 수상 항해 모드 (강화 동막행)</button></div>
</section>
</div>
<div>
<section class="panel">
<h2>운전 계기판</h2>
<div class="status">운행 상태: <b id="state">정차</b></div>
<div class="status">문 상태: <b id="doorState">모든 문 닫힘</b></div>
<div class="status">수소: <b id="h2Text">33.00 kg</b><div class="bar"><div id="h2Bar" class="fill"></div></div>배터리: <b id="batteryText">47.0 kWh</b><div class="bar"><div id="batteryBar" class="fill blue"></div></div></div>
<div class="section"><h2>변속기</h2><div class="gears" id="gearButtons"></div><div class="row" style="margin-top:8px"><button id="clutchBtn">클러치 Shift</button><span id="gearHelp" class="muted">수동변속 모드</span></div></div>
<div class="section">
<h2>출입문 · 부저 설정</h2>
<div class="row"><button id="openL">좌측 문 열기 (W+O)</button><button id="closeL">좌측 문 닫기 (W+P)</button></div>
<div class="row" style="margin-top:7px"><button id="openR">우측 문 열기 (E+O)</button><button id="closeR">우측 문 닫기 (E+P)</button></div>
<div class="freq">
<div><div class="between"><label for="openFreq">열림 부저</label><b><span id="openFreqText">500</span> Hz</b></div><input id="openFreq" type="range" min="500" max="580" step="10" value="500"></div>
<div><div class="between"><label for="closeFreq">닫힘 부저</label><b><span id="closeFreqText">580</span> Hz</b></div><input id="closeFreq" type="range" min="500" max="580" step="10" value="580"></div>
</div>
<div class="row"><button id="testOpen">열림 부저 시험</button><button id="testClose">닫힘 부저 시험</button></div>
</div>
</section>
<section class="panel" style="margin-top:14px"><div class="between"><h2>운행 기록</h2><button id="clearLog">기록 지우기</button></div><div id="log"></div></section>
</div></div>
</main>
<script>
"use strict";
const $=id=>document.getElementById(id),clamp=(v,a,b)=>Math.max(a,Math.min(b,v)),pressed=new Set();
const GEAR_MAP={Digit5:"1",Numpad5:"1",Digit6:"2",Numpad6:"2",Digit7:"3",Numpad7:"3",Digit8:"4",Numpad8:"4",Digit9:"5",Numpad9:"5",Digit4:"N",Numpad4:"N",Digit3:"R1",Numpad3:"R1",Digit2:"R2",Numpad2:"R2",Digit1:"R3",Numpad1:"R3"};
let s,audioCtx=null,zoomLevel=1.0,showChart=true,showDepth=true,showRoute=true;

function freshState(){
 return {speed:0,distance:0,lane:3,steer:0,mode:"manual",gear:"D1",manualGear:"1",clutch:false,leftDoor:false,rightDoor:false,doorBusy:false,signal:"off",hazard:false,passengers:0,h2:33,battery:47,paused:false,finished:false,log:[],terrain:"land",doorTimer:null}
}
s=freshState();

function initAudio(){
 if(!audioCtx){ audioCtx=new(window.AudioContext||window.webkitAudioContext)(); }
 if(audioCtx.state==="suspended"){ audioCtx.resume(); }
}
$("audioInitBtn").onclick=()=>{ initAudio(); notify("🔊 오디오 시스템 활성화됨"); };

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

function log(m){s.log.unshift("["+new Date().toLocaleTimeString("ko-KR",{hour12:false})+"] "+m);s.log=s.log.slice(0,50);$("log").innerHTML=s.log.map(v=>"<div>"+v+"</div>").join("")}
function notify(m){$("roadStatus").textContent=m;log(m)}

// --- 💾 위치 저장 및 📂 불러오기 기능 추가 ---
$("savePosBtn").onclick=()=>{
 try {
  const savePayload = { state: s, zoom: zoomLevel };
  localStorage.setItem("yeondoo_saved_position", JSON.stringify(savePayload));
  notify(`💾 현재 위치 저장 완료 (${s.terrain === "water" ? s.distance.toFixed(2) + " NM" : s.distance.toFixed(1) + " km"})`);
 } catch(e) {
  notify("❌ 위치 저장 실패");
 }
};

$("loadPosBtn").onclick=()=>{
 try {
  const savedData = localStorage.getItem("yeondoo_saved_position");
  if(savedData){
   const parsed = JSON.parse(savedData);
   s = parsed.state;
   zoomLevel = parsed.zoom || 1.0;
   
   if(s.terrain === "water"){
    $("waterMode").click();
   } else {
    $("landMode").click();
   }
   notify("📂 저장된 위치 및 데이터를 성공적으로 불러왔습니다.");
   render();
  } else {
   notify("⚠️ 저장된 위치 데이터가 없습니다.");
  }
 } catch(e) {
  notify("❌ 위치 불러오기 오류 발생");
 }
};

function doorBeep(kind){
 const freq = kind==="open" ? $("openFreq").value : $("closeFreq").value;
 playSound(freq, "square", kind==="open"?0.25:0.35);
}

function setDoor(side,open){
 if(s.doorBusy)return;
 if(open){
  if(side==="left")s.leftDoor=true;else s.rightDoor=true;
  doorBeep("open"); notify((side==="left"?"좌측":"우측")+" 문 열림"); render(); return;
 }
 const opened=side==="left"?s.leftDoor:s.rightDoor;
 if(!opened)return;
 s.doorBusy=true; doorBeep("close"); notify("문 닫는 중..."); render();
 clearTimeout(s.doorTimer);
 s.doorTimer=setTimeout(()=>{
  s.doorBusy=false;
  if(side==="left")s.leftDoor=false;else s.rightDoor=false;
  notify((side==="left"?"좌측":"우측")+" 문 닫힘"); render();
 },1000);
}

$("landMode").onclick=()=>{
 s.terrain="land"; s.speed=0; zoomLevel=1.0;
 $("landMode").classList.add("active"); $("waterMode").classList.remove("active");
 $("ecdisToolbar").style.display="none"; $("zoomControls").style.display="none"; $("highwaySection").style.display="block";
 notify("🛣️ 육상 주행 모드 전환 (경부고속도로)"); render();
};

$("waterMode").onclick=()=>{
 s.terrain="water"; s.speed=0; zoomLevel=1.0;
 $("waterMode").classList.add("active"); $("landMode").classList.remove("active");
 $("ecdisToolbar").style.display="flex"; $("zoomControls").style.display="flex"; $("highwaySection").style.display="none";
 notify("🌊 개방해(海) 시천나루 ➔ 동검도 ➔ 강화 동막해변 수상 항해 시작"); render();
};

function update(dt){
 if(s.paused||s.finished)return;

 const steerRate=35;
 if(pressed.has("ArrowLeft")){s.steer=clamp(s.steer-steerRate*dt,-60,60);}
 else if(pressed.has("ArrowRight")){s.steer=clamp(s.steer+steerRate*dt,-60,60);}
 else{s.steer+=(0-s.steer)*Math.min(1,dt*4);}

 const accel=pressed.has("ArrowUp"), brake=pressed.has("ArrowDown");
 const maxSpd = s.terrain==="water" ? 30 : 110;

 if(accel){s.speed+= (s.terrain==="water"?7.0:12.0)*dt;}
 else if(!accel&&!brake){s.speed=Math.max(0,s.speed-(s.terrain==="water"?1.2:4.0)*dt);}
 if(brake){s.speed=Math.max(0,s.speed-18.0*dt);}
 s.speed=clamp(s.speed,0,maxSpd);

 const distInc = s.terrain==="water" ? (s.speed*1.852*dt/3600) : (s.speed*dt/3600);
 if(distInc>0 && s.speed>0){
  s.distance+=distInc;
  s.h2=Math.max(0,s.h2-distInc*0.01);
 }
 render();
}

function drawRoad(){
 const c=$("road"),x=c.getContext("2d"),w=c.width,h=c.height;x.clearRect(0,0,w,h);

 if(s.terrain==="land"){
  const g=x.createLinearGradient(0,0,0,h*0.55); g.addColorStop(0,"#17344b"); g.addColorStop(1,"#8bb3c3");
  x.fillStyle=g; x.fillRect(0,0,w,h);
  x.fillStyle="#263d36"; x.fillRect(0,h*0.4,w,h*0.6);
  const hy=h*0.4, ny=h*0.96, cx=w/2, rn=w*0.94, rf=w*0.22;
  x.fillStyle="#343b43"; x.beginPath(); x.moveTo(cx-rf/2,hy); x.lineTo(cx+rf/2,hy); x.lineTo(cx+rn/2,ny); x.lineTo(cx-rn/2,ny); x.closePath(); x.fill();
  x.strokeStyle="#f7d86d"; x.lineWidth=4; x.beginPath(); x.moveTo(cx-rn/2,ny); x.lineTo(cx-rf/2,hy); x.moveTo(cx+rn/2,ny); x.lineTo(cx+rf/2,hy); x.stroke();
  return;
 }

 // 강화 동막·동검도 개방해 ECDIS 지도
 let progress=Math.min(1.0, s.distance/10.0);
 let camX=w*0.25+(w*0.2)*progress, camY=h*0.18+(h*0.7)*progress;

 x.save();
 x.translate(w/2 - camX*(zoomLevel-1), h/2 - camY*(zoomLevel-1));
 x.scale(zoomLevel, zoomLevel);

 if(showChart){
  x.fillStyle="#b4dbdd"; x.fillRect(-w*0.5, -h*0.5, w*3, h*3);
  x.fillStyle="#a2c896";
  x.beginPath(); x.moveTo(0, h*0.2); x.lineTo(w*0.4, h*0.22); x.lineTo(w*0.35, h*0.8); x.lineTo(0, h*0.8); x.closePath(); x.fill();
  x.fillStyle="#e4c979";
  x.beginPath(); x.moveTo(0, 0); x.lineTo(w*0.32, 0); x.lineTo(w*0.28, h*1.2); x.lineTo(0, h*1.2); x.closePath(); x.fill();
  x.beginPath(); x.ellipse(w*0.52, h*0.55, w*0.08, h*0.12, 0, 0, Math.PI*2); x.fill();
  x.beginPath(); x.ellipse(w*0.65, h*0.32, w*0.05, h*0.08, 0, 0, Math.PI*2); x.fill();
 }

 if(showRoute){
  x.strokeStyle="#c026d3"; x.lineWidth=2.5; x.setLineDash([8,5]);
  x.beginPath(); x.moveTo(w*0.25, h*0.18); x.quadraticCurveTo(w*0.4, h*0.38, w*0.48, h*0.55); x.quadraticCurveTo(w*0.45, h*0.75, w*0.35, h*0.9); x.stroke();
  x.setLineDash([]);
 }

 if(showDepth){
  x.fillStyle="#0f172a"; x.font="bold 13px system-ui";
  x.fillText("동검도", w*0.48, h*0.52); x.fillText("세어도", w*0.62, h*0.29); x.fillText("강화 동막해변", w*0.2, h*0.85);
  x.fillStyle="#1b3846"; x.font="11px sans-serif"; x.fillText("MS", w*0.35, h*0.45); x.fillText("M", w*0.42, h*0.68);
 }

 let curX = w*0.25 + (w*0.2) * progress + (s.steer * 0.8);
 let curY = h*0.18 + (h*0.7) * progress;
 x.fillStyle="#dc2626"; x.beginPath(); x.arc(curX, curY, 8, 0, Math.PI*2); x.fill();
 x.fillStyle="#ffffff"; x.beginPath(); x.arc(curX, curY, 3, 0, Math.PI*2); x.fill();
 x.fillStyle="#000000"; x.font="bold 12px system-ui"; x.fillText("🚌 연두 FCEV (동막행)", curX + 10, curY + 4);

 x.restore();
 x.fillStyle="rgba(228,201,121,0.95)"; x.fillRect(w - 330, h - 26, 320, 20);
 x.fillStyle="#000"; x.font="bold 11px sans-serif"; x.fillText("본 바다지도(해아름)는 항해용으로 사용할 수 없습니다.", w - 322, h - 12);
}

function render(){
 const isWater = s.terrain==="water";
 $("speed").textContent=Math.round(s.speed);
 $("distance").textContent=s.distance.toFixed(1);
 $("speedUnit").textContent = isWater ? "kn" : "km/h";
 $("distUnit").textContent = isWater ? "NM" : "km";
 $("h2Text").textContent=s.h2.toFixed(2)+" kg";
 $("steerDisplay").textContent=s.steer<0?`좌 ${Math.abs(Math.round(s.steer))}°`:s.steer>0?`우 ${Math.round(s.steer)}°`:`중앙 0°`;
 $("terrainState").textContent=isWater ? "수상 · 강화 동막해변 항로" : "육상 · 경부고속도로";
 $("doorState").textContent=s.doorBusy?"문 닫는 중":s.leftDoor&&s.rightDoor?"양쪽 문 열림":s.leftDoor?"좌측 문 열림":s.rightDoor?"우측 문 열림":"모든 문 닫힘";

 drawRoad();
}

$("layerChart").onclick=()=>{showChart=!showChart;$("layerChart").classList.toggle("active",showChart);render()};
$("layerDepth").onclick=()=>{showDepth=!showDepth;$("layerDepth").classList.toggle("active",showDepth);render()};
$("layerRoute").onclick=()=>{showRoute=!showRoute;$("layerRoute").classList.toggle("active",showRoute);render()};

$("road").addEventListener("wheel",e=>{if(s.terrain==="water"){e.preventDefault();zoomLevel=e.deltaY<0?Math.min(3.0,zoomLevel+0.1):Math.max(0.5,zoomLevel-0.1);render();}},{passive:false});
$("zoomIn").onclick=()=>{zoomLevel=Math.min(3.0,zoomLevel+0.2);render()};
$("zoomOut").onclick=()=>{zoomLevel=Math.max(0.5,zoomLevel-0.1);render()};
$("zoomReset").onclick=()=>{zoomLevel=1.0;render()};

$("horn").onclick=()=>{playSound(140, "sawtooth", 0.6); notify("📢 경적 뿜- 울림");};
$("testOpen").onclick=()=>doorBeep("open");
$("testClose").onclick=()=>doorBeep("close");
$("openFreq").oninput=()=>{$("openFreqText").textContent=$("openFreq").value;};
$("closeFreq").oninput=()=>{$("closeFreqText").textContent=$("closeFreq").value;};

$("openL").onclick=()=>setDoor("left",true);$("closeL").onclick=()=>setDoor("left",false);
$("openR").onclick=()=>setDoor("right",true);$("closeR").onclick=()=>setDoor("right",false);

function bindHold(id,key){const el=$(id);el.addEventListener("pointerdown",e=>{e.preventDefault();pressed.add(key)});const r=()=>pressed.delete(key);el.addEventListener("pointerup",r);el.addEventListener("pointercancel",r)}
bindHold("accelerate","ArrowUp");bindHold("brake","ArrowDown");bindHold("left","ArrowLeft");bindHold("right","ArrowRight");

$("pauseBtn").onclick=()=>{s.paused=!s.paused;notify(s.paused?"일시정지":"운행 재개")};
$("resetBtn").onclick=()=>{s=freshState();zoomLevel=1.0;notify("초기화 완료");render()};

window.addEventListener("keydown",e=>{
 if(["ArrowUp","ArrowDown","ArrowLeft","ArrowRight","Space"].includes(e.code))e.preventDefault();
 pressed.add(e.code); initAudio();
});
window.addEventListener("keyup",e=>pressed.delete(e.code));
window.addEventListener("blur",()=>{pressed.clear();s.steer=0;render();});

let last=performance.now();
function frame(now){const dt=Math.min((now-last)/1000,.05);last=now;update(dt);requestAnimationFrame(frame)}
log("연두 FCEV 수륙양용 시뮬레이터 로드 완료.");
render();requestAnimationFrame(frame);
</script>
</body>
</html>
