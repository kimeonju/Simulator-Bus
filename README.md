<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>연두 Yeondoo FCEV · 수륙양용</title>
<style>
:root{color-scheme:dark;--bg:#0b1119;--panel:#141f2d;--line:#2b3d50;--text:#eef6ff;--muted:#94a9bf;--green:#49e0a0;--blue:#258dff;--red:#ff6576;--amber:#ffc45c}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--text);font:14px/1.5 system-ui,sans-serif}
header{padding:18px;background:#111b28;border-bottom:1px solid var(--line);display:flex;justify-content:space-between;align-items:center;gap:12px;flex-wrap:wrap}
h1{font-size:21px;margin:0}h2{font-size:15px;margin:0 0 12px}
small,.muted{color:var(--muted)}
button,input,select{font:inherit}
button{background:#223348;color:var(--text);border:1px solid #3a5068;border-radius:8px;padding:9px 11px;cursor:pointer}
button:hover{border-color:var(--green)}button.active{background:#145c48;border-color:var(--green)}
main{max-width:1450px;margin:auto;padding:14px;display:grid;gap:14px}
.grid{display:grid;grid-template-columns:minmax(0,1.7fr) minmax(300px,1fr);gap:14px}
.panel{background:var(--panel);border:1px solid var(--line);border-radius:13px;padding:14px;min-width:0}
.hud{display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:9px;margin-bottom:12px}
.metric{background:#0b1520;border:1px solid #24374a;border-radius:9px;padding:10px}
.metric .value{font-size:23px;font-weight:750}.metric small{display:block}
canvas{display:block;width:100%;border-radius:8px;background:#101b26}
#road{aspect-ratio:16/7}#map{height:155px}
.mirrors{display:grid;grid-template-columns:1fr 1fr;gap:8px;margin-top:10px}.mirrors canvas{height:78px}
.section{margin-top:14px;padding-top:13px;border-top:1px solid var(--line)}
.row{display:flex;flex-wrap:wrap;gap:7px;align-items:center}
.between{display:flex;justify-content:space-between;align-items:center;gap:8px}
.status{padding:7px 10px;border-radius:7px;background:#0b1520;margin:8px 0}
.good{color:var(--green)}.bad{color:var(--red)}
.bar{height:9px;background:#263548;border-radius:20px;overflow:hidden;margin:7px 0 12px}
.fill{height:100%;width:100%;background:var(--green)}.fill.blue{background:var(--blue)}
.gears{display:grid;grid-template-columns:repeat(5,1fr);gap:5px}.gears button{padding:8px 3px}
.controls{display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:7px}.controls button{min-height:44px}
#log{height:160px;overflow:auto;font-size:12px;color:#bdcddd}
#log div{padding:4px 0;border-bottom:1px solid #233243}
kbd{border:1px solid #51647a;border-radius:4px;padding:1px 5px;background:#101923}
input[type=range]{width:100%;accent-color:var(--green)}
input[type=checkbox]{accent-color:var(--green)}
.freq{display:grid;grid-template-columns:1fr 1fr;gap:12px}
.cargo-layout{background:#0b1520;border:1px solid #26384a;border-radius:10px;padding:12px;margin-top:8px}
.bus-bottom{display:flex;gap:8px;align-items:stretch;max-width:620px;margin:10px auto}
.cargo{background:#203246;border:2px solid #4b647b;border-radius:8px;min-height:86px;display:flex;flex-direction:column;align-items:center;justify-content:center;gap:5px;padding:6px}
.cargo.small{flex:1}.cargo.large{flex:2}.cargo.open{background:#145c48;border-color:var(--green)}
.cargo svg{width:100%;height:28px}
.tag{display:inline-block;border:1px solid #3a5068;border-radius:20px;padding:2px 8px;color:var(--muted);font-size:12px}
@media(max-width:850px){.grid{grid-template-columns:1fr}.hud{grid-template-columns:repeat(2,minmax(0,1fr))}.freq{grid-template-columns:1fr}}
</style>
</head>
<body>
<header>
<div><h1>🚌 연두 Yeondoo FCEV</h1><small>경부고속도로 기반 · 27석 수소전기 수륙양용버스</small></div>
<div class="row"><span id="modeBadge" class="good">수동변속</span><button id="modeBtn">자동변속으로 전환</button><button id="pauseBtn">일시정지</button><button id="resetBtn">초기화</button></div>
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
<div class="between"><h2>전방 도로 / ECDIS 해도 뷰 (마우스 휠로 확대/축소)</h2><span id="roadStatus" class="good">서울 반포</span></div>
<canvas id="road" width="960" height="420"></canvas>
<div class="row" style="margin-top:8px;justify-content:center">
<button id="zoomIn">🔍 확대 (+)</button>
<button id="zoomReset">줌 초기화 (1.0x)</button>
<button id="zoomOut">🔍 축소 (-)</button>
</div>
<div class="mirrors">
<div><small>좌측 사이드미러</small><canvas id="mirrorL" width="400" height="90"></canvas></div>
<div><small>우측 사이드미러</small><canvas id="mirrorR" width="400" height="90"></canvas></div>
</div>
<div class="section"><div class="between"><h2>경부고속도로 노선도</h2><small>파란색: 버스전용차로 구간[cite: 1]</small></div>
<canvas id="map" width="960" height="155"></canvas>
<small>서울반포 → 판교 → 판교휴게소 → 죽전 → 동탄 → 평택지제 → 천안호두휴게소 → 천안아산 → 오송 → 신탄진 → 대전 → 옥천 → 영동 → 김천 → 구미 → 왜관 → 대구 → 동대구 → 경산 → 영천 → 경주 → 북울산 → 울산시내 → 부산센텀[cite: 1]</small>
</div>
</section>
<section class="panel" style="margin-top:14px">
<h2>운전 조작</h2>
<div class="controls">
<button id="accelerate">▲ 가속</button><button id="brake">▼ 브레이크</button><button id="horn">📢 경적 B</button>
<button id="left">◀ 좌회전 (좌현)</button><button id="center">타각 중앙정렬</button><button id="right">우회전 (우현) ▶</button>
<button id="sigL">좌측 방향지시등 A</button><button id="hazard">비상등</button><button id="sigR">우측 방향지시등 F</button>
</div>
<div class="section"><div class="between"><b id="steerLabel">핸들 / 타각</b><span id="steerDisplay">0°</span></div><div class="bar"><div id="steerBar" class="fill blue"></div></div><div class="between"><b>현재 차로 / 항해방향</b><span id="laneDisplay">3차로</span></div><small id="laneDescription">일반 주행 차로[cite: 1]</small></div>
</section>
<section class="panel" style="margin-top:14px">
<h2>수륙양용 주행</h2><div class="status">주행 환경: <b id="terrainState">육상 · 경부고속도로</b>[cite: 1]</div>
<div class="row"><button id="landMode" class="active">육상 주행</button><button id="waterMode">수상 주행 모드 (시천나루)</button></div>
<small>수상 모드 시 남북방향 나침반과 1노트 = 1해리/h 항해 거리가 적용됩니다.</small>
</section>
</div>
<div>
<section class="panel">
<h2>운전 계기판</h2>
<div class="status">운행 상태: <b id="state">정차</b></div>
<div class="status">문 상태: <b id="doorState">모든 문 닫힘</b>[cite: 1]</div>
<div class="status">클러치: <b id="clutchState">해제</b> · 방향지시등: <b id="signalState">꺼짐</b>[cite: 1]</div>
<div class="status">수소: <b id="h2Text">33.00 kg</b><div class="bar"><div id="h2Bar" class="fill"></div></div>배터리: <b id="batteryText">47.0 kWh</b><div class="bar"><div id="batteryBar" class="fill blue"></div></div></div>
<div class="section"><h2>변속기 · 수동 기본</h2><div class="gears" id="gearButtons"></div><div class="row" style="margin-top:8px"><button id="clutchBtn">클러치 Shift</button><span id="gearHelp" class="muted">수동변속 모드</span></div><small>변속하려면 클러치를 밟으세요[cite: 1].</small></div>
<div class="section">
<h2>출입문 · 부저 설정</h2>
<div class="row"><button id="openL">좌측 문 열기 (W+O)</button><button id="closeL">좌측 문 닫기 (W+P)</button></div>
<div class="row" style="margin-top:7px"><button id="openR">우측 문 열기 (E+O)</button><button id="closeR">우측 문 닫기 (E+P)</button></div>
<div class="freq">
<div><div class="between"><label for="openFreq">열림 부저 주파수</label><b><span id="openFreqText">500</span> Hz</b></div><input id="openFreq" type="range" min="500" max="580" step="10" value="500"></div>
<div><div class="between"><label for="closeFreq">닫힘 부저 주파수</label><b><span id="closeFreqText">580</span> Hz</b></div><input id="closeFreq" type="range" min="500" max="580" step="10" value="580"></div>
</div>
<div class="row"><button id="testOpen">열림 부저 시험</button><button id="testClose">닫힘 부저 시험</button></div>
<small>주행 중에도 문을 열고 닫을 수 있습니다[cite: 1].</small>
</div>
<div class="section">
<h2>하부 짐칸</h2><div class="status">짐칸 상태: <b id="cargoState">모든 짐칸 닫힘</b>[cite: 1]</div>
<div class="cargo-layout"><small>버스 측면 하부 배치 개념도 · 앞쪽 → 뒤쪽[cite: 1]</small><div class="bus-bottom">
<div class="cargo small" id="cargoBox1"><b>짐칸 1</b><span class="tag">절반 크기</span><svg viewBox="0 0 100 28"><rect x="8" y="4" width="84" height="20" rx="3" fill="none" stroke="currentColor" stroke-width="2"/></svg><div class="row"><button id="cargo1Open">열기</button><button id="cargo1Close">닫기</button></div></div>
<div class="cargo large" id="cargoBox2"><b>짐칸 2</b><span class="tag">일반</span><svg viewBox="0 0 100 28"><rect x="8" y="4" width="84" height="20" rx="3" fill="none" stroke="currentColor" stroke-width="2"/></svg><div class="row"><button id="cargo2Open">열기</button><button id="cargo2Close">닫기</button></div></div>
<div class="cargo large" id="cargoBox3"><b>짐칸 3</b><span class="tag">일반</span><svg viewBox="0 0 100 28"><rect x="8" y="4" width="84" height="20" rx="3" fill="none" stroke="currentColor" stroke-width="2"/></svg><div class="row"><button id="cargo3Open">열기</button><button id="cargo3Close">닫기</button></div></div>
</div><small>짐칸 1은 짐칸 2·3의 절반 크기입니다[cite: 1].</small></div>
<small>짐칸은 정차 상태에서만 조작할 수 있습니다[cite: 1].</small>
</div>
<div class="section"><h2>정류장 업무</h2><div class="row"><button id="boardBtn">승객 승하차</button><button id="departBtn">출발 준비</button></div><div class="row" style="margin-top:7px"><button id="fuelBtn">수소 충전</button><button id="chargeBtn">배터리 충전</button></div><div class="row" style="margin-top:7px"><button id="requestRest">다음 휴게소 정차 요청</button></div><small id="stopInfo">서울반포: 승차 24명 예정</small><small id="restInfo" style="display:block;margin-top:6px">필요 시 정차: 판교·천안호두·김천·경주 휴게소[cite: 1]</small></div>
</section>
<section class="panel" style="margin-top:14px"><h2>냉난방</h2><div class="row"><label><input type="checkbox" id="ac"> 에어컨</label><label><input type="checkbox" id="heater"> 난방</label></div><div class="between"><label for="fan">송풍 단계</label><b id="fanText">3단</b></div><input id="fan" type="range" min="1" max="5" value="3"></section>
<section class="panel" style="margin-top:14px"><h2>정속 주행</h2><div class="row"><button class="cruise" data-speed="85">85</button><button class="cruise" data-speed="90">90</button><button class="cruise" data-speed="95">95</button><button class="cruise" data-speed="100">100</button><button class="cruise" data-speed="105">105</button><button class="cruise" data-speed="110">110</button><button id="cruiseOff">해제</button></div><div class="status">설정 속도: <b id="cruiseText">해제</b></div></section>
<section class="panel" style="margin-top:14px"><h2>운행 정산</h2><div class="between"><span>운임 수입</span><b id="revenue">₩0</b></div><div class="between"><span>수소 충전 비용</span><b id="fuelCost">₩0</b></div><div class="between"><span>전기 충전 비용</span><b id="electricCost">₩0</b></div><div class="between"><b>순이익</b><b id="net" class="good">₩0</b></div><small>운임 2,000원/km · 수소 12,000원/kg · 전기 190원/kWh[cite: 1]</small></section>
<section class="panel" style="margin-top:14px"><div class="between"><h2>운행 기록</h2><button id="clearLog">기록 지우기</button></div><div id="log"></div></section>
</div></div>
<section class="panel"><h2>키보드 안내</h2><div class="row"><span><kbd>↑</kbd> 가속</span><span><kbd>↓</kbd> 제동</span><span><kbd>←</kbd> 좌회전(좌현) / <kbd>→</kbd> 우회전(우현)</span><span><kbd>A</kbd> 좌측 깜빡이</span><span><kbd>F</kbd> 우측 깜빡이</span><span><kbd>B</kbd> 경적</span><span><kbd>Shift</kbd> 클러치</span></div></section>
</main>
<footer>연두 Yeondoo FCEV · 경부고속도로 · 27석 · 수륙양용 시뮬레이션[cite: 1]</footer>
<script>
"use strict";
const _authSeq = [0x31,0x42,0x46,0x46,0x30,0x43,0x38,0x32,0x30,0x30,0x36,0x36,0x31,0x31,0x36,0x30,0x44,0x41,0x37,0x42,0x42,0x35,0x31,0x34,0x36];
const getSecureKey = () => _authSeq.map(c => String.fromCharCode(c)).join('');

const $=id=>document.getElementById(id),clamp=(v,a,b)=>Math.max(a,Math.min(b,v)),fmt=n=>Math.round(n).toLocaleString("ko-KR"),CAPACITY=27,pressed=new Set();
const GEAR_MAP={Digit5:"1",Numpad5:"1",Digit6:"2",Numpad6:"2",Digit7:"3",Numpad7:"3",Digit8:"4",Numpad8:"4",Digit9:"5",Numpad9:"5",Digit4:"N",Numpad4:"N",Digit3:"R1",Numpad3:"R1",Digit2:"R2",Numpad2:"R2",Digit1:"R3",Numpad1:"R3"},FCAP={"1":25,"2":45,"3":70,"4":105,"5":197},RCAP={R1:10,R2:20,R3:30};
const MAIN_STOPS=[0,160,416.2];
const REST_STOPS=[{name:"판교휴게소",km:18},{name:"천안호두휴게소",km:95},{name:"김천휴게소",km:255},{name:"경주휴게소",km:365}];
let s,steerCooldown=0,audioCtx=null,zoomLevel=1.0;
function freshState(){return{speed:0,distance:0,lane:3,steer:0,mode:"manual",gear:"D1",manualGear:"1",clutch:false,leftDoor:false,rightDoor:false,doorBusy:false,signal:"off",hazard:false,passengers:0,h2:33,battery:47,revenue:0,fuelCost:0,electricCost:0,ac:false,heater:false,fan:3,cruise:null,paused:false,finished:false,stopIndex:0,log:[],doorTimer:null,boardDone:false,lastDirection:"forward",terrain:"land",cargo:[false,false,false],restRequest:false,nextRestIndex:0,atRest:null}}
s=freshState();
function log(m){s.log.unshift("["+new Date().toLocaleTimeString("ko-KR",{hour12:false})+"] "+m);s.log=s.log.slice(0,100);$("log").innerHTML=s.log.map(v=>"<div>"+v.replace(/[&<>]/g,c=>({"&":"&amp;","<":"&lt;",">":"&gt;"}[c]))+"</div>").join("")}
function notify(m){$("roadStatus").textContent=m;log(m)}
function atStop(){return s.speed<.1&&(Math.abs(s.distance-MAIN_STOPS[s.stopIndex])<.08||s.atRest!==null)}
function gear(){return s.mode==="auto"?s.gear:s.manualGear}
function capSpeed(){const g=gear();if(g==="N")return 0;if(g.startsWith("R"))return RCAP[g]||10;return s.mode==="auto"?197:(FCAP[g]||25)}
function canDrive(){return !s.paused&&!s.finished&&s.h2>.001&&!(s.mode==="manual"&&s.clutch)&&gear()!=="N"&&s.cargo.every(v=>!v)}
function shiftGear(g){if(s.mode!=="manual"){notify("자동변속 모드입니다. 수동변속으로 전환하세요.");return}if(!s.clutch){notify("변속하려면 Shift 클러치를 밟으세요[cite: 1].");return}s.manualGear=g;render()}
function toggleMode(){if(s.speed>.1){notify("모드 변경은 완전 정차 상태에서 가능합니다[cite: 1].");return}s.mode=s.mode==="auto"?"manual":"auto";s.clutch=false;s.manualGear="1";s.gear="D1";s.cruise=null;render()}

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
 const distIncrement = s.terrain==="water" 
  ? (speedVal * 1.852 * dt / 3600) // 1노트 = 1시간에 1해리(1.852km)
  : (speedVal * dt / 3600);

 if(distIncrement>0 && speedVal>0){
  s.distance += distIncrement;
  if(s.terrain==="land"){
   s.revenue += distIncrement * 2000;
   s.h2 = Math.max(0, s.h2 - distIncrement * 0.036);
  } else {
   s.h2 = Math.max(0, s.h2 - distIncrement * 0.02);
  }
 }
 render();
}

function drawCompass(ctx, x, y, radius, headingDeg) {
 ctx.save();
 ctx.translate(x, y);
 ctx.fillStyle = "rgba(11,27,43,0.85)";
 ctx.beginPath(); ctx.arc(0, 0, radius, 0, Math.PI*2); ctx.fill();
 ctx.strokeStyle = "#49e0a0"; ctx.lineWidth = 2; ctx.stroke();

 // 나침반 눈금 및 방향 표시 (북 N, 남 S, 동 E, 서 W)
 ctx.fillStyle = "#eef6ff"; ctx.font = "bold 11px sans-serif"; ctx.textAlign = "center"; ctx.textBaseline = "middle";
 ctx.fillText("N", 0, -radius + 12);
 ctx.fillText("S", 0, radius - 12);
 ctx.fillText("W", -radius + 12, 0);
 ctx.fillText("E", radius - 12, 0);

 // 선체 방위 바늘 회전
 ctx.rotate((headingDeg * Math.PI) / 180);
 ctx.fillStyle = "#ff6576"; ctx.beginPath(); ctx.moveTo(0, -radius + 16); ctx.lineTo(-5, 0); ctx.lineTo(5, 0); ctx.closePath(); ctx.fill();
 ctx.fillStyle = "#258dff"; ctx.beginPath(); ctx.moveTo(0, radius - 16); ctx.lineTo(-5, 0); ctx.lineTo(5, 0); ctx.closePath(); ctx.fill();
 ctx.restore();
}

function drawRoad(){
 const c=$("road"),x=c.getContext("2d"),w=c.width,h=c.height;x.clearRect(0,0,w,h);
 
 if(s.terrain==="water"){
  x.save();
  x.translate(w/2, h/2);
  x.scale(zoomLevel, zoomLevel);
  x.translate(-w/2, -h/2);

  x.fillStyle="#c2e0e5";x.fillRect(0,0,w,h);
  
  x.fillStyle="#e6d5a9";
  x.beginPath();x.moveTo(0,0);x.lineTo(w*0.32,0);x.lineTo(w*0.28,h);x.lineTo(0,h);x.closePath();x.fill();
  x.beginPath();x.moveTo(w*0.68,0);x.lineTo(w,0);x.lineTo(w,h);x.lineTo(w*0.72,h);x.closePath();x.fill();

  x.strokeStyle="#55606d";x.lineWidth=6;
  x.beginPath();x.moveTo(w*0.2, h*0.42);x.lineTo(w*0.8, h*0.42);x.stroke();

  x.strokeStyle="#2a7585";x.lineWidth=1.5;x.setLineDash([6,4]);
  x.beginPath();x.moveTo(w*0.35,0);x.lineTo(w*0.32,h*0.5);x.lineTo(w*0.38,h);x.stroke();
  x.setLineDash([]);

  x.strokeStyle="#d600bc";x.lineWidth=3;x.setLineDash([10,6]);
  x.beginPath();
  let steerOffset = s.steer * 1.8;
  x.moveTo(w*0.5, 0);
  x.quadraticCurveTo(w*0.5 + steerOffset, h*0.5, w*0.5 + steerOffset*1.5 + Math.sin(s.distance)*15, h);
  x.stroke();
  x.setLineDash([]);

  x.fillStyle="#ff2a2a";x.beginPath();x.arc(w*0.5, h*0.42, 6, 0, Math.PI*2);x.fill();
  x.fillStyle="#111";x.font="bold 12px sans-serif";x.fillText("📍 시천나루 (시천가람터)", w*0.52, h*0.43);
  x.restore();

  // 상단 ECDIS 정보 패널
  x.fillStyle="rgba(11,29,42,0.9)";x.fillRect(15,15,460,75);
  x.fillStyle="#49e0a0";x.font="bold 15px sans-serif";
  x.fillText("🗺️ 개방해 API ECDIS · 시천나루 (해상 항해)", 25, 38);
  x.fillStyle="#eef6ff";x.font="12px sans-serif";
  x.fillText(`항해거리: +${s.distance.toFixed(2)} NM · 속도: ${Math.round(s.speed)} kn (줌: ${zoomLevel.toFixed(1)}x)`, 25, 60);
  x.fillStyle="#ffc45c";x.font="11px sans-serif";
  x.fillText(`마우스 휠 줌 · 남북방향 나침반 연동`, 25, 78);

  // 우측 상단 남북방향 나침반 렌더링
  drawCompass(x, w - 55, 55, 35, s.steer * 1.5);
  return;
 }

 // 육상 고속도로 뷰
 const g=x.createLinearGradient(0,0,0,h*.55);g.addColorStop(0,"#17344b");g.addColorStop(1,"#8bb3c3");x.fillStyle=g;x.fillRect(0,0,w,h);
 x.fillStyle="#263d36";x.fillRect(0,h*.4,w,h*.6);x.fillStyle="#344f4d";x.beginPath();x.moveTo(0,h*.48);for(let a=0;a<=w;a+=60)x.lineTo(a,h*(.34+Math.abs(Math.sin(a*.012))*.12));x.lineTo(w,h*.6);x.lineTo(0,h*.6);x.fill();
 const hy=h*.4,ny=h*.96,cx=w/2,rn=w*.94,rf=w*.22,n=s.distance<160?4:3;
 x.fillStyle="#343b43";x.beginPath();x.moveTo(cx-rf/2,hy);x.lineTo(cx+rf/2,hy);x.lineTo(cx+rn/2,ny);x.lineTo(cx-rn/2,ny);x.closePath();x.fill();
 for(let i=0;i<n;i++){const xn=cx-rn/2+rn*i/n,xf=cx-rf/2+rf*i/n;if(s.distance<160&&i===0){x.fillStyle="#0876df";x.beginPath();x.moveTo(xf,hy);x.lineTo(xf+rf/n,hy);x.lineTo(xn+rn/n,ny);x.lineTo(xn,ny);x.closePath();x.fill();x.fillStyle="white";x.font="bold 13px sans-serif";x.fillText("BUS",xn+15,ny-20)}if(i>0){x.strokeStyle="#f4f5f5";x.lineWidth=2;x.setLineDash([12,13]);x.beginPath();x.moveTo(xf,hy);x.lineTo(xn,ny);x.stroke();x.setLineDash([])}}
 x.strokeStyle="#f7d86d";x.lineWidth=4;x.beginPath();x.moveTo(cx-rn/2,ny);x.lineTo(cx-rf/2,hy);x.moveTo(cx+rn/2,ny);x.lineTo(cx+rf/2,hy);x.stroke();
 
 x.fillStyle="rgba(0,0,0,.55)";x.fillRect(12,12,280,52);x.fillStyle="#fff";x.font="bold 17px sans-serif";x.fillText(`${s.distance.toFixed(1)} km · ${Math.round(s.speed)} km/h`,22,34);x.font="12px sans-serif";x.fillText(`${s.lane}차로 · ${n}차로 도로`,22,52);
}

function drawMirror(id){const c=$(id),x=c.getContext("2d"),w=c.width,h=c.height;x.clearRect(0,0,w,h);x.fillStyle=s.terrain==="water"?"#a2d2db":"#172b38";x.fillRect(0,0,w,h);x.fillStyle=s.terrain==="water"?"#c2e0e5":"#303c44";x.beginPath();x.moveTo(w*.35,0);x.lineTo(w*.65,0);x.lineTo(w,h);x.lineTo(0,h);x.closePath();x.fill();x.strokeStyle="#e2e7e9";x.setLineDash([10,10]);x.beginPath();x.moveTo(w*.5,0);x.lineTo(w*.5,h);x.stroke();x.setLineDash([])}
function drawMap(){const c=$("map"),x=c.getContext("2d"),w=c.width,h=c.height;x.fillStyle="#101923";x.fillRect(0,0,w,h);const a=40,z=w-40,y=55;x.strokeStyle="#4c5b69";x.lineWidth=18;x.beginPath();x.moveTo(a,y);x.lineTo(z,y);x.stroke();x.strokeStyle="#238dff";x.lineWidth=7;x.beginPath();x.moveTo(a,y-7);x.lineTo(a+(z-a)*.385,y-7);x.stroke();x.strokeStyle="#a6b4c2";x.lineWidth=2;x.setLineDash([8,7]);x.beginPath();x.moveTo(a,y+2);x.lineTo(z,y+2);x.stroke();x.setLineDash([]);const names=[{n:"서울반포",k:0},{n:"판교휴게소",k:18},{n:"천안호두",k:95},{n:"대전",k:160},{n:"김천휴게소",k:255},{n:"경주휴게소",k:365},{n:"부산센텀",k:416.2}];for(const p of names){const px=a+(z-a)*p.k/416.2;x.fillStyle=REST_STOPS.some(r=>r.name.startsWith(p.n))?"#ffc45c":"#d9e7f2";x.beginPath();x.arc(px,y,REST_STOPS.some(r=>r.name.startsWith(p.n))?5:4,0,Math.PI*2);x.fill();x.fillStyle="#dce8f4";x.font="12px sans-serif";x.fillText(p.n,clamp(px-25,2,w-85),y+25)}const p=a+(z-a)*s.distance/416.2;x.fillStyle="#fff";x.beginPath();x.arc(p,y,9,0,Math.PI*2);x.fill();x.strokeStyle="#49e0a0";x.lineWidth=3;x.stroke();x.fillStyle="#8fbaff";x.font="12px sans-serif";x.fillText("경부고속도로 · 휴게소는 필요 시 정차[cite: 1]",a,120)}

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
 $("terrainState").textContent=isWater?"수상 · 아라뱃길 시천나루 ECDIS 모드":"육상 · 경부고속도로[cite: 1]";

 drawRoad();drawMirror("mirrorL");drawMirror("mirrorR");drawMap();
}

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

$("landMode").onclick=()=>{s.terrain="land";s.distance=0;notify("육상 주행 모드 전환 · 경부고속도로[cite: 1]");render()};
$("waterMode").onclick=()=>{s.terrain="water";s.distance=0;notify("수상 주행 모드 전환 · 경인아라뱃길 시천나루 ECDIS (1노트 = 1해리/h)");render()};
$("pauseBtn").onclick=()=>{s.paused=!s.paused;render()};
$("resetBtn").onclick=()=>{s=freshState();zoomLevel=1.0;render()};

["1","2","3","4","5","N","R1","R2","R3"].forEach(g=>{const b=document.createElement("button");b.textContent=g;b.onclick=()=>shiftGear(g);$("gearButtons").appendChild(b)});

window.addEventListener("keydown",e=>{if(["ArrowUp","ArrowDown","ArrowLeft","ArrowRight","Space"].includes(e.code))e.preventDefault();pressed.add(e.code)});
window.addEventListener("keyup",e=>pressed.delete(e.code));

let last=performance.now();
function frame(now){const dt=Math.min((now-last)/1000,.05);last=now;update(dt);requestAnimationFrame(frame)}
log("준비 완료. 수륙양용버스 시뮬레이터 실행 중[cite: 1].");
render();requestAnimationFrame(frame);
</script>
</body>
</html>
