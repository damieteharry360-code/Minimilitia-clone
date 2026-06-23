<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no, viewport-fit=cover">
<title>JETPACK ARENA</title>
<style>
  :root{
    --bg:#0c0c1a;
    --panel:#11142a;
    --accent:#ffd400;
    --accent2:#00dcdc;
    --text:#eef0ff;
    --muted:#8d92b8;
  }
  *{ box-sizing:border-box; -webkit-tap-highlight-color:transparent; }
  html,body{
    margin:0; padding:0; width:100%; height:100%;
    background:var(--bg); overflow:hidden;
    font-family:"Segoe UI",Arial,Helvetica,sans-serif; color:var(--text);
    touch-action:none; user-select:none;
  }
  #stage{
    position:fixed; inset:0; display:flex; align-items:center; justify-content:center;
    background:radial-gradient(circle at 50% 30%, #161a36 0%, #07070f 70%);
  }
  canvas{ display:block; image-rendering:optimizeSpeed; background:#1e1e3c; box-shadow:0 0 60px rgba(0,0,0,.6); }

  /* ── Overlays ───────────────────────────────────────────── */
  .overlay{
    position:fixed; inset:0; display:flex; flex-direction:column; align-items:center; justify-content:center;
    background:rgba(6,7,16,.92); z-index:50; text-align:center; padding:20px;
  }
  .hidden{ display:none !important; }
  .title{
    font-size:clamp(34px,8vw,64px); font-weight:800; letter-spacing:2px; margin:0 0 4px;
    color:var(--accent); text-shadow:0 0 22px rgba(255,212,0,.45);
  }
  .subtitle{ color:var(--muted); margin:0 0 26px; font-size:clamp(13px,2.6vw,16px); letter-spacing:1px;}
  .panel{
    background:var(--panel); border:1px solid #2a2e54; border-radius:14px; padding:20px 26px;
    max-width:520px; width:92%; margin-bottom:18px;
  }
  .panel h3{ margin:0 0 10px; color:var(--accent2); font-size:15px; letter-spacing:1px;}
  .row{ display:flex; justify-content:center; gap:10px; flex-wrap:wrap; margin-top:10px;}
  .opt-btn{
    background:#1b1f40; border:1px solid #34396b; color:var(--text); padding:10px 16px; border-radius:10px;
    font-weight:700; cursor:pointer; font-size:14px;
  }
  .opt-btn.active{ background:var(--accent); color:#1a1400; border-color:var(--accent); }
  .big-btn{
    background:linear-gradient(180deg,#ffe24d,#ffb900); color:#1a1400; border:none; padding:16px 46px;
    border-radius:12px; font-size:20px; font-weight:800; letter-spacing:1px; cursor:pointer;
    box-shadow:0 6px 0 #8a6a00, 0 10px 24px rgba(0,0,0,.4); margin-top:6px;
  }
  .big-btn:active{ transform:translateY(4px); box-shadow:0 2px 0 #8a6a00; }
  .hint{ color:var(--muted); font-size:12px; margin-top:14px; max-width:480px; line-height:1.5;}
  .winner{ font-size:clamp(26px,6vw,40px); font-weight:800; margin-bottom:6px; }

  /* ── HUD ────────────────────────────────────────────────── */
  #hud{ position:absolute; inset:0; pointer-events:none; z-index:20; }
  .bar-wrap{ position:absolute; top:14px; left:14px; width:220px; }
  .bar-label{ font-size:11px; font-weight:700; letter-spacing:1px; color:var(--muted); margin-bottom:2px;}
  .bar{ height:12px; border-radius:6px; background:#000; border:1px solid #2c2f55; overflow:hidden; margin-bottom:7px;}
  .bar-fill{ height:100%; border-radius:6px; transition:width .12s linear; }
  #hpFill{ background:linear-gradient(90deg,#ff4b4b,#ff9d4b); }
  #fuelFill{ background:linear-gradient(90deg,#2c8bff,#36d6ff); }
  #weaponTag{
    position:absolute; top:84px; left:14px; font-size:13px; font-weight:800; letter-spacing:1px;
    padding:4px 10px; border-radius:6px; background:#000a; border:1px solid #2c2f55;
  }
  #score{
    position:absolute; top:14px; right:14px; background:#000a; border:1px solid #2c2f55; border-radius:10px;
    padding:8px 12px; font-size:12px; min-width:120px; text-align:left;
  }
  #score div{ display:flex; justify-content:space-between; gap:14px; padding:1px 0; }
  #killfeed{
    position:absolute; bottom:14px; left:0; right:0; text-align:center; font-size:13px; color:#fff;
    display:flex; flex-direction:column; align-items:center; gap:3px;
  }
  #killfeed span{ background:#000a; padding:3px 10px; border-radius:6px; opacity:0; transition:opacity .3s; }
  #slotBar{ position:absolute; top:14px; left:50%; transform:translateX(-50%); display:flex; gap:6px; pointer-events:auto;}
  .slotBtn{
    width:54px; height:54px; border-radius:10px; border:2px solid #2c2f55; background:#000a;
    color:#fff; font-size:10px; font-weight:800; display:flex; flex-direction:column; align-items:center;
    justify-content:center; cursor:pointer; gap:2px;
  }
  .slotBtn.sel{ border-color:var(--accent); box-shadow:0 0 10px rgba(255,212,0,.5); }
  .slotBtn small{ opacity:.75; font-weight:600;}

  /* swap prompt */
  #swapPanel{
    position:absolute; left:50%; top:50%; transform:translate(-50%,-50%); width:300px; pointer-events:auto;
    background:#0c0e22ee; border:2px solid var(--accent); border-radius:12px; padding:14px; z-index:40;
  }
  #swapPanel h4{ margin:0 0 8px; font-size:13px; color:var(--accent); letter-spacing:1px;}
  .swapRow{
    display:flex; justify-content:space-between; align-items:center; background:#1b1f40; border:1px solid #34396b;
    border-radius:8px; padding:9px 12px; margin-bottom:7px; cursor:pointer; font-size:13px; font-weight:700;
  }
  .swapRow:active{ background:#272c55; }
  .swapCancel{ text-align:center; color:var(--muted); font-size:12px; margin-top:4px; cursor:pointer; }

  /* ── Touch controls ─────────────────────────────────────── */
  #touchUI{ position:absolute; inset:0; z-index:30; pointer-events:none; }
  .stick{
    position:absolute; width:120px; height:120px; border-radius:50%; background:#ffffff10;
    border:2px solid #ffffff30; pointer-events:auto; touch-action:none;
  }
  .stick .nub{
    position:absolute; width:54px; height:54px; border-radius:50%; background:#ffffff35;
    top:33px; left:33px; border:2px solid #ffffff55;
  }
  #moveStick{ left:24px; bottom:24px; }
  #aimStick{ right:24px; bottom:24px; background:#ff4b4b14; border-color:#ff4b4b40; }
  #aimStick .nub{ background:#ff4b4b30; border-color:#ff4b4b66; }
  .actionBtn{
    position:absolute; pointer-events:auto; touch-action:none; border-radius:50%; border:2px solid #ffffff40;
    background:#ffffff14; color:#fff; font-size:11px; font-weight:800; display:flex; align-items:center;
    justify-content:center; letter-spacing:.5px;
  }
  #jumpBtn{ width:64px; height:64px; right:170px; bottom:90px; background:#36d6ff1c; border-color:#36d6ff55; }
  #jetBtn{ width:64px; height:64px; right:170px; bottom:18px; background:#ffd4001c; border-color:#ffd40066; }
  .pressed{ filter:brightness(1.6); }

  #muteBtn{
    position:absolute; top:14px; right:14px; pointer-events:auto; background:#000a; border:1px solid #2c2f55;
    color:#fff; width:34px; height:34px; border-radius:8px; cursor:pointer; font-size:15px; z-index:25;
    display:flex; align-items:center; justify-content:center;
  }
</style>
</head>
<body>

<div id="stage">
  <canvas id="game" width="1200" height="700"></canvas>
  <div id="hud">
    <div class="bar-wrap">
      <div class="bar-label">HEALTH</div>
      <div class="bar"><div class="bar-fill" id="hpFill" style="width:100%"></div></div>
      <div class="bar-label">FUEL</div>
      <div class="bar"><div class="bar-fill" id="fuelFill" style="width:100%"></div></div>
    </div>
    <div id="weaponTag">PISTOL · ∞</div>
    <div id="slotBar"></div>
    <div id="score"></div>
    <div id="killfeed"></div>
    <div id="swapPanel" class="hidden"></div>
  </div>
  <button id="muteBtn">🔊</button>
  <div id="touchUI" class="hidden">
    <div class="stick" id="moveStick"><div class="nub"></div></div>
    <div class="stick" id="aimStick"><div class="nub"></div></div>
    <div class="actionBtn" id="jumpBtn">JUMP</div>
    <div class="actionBtn" id="jetBtn">JET</div>
  </div>
</div>

<!-- ── Title screen ─────────────────────────────────────────── -->
<div class="overlay" id="titleScreen">
  <h1 class="title">JETPACK ARENA</h1>
  <p class="subtitle">a Mini-Militia-style jetpack shooter · plays on phone &amp; PC</p>

  <div class="panel">
    <h3>OPPONENTS</h3>
    <div class="row" id="botRow">
      <button class="opt-btn" data-bots="1">1 BOT</button>
      <button class="opt-btn active" data-bots="2">2 BOTS</button>
      <button class="opt-btn" data-bots="3">3 BOTS</button>
    </div>
  </div>
  <div class="panel">
    <h3>FIRST TO</h3>
    <div class="row" id="scoreRow">
      <button class="opt-btn" data-score="5">5 KILLS</button>
      <button class="opt-btn active" data-score="10">10 KILLS</button>
      <button class="opt-btn" data-score="15">15 KILLS</button>
    </div>
  </div>

  <button class="big-btn" id="startBtn">DROP IN</button>
  <p class="hint" id="controlHint">Detecting input method…</p>
</div>

<!-- ── Game over screen ─────────────────────────────────────── -->
<div class="overlay hidden" id="overScreen">
  <div class="winner" id="winnerText">—</div>
  <p class="subtitle" id="finalScores"></p>
  <button class="big-btn" id="restartBtn">PLAY AGAIN</button>
</div>

<script>
"use strict";
/* ════════════════════════════════════════════════════════════════════════
   JETPACK ARENA — single-file browser port
   Mechanics ported from the supplied pygame source (weapons, platforms,
   particles, bullets, loot boxes, armoury-swap prompt). Movement physics,
   collisions, AI, HUD, the game loop, and touch/desktop input are new,
   since the original script cut off before any of that was implemented.
   ════════════════════════════════════════════════════════════════════════ */

const WORLD_W = 1200, WORLD_H = 700;
const MAX_SLOTS = 4;
const GRAVITY = 0.6;
const MAX_FALL = 16;

const COLORS = {
  ground:"#3ca44b", platform:"#5d6b8a", platformEdge:"#aab4d4",
  sky1:"#161a36", sky2:"#06070f"
};

const WEAPONS = {
  pistol:  { damage:15, speed:18, ammo:30, fireRate:14, color:"#ffdc00", size:4, spread:2,  pellets:1 },
  rifle:   { damage:18, speed:21, ammo:50, fireRate:7,  color:"#ff8c00", size:4, spread:3,  pellets:1 },
  shotgun: { damage:11, speed:15, ammo:16, fireRate:32, color:"#e63232", size:5, spread:14, pellets:5 },
  sniper:  { damage:55, speed:28, ammo:10, fireRate:46, color:"#00dcdc", size:4, spread:0,  pellets:1 },
};
const WEAPON_LIST = Object.keys(WEAPONS);

const PLATFORMS = [
  {x:0,    y:WORLD_H-40, w:WORLD_W, h:40},
  {x:100,  y:540, w:220, h:18},
  {x:400,  y:480, w:200, h:18},
  {x:700,  y:540, w:220, h:18},
  {x:250,  y:390, w:180, h:18},
  {x:550,  y:330, w:180, h:18},
  {x:850,  y:390, w:180, h:18},
  {x:100,  y:260, w:150, h:18},
  {x:700,  y:260, w:150, h:18},
  {x:430,  y:190, w:200, h:18},
  {x:950,  y:200, w:180, h:18},
  {x:0,    y:200, w:180, h:18},
];

const PLAYER_COLORS = ["#50c8ff","#ff5050","#7ee37e","#e07eff"];
const NAMES = ["YOU","RAZOR","VIPER","GHOST","WOLF"];

function rectsOverlap(a,b){ return a.x < b.x+b.w && a.x+a.w > b.x && a.y < b.y+b.h && a.y+a.h > b.y; }
function clamp(v,a,b){ return Math.max(a, Math.min(b,v)); }
function rand(a,b){ return a + Math.random()*(b-a); }
function choice(arr){ return arr[Math.floor(Math.random()*arr.length)]; }

/* ── tiny WebAudio SFX (no external assets) ─────────────────────────── */
const SFX = (()=>{
  let ctx=null, muted=false;
  function ensure(){ if(!ctx){ try{ ctx = new (window.AudioContext||window.webkitAudioContext)(); }catch(e){} } }
  function blip(freq, dur, type, vol, slideTo){
    if(muted) return; ensure(); if(!ctx) return;
    const o=ctx.createOscillator(), g=ctx.createGain();
    o.type=type||"square"; o.frequency.value=freq;
    if(slideTo) o.frequency.exponentialRampToValueAtTime(slideTo, ctx.currentTime+dur);
    g.gain.value=vol; g.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime+dur);
    o.connect(g); g.connect(ctx.destination);
    o.start(); o.stop(ctx.currentTime+dur);
  }
  function noise(dur, vol){
    if(muted) return; ensure(); if(!ctx) return;
    const buf = ctx.createBuffer(1, ctx.sampleRate*dur, ctx.sampleRate);
    const d = buf.getChannelData(0);
    for(let i=0;i<d.length;i++) d[i]=(Math.random()*2-1)*(1-i/d.length);
    const src=ctx.createBufferSource(); src.buffer=buf;
    const g=ctx.createGain(); g.gain.value=vol;
    src.connect(g); g.connect(ctx.destination); src.start();
  }
  return {
    shoot(weapon){ const f = weapon==="sniper"?180:weapon==="shotgun"?90:weapon==="rifle"?260:320;
      blip(f, 0.09, "square", 0.05, f*0.4); },
    hit(){ noise(0.08, 0.10); },
    jump(){ blip(220,0.12,"triangle",0.06,440); },
    pickup(){ blip(440,0.12,"sine",0.07,880); },
    death(){ blip(140,0.35,"sawtooth",0.08,40); },
    win(){ blip(523,0.5,"square",0.07,1046); },
    toggleMute(v){ muted = v; },
  };
})();

/* ════════════════════════════════════════════════════════════════════
   PARTICLE
   ════════════════════════════════════════════════════════════════════ */
class Particle{
  constructor(x,y,color,vx,vy,life,size){
    this.x=x; this.y=y; this.color=color;
    this.vx = vx!==undefined? vx : rand(-4,4);
    this.vy = vy!==undefined? vy : rand(-6,1);
    this.life=life||30; this.maxLife=this.life; this.size=size||4;
  }
  update(){ this.x+=this.vx; this.y+=this.vy; this.vy+=0.25; this.life--; }
  draw(ctx){
    const s = Math.max(1, this.size*this.life/this.maxLife);
    ctx.globalAlpha = clamp(this.life/this.maxLife,0,1);
    ctx.fillStyle=this.color;
    ctx.beginPath(); ctx.arc(this.x,this.y,s,0,7); ctx.fill();
    ctx.globalAlpha=1;
  }
}

/* ════════════════════════════════════════════════════════════════════
   BULLET
   ════════════════════════════════════════════════════════════════════ */
class Bullet{
  constructor(x,y,angleDeg,weaponName,ownerId){
    const w = WEAPONS[weaponName];
    this.x=x; this.y=y;
    const rad = (angleDeg + rand(-w.spread,w.spread)) * Math.PI/180;
    this.vx = Math.cos(rad)*w.speed;
    this.vy = Math.sin(rad)*w.speed;
    this.damage=w.damage; this.color=w.color; this.size=w.size;
    this.owner=ownerId; this.alive=true; this.trail=[];
  }
  update(game){
    this.trail.push([this.x,this.y]); if(this.trail.length>6) this.trail.shift();
    this.x+=this.vx; this.y+=this.vy; this.vy+=0.08;
    if(!(this.x>0 && this.x<WORLD_W && this.y>-50 && this.y<WORLD_H+50)) this.alive=false;
    for(const p of PLATFORMS){
      if(this.x>p.x && this.x<p.x+p.w && this.y>p.y && this.y<p.y+p.h){
        this.alive=false;
        for(let i=0;i<4;i++) game.particles.push(new Particle(this.x,this.y,"#cfd4e8",rand(-2,2),rand(-2,2),16,3));
        break;
      }
    }
  }
  draw(ctx){
    const n=Math.max(this.trail.length,1);
    this.trail.forEach((pt,i)=>{
      const r=(i+1)/n;
      ctx.globalAlpha=r*0.7;
      ctx.fillStyle=this.color;
      ctx.beginPath(); ctx.arc(pt[0],pt[1],Math.max(1,this.size*r*0.6),0,7); ctx.fill();
    });
    ctx.globalAlpha=1;
    ctx.fillStyle=this.color;
    ctx.beginPath(); ctx.arc(this.x,this.y,this.size,0,7); ctx.fill();
  }
}

/* ════════════════════════════════════════════════════════════════════
   WEAPON PICKUP  (small bobbing crate spawned on platforms)
   ════════════════════════════════════════════════════════════════════ */
class WeaponPickup{
  constructor(x,y,weapon){
    this.x=x; this.y=y; this.weapon=weapon; this.alive=true; this.bob=rand(0,360); this.life=1400;
  }
  update(){ this.bob=(this.bob+2)%360; this.life--; if(this.life<=0) this.alive=false; }
  rect(){ return {x:this.x-16,y:this.y-16,w:32,h:32}; }
  draw(ctx){
    const off = Math.sin(this.bob*Math.PI/180)*5;
    const cx=this.x, cy=this.y+off;
    const col = WEAPONS[this.weapon].color;
    ctx.fillStyle="#1c1f3a"; ctx.strokeStyle=col; ctx.lineWidth=2;
    roundRect(ctx,cx-18,cy-14,36,28,6); ctx.fill(); ctx.stroke();
    ctx.fillStyle="#fff"; ctx.font="bold 11px Arial"; ctx.textAlign="center"; ctx.textBaseline="middle";
    ctx.fillText(this.weapon.slice(0,3).toUpperCase(), cx, cy+1);
  }
}

/* ════════════════════════════════════════════════════════════════════
   LOOT BOX  (dropped on death — contains the fallen fighter's slots)
   ════════════════════════════════════════════════════════════════════ */
class LootBox{
  constructor(x,y,slots,ownerColor){
    this.x=clamp(x,30,WORLD_W-30); this.y=clamp(y,30,WORLD_H-60);
    this.slots=slots.slice(); this.color=ownerColor; this.alive=true;
    this.bob=rand(0,360); this.glow=0; this.glowDir=1; this.life=900;
  }
  rect(){ return {x:this.x-18,y:this.y-18,w:36,h:36}; }
  update(){
    this.bob=(this.bob+2)%360;
    this.glow+=this.glowDir*3; if(this.glow>=80||this.glow<=0) this.glowDir*=-1;
    this.life--; if(this.life<=0) this.alive=false;
  }
  draw(ctx){
    const off=Math.sin(this.bob*Math.PI/180)*5, cx=this.x, cy=this.y+off, h=18;
    ctx.globalAlpha = this.glow/160;
    ctx.fillStyle=this.color;
    ctx.beginPath(); ctx.arc(cx,cy,30,0,7); ctx.fill();
    ctx.globalAlpha=1;
    ctx.fillStyle="#1c1f3a"; ctx.strokeStyle=this.color; ctx.lineWidth=3;
    roundRect(ctx,cx-h,cy-h,36,36,6); ctx.fill(); ctx.stroke();
    ctx.beginPath(); ctx.moveTo(cx-h+2,cy); ctx.lineTo(cx+h-2,cy);
    ctx.moveTo(cx,cy-h+2); ctx.lineTo(cx,cy+h-2); ctx.strokeStyle=this.color; ctx.lineWidth=2; ctx.stroke();
    ctx.fillStyle="#fff"; ctx.font="bold 10px Arial"; ctx.textAlign="center";
    ctx.fillText("LOOT", cx, cy+h+12);
    const ratio=this.life/900;
    ctx.fillStyle="#000"; ctx.fillRect(cx-h,cy+h+18,36,4);
    ctx.fillStyle="#ffdc00"; ctx.fillRect(cx-h,cy+h+18,36*ratio,4);
  }
}

/* ════════════════════════════════════════════════════════════════════
   ARMOURY SWAP PROMPT (full inventory -> choose a slot to replace)
   ════════════════════════════════════════════════════════════════════ */
class SwapPrompt{
  constructor(player,newWeapon,newAmmo,sourceRef,game){
    this.player=player; this.newWeapon=newWeapon; this.newAmmo=newAmmo;
    this.sourceRef=sourceRef; this.game=game; this.active=true;
  }
  choose(index){
    const p=this.player;
    if(index<0||index>=p.slots.length) return;
    const oldName=p.slots[index].name;
    p.slots[index]={name:this.newWeapon, ammo:this.newAmmo};
    if(p.weapon===oldName) p.weapon=this.newWeapon;
    if(this.sourceRef) this.sourceRef.alive=false;
    for(let i=0;i<14;i++) this.game.particles.push(new Particle(p.x+p.w/2,p.y+p.h/2,WEAPONS[this.newWeapon].color,undefined,undefined,28,5));
    SFX.pickup();
    this.active=false;
  }
  cancel(){ this.active=false; }
}

/* ════════════════════════════════════════════════════════════════════
   PLAYER  (human-controlled or AI bot)
   ════════════════════════════════════════════════════════════════════ */
class Player{
  constructor(x,y,id,color,name,isBot){
    this.id=id; this.color=color; this.name=name; this.isBot=isBot;
    this.w=28; this.h=44;
    this.x=x; this.y=y; this.vx=0; this.vy=0;
    this.onGround=false; this.facing=1; this.angle=0;
    this.fuel=100; this.maxFuel=100; this.jetting=false;
    this.hp=100; this.maxHp=100;
    this.slots=[{name:"pistol", ammo:WEAPONS.pistol.ammo}];
    this.weapon="pistol";
    this.fireCd=0; this.alive=true; this.respawnTimer=0;
    this.kills=0; this.deaths=0; this.hurtFlash=0; this.walkTick=0;
    // AI state
    this.aiTargetX = x; this.aiTargetY = y; this.aiRetargetT = 0; this.aiJetT=0;
  }
  rect(){ return {x:this.x, y:this.y, w:this.w, h:this.h}; }
  slotIndex(name){ return this.slots.findIndex(s=>s.name===name); }
  hasWeapon(name){ return this.slotIndex(name) !== -1; }
  activeAmmo(){ const i=this.slotIndex(this.weapon); return i!==-1? this.slots[i].ammo : 0; }
  setActiveAmmo(v){ const i=this.slotIndex(this.weapon); if(i!==-1) this.slots[i].ammo=v; }
  armouryFull(){ return this.slots.length>=MAX_SLOTS; }

  tryAddWeapon(name){
    if(this.hasWeapon(name)){
      const i=this.slotIndex(name);
      this.slots[i].ammo += WEAPONS[name].ammo;
      return "stacked";
    }
    if(!this.armouryFull()){
      this.slots.push({name, ammo:WEAPONS[name].ammo});
      this.weapon=name;
      return "added";
    }
    return "full";
  }
  lootAll(lootSlots){
    const overflow=[];
    for(const s of lootSlots){
      if(this.hasWeapon(s.name)){
        const i=this.slotIndex(s.name); this.slots[i].ammo += s.ammo;
      } else if(!this.armouryFull()){
        this.slots.push({name:s.name, ammo:s.ammo}); this.weapon=s.name;
      } else overflow.push(s);
    }
    return overflow;
  }
  cycleWeapon(dir=1){
    if(this.slots.length<2) return;
    let idx=this.slotIndex(this.weapon);
    idx=(idx+dir+this.slots.length)%this.slots.length;
    this.weapon=this.slots[idx].name;
  }
  gunTip(){
    const cx=this.x+this.w/2, shoulderY=this.y+18;
    const rad=this.angle*Math.PI/180;
    const handX=cx+Math.cos(rad)*16, handY=shoulderY+Math.sin(rad)*10;
    return [handX+Math.cos(rad)*14, handY+Math.sin(rad)*14];
  }
  shoot(game){
    if(!this.slots.length) return;
    const ammo=this.activeAmmo();
    if(ammo<=0){ return; }
    const w=WEAPONS[this.weapon];
    this.fireCd=w.fireRate;
    const cost=Math.min(w.pellets, ammo);
    this.setActiveAmmo(ammo-cost);
    const [gx,gy]=this.gunTip();
    for(let i=0;i<cost;i++) game.bullets.push(new Bullet(gx,gy,this.angle,this.weapon,this.id));
    SFX.shoot(this.weapon);
    for(let i=0;i<3;i++) game.particles.push(new Particle(gx,gy,w.color,Math.cos(this.angle*Math.PI/180)*2,Math.sin(this.angle*Math.PI/180)*2,10,3));
  }

  applyPhysics(game, moveX, jumpPressed, jetHeld){
    if(!this.alive){
      this.respawnTimer--;
      if(this.respawnTimer<=0) game.respawn(this);
      return;
    }
    // horizontal
    this.vx += moveX*1.1;
    this.vx *= 0.84;
    this.vx = clamp(this.vx,-7.5,7.5);
    if(Math.abs(moveX)>0.05) this.facing = moveX>0?1:-1;
    if(Math.abs(this.vx)>0.4 && this.onGround) this.walkTick+=Math.abs(this.vx)*0.3;

    if(jumpPressed && this.onGround){ this.vy=-13.5; this.onGround=false; SFX.jump(); }

    this.jetting = jetHeld && this.fuel>0;
    if(this.jetting){
      this.vy -= 0.95;
      this.fuel = Math.max(0,this.fuel-1.2);
      for(let i=0;i<2;i++) game.particles.push(new Particle(this.x+this.w/2+rand(-4,4), this.y+this.h, "#ff8c20", rand(-1,1), rand(2,5), 18, 4));
    } else {
      this.fuel = Math.min(this.maxFuel, this.fuel + (this.onGround?0.8:0.3));
    }

    this.vy = Math.min(this.vy+GRAVITY, MAX_FALL);
    this.x += this.vx; this.y += this.vy;
    this.x = clamp(this.x, 0, WORLD_W-this.w);
    if(this.y>WORLD_H+100){ // fell off world
      this.hp=0;
    }

    // platform collision (land on top only)
    this.onGround=false;
    for(const p of PLATFORMS){
      const feet = this.y+this.h;
      if(this.vy>=0 && feet>=p.y && feet<=p.y+p.h+this.vy+1 &&
         this.x+this.w*0.7 > p.x && this.x+this.w*0.3 < p.x+p.w){
        this.y = p.y-this.h; this.vy=0; this.onGround=true;
      }
    }

    if(this.fireCd>0) this.fireCd--;
    if(this.hurtFlash>0) this.hurtFlash--;

    if(this.hp<=0) game.killPlayer(this, game.lastDamagedBy[this.id]);
  }

  draw(ctx){
    if(!this.alive) return;
    const cx=this.x+this.w/2, topY=this.y;
    ctx.save();
    if(this.hurtFlash>0){ ctx.globalAlpha = 0.6+0.4*Math.sin(Date.now()/30); }

    // jet flame
    if(this.jetting){
      ctx.fillStyle="#ff9d2e";
      ctx.beginPath();
      ctx.moveTo(cx-6, topY+this.h);
      ctx.lineTo(cx+6, topY+this.h);
      ctx.lineTo(cx, topY+this.h+10+rand(0,6));
      ctx.fill();
    }

    // legs (simple walk wiggle)
    const swing = this.onGround && Math.abs(this.vx)>0.5 ? Math.sin(this.walkTick)*7 : 0;
    ctx.strokeStyle="#2a2a2a"; ctx.lineWidth=6; ctx.lineCap="round";
    ctx.beginPath();
    ctx.moveTo(cx-5, topY+26); ctx.lineTo(cx-5+swing*0.3, topY+42);
    ctx.moveTo(cx+5, topY+26); ctx.lineTo(cx+5-swing*0.3, topY+42);
    ctx.stroke();

    // torso
    ctx.fillStyle=this.color;
    roundRect(ctx, cx-9, topY+10, 18, 20, 5); ctx.fill();

    // head
    ctx.fillStyle="#d8a878";
    ctx.beginPath(); ctx.arc(cx, topY+5, 8, 0, 7); ctx.fill();
    ctx.fillStyle=this.color;
    ctx.beginPath(); ctx.arc(cx, topY+2, 8, Math.PI, 0); ctx.fill(); // helmet cap

    // arm + gun
    const rad=this.angle*Math.PI/180;
    const shoulderY=topY+18;
    const handX=cx+Math.cos(rad)*16, handY=shoulderY+Math.sin(rad)*10;
    ctx.strokeStyle=this.color; ctx.lineWidth=6; ctx.lineCap="round";
    ctx.beginPath(); ctx.moveTo(cx,shoulderY); ctx.lineTo(handX,handY); ctx.stroke();
    ctx.strokeStyle=WEAPONS[this.weapon].color; ctx.lineWidth=4;
    ctx.beginPath(); ctx.moveTo(handX,handY); ctx.lineTo(handX+Math.cos(rad)*14, handY+Math.sin(rad)*14); ctx.stroke();

    // jetpack tank
    ctx.fillStyle="#444"; ctx.fillRect(cx-12, topY+12, 6, 16);
    ctx.fillStyle="#444"; ctx.fillRect(cx+6, topY+12, 6, 16);

    ctx.restore();

    // name tag
    ctx.fillStyle="#fff"; ctx.font="bold 11px Arial"; ctx.textAlign="center";
    ctx.fillText(this.name, cx, topY-8);

    // mini hp bar above bots
    if(this.isBot){
      ctx.fillStyle="#000"; ctx.fillRect(cx-16, topY-20, 32, 4);
      ctx.fillStyle="#ff5050"; ctx.fillRect(cx-16, topY-20, 32*this.hp/this.maxHp, 4);
    }
  }
}

function roundRect(ctx,x,y,w,h,r){
  ctx.beginPath();
  ctx.moveTo(x+r,y);
  ctx.arcTo(x+w,y,x+w,y+h,r);
  ctx.arcTo(x+w,y+h,x,y+h,r);
  ctx.arcTo(x,y+h,x,y,r);
  ctx.arcTo(x,y,x+w,y,r);
  ctx.closePath();
}

/* ════════════════════════════════════════════════════════════════════
   GAME ORCHESTRATOR
   ════════════════════════════════════════════════════════════════════ */
class Game{
  constructor(numBots, scoreLimit){
    this.players=[];
    this.particles=[]; this.bullets=[]; this.pickups=[]; this.lootboxes=[];
    this.killFeed=[];
    this.scoreLimit=scoreLimit;
    this.pickupTimer=90;
    this.over=false;
    this.lastDamagedBy={};
    this.swapPrompt=null;

    const spawns = [[60,500],[1080,500],[560,140],[560,560]];
    this.human = new Player(spawns[0][0], spawns[0][1], 0, PLAYER_COLORS[0], NAMES[0], false);
    this.players.push(this.human);
    for(let i=0;i<numBots;i++){
      const sp = spawns[(i+1)%spawns.length];
      this.players.push(new Player(sp[0], sp[1], i+1, PLAYER_COLORS[(i+1)%PLAYER_COLORS.length], NAMES[(i+1)%NAMES.length], true));
    }
  }

  spawnPoint(){
    const spawns = [[60,500],[1080,500],[560,140],[60,140],[1080,140],[560,560]];
    return choice(spawns);
  }

  respawn(p){
    const [sx,sy]=this.spawnPoint();
    p.x=sx; p.y=sy; p.vx=0; p.vy=0; p.hp=p.maxHp; p.fuel=p.maxFuel;
    p.slots=[{name:"pistol", ammo:WEAPONS.pistol.ammo}]; p.weapon="pistol";
    p.alive=true; p.hurtFlash=0;
  }

  killPlayer(victim, killerId){
    if(!victim.alive) return;
    victim.alive=false;
    victim.deaths++;
    victim.respawnTimer=90;
    SFX.death();
    const killer = this.players.find(pl=>pl.id===killerId && pl.id!==victim.id);
    if(killer){
      killer.kills++;
      this.pushFeed(`${killer.name} eliminated ${victim.name}`);
      if(killer.kills>=this.scoreLimit) this.endGame(killer);
    } else {
      this.pushFeed(`${victim.name} was eliminated`);
    }
    for(let i=0;i<26;i++) this.particles.push(new Particle(victim.x+victim.w/2, victim.y+victim.h/2, victim.color, rand(-6,6), rand(-6,2), 36, 5));
    this.lootboxes.push(new LootBox(victim.x+victim.w/2, victim.y+victim.h/2, victim.slots, victim.color));
  }

  pushFeed(msg){
    this.killFeed.push({msg, life:180});
    if(this.killFeed.length>4) this.killFeed.shift();
  }

  endGame(winner){
    this.over=true; this.winner=winner; SFX.win();
  }

  trySpawnPickup(){
    this.pickupTimer--;
    if(this.pickupTimer<=0 && this.pickups.length<5){
      const plats = PLATFORMS.slice(1); // skip ground
      const p = choice(plats);
      this.pickups.push(new WeaponPickup(p.x+p.w/2, p.y-20, choice(WEAPON_LIST)));
      this.pickupTimer = Math.floor(rand(240,420));
    }
  }

  handlePickupCollect(player, weaponName, sourceRef){
    const result = player.tryAddWeapon(weaponName);
    if(result==="full"){
      this.swapPrompt = new SwapPrompt(player, weaponName, WEAPONS[weaponName].ammo, sourceRef, this);
    } else {
      if(sourceRef) sourceRef.alive=false;
      SFX.pickup();
    }
  }

  handleLootCollect(player, box){
    const overflow = player.lootAll(box.slots);
    box.alive=false;
    SFX.pickup();
    if(overflow.length){
      const first=overflow[0];
      this.swapPrompt = new SwapPrompt(player, first.name, first.ammo, null, this);
    }
  }

  update(input){
    if(this.over) return;
    this.trySpawnPickup();

    for(const pl of this.players){
      if(pl.isBot) this.updateBot(pl);
    }

    // human input -> physics
    let moveX=0;
    if(input.human){
      moveX = input.human.moveX;
      this.human.angle = input.human.angle;
      this.human.facing = input.human.facing;
      this.human.applyPhysics(this, moveX, input.human.jump, input.human.jet);
      if(input.human.fire && this.human.alive && !this.swapPrompt) this.human.shoot(this);
    }
    for(const pl of this.players){
      if(pl.isBot) pl.applyPhysics(this, pl._aiMoveX||0, pl._aiJump||false, pl._aiJet||false);
    }

    // bullets
    for(const b of this.bullets){
      b.update(this);
      if(!b.alive) continue;
      for(const pl of this.players){
        if(!pl.alive || pl.id===b.owner) continue;
        const r=pl.rect();
        if(b.x>r.x && b.x<r.x+r.w && b.y>r.y && b.y<r.y+r.h){
          pl.hp -= b.damage;
          pl.hurtFlash=10;
          this.lastDamagedBy[pl.id]=b.owner;
          b.alive=false;
          SFX.hit();
          for(let i=0;i<8;i++) this.particles.push(new Particle(b.x,b.y,"#ff5050",rand(-3,3),rand(-3,3),16,3));
        }
      }
    }
    this.bullets = this.bullets.filter(b=>b.alive);
    this.particles.forEach(p=>p.update());
    this.particles = this.particles.filter(p=>p.life>0);

    // pickups
    this.pickups.forEach(p=>p.update());
    for(const pk of this.pickups){
      if(!pk.alive) continue;
      for(const pl of this.players){
        if(!pl.alive) continue;
        if(rectsOverlap(pl.rect(), pk.rect())){
          this.handlePickupCollect(pl, pk.weapon, pk);
          break;
        }
      }
    }
    this.pickups = this.pickups.filter(p=>p.alive);

    // lootboxes
    this.lootboxes.forEach(l=>l.update());
    for(const lb of this.lootboxes){
      if(!lb.alive) continue;
      for(const pl of this.players){
        if(!pl.alive) continue;
        if(rectsOverlap(pl.rect(), lb.rect())){
          this.handleLootCollect(pl, lb);
          break;
        }
      }
    }
    this.lootboxes = this.lootboxes.filter(l=>l.alive);

    this.killFeed.forEach(k=>k.life--);
    this.killFeed = this.killFeed.filter(k=>k.life>0);
  }

  /* ── very small AI brain ─────────────────────────────────────── */
  updateBot(bot){
    if(!bot.alive){ bot._aiMoveX=0; bot._aiJump=false; bot._aiJet=false; return; }
    bot.aiRetargetT--;
    if(bot.aiRetargetT<=0){
      const p = choice(PLATFORMS);
      bot.aiTargetX = rand(p.x+10, p.x+p.w-10);
      bot.aiTargetY = p.y;
      bot.aiRetargetT = Math.floor(rand(90,180));
    }
    // prefer chasing nearest alive enemy if close & has ammo
    let target=null, bestD=99999;
    for(const pl of this.players){
      if(pl===bot || !pl.alive) continue;
      const d=Math.hypot(pl.x-bot.x, pl.y-bot.y);
      if(d<bestD){ bestD=d; target=pl; }
    }

    let moveX=0, wantJump=false, wantJet=false;
    const dx = bot.aiTargetX - (bot.x+bot.w/2);
    moveX = Math.abs(dx)>8 ? clamp(dx/40,-1,1) : 0;

    if(bot.aiTargetY < bot.y-20 && bot.fuel>20) wantJet = Math.random()<0.7;
    if(bot.onGround && Math.random()<0.02) wantJump=true;

    if(target && bestD<480){
      const dxp=target.x-bot.x, dyp=(target.y+target.h/2)-(bot.y+bot.h/2);
      bot.angle = Math.atan2(dyp,dxp)*180/Math.PI + rand(-6,6);
      bot.facing = Math.abs(bot.angle)>90 ? -1 : 1;
      if(bot.fireCd<=0 && Math.random()<0.85) bot.shoot(this);
      // try to keep some distance / line up
      if(bestD<140) moveX = -Math.sign(dxp)*0.6;
      else if(bestD>320) moveX = Math.sign(dxp)*0.8;
    }

    // pick up nearby loot if armoury not full
    if(!bot.armouryFull()){
      for(const pk of this.pickups){
        const d=Math.hypot(pk.x-bot.x, pk.y-bot.y);
        if(d<260){ moveX = clamp((pk.x-bot.x)/40,-1,1); break; }
      }
    }

    if(bot.fireCd>0) bot.fireCd--;
    bot._aiMoveX=moveX; bot._aiJump=wantJump; bot._aiJet=wantJet;
  }

  draw(ctx){
    // sky
    const grad=ctx.createLinearGradient(0,0,0,WORLD_H);
    grad.addColorStop(0,COLORS.sky1); grad.addColorStop(1,COLORS.sky2);
    ctx.fillStyle=grad; ctx.fillRect(0,0,WORLD_W,WORLD_H);
    // faint stars
    ctx.fillStyle="#ffffff14";
    for(let i=0;i<40;i++){ const sx=(i*97)%WORLD_W, sy=(i*53)%(WORLD_H-100); ctx.fillRect(sx,sy,2,2); }

    // platforms
    for(const p of PLATFORMS){
      ctx.fillStyle = p.h>30 ? COLORS.ground : COLORS.platform;
      roundRect(ctx,p.x,p.y,p.w,p.h, p.h>30?0:5); ctx.fill();
      ctx.fillStyle = COLORS.platformEdge;
      ctx.fillRect(p.x, p.y, p.w, 3);
    }

    this.pickups.forEach(p=>p.draw(ctx));
    this.lootboxes.forEach(l=>l.draw(ctx));
    this.players.forEach(p=>p.draw(ctx));
    this.bullets.forEach(b=>b.draw(ctx));
    this.particles.forEach(p=>p.draw(ctx));
  }
}

/* ════════════════════════════════════════════════════════════════════
   INPUT / UI WIRING
   ════════════════════════════════════════════════════════════════════ */
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");
const isTouch = window.matchMedia("(pointer: coarse)").matches || ("ontouchstart" in window) || window.innerWidth<880;

function resize(){
  const margin = 0;
  const availW = window.innerWidth - margin, availH = window.innerHeight - margin;
  const scale = Math.min(availW/WORLD_W, availH/WORLD_H);
  canvas.style.width = (WORLD_W*scale)+"px";
  canvas.style.height = (WORLD_H*scale)+"px";
}
window.addEventListener("resize", resize);
window.addEventListener("orientationchange", resize);
resize();

const keys = {};
window.addEventListener("keydown", e=>{
  keys[e.code]=true;
  if(["Space","ArrowUp","ArrowDown","ArrowLeft","ArrowRight"].includes(e.code)) e.preventDefault();
  if(game && game.swapPrompt && game.swapPrompt.active){
    if(e.code==="Escape") game.swapPrompt.cancel();
    const map={Digit1:0,Digit2:1,Digit3:2,Digit4:3};
    if(e.code in map) game.swapPrompt.choose(map[e.code]);
  } else if(["Digit1","Digit2","Digit3","Digit4"].includes(e.code)){
    const idx = Number(e.code.slice(-1))-1;
    if(game && game.human.slots[idx]) game.human.weapon = game.human.slots[idx].name;
  } else if(e.code==="KeyQ"){
    if(game) game.human.cycleWeapon(-1);
  } else if(e.code==="KeyE"){
    if(game) game.human.cycleWeapon(1);
  }
}, {passive:false});
window.addEventListener("keyup", e=>{ keys[e.code]=false; });

let mouseWorld={x:600,y:350}, mouseDown=false;
canvas.addEventListener("mousemove", e=>{
  const r=canvas.getBoundingClientRect();
  mouseWorld.x = (e.clientX-r.left) * (WORLD_W/r.width);
  mouseWorld.y = (e.clientY-r.top) * (WORLD_H/r.height);
});
canvas.addEventListener("mousedown", e=>{
  mouseDown=true;
  handleCanvasTap(e.clientX, e.clientY);
});
window.addEventListener("mouseup", ()=> mouseDown=false);

function canvasToWorld(clientX, clientY){
  const r=canvas.getBoundingClientRect();
  return { x:(clientX-r.left)*(WORLD_W/r.width), y:(clientY-r.top)*(WORLD_H/r.height) };
}

function handleCanvasTap(clientX, clientY){
  if(!game) return;
  if(game.swapPrompt && game.swapPrompt.active){
    // swap handled via DOM buttons (#swapPanel), not canvas — see renderSwapPanel
    return;
  }
}

/* ── virtual joysticks (touch + mouse fallback) ─────────────────── */
function makeStick(el){
  const state={x:0,y:0,active:false};
  const nub = el.querySelector(".nub");
  let activeId=null;
  function start(id,clientX,clientY){
    activeId=id; state.active=true; move(clientX,clientY);
  }
  function move(clientX,clientY){
    const r=el.getBoundingClientRect();
    const cx=r.left+r.width/2, cy=r.top+r.height/2;
    let dx=clientX-cx, dy=clientY-cy;
    const max=r.width/2;
    const dist=Math.hypot(dx,dy);
    if(dist>max){ dx=dx/dist*max; dy=dy/dist*max; }
    state.x = dx/max; state.y = dy/max;
    nub.style.left = (33+dx)+"px"; nub.style.top = (33+dy)+"px";
  }
  function end(id){
    if(id!==undefined && id!==activeId) return;
    activeId=null; state.active=false; state.x=0; state.y=0;
    nub.style.left="33px"; nub.style.top="33px";
  }
  el.addEventListener("pointerdown", e=>{ el.setPointerCapture(e.pointerId); start(e.pointerId,e.clientX,e.clientY); });
  el.addEventListener("pointermove", e=>{ if(activeId===e.pointerId) move(e.clientX,e.clientY); });
  el.addEventListener("pointerup", e=> end(e.pointerId));
  el.addEventListener("pointercancel", e=> end(e.pointerId));
  return state;
}
const moveStickEl=document.getElementById("moveStick");
const aimStickEl=document.getElementById("aimStick");
const moveState = makeStick(moveStickEl);
const aimState = makeStick(aimStickEl);

function makeHoldBtn(el){
  const state={held:false};
  el.addEventListener("pointerdown", e=>{ el.setPointerCapture(e.pointerId); state.held=true; el.classList.add("pressed"); });
  el.addEventListener("pointerup", ()=>{ state.held=false; el.classList.remove("pressed"); });
  el.addEventListener("pointercancel", ()=>{ state.held=false; el.classList.remove("pressed"); });
  return state;
}
const jumpState = makeHoldBtn(document.getElementById("jumpBtn"));
const jetState = makeHoldBtn(document.getElementById("jetBtn"));

if(isTouch){
  document.getElementById("touchUI").classList.remove("hidden");
}

/* ── mute ─────────────────────────────────────────────────────────── */
let muted=false;
document.getElementById("muteBtn").addEventListener("click", ()=>{
  muted=!muted; SFX.toggleMute(muted);
  document.getElementById("muteBtn").textContent = muted? "🔇":"🔊";
});

/* ── title screen option buttons ──────────────────────────────────── */
let chosenBots=2, chosenScore=10;
document.getElementById("botRow").addEventListener("click", e=>{
  const b=e.target.closest(".opt-btn"); if(!b) return;
  document.querySelectorAll("#botRow .opt-btn").forEach(x=>x.classList.remove("active"));
  b.classList.add("active"); chosenBots=Number(b.dataset.bots);
});
document.getElementById("scoreRow").addEventListener("click", e=>{
  const b=e.target.closest(".opt-btn"); if(!b) return;
  document.querySelectorAll("#scoreRow .opt-btn").forEach(x=>x.classList.remove("active"));
  b.classList.add("active"); chosenScore=Number(b.dataset.score);
});
document.getElementById("controlHint").textContent = isTouch
  ? "Left stick to move · right stick to aim & fire (hold) · JUMP / JET buttons · tap a weapon icon to switch."
  : "A/D or ←/→ to move · W / Space to jump · Shift or Ctrl to jet-pack · mouse to aim · click to fire · 1-4 to switch weapons.";

/* ════════════════════════════════════════════════════════════════════
   MAIN LOOP
   ════════════════════════════════════════════════════════════════════ */
let game=null;
let lastSwapKeys=[];

function startGame(){
  game = new Game(chosenBots, chosenScore);
  document.getElementById("titleScreen").classList.add("hidden");
  document.getElementById("overScreen").classList.add("hidden");
  requestAnimationFrame(loop);
}
document.getElementById("startBtn").addEventListener("click", startGame);
document.getElementById("restartBtn").addEventListener("click", startGame);

function gatherHumanInput(){
  const h=game.human;
  let moveX=0;
  if(isTouch){
    moveX = Math.abs(moveState.x)>0.12 ? moveState.x : 0;
  } else {
    if(keys["KeyA"]||keys["ArrowLeft"]) moveX-=1;
    if(keys["KeyD"]||keys["ArrowRight"]) moveX+=1;
  }

  let angle = h.angle, facing=h.facing, fire=false;
  if(isTouch){
    if(aimState.active && Math.hypot(aimState.x,aimState.y)>0.18){
      angle = Math.atan2(aimState.y, aimState.x)*180/Math.PI;
      facing = Math.abs(angle)>90? -1:1;
      fire = true;
    }
  } else {
    const cx=h.x+h.w/2, cy=h.y+18;
    angle = Math.atan2(mouseWorld.y-cy, mouseWorld.x-cx)*180/Math.PI;
    facing = Math.abs(angle)>90? -1:1;
    fire = mouseDown;
  }

  let jump=false, jet=false;
  if(isTouch){
    jump = jumpState.held; jet = jetState.held;
  } else {
    jump = !!(keys["KeyW"]||keys["Space"]||keys["ArrowUp"]);
    jet = !!(keys["ShiftLeft"]||keys["ShiftRight"]||keys["ControlLeft"]||keys["ControlRight"]);
  }

  return {human:{moveX, angle, facing, fire, jump, jet}};
}

function updateHUD(){
  const h=game.human;
  document.getElementById("hpFill").style.width = Math.max(0,h.hp)+"%";
  document.getElementById("fuelFill").style.width = Math.max(0,h.fuel)+"%";
  const tag=document.getElementById("weaponTag");
  const w=WEAPONS[h.weapon];
  tag.textContent = `${h.weapon.toUpperCase()} · ${h.activeAmmo()}`;
  tag.style.borderColor = w.color; tag.style.color = w.color;

  // slot bar
  const bar=document.getElementById("slotBar");
  bar.innerHTML="";
  h.slots.forEach(s=>{
    const btn=document.createElement("div");
    btn.className="slotBtn"+(s.name===h.weapon?" sel":"");
    btn.style.borderColor = WEAPONS[s.name].color;
    btn.innerHTML = `<div>${s.name.slice(0,3).toUpperCase()}</div><small>${s.ammo}</small>`;
    btn.addEventListener("pointerdown", ()=>{ h.weapon=s.name; });
    bar.appendChild(btn);
  });

  // scoreboard
  const sc=document.getElementById("score");
  sc.innerHTML = "<div style='color:#8d92b8;font-weight:700;margin-bottom:3px;'>SCOREBOARD</div>" +
    game.players.slice().sort((a,b)=>b.kills-a.kills).map(p=>
      `<div><span style="color:${p.color}">${p.name}</span><span>${p.kills}</span></div>`
    ).join("");

  // kill feed
  const kf=document.getElementById("killfeed");
  kf.innerHTML="";
  game.killFeed.forEach(k=>{
    const s=document.createElement("span");
    s.textContent=k.msg; s.style.opacity = Math.min(1,k.life/40);
    kf.appendChild(s);
  });

  // swap panel
  const panel=document.getElementById("swapPanel");
  if(game.swapPrompt && game.swapPrompt.active){
    panel.classList.remove("hidden");
    const sp=game.swapPrompt;
    panel.innerHTML = `<h4>ARMOURY FULL — pick a slot to replace</h4>` +
      h.slots.map((s,i)=>`<div class="swapRow" data-i="${i}" style="border-color:${WEAPONS[s.name].color}">
          <span>${isTouch? "":("["+(i+1)+"] ")}${s.name.toUpperCase()} ·${s.ammo}</span>
          <span style="color:${WEAPONS[sp.newWeapon].color}">→ ${sp.newWeapon.toUpperCase()}</span>
        </div>`).join("") +
      `<div class="swapCancel" id="swapCancelBtn">${isTouch?"tap to cancel":"[ESC] cancel"}</div>`;
    panel.querySelectorAll(".swapRow").forEach(row=>{
      row.addEventListener("pointerdown", ()=> sp.choose(Number(row.dataset.i)));
    });
    document.getElementById("swapCancelBtn").addEventListener("pointerdown", ()=> sp.cancel());
  } else {
    panel.classList.add("hidden");
  }
}

function showGameOver(){
  document.getElementById("overScreen").classList.remove("hidden");
  document.getElementById("winnerText").textContent =
    (game.winner.id===0? "YOU WIN! 🏆" : game.winner.name+" WINS");
  document.getElementById("winnerText").style.color = game.winner.color;
  document.getElementById("finalScores").textContent =
    game.players.slice().sort((a,b)=>b.kills-a.kills).map(p=>`${p.name} ${p.kills}`).join("   ·   ");
}

let lastT=0;
function loop(t){
  if(!game) return;
  const dt = Math.min(2, (t-lastT)/16.67 || 1); lastT=t;
  const input = gatherHumanInput();
  for(let i=0;i<1;i++) game.update(input); // fixed-ish step (dt smoothing kept simple)
  ctx.clearRect(0,0,WORLD_W,WORLD_H);
  game.draw(ctx);
  updateHUD();
  if(game.over){ showGameOver(); return; }
  requestAnimationFrame(loop);
}
</script>
</body>
</html>

