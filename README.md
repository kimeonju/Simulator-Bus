<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>국립해양조사원 개방해(海) · 수륙양용 ECDIS</title>
<style>
:root{color-scheme:dark;--bg:#090e14;--panel:#111923;--line:#223244;--text:#eef6ff;--muted:#8fa4bc;--green:#22c55e;--blue:#0284c7;--red:#ef4444;--amber:#f59e0b;--khoa:#0f2c42}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--text);font:13px/1.4 system-ui,sans-serif}
header{padding:12px 18px;background:#0d1520;border-bottom:1px solid var(--line);display:flex;justify-content:space-between;align-items:center;gap:12px;flex-wrap:wrap}
h1{font-size:18px;margin:0;display:flex;align-items:center;gap:8px}h2{font-size:14px;margin:0 0 10px}
small,.muted{color:var(--muted)}
button,input,select{font:inherit}
button{background:#182636;color:var(--text);border:1px solid #2e435c;border-radius:6px;padding:7px 11px;cursor:pointer}
button:hover{border-color:var(--green)}button.active{background:#0e4a30;border-color:var(--green)}
main{max-width:1480px;margin:auto;padding:12px;display:grid;gap:12px}
.grid{display:grid;grid-template-columns:minmax(0,1.8fr) minmax(320px,1fr);gap:12px}
.panel{background:var(--panel);border:1px solid var(--line);border-radius:10px;padding:12px;min-width:0}
.hud{display:grid;grid-template-columns:repeat(4,minmax(0,1fr));gap:8px;margin-bottom:10px}
.metric{background:#0b131c;border:1px solid #1e2e3e;border-radius:8px;padding:9px}
.metric .value{font-size:21px;font-weight:750}.metric small{display:block}
canvas{display:block;width:100%;border-radius:6px;background:#c2e0e5}
#road{aspect-ratio:16/7.5;border:2px solid #2e435c}#map{height:140px}
.mirrors{display:grid;grid-template-columns:1fr 1fr;gap:8px;margin-top:8px}.mirrors canvas{height:72px}
.section{margin-top:12px;padding-top:11px;border-top:1px solid var(--line)}
.row{display:flex;flex-wrap:wrap;gap:6px;align-items:center}
.between{display:flex;justify-content:space-between;align-items:center;gap:8px}
.status{padding:6px 9px;border-radius:6px;background:#0b131c;margin:6px 0}
.good{color:var(--green)}.bad{color:var(--red)}
.bar{height:8px;background:#1b2836;border-radius:10px;overflow:hidden;margin:6px 0 10px}
.fill{height:100%;width:100%;background:var(--green)}.fill.blue{background:var(--blue)}
.gears{display:grid;grid-template-columns:repeat(5,1fr);gap:4px}.gears button{padding:7px 2px}
.controls{display:grid;grid-template-columns:repeat(3,minmax(0,1fr));gap:6px}.controls button{min-height:40px}
#log{height:140px;overflow:auto;font-size:11px;color:#a8bed6}
#log div{padding:3px 0;border-bottom:1px solid #1a293a}
kbd{border:1px solid #435870;border-radius:3px;padding:1px 4px;background:#0d1520;font-size:11px}
input[type=range]{width:100%;accent-color:var(--green)}
input[type=checkbox]{accent-color:var(--green)}
.freq{display:grid;grid-template-columns:1fr 1fr;gap:10px}
.cargo-layout{background:#0b131c;border:1px solid #1e2e3e;border-radius:8px;padding:10px;margin-top:6px}
.bus-bottom{display:flex;gap:6px;align-items:stretch;max-width:580px;margin:8px auto}
.cargo{background:#182636;border:2px solid #384f69;border-radius:6px;min-height:75px;display:flex;flex-direction:column;align-items:center;justify-content:center;gap:4px;padding:5px}
.cargo.small{flex:1}.cargo.large{flex:2}.cargo.open{background:#0e4a30;border-color:var(--green)}
.cargo svg{width:100%;height:24px}
.tag{display:inline-block;border:1px solid #2e435c;border-radius:15px;padding:1px 6px;color:var(--muted);font-size:11px}
/* 개방해 웹사이트 스타일 툴바 */
.khoa-toolbar{background:#091420;border:1px solid var(--line);border-radius:6px;padding:6px 10px;display:flex;gap:10px;align-items:center;margin-bottom:8px;font-size:12px}
.khoa-toolbar span{color:var(--amber);font-weight:bold}
@media(max-width:850px){.grid{grid-template-columns:1fr}.hud{grid-template-columns:repeat(2,minmax(0,1fr))}.freq{grid-template-columns:1fr}}
</style>
</head>
<body>
<header>
<div><h1>🌊 국립해양조사원 개방해(海) GIS</h1><small>수륙양용 버스 항해 시스템 · 경인아라뱃길 시천나루</small></div>
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
<div class="khoa-toolbar">
<span>KHOA 개방해 웹맵</span>
<button id="layerToggle" class="active">전자해도(ECDIS)</button>
<button id="depthToggle">수심격자</button>
<button id="atonToggle">항로표지(부표)</button>
<span style="margin-left:auto;color:var(--muted)">EPSG:5186 (TM)</span>
</div>
<div class="between"><h2>개방해 전자해도 뷰 (시천나루 수운수로)</h2><span id="roadStatus" class="good">시천나루 대기 중</span></div>
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
<div class="section"><div class="between"><h2>경부고속도로 노선도 (육상 모드용)</h2><small>파란색: 버스전용차로</small></div>
<canvas id="map" width="960" height="140"></canvas>
<small>서울반포 → 판교 → 천안호두휴게소 → 대전 → 김천휴게소 → 경주휴게소 → 부산센텀</small>
</div>
</section>
<section class="panel" style="margin-top:12px">
<h2>운전 조작</h2>
<div class="controls">
<button id="accelerate">▲ 가속</button><button id="brake">▼ 브레이크</button><button id="horn">📢 경적 B</button>
<button id="left">◀ 좌회전 (좌현)</button><button id="center">타각 중앙정렬</button><button id="right">우회전 (우현) ▶</button>
<button id="sigL">좌측 깜빡이 A</button><button id="hazard">비상등</button><button id="sigR">우측 깜빡이 F</button>
</div>
<div class="section"><div class="between"><b id="steerLabel">핸들 / 타각</b><span id="steerDisplay">0°</span></div><div class="bar"><div id="steerBar" class="fill blue"></div></div><div class="between"><b>현재 차로 / 항해방향</b><span id="laneDisplay">3차로</span></div><small id="laneDescription">일반 주행 차로</small></div>
</section>
<section class="panel" style="margin-top:12px">
<h2>수륙양용 주행</h2><div class="status">주행 환경: <b id="terrainState">육상 · 경부고속도로</b></div>
<div class="row"><button id="landMode" class="active">육상 주행</button><button id="waterMode">수상 주행 모드 (시천나루 출항)</button></div>
<small>수상 모드 시 국립해양조사원 개방해(海) GIS 웹맵 시스템 화면으로 자동 전환됩니다.</small>
</section>
</div>
<div>
<section class="panel">
<h2>운전 계기판</h2>
<div class="status">운행 상태: <b id="state">정차</b></div>
<div class="status">문 상태: <b id="doorState">모든 문 닫힘</b></div>
<div class="status">클러치: <b id="clutchState">해제</b> · 방향지시등: <b id="signalState">꺼짐</b></div>
<div class="status">수소: <b id="h2Text">33.00 kg</b><div class="bar"><div id="h2Bar" class="fill"></div></div>배터리: <b id="batteryText">47.0 kWh</b><div class="bar"><div id="batteryBar" class="fill blue"></div></div></div>
<div class="section"><h2>변속기 · 수동 기본</h2><div class="gears" id="gearButtons"></div><div class="row" style="margin-top:6px"><button id="clutchBtn">클러치 Shift</button><span id="gearHelp" class="muted">수동변속 모드</span></div><small>변속하려면 클러치를 밟으세요.</small></div>
<div class="section">
<h2>출입문 · 부저 설정</h2>
<div class="row"><button id="openL">좌측 문 열기 (W+O)</button><button id="closeL">좌측 문 닫기 (W+P)</button></div>
<div class="row" style="margin-top:6px"><button id="openR">우측 문 열기 (E+O)</button><button id="closeR">우측 문 닫기 (E+P)</button></div>
<div class="freq" style="margin-top:8px">
<div><div class="between"><label for="openFreq">열림 부저</label><b><span id="openFreqText">500</span>Hz</b></div><input id="openFreq" type="range" min="500" max="580" step="10" value="500"></div>
<div><div class="between"><label for="closeFreq">닫힘 부저</label><b><span id="closeFreqText">580</span>Hz</b></div><input id="closeFreq" type="range" min="500" max="580" step="10" value="580"></div>
</div>
<div class="row" style="margin-top:6px"><button id="testOpen">열림 시험</button><button id="testClose">닫힘 시험</button></div>
</div>
<div class="section">
<h2>하부 짐칸</h2><div class="status">짐칸 상태: <b id="cargoState">모든 짐칸 닫힘</b></div>
<div class="cargo-layout"><small>버스 측면 하부 배치 · 앞 → 뒤</small><div class="bus-bottom">
<div class="cargo small" id="cargoBox1"><b>짐칸 1</b><span class="tag">절반</span><svg viewBox="0 0 100 24"><rect x="6" y="3" width="88" height="18" rx="2" fill="none" stroke="currentColor" stroke-width="2"/></svg><div class="row"><button id="cargo1Open">열기</button><button id="cargo1Close">닫기</button></div></div>
<div class="cargo large" id="cargoBox2"><b>짐칸 2</b><span class="tag">일반</span><svg viewBox="0 0 100 24"><rect x="6" y="3" width="88" height="18" rx="2" fill="none" stroke="currentColor" stroke-width="2"/></svg><div class="row"><button id="cargo2Open">열기</button><button id="cargo2Close">닫기</button></div></div>
<div class="cargo large" id="cargoBox3"><b>짐칸 3</b><span class="tag">일반</span><svg viewBox="0 0 100 24"><rect x="6" y="3" width="88" height="18" rx="2" fill="none" stroke="currentColor" stroke-width="2"/></svg><div class="row"><button id="cargo3Open">열기</button><button id="cargo3Close">닫기</button></div></div>
</div></div>
</div>
<div class="section"><h2>정류장 업무</h2><div class="row"><button id="boardBtn">승객 승하차</button><button id="departBtn">출발 준비</button></div><div class="row" style="margin-top:6px"><button id="fuelBtn">수소 충전</button><button id="chargeBtn">배터리 충전</button></div><div class="row" style="margin-top:6px"><button id="requestRest">휴게소 정차 요청</button></div><small id="stopInfo">서울반포: 승차 24명 예정</small></div>
</section>
<section class="panel" style="margin-top:12px"><h2>냉난방</h2><div class="row"><label><input type="checkbox" id="ac"> 에어컨</label><label><input type="checkbox" id="heater"> 난방</label></div><div class="between"><label for="fan">송풍 단계</label><b id="fanText">3단</b></div><input id="fan" type="range" min="1" max="5" value="3"></section>
<section class="panel" style="margin-top:12px"><h2>정속 주행</h2><div class="row"><button class="cruise" data-speed="85">85</button><button class="cruise" data-speed="90">90</button><button class="cruise" data-speed="95">95</button><button class="cruise" data-speed="100">100</button><button class="cruise" data-speed="105">105</button><button class="cruise" data-speed="110">110</button><button id="cruiseOff">해제</button></div><div class="status">설정 속도: <b id="cruiseText">해제</b></div></section>
<section class="panel" style="margin-top:12px"><h2>운행 정산</h2><div class="between"><span>운임 수입</span><b id="revenue">₩0</b></div><div class="between"><span>수소 충전 비용</span><b id="fuelCost">₩0</b></div><div class="between"><span>전기 충전 비용</span><b id="electricCost">₩0</b></div><div class="between"><b>순이익</b><b id="net" class="good">₩0</b></div><small>운임 2,000원/km · 수소 12,000원/kg · 전기 190원/kWh</small></section>
<section class="panel" style="margin-top:12px"><div class="between"><h2>운행 기록</h2><button id="clearLog">기록 지우기</button></div><div id="log"></div></section>
</div></div>
<section class="panel"><h2>키보드 안내</h2><div class="row"><span><kbd>↑</kbd> 가속</span><span><kbd>↓</kbd> 제동</span><span><kbd>←</kbd> 좌회전(좌현) / <kbd>→</kbd> 우회전(우현)</span><span><kbd>A</kbd> 좌측 깜빡이</span><span><kbd>F</kbd> 우측 깜빡이</span><span><kbd>B</kbd> 경적</span><span><kbd>Shift</kbd> 클러치</span></div></section>
</main>
<footer>국립해양조사원 개방해(海) GIS · 수륙양용 시뮬레이터</footer>
<script>
"use strict";
const $=id=>document.getElementById(id),clamp=(v,a,b)=>Math.max(a,Math.min(b,v)),fmt=n=>Math.round(n).toLocaleString("ko-KR"),CAPACITY=27,pressed=new Set();
const GEAR_MAP={Digit5:"1",Numpad5:"1",Digit6:"2",Numpad6:"2",Digit7:"3",Numpad7:"3",Digit8:"4",Numpad8:"4",Digit9:"5",Numpad9:"5",Digit4:"N",Numpad4:"N",Digit3:"R1",Numpad3:"R1",Digit2:"R2",Numpad2:"R2",Digit1:"R3",Numpad1:"R3"},FCAP={"1":25,"2":45,"3":70,"4":105,"5":197},RCAP={R1:10,R2:20,R3:30};
const MAIN_STOPS=[0,160,416.2];
const REST_STOPS=[{name:"판교휴게소",km:18},{name:"천안호두휴게소",km:95},{name:"김천휴게소",km:255},{name:"경주휴게소",km:365}];
let s,steerCooldown=0,audioCtx=null,zoomLevel=1.0,showDepth=true,showAton=true;
function freshState(){return{speed:0,distance:0,lane:3,steer:0,mode:"manual",gear:"D1",manualGear:"1",clutch:false,leftDoor:false,rightDoor:false,doorBusy:false,signal:"off",hazard:false,passengers:0,h2:33,battery:47,revenue:0,fuelCost:0,electricCost:0,ac:false,heater:false,fan:3,cruise:null,paused:false,finished:false,stopIndex:0,log:[],doorTimer:null,boardDone:false,lastDirection:"forward",terrain:"land",cargo:[false,false,false],restRequest:false,nextRestIndex:0,atRest:null}}
s=freshState();
function log(m){s.log.unshift("["+new Date().toLocaleTimeString("ko-KR",{hour12:false})+"] "+m);s.log=s.log.slice(0,100);$("log").innerHTML=s.log.map(v=>"<div>"+v.replace(/[&<>]/g,c=>({"&":"&amp;","<":"&lt;",">":"&gt;"}[c]))+"</div>").join("")}
function notify(m){$("roadStatus").textContent=m;log(m)}
function atStop(){return s.speed<.1&&(Math.abs(s.distance-MAIN_STOPS[s.stopIndex])<.08||s.atRest!==null)}
function gear(){return s.mode==="auto"?s.gear:s.manualGear}
function capSpeed(){const g=gear();if(g==="N")return 0;if(g.startsWith("R"))return RCAP[g]||10;return s.mode==="auto"?197:(FCAP[g]||25)}
function canDrive(){return !s.paused&&!s.finished&&s.h2>.001&&!(s.mode==="manual"&&s.clutch)&&gear()!=="N"&&s.cargo.every(v=>!v)}
function shiftGear(g){if(s.mode!=="manual"){notify("자동변속 모드입니다.");return}if(!s.clutch){notify("클러치를 밟으세요.");return}s.manualGear=g;render()}
function toggleMode(){if(s.speed>.1){notify("정차 후 모드 변경이 가능합니다.");return}s.mode=s.mode==="auto"?"manual":"auto";s.clutch=false;s.manualGear="1";s.gear="D1";s.cruise=null;render()}

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
 ctx.fillStyle = "rgba(15,44,66,0.9)";
 ctx.beginPath(); ctx.arc(0, 0, radius, 0, Math.PI*2); ctx.fill();
 ctx.strokeStyle = "#38bdf8"; ctx.lineWidth = 2; ctx.stroke();
 ctx.fillStyle = "#eef6ff"; ctx.font = "bold 11px sans-serif"; ctx.textAlign = "center"; ctx.textBaseline = "middle";
 ctx.fillText("N", 0, -radius + 11);
 ctx.fillText("S", 0, radius - 11);
 ctx.fillText("W", -radius + 11, 0);
 ctx.fillText("E", radius - 11, 0);
 ctx.rotate((headingDeg * Math.PI) / 180);
 ctx.fillStyle = "#ef4444"; ctx.beginPath(); ctx.moveTo(0, -radius + 15); ctx.lineTo(-4, 0); ctx.lineTo(4, 0); ctx.closePath(); ctx.fill();
 ctx.fillStyle = "#0284c7"; ctx.beginPath(); ctx.moveTo(0, radius - 15); ctx.lineTo(-4, 0); ctx.lineTo(4, 0); ctx.closePath(); ctx.fill();
 ctx.restore();
}

function drawRoad(){
 const c=$("road"),x=c.getContext("2d"),w=c.width,h=c.height;x.clearRect(0,0,w,h);
 
 if(s.terrain==="water"){
  x.save();
  x.translate(w/2, h/2);
  x.scale(zoomLevel, zoomLevel);
  x.translate(-w/2, -h/2);

  // 1. 국립해양조사원 개방해(海) GIS 전자해도 표준 수역 배경
  x.fillStyle="#b8e0e6";x.fillRect(0,0,w,h);

  // 개방해 위경도 그리드 라인
  x.strokeStyle="rgba(14,116,144,0.18)";x.lineWidth=1;
  for(let gx=0; gx<w; gx+=45){ x.beginPath(); x.moveTo(gx, 0); x.lineTo(gx, h); x.stroke(); }
  for(let gy=0; gy<h; gy+=45){ x.beginPath(); x.moveTo(0, gy); x.lineTo(w, gy); x.stroke(); }
  
  // 2. 경인아라뱃길 실제 양안 지형 (연한 베이지/육지표시)
  x.fillStyle="#f3e8cb";
  x.beginPath();x.moveTo(0,0);x.lineTo(w*0.32,0);x.lineTo(w*0.28,h);x.lineTo(0,h);x.closePath();x.fill();
  x.beginPath();x.moveTo(w*0.68,0);x.lineTo(w,0);x.lineTo(w,h);x.lineTo(w*0.72,h);x.closePath();x.fill();

  x.strokeStyle="#8c7355";x.lineWidth=2;
  x.beginPath();x.moveTo(w*0.32,0);x.lineTo(w*0.28,h);x.stroke();
  x.beginPath();x.moveTo(w*0.68,0);x.lineTo(w*0.72,h);x.stroke();

  // 3. 시천교 교량 시설
  x.strokeStyle="#475569";x.lineWidth=8;
  x.beginPath();x.moveTo(w*0.2, h*0.42);x.lineTo(w*0.8, h*0.42);x.stroke();
  x.fillStyle="#334155";
  x.fillRect(w*0.3, h*0.42-6, 12, 12);
  x.fillRect(w*0.7, h*0.42-6, 12, 12);

  // 4. 수심 격자 및 등심선 (토글 연동)
  if(showDepth){
   x.strokeStyle="#0e7490";x.lineWidth=1.5;x.setLineDash([6,4]);
   x.beginPath();x.moveTo(w*0.36,0);x.lineTo(w*0.33,h*0.5);x.lineTo(w*0.4,h);x.stroke();
   x.setLineDash([]);

   x.fillStyle="#0f172a";x.font="bold 11px sans-serif";
   x.fillText("6.3m", w*0.44, h*0.22);
   x.fillText("5.8m", w*0.55, h*0.62);
   x.fillText("6.1m", w*0.48, h*0.85);
  }

  // 5. 항로표지(AtoN) 부표 (토글 연동)
  if(showAton){
   x.fillStyle="#22c55e"; x.beginPath(); x.arc(w*0.35, h*0.3, 5, 0, Math.PI*2); x.fill(); // 녹색 좌현등부표
   x.fillStyle="#ef4444"; x.beginPath(); x.arc(w*0.65, h*0.3, 5, 0, Math.PI*2); x.fill(); // 홍색 우현등부표
   x.fillStyle="#22c55e"; x.beginPath(); x.arc(w*0.37, h*0.7, 5, 0, Math.PI*2); x.fill();
   x.fillStyle="#ef4444"; x.beginPath(); x.arc(w*0.63, h*0.7, 5, 0, Math.PI*2); x.fill();
  }

  // 6. 마젠타 항로선 (주운수로 안전 경로)
  x.strokeStyle="#c026d3";x.lineWidth=3;x.setLineDash([10,6]);
  x.beginPath();
  let steerOffset = s.steer * 1.8;
  x.moveTo(w*0.5, 0);
  x.quadraticCurveTo(w*0.5 + steerOffset, h*0.5, w*0.5 + steerOffset*1.5 + Math.sin(s.distance)*15, h);
  x.stroke();
  x.setLineDash([]);

  // 7. 시천나루 출발점 마크
  x.fillStyle="#dc2626";x.beginPath();x.arc(w*0.5, h*0.42, 7, 0, Math.PI*2);x.fill();
  x.fillStyle="#fff";x.beginPath();x.arc(w*0.5, h*0.42, 3, 0, Math.PI*2);x.fill();
  x.fillStyle="#0f172a";x.font="bold 12px sans-serif";
  x.fillText("📍 시천나루 선착장 (시천가람터)", w*0.52, h*0.43);
  x.restore();

  // 개방해 사이트 표준 GIS 오버레이 정보창
  x.fillStyle="rgba(15,23,42,0.92)";x.fillRect(14,14,480,72);
  x.fillStyle="#38bdf8";x.font="bold 14px sans-serif";
  x.fillText("🗺️ 국립해양조사원 개방해(海) GIS Portal", 22, 36);
  x.fillStyle="#eef6ff";x.font="12px sans-serif";
  x.fillText(`항해거리: +${s.distance.toFixed(2)} NM · 속도: ${Math.round(s.speed)} kn (줌: ${zoomLevel.toFixed(1)}x)`, 22, 56);
  x.fillStyle="#f59e0b";x.font="11px sans-serif";
  x.fillText(`위치: 경인아라뱃길 수운수로 3.2km 지점`, 22, 74);

  // 하단 개방해 스케일 바 (Scale Bar)
  x.fillStyle="rgba(15,23,42,0.85)";x.fillRect(w - 150, h - 30, 138, 22);
  x.strokeStyle="#38bdf8";x.lineWidth=1;x.strokeRect(w - 150, h - 30, 138, 22);
  x.fillStyle="#eef6ff";x.font="10px sans-serif";
  x.fillText("┣━━ 500m ━━┫ KHOA", w - 142, h - 15);

  drawCompass(x, w - 50, 55, 32, s.steer * 1.5);
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

function drawMirror(id){const c=$(id),x=c.getContext("2d"),w=c.width,h=c.height;x.clearRect(0,0,w,h);x.fillStyle=s.terrain==="water"?"#b8e0e6":"#172b38";x.fillRect(0,0,w,h);x.fillStyle=s.terrain==="water"?"#c2e0e5":"#303c44";x.beginPath();x.moveTo(w*.35,0);x.lineTo(w*.65,0);x.lineTo(w,h);x.lineTo(0,h);x.closePath();x.fill();x.strokeStyle="#e2e7e9";x.setLineDash([10,10]);x.beginPath();x.moveTo(w*.5,0);x.lineTo(w*.5,h);x.stroke();x.setLineDash([])}
function drawMap(){const c=$("map"),x=c.getContext("2d"),w=c.width,h=c.height;x.fillStyle="#0d1520";x.fillRect(0,0,w,h);const a=35,z=w-35,y=48;x.strokeStyle="#334155";x.lineWidth=14;x.beginPath();x.moveTo(a,y);x.lineTo(z,y);x.stroke();x.strokeStyle="#0284c7";x.lineWidth=5;x.beginPath();x.moveTo(a,y-5);x.lineTo(a+(z-a)*.385,y-5);x.stroke();const names=[{n:"서울반포",k:0},{n:"판교",k:18},{n:"대전",k:160},{n:"부산센텀",k:416.2}];for(const p of names){const px=a+(z-a)*p.k/416.2;x.fillStyle="#94a3b8";x.beginPath();x.arc(px,y,4,0,Math.PI*2);x.fill();x.fillStyle="#cbd5e1";x.font="11px sans-serif";x.fillText(p.n,clamp(px-20,2,w-70),y+22)}const p=a+(z-a)*s.distance/416.2;x.fillStyle="#fff";x.beginPath();x.arc(p,y,7,0,Math.PI*2);x.fill();x.strokeStyle="#22c55e";x.lineWidth=2;x.stroke()}

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
 $("terrainState").textContent=isWater?"수상 · 국립해양조사원 개방해(海) GIS":"육상 · 경부고속도로";

 drawRoad();drawMirror("mirrorL");drawMirror("mirrorR");drawMap();
}

$("savePosBtn").onclick = () => { try { localStorage.setItem("yeondoo_sim_state", JSON.stringify({ state: s, zoom: zoomLevel })); notify("💾 위치 저장 완료"); }catch(e){} };
$("loadPosBtn").onclick = () => { try { const saved = localStorage.getItem("yeondoo_sim_state"); if(saved){ const p = JSON.parse(saved); s = p.state; zoomLevel = p.zoom || 1.0; notify("📂 불러오기 완료"); render(); } }catch(e){} };

$("depthToggle").onclick = () => { showDepth = !showDepth; $("depthToggle").classList.toggle("active", showDepth); notify(showDepth ? "수심격자 표시 켜짐" : "수심격자 표시 꺼짐"); render(); };
$("atonToggle").onclick = () => { showAton = !showAton; $("atonToggle").classList.toggle("active", showAton); notify(showAton ? "항로표지(부표) 켜짐" : "항로표지(부표) 꺼짐"); render(); };

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
$("waterMode").onclick=()=>{s.terrain="water";s.distance=0;notify("개방해(海) GIS 시천나루 항해 시작");render()};
$("pauseBtn").onclick=()=>{s.paused=!s.paused;render()};
$("resetBtn").onclick=()=>{s=freshState();zoomLevel=1.0;render()};

["1","2","3","4","5","N","R1","R2","R3"].forEach(g=>{const b=document.createElement("button");b.textContent=g;b.onclick=()=>shiftGear(g);$("gearButtons").appendChild(b)});

window.addEventListener("keydown",e=>{if(["ArrowUp","ArrowDown","ArrowLeft","ArrowRight","Space"].includes(e.code))e.preventDefault();pressed.add(e.code)});
window.addEventListener("keyup",e=>pressed.delete(e.code));

let last=performance.now();
function frame(now){const dt=Math.min((now-last)/1000,.05);last=now;update(dt);requestAnimationFrame(frame)}
log("국립해양조사원 개방해(海) GIS 연동 완료.");
render();requestAnimationFrame(frame);
</script>
</body>
</html>
