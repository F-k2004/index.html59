<!DOCTYPE html>
<html lang="fa">
<head>
<meta charset="UTF-8">
<title>🧠 Voyager AI – Data Transmission Manager</title>
<style>
html,body{margin:0;overflow:hidden;background:#00020a;font-family:system-ui}
canvas{display:block}
.hud{
 position:absolute;left:16px;top:16px;
 padding:14px 18px;border-radius:14px;
 background:rgba(255,255,255,0.06);
 backdrop-filter:blur(10px);
 color:#d9f3ff;font-size:13px;min-width:420px;
}
.good{color:#9ff0ff}
.mid{color:#ffd29f}
.bad{color:#ff9f9f}
.warn{color:#ffb366}
.ai{color:#8fffcb}
</style>
</head>
<body>

<canvas id="c"></canvas>
<div class="hud" id="hud"></div>

<script>
const c=document.getElementById("c");
const ctx=c.getContext("2d");
let w,h;
function resize(){w=c.width=innerWidth;h=c.height=innerHeight}
resize();addEventListener("resize",resize);

// Constants
const AU=65;
const sun={x:w/2,y:h/2};
const terminationShock=85;
const heliopause=120;

// Planets
const earth={a:AU*1,angle:0,r:6};

// Spacecraft
const ship={
 distance:AU*1,
 angle:0.4,
 velocity:0.22,
 buffer:0, // MB stored
 phase:"INNER SOLAR SYSTEM"
};

// Radio
const txPower=210;

// Storm
let storm={active:false,intensity:0};

// Helpers
function pos(a,ang){
 return {x:sun.x+Math.cos(ang)*a,y:sun.y+Math.sin(ang)*a};
}

// Storm logic
function updateStorm(){
 if(ship.distance/AU<20 && Math.random()<0.001){
  storm.active=true;
  storm.intensity=1+Math.random()*3;
 }
 if(storm.active){
  storm.intensity*=0.995;
  if(storm.intensity<0.1) storm.active=false;
 }
}

// Link model
function link(distAU){
 let signal=txPower/(distAU*distAU*140);
 let noise=0.1+Math.random()*0.06;
 if(distAU>terminationShock) noise+=0.2;
 if(distAU>heliopause) noise+=0.3;
 if(storm.active) noise+=storm.intensity*0.5;
 return {snr:signal/noise,latency:distAU*8.3};
}

// 🧠 AI DECISION SYSTEM
function aiDecision(snr){
 if(storm.active) return "HOLD (SOLAR STORM)";
 if(snr>4) return "LIVE TRANSMISSION";
 if(snr>2) return "COMPRESSED DATA";
 if(snr>1) return "STORE & WAIT";
 return "TRANSMISSION HALTED";
}

// Data handling
function manageData(snr){
 let decision=aiDecision(snr);
 if(decision==="LIVE TRANSMISSION"){
  ship.buffer=Math.max(0,ship.buffer-0.4);
 }
 else if(decision==="COMPRESSED DATA"){
  ship.buffer=Math.max(0,ship.buffer-0.2);
 }
 else if(decision==="STORE & WAIT"){
  ship.buffer+=0.15;
 }
 else{
  ship.buffer+=0.25;
 }
 return decision;
}

// Update
function update(){
 earth.angle+=0.0012;
 ship.distance+=ship.velocity;
 ship.angle+=0.002;

 const dAU=ship.distance/AU;
 ship.phase=
  dAU<terminationShock?"SOLAR WIND":
  dAU<heliopause?"TERMINATION SHOCK":
  "INTERSTELLAR SPACE";

 updateStorm();
}

// Draw
function draw(){
 ctx.fillStyle="rgba(0,2,10,0.35)";
 ctx.fillRect(0,0,w,h);

 update();

 const pe=pos(earth.a,earth.angle);
 const pship={x:sun.x+Math.cos(ship.angle)*ship.distance,
              y:sun.y+Math.sin(ship.angle)*ship.distance};

 // Boundaries
 ctx.strokeStyle="rgba(255,180,120,0.25)";
 ctx.beginPath();ctx.arc(sun.x,sun.y,terminationShock*AU,0,Math.PI*2);ctx.stroke();
 ctx.strokeStyle="rgba(160,120,255,0.25)";
 ctx.beginPath();ctx.arc(sun.x,sun.y,heliopause*AU,0,Math.PI*2);ctx.stroke();

 // Sun
 ctx.fillStyle="#ffcc66";
 ctx.beginPath();ctx.arc(sun.x,sun.y,12,0,Math.PI*2);ctx.fill();

 // Earth
 ctx.fillStyle="#2a7fff";
 ctx.beginPath();ctx.arc(pe.x,pe.y,6,0,Math.PI*2);ctx.fill();

 // Ship
 ctx.fillStyle="#fff";
 ctx.beginPath();ctx.arc(pship.x,pship.y,3,0,Math.PI*2);ctx.fill();

 const distAU=Math.hypot(pship.x-pe.x,pship.y-pe.y)/AU;
 const q=link(distAU);
 const ai=manageData(q.snr);

 let cls=q.snr>4?"good":q.snr>2?"mid":"bad";

 // Link
 ctx.strokeStyle=
  cls==="good"?"rgba(120,220,255,0.5)":
  cls==="mid" ?"rgba(255,200,120,0.5)":
               "rgba(255,120,120,0.5)";
 ctx.beginPath();
 ctx.moveTo(pe.x,pe.y);
 ctx.lineTo(pship.x,pship.y);
 ctx.stroke();

 // HUD
 document.getElementById("hud").innerHTML=`
🧠 AI Data Transmission Manager<br>
🚀 Phase: <b>${ship.phase}</b><br>
📏 Distance: ${distAU.toFixed(1)} AU<br>
📊 SNR: <span class="${cls}">${q.snr.toFixed(2)}</span><br>
🕒 Latency: ${(q.latency/60).toFixed(2)} h<br>
💾 Buffer: ${ship.buffer.toFixed(2)} MB<br>
🌞 Solar Storm: ${storm.active?'<span class="warn">ACTIVE</span>':'NO'}<br>
🤖 AI Decision: <span class="ai">${ai}</span>
`;

 requestAnimationFrame(draw);
}

draw();
</script>
</body>
</html>
