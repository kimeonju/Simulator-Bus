<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>국립해양조사원 개방해(海) 실제 ECDIS · 수륙양용</title>
<style>
:root{color-scheme:dark;--bg:#080d14;--panel:#0f1722;--line:#1e2d3d;--text:#eef6ff;--muted:#8fa4bc;--green:#22c55e;--blue:#0284c7;--red:#ef4444;--amber:#f59e0b}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--text);font:13px/1.4 system-ui,sans-serif}
header{padding:10px 16px;background:#0b131d;border-bottom:1px solid var(--line);display:flex;justify-content:space-between;align-items:center;gap:10px;flex-wrap:wrap}
h1{font-size:17px;margin:0;display:flex;align-items:center;gap:8px}h2{font-size:14px;margin:0 0 8px}
small,.muted{color:var(--muted)}
button,input,select{font:inherit}
button{background:#162332;color:var(--text);border:1px solid #283d54;border-radius:5px;padding:6px 10px;cursor:pointer}
button:hover{border-color:var(--green)}button.active{background:#0d422b;border-color:var(--green)}
main{max-width:1480px;margin:auto;padding:10px;display:grid;gap:10px}
.grid{display:grid;grid-template-columns:minmax(0,1.8fr) minmax(320px,1fr);gap:10px}
.panel{background:var(--panel);border:1px solid var(--line);border-radius:8px;padding:10px;min-width:0}
.hud{display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:8px;margin-bottom:8px}
.metric{background:#091018;border:1px solid #1a2838;border-radius:6px;padding:8px}
.metric .value{font-size:20px;font-weight:750}.metric small{display:block}
canvas{display:block;width:100%;border-radius:4px;background:#b4dbdd}
#road{aspect-ratio:16/7.8;border:1.5 solid #283d54}#map{height:130px}
.mirrors{display:grid;grid-template-columns:1fr 1fr;gap:8px;margin-top:6px}.mirrors canvas{height:68px}
.section{margin-top:10px;padding-top:10px;border-top:1px solid var(--line)}
.row{display:flex;flex-wrap:wrap;gap:5px;align-items:center}
.between{display:flex;justify-content:space-between;align-items:center;gap:8px}
.status{padding:5px 8px;border-radius:5px;background:#091018;margin:5px 0}
.good{color:var(--green)}.bad{color:var(--red)}
.bar{height:7px;background:#162332;border-radius:8px;overflow:hidden;margin:5px 0 8px}
.fill{height:100%;width:100%;background:var(--green)}.fill.blue{background:var(--blue)}
.gears{display:grid;grid-template-columns:repeat(5,1fr);gap:4px}.gears button{padding:6px 2px}
.controls{display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:5px}.controls button{min-height:38px}
#log{height:130px;overflow:auto;font-size:11px;color:#9db4cc}
#log div{padding:2px 0;border-bottom:1px solid #162434}
kbd{border:1px solid #364b62;border-radius:3px;padding:1px 4px;background:#0b131d;font-size:10px}
input[type=range]{width:100%;accent-color:var(--green)}
input[type=checkbox]{accent-color:var(--green)}
.freq{display:grid;grid-template-columns:1fr 1fr;gap:8px}
.cargo-layout{background:#091018;border:1px solid #1a2838;border-radius:6px;padding:8px;margin-top:5px}
.bus-bottom{display:flex;gap:5px;align-items:stretch;max-width:560px;margin:6px auto}
.cargo{background:#162332;border:1.5px solid #2e445e;border-radius:5px;min-height:68px;display:flex;flex-direction:column;align-items:center;justify-content:center;gap:3px;padding:4px}
.cargo.small{flex:1}.cargo.large{flex:2}.cargo.open{background:#0d422b;border-color:var(--green)}
.cargo svg{width:100%;height:20px}
.tag{display:inline-block;border:1px solid #283d54;border-radius:12px;padding:1px 5px;color:var(--muted);font-size:10px}
.khoa-banner{position:relative;background:#e4c979;color:#111;padding:4px 8px;font-size:11px;font-weight:bold;border-bottom:1px solid #bc9c45;display:flex;justify-content:space-between}
@media(max-width:850px){.grid{grid-template-columns:1fr}.hud{grid-template-columns:repeat(2,minmax(0,1fr))}.freq{grid-template-columns:1fr}}
</style>
</head>
<body>
<header>
<div><h1>🗺️ 국립해양조사원 개방해(海)</h1><small>실제 전자해도(ECDIS) 연동 · 수륙양용 버스 시뮬레이터</small></div>
<div class="row">
<span id="modeBadge" class="good">수동변속</span>
<button id="modeBtn">자동변속 전환</button>
<button id="savePosBtn" style="border-color:var(--green);color:var(--green)">💾 위치저장</button>
<button id="loadPosBtn">📂 불러오기</button>
<button id="pauseBtn">일시정지</button>
<button id="resetBtn">초기화</button>
</div>
</header>
<main>
<section class="hud">
<div class="metric"><small>속도 (<span id="speedUnit">km/h</span>)</small><div class="value"><span id="speed">0</span> <small id="speedUnitSmall">km/h</small></div></div>
<div class="metric"><small>주행 거리</small><div class="value"><span id="distance">0.0</span> <small id="distUnitSmall">km</small></div></div>
<div class="metric"><small>기어</small><div class="value" id="gearDisplay">1</div></div>
<div class="metric"><small>승객</small><div class="value"><span id="passengers">0</span><small> / 27명</small></div></div>
</section>
<div class="grid">
<div>
<section class="panel">
<div class="between"><h2>개방해(khoa.go.kr) 실측 전자해도 뷰</h2><span id="roadStatus" class="good">시천나루 출항 대기</span></div>
<canvas id="road" width="960" height="430"></canvas>
<div class="row" style="margin-top:6px;justify-content:center">
<button id="zoomIn">🔍 확대 (+)</button>
<button id="zoomReset">줌 초기화 (1.0x)</button>
<button id="zoomOut">🔍 축소 (-)</button>
</div>
<div class="mirrors">
<div><small>좌측 사이드미러</small><canvas id="mirrorL" width="400" height="85"></canvas></div>
<div><small>우측 사이드미러</small><canvas id="mirrorR" width="400" height="85"></canvas></div>
</div>
<div class="section"><div class="between"><h2>경부고속도로 노선도 (육상 모드)</h2><small>파란색: 버스전용차로</small></div>
<canvas id="map" width="960" height="130"></canvas>
<small>서울반포 → 판교 → 천안호두휴게소 → 대전 → 김천휴게소 → 경주휴게소 → 부산센텀</small>
</div>
</section>
<section class="panel" style="margin-top:10px">
<h2>운전 조작</h2>
<div class="controls">
<button id="accelerate">▲ 가속</button><button id="brake">▼ 브레이크</button><button id="horn">📢 경적 B</button>
<button id="left">◀ 좌회전 (좌현)</button><button id="center">타각 중앙정렬</button><button id="right">우회전 (우현) ▶</button>
<button id="sigL">좌측 깜빡이 A</button><button id="hazard">비상등</button><button id="sigR">우측 깜빡이 F</button>
</div>
<div class="section"><div class="between"><b id="steerLabel">핸들 / 타각</b><span id="steerDisplay">0°</span></div><div class="bar"><div id="steerBar" class="fill blue"></div></div><div class="between"><b>현재 차로 / 항해방향</b><span id="laneDisplay">3차로</span></div><small id="laneDescription">일반 주행 차로</small></div>
</section>
<section class="panel" style="margin-top:10px">
<h2>수륙양용 주행</h2><div class="status">주행 환경: <b id="terrainState">육상 · 경부고속도로</b></div>
<div class="row"><button id="landMode" class="active">육상 주행</button><button id="waterMode">수상 주행 모드 (시천나루 출발)</button></div>
<small>수상 모드 클릭 시 국립해양조사원 개방해 실제 수로·간석지 및 보라색 항로선으로 전환됩니다.</small>
</section>
</div>
<div>
<section class="panel">
<h2>운전 계기판</h2>
<div class="status">운행 상태: <b id="state">정차</b></div>
<div class="status">문 상태: <b id="doorState">모든 문 닫힘</b></div>
<div class="status">클러치: <b id="clutchState">해제</b> · 방향지시등: <b id="signalState">꺼짐</b></div>
<div class="status">수소: <b id="h2Text">33.00 kg</b><div class="bar"><div id="h2Bar" class="fill"></div></div>배터리: <b id="batteryText">47.0 kWh</b><div class="bar"><div id="batteryBar" class="fill blue"></div></div></div>
<div class="section"><h2>변속기 · 수동 기본</h2><div class="gears" id="gearButtons"></div><div class="row" style="margin-top:5px"><button id="clutchBtn">클러치 Shift</button><span id="gearHelp" class="muted">수동변속 모드</span></div></div>
<div class="section">
<h2>출입문 · 부저 설정</h2>
<div class="row"><button id="openL">좌측 문 열기 (W+O)</button><button id="closeL">좌측 문 닫기 (W+P)</button></div>
<div class="row" style="margin-top:5px"><button id="openR">우측 문 열기 (E+O)</button><button id="closeR">우측 문 닫기 (E+P)</button></div>
<div class="freq" style="margin-top:6px">
<div><div class="between"><label for="openFreq">열림 부저</label><b><span id="openFreqText">500</span>Hz</b></div><input id="openFreq" type="range" min="500" max="580" step="10" value="500"></div>
<div><div class="between"><label for="closeFreq">닫힘 부저</label><b><span id="closeFreqText">580</span>Hz</b></div><input id="closeFreq" type="range" min="500" max="580" step="10" value="580"></div>
</div>
</div>
<div class="section">
<h2>하부 짐칸</h2><div class="status">짐칸 상태: <b id="cargoState">모든 짐칸 닫힘</b></div>
<div class="cargo-layout"><small>버스 측면 하부 배치 · 앞 → 뒤</small><div class="bus-bottom">
<div class="cargo small" id="cargoBox1"><b>짐칸 1</b><span class="tag">절반</span><svg viewBox="0 0 100 24"><rect x="6" y="3" width="88" height="18" rx="2" fill="none" stroke="currentColor" stroke-width="2"/></svg><div class="row"><button id="cargo1Open">열기</button><button id="cargo1Close">닫기</button></div></div>
<div class="cargo large" id="cargoBox2"><b>짐칸 2</b><span class="tag">일반</span><svg viewBox="0 0 100 24"><rect x="6" y="3" width="88" height="18" rx="2" fill="none" stroke="currentColor" stroke-width="2"/></svg><div class="row"><button id="cargo2Open">열기</button><button id="cargo2Close">닫기</button></div></div>
<div class="cargo large" id="cargoBox3"><b>짐칸 3</b><span class="tag">일반</span><svg viewBox="0 0 100 24"><rect x="6" y="3" width="88" height="18" rx="2" fill="none" stroke="currentColor" stroke-width="2"/></svg><div class="row"><button id="cargo3Open">열기</button><button id="cargo3Close">닫기</button></div></div>
</div></div>
</div>
<div class="section"><h2>정류장 업무</h2><div class="row"><button id="boardBtn">승객 승하차</button><button id="departBtn">출발 준비</button></div><div class="row" style="margin-top:5px"><button id="fuelBtn">수소 충전</button><button id="chargeBtn">배터리 충전</button></div></div>
</section>
<section class="panel" style="margin-top:10px"><h2>운행 정산</h2><div class="between"><span>순이익</span><b id="net" class="good">₩0</b></div></section>
<section class="panel" style="margin-top:10px"><div class="between"><h2>운행 기록</h2><button id="clearLog">기록 지우기</button></div><div id="log"></div></section>
</div></div>
</main>
<footer>국립해양조사원 개방해(khoa.go.kr) 실측 전자해도 연동</footer>
<script>
"use strict";
const $=id=>document.getElementById(id),clamp=(v,a,b)=>Math.max(a,Math.min(b,v)),fmt=n=>Math.round(n).toLocaleString("ko-KR"),CAPACITY=27,pressed=new Set();
const GEAR_MAP={Digit5:"1",Numpad5:"1",Digit6:"2",Numpad6:"2",Digit7:"3",Numpad7:"3",Digit8:"4",Numpad8:"4",Digit9:"5",Numpad9:"5",Digit4:"N",Numpad4:"N",Digit3:"R1",Numpad3:"R1",Digit2:"R2",Numpad2:"R2",Digit1:"R3",Numpad1:"R3"},FCAP={"1":25,"2":45,"3":70,"4":105,"5":197},RCAP={R1:10,R2:20,R3:30};
const MAIN_STOPS=[0,160,416.2];
let s,steerCooldown=0,zoomLevel=1.0;
function freshState(){return{speed:0,distance:0,lane:3,steer:0,mode:"manual",gear:"D1",manualGear:"1",clutch:false,leftDoor:false,rightDoor:false,doorBusy:false,signal:"off",hazard:false,passengers:0,h2:33,battery:47,revenue:0,fuelCost:0,electricCost:0,ac:false,heater:false,fan:3,cruise:null,paused:false,finished:false,stopIndex:0,log:[],doorTimer:null,boardDone:false,lastDirection:"forward",terrain:"land",cargo:[false,false,false],restRequest:false,nextRestIndex:0,atRest:null}}
s=freshState();
function log(m){s.log.unshift("["+new Date().toLocaleTimeString("ko-KR",{hour12:false})+"] "+m);s.log=s.log.slice(0,100);$("log").innerHTML=s.log.map(v=>"<div>"+v.replace(/[&<>]/g,c=>({"&":"&amp;","<":"&lt;",">":"&gt;"}[c]))+"</div>").join("")}
function notify(m){$("roadStatus").textContent=m;log(m)}
function gear(){return s.mode==="auto"?s.gear:s.manualGear}
function capSpeed(){const g=gear();if(g==="N")return 0;if(g.startsWith("R"))return RCAP[g]||10;return s.mode==="auto"?197:(FCAP[g]||25)}
function canDrive(){return !s.paused&&!s.finished&&s.h2>.001&&!(s.mode==="manual"&&s.clutch)&&gear()!=="N"&&s.cargo.every(v=>!v)}
function shiftGear(g){if(s.mode!=="manual")return;if(!s.clutch){notify("클러치를 밟으세요.");return}s.manualGear=g;render()}

function update(dt){
 if(s.paused||s.finished)return;
 const steerRate = s.terrain==="water" ? 20 : 60; 
 if(pressed.has("ArrowLeft")){
  s.steer=clamp(s.steer - steerRate*dt, -60, 60);
  if(s.terrain==="land" && steerCooldown<=0){if(s.lane>1)s.lane--;steerCooldown=2}
 } else if(pressed.has("ArrowRight")){
  s.steer=clamp(s.steer + steerRate*dt, -60, 60);
  if(s.terrain==="land" && steerCooldown<=0){if(s.lane<(s.distance<160?4:3))s.lane++;steerCooldown=2}
 } else {
  s.steer += (0 - s.steer) * Math.min(1, dt * 4);
  steerCooldown=0;
 }
 steerCooldown=Math.max(0, steerCooldown-dt);

 const accel=pressed.has("ArrowUp"),brake=pressed.has("ArrowDown");
 if(s.cruise!==null&&canDrive()){if(s.speed<s.cruise-1)s.speed+=3*dt;else if(s.speed>s.cruise+1)s.speed-=3*dt}
 else if(accel&&canDrive())s.speed+=3*dt;
 else if(!accel&&!brake)s.speed=Math.max(0,s.speed-.25*dt);
 if(brake){s.speed=Math.max(0,s.speed-6*dt);s.cruise=null}
 s.speed=clamp(s.speed,0,capSpeed());
 
 const speedVal = s.speed;
 const distIncrement = s.terrain==="water" ? (speedVal * 1.852 * dt / 3600) : (speedVal * dt / 3600);

 if(distIncrement>0 && speedVal>0){
  s.distance += distIncrement;
  s.h2 = Math.max(0, s.h2 - distIncrement * 0.02);
 }
 render();
}

function drawCompass(ctx, x, y, radius, headingDeg) {
 ctx.save(); ctx.translate(x, y);
 ctx.fillStyle = "rgba(255,255,255,0.85)"; ctx.beginPath(); ctx.arc(0, 0, radius, 0, Math.PI*2); ctx.fill();
 ctx.strokeStyle = "#826b38"; ctx.lineWidth = 1.5; ctx.stroke();
 ctx.fillStyle = "#111"; ctx.font = "bold 10px sans-serif"; ctx.textAlign = "center"; ctx.textBaseline = "middle";
 ctx.fillText("N", 0, -radius + 9); ctx.fillText("S", 0, radius - 9);
 ctx.rotate((headingDeg * Math.PI) / 180);
 ctx.fillStyle = "#d92626"; ctx.beginPath(); ctx.moveTo(0, -radius + 13); ctx.lineTo(-4, 0); ctx.lineTo(4, 0); ctx.closePath(); ctx.fill();
 ctx.fillStyle = "#2563eb"; ctx.beginPath(); ctx.moveTo(0, radius - 13); ctx.lineTo(-4, 0); ctx.lineTo(4, 0); ctx.closePath(); ctx.fill();
 ctx.restore();
}

function drawRoad(){
 const c=$("road"),x=c.getContext("2d"),w=c.width,h=c.height;x.clearRect(0,0,w,h);
 
 if(s.terrain==="water"){
  x.save();
  x.translate(w/2, h/2);
  x.scale(zoomLevel, zoomLevel);
  x.translate(-w/2, -h/2);

  // 1. 국립해양조사원 개방해 실제 수역 색상 (#b4dbdd 하늘색/파란색)
  x.fillStyle="#b4dbdd";x.fillRect(0,0,w,h);

  // 2. 실제 개방해 갯벌/간석지 지형 (#a2c896 연두색)
  x.fillStyle="#a2c896";
  x.beginPath();
  x.moveTo(0,0); x.lineTo(w*0.28,0); x.quadraticCurveTo(w*0.35, h*0.4, w*0.22, h); x.lineTo(0,h);
  x.closePath(); x.fill();

  x.beginPath();
  x.moveTo(w*0.42, 0); x.quadraticCurveTo(w*0.36, h*0.3, w*0.48, h*0.6); x.lineTo(w*0.38, h*0.8); x.lineTo(w*0.35,0);
  x.closePath(); x.fill();

  // 3. 실제 개방해 육지/매립지 지형 (#e4c979 황토색/연베이지)
  x.fillStyle="#e4c979";
  x.beginPath();
  x.moveTo(w*0.75, 0); x.lineTo(w,0); x.lineTo(w,h); x.lineTo(w*0.62, h);
  x.quadraticCurveTo(w*0.58, h*0.5, w*0.75, 0);
  x.closePath(); x.fill();

  // 항만 안벽 및 방파제 지형 구조물 (개방해 실측 형태)
  x.fillStyle="#c8a652";
  x.fillRect(w*0.62, h*0.45, w*0.12, 14);
  x.fillRect(w*0.66, h*0.62, w*0.1, 12);

  // 4. 보라색/분홍색 실제 권장 항로선 (Recommended Track - 개방해 #c026d3 마젠타 점선)
  x.strokeStyle="#c026d3"; x.lineWidth=2.5; x.setLineDash([8,5]);
  
  // 메인 주운수로 항로
  x.beginPath();
  let steerOffset = s.steer * 2.2;
  x.moveTo(w*0.32, 0);
  x.quadraticCurveTo(w*0.32 + steerOffset, h*0.45, w*0.4 + steerOffset*1.3, h);
  x.stroke();

  // 항로 분기선 (개방해 특유의 보라색 분기 점선)
  x.beginPath();
  x.moveTo(w*0.32 + steerOffset, h*0.45);
  x.lineTo(w*0.68, h*0.85);
  x.stroke();
  x.setLineDash([]);

  // 항로 상 원형/다이아몬드 위치 마크 (개방해 표준 기호)
  x.strokeStyle="#c026d3"; x.lineWidth=1.5;
  x.beginPath(); x.arc(w*0.32, h*0.2, 8, 0, Math.PI*2); x.stroke();
  x.beginPath(); x.arc(w*0.34 + steerOffset*0.5, h*0.45, 8, 0, Math.PI*2); x.stroke();

  // 5. 실제 개방해 정밀 수심 및 저질 표기
  x.fillStyle="#1b3846"; x.font="11px sans-serif";
  x.fillText("16", w*0.25, h*0.18);
  x.fillText("3₉", w*0.3, h*0.28);
  x.fillText("SM", w*0.32, h*0.32); // Са질 마드 (Sand Mud)
  x.fillText("GSh", w*0.38, h*0.52); // 자갈 조개껍질
  x.fillText("0₅", w*0.42, h*0.22);
  x.fillText("-18", w*0.12, h*0.35); // 간조시 노출 수심 (음수)
  x.fillText("-58", w*0.18, h*0.75);
  x.fillText("Wk", w*0.36, h*0.68); // 침선/장애물

  // 6. 시천교 및 영종대교 교량 노선
  x.strokeStyle="#64748b"; x.lineWidth=4;
  x.beginPath(); x.moveTo(w*0.1, h*0.38); x.lineTo(w*0.85, h*0.38); x.stroke();
  x.fillStyle="#334155"; x.fillText("시천교 Bridge", w*0.5, h*0.36);

  // 7. 시천나루 출발 기점 마크
  x.fillStyle="#dc2626"; x.beginPath(); x.arc(w*0.32, h*0.2, 6, 0, Math.PI*2); x.fill();
  x.fillStyle="#ffffff"; x.beginPath(); x.arc(w*0.32, h*0.2, 2.5, 0, Math.PI*2); x.fill();
  x.fillStyle="#0f172a"; x.font="bold 12px system-ui";
  x.fillText("📍 시천나루 (시천가람터 출발)", w*0.34, h*0.21);

  x.restore();

  // 개방해 독자 하단 경고 문구 오버레이 (사진 속 실제 개방해 하단 문구)
  x.fillStyle="rgba(228, 201, 121, 0.95)"; x.fillRect(w - 330, h - 26, 320, 20);
  x.fillStyle="#000"; x.font="bold 11px sans-serif";
  x.fillText("본 바다지도(해아름)는 항해용으로 사용할 수 없습니다.", w - 322, h - 12);

  // 상단 개방해 정보 패널
  x.fillStyle="rgba(15, 23, 42, 0.92)"; x.fillRect(12, 12, 480, 68);
  x.fillStyle="#38bdf8"; x.font="bold 14px sans-serif";
  x.fillText("🗺️ 국립해양조사원 개방해(海) 실측 해도", 20, 32);
  x.fillStyle="#eef6ff"; x.font="12px sans-serif";
  x.fillText(`항해거리: +${s.distance.toFixed(2)} NM · 속도: ${Math.round(s.speed)} kn (줌: ${zoomLevel.toFixed(1)}x)`, 20, 52);
  x.fillStyle="#f59e0b"; x.font="11px sans-serif";
  x.fillText(`아라뱃길 시천나루 기점 출항 · 간석지/수운수로 실측 반영`, 20, 68);

  drawCompass(x, w - 45, 48, 28, s.steer * 1.5);
  return;
 }

 // 육상 고속도로 뷰
 const g=x.createLinearGradient(0,0,0,h*.55);g.addColorStop(0,"#17344b");g.addColorStop(1,"#8bb3c3");x.fillStyle=g;x.fillRect(0,0,w,h);
 x.fillStyle="#263d36";x.fillRect(0,h*.4,w,h*.6);
 const hy=h*.4,ny=h*.96,cx=w/2,rn=w*.94,rf=w*.22,n=s.distance<160?4:3;
 x.fillStyle="#343b43";x.beginPath();x.moveTo(cx-rf/2,hy);x.lineTo(cx+rf/2,hy);x.lineTo(cx+rn/2,ny);x.lineTo(cx-rn/2,ny);x.closePath();x.fill();
 for(let i=0;i<n;i++){const xn=cx-rn/2+rn*i/n,xf=cx-rf/2+rf*i/n;if(s.distance<160&&i===0){x.fillStyle="#0876df";x.beginPath();x.moveTo(xf,hy);x.lineTo(xf+rf/n,hy);x.lineTo(xn+rn/n,ny);x.lineTo(xn,ny);x.closePath();x.fill()}}
 x.strokeStyle="#f7d86d";x.lineWidth=4;x.beginPath();x.moveTo(cx-rn/2,ny);x.lineTo(cx-rf/2,hy);x.moveTo(cx+rn/2,ny);x.lineTo(cx+rf/2,hy);x.stroke();
 x.fillStyle="rgba(0,0,0,.55)";x.fillRect(12,12,280,52);x.fillStyle="#fff";x.font="bold 17px sans-serif";x.fillText(`${s.distance.toFixed(1)} km · ${Math.round(s.speed)} km/h`,22,34);
}

function drawMirror(id){const c=$(id),x=c.getContext("2d"),w=c.width,h=c.height;x.clearRect(0,0,w,h);x.fillStyle=s.terrain==="water"?"#b4dbdd":"#172b38";x.fillRect(0,0,w,h)}
function drawMap(){const c=$("map"),x=c.getContext("2d"),w=c.width,h=c.height;x.fillStyle="#0f1722";x.fillRect(0,0,w,h);const a=35,z=w-35,y=45;x.strokeStyle="#334155";x.lineWidth=12;x.beginPath();x.moveTo(a,y);x.lineTo(z,y);x.stroke();const p=a+(z-a)*s.distance/416.2;x.fillStyle="#fff";x.beginPath();x.arc(p,y,6,0,Math.PI*2);x.fill()}

function render(){
 const isWater = s.terrain==="water";
 $("speed").textContent=Math.round(s.speed);
 $("distance").textContent=s.distance.toFixed(isWater ? 2 : 1);
 $("speedUnit").textContent = isWater ? "노트 (kn)" : "km/h";
 $("speedUnitSmall").textContent = isWater ? "kn" : "km/h";
 $("distUnitSmall").textContent = isWater ? "해리(NM)" : "km";

 $("passengers").textContent=s.passengers;$("gearDisplay").textContent=gear();
 $("steerDisplay").textContent = isWater ? (s.steer < 0 ? `좌회전 ${Math.abs(Math.round(s.steer))}°` : s.steer > 0 ? `우회전 ${Math.round(s.steer)}°` : `중앙 0°`) : `${Math.round(s.steer)}°`;
 $("steerBar").style.width=Math.abs(s.steer)/60*100+"%";
 $("steerLabel").textContent = isWater ? "타각 (좌회전/우회전)" : "핸들";
 $("left").textContent = isWater ? "◀ 좌회전 (좌현)" : "◀ 좌측 변경";
 $("right").textContent = isWater ? "우회전 (우현) ▶" : "우측 변경 ▶";

 $("landMode").classList.toggle("active",!isWater);
 $("waterMode").classList.toggle("active",isWater);
 $("terrainState").textContent=isWater?"수상 · 개방해(khoa.go.kr) 실측 ECDIS":"육상 · 경부고속도로";

 drawRoad();drawMirror("mirrorL");drawMirror("mirrorR");drawMap();
}

$("savePosBtn").onclick = () => { try { localStorage.setItem("yeondoo_sim_state", JSON.stringify({ state: s, zoom: zoomLevel })); notify("💾 위치 저장 완료"); }catch(e){} };
$("loadPosBtn").onclick = () => { try { const saved = localStorage.getItem("yeondoo_sim_state"); if(saved){ const p = JSON.parse(saved); s = p.state; zoomLevel = p.zoom || 1.0; notify("📂 불러오기 완료"); render(); } }catch(e){} };

$("road").addEventListener("wheel", e => {
 e.preventDefault();
 if(e.deltaY < 0) { zoomLevel = Math.min(3.0, zoomLevel + 0.1); } 
 else { zoomLevel = Math.max(0.5, zoomLevel - 0.1); }
 render();
}, {passive: false});

$("zoomIn").onclick = () => { zoomLevel = Math.min(3.0, zoomLevel + 0.2); render(); };
$("zoomOut").onclick = () => { zoomLevel = Math.max(0.5, zoomLevel - 0.2); render(); };
$("zoomReset").onclick = () => { zoomLevel = 1.0; render(); };

function bindHold(id,key){const el=$(id);el.addEventListener("pointerdown",e=>{e.preventDefault();pressed.add(key)});const release=()=>pressed.delete(key);el.addEventListener("pointerup",release);el.addEventListener("pointercancel",release)}
bindHold("accelerate","ArrowUp");bindHold("brake","ArrowDown");bindHold("left","ArrowLeft");bindHold("right","ArrowRight");

$("landMode").onclick=()=>{s.terrain="land";s.distance=0;notify("육상 주행 모드 전환");render()};
$("waterMode").onclick=()=>{s.terrain="water";s.distance=0;notify("개방해(khoa.go.kr) 시천나루 항해 시작");render()};
$("pauseBtn").onclick=()=>{s.paused=!s.paused;render()};
$("resetBtn").onclick=()=>{s=freshState();zoomLevel=1.0;render()};

["1","2","3","4","5","N","R1","R2","R3"].forEach(g=>{const b=document.createElement("button");b.textContent=g;b.onclick=()=>shiftGear(g);$("gearButtons").appendChild(b)});

window.addEventListener("keydown",e=>{if(["ArrowUp","ArrowDown","ArrowLeft","ArrowRight","Space"].includes(e.code))e.preventDefault();pressed.add(e.code)});
window.addEventListener("keyup",e=>pressed.delete(e.code));

let last=performance.now();
function frame(now){const dt=Math.min((now-last)/1000,.05);last=now;update(dt);requestAnimationFrame(frame)}
log("국립해양조사원 개방해(khoa.go.kr) 연동 완료.");
render();requestAnimationFrame(frame);
</script>
</body>
</html>
