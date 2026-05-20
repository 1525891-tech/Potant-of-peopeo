<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<title>Potant of Peopeo</title>
<style>
body {
  margin: 0;
  overflow: hidden;
  background: #050505;
  font-family: Arial, sans-serif;
}
canvas { display: block; }
#ui {
  position: absolute;
  top: 10px;
  left: 10px;
  color: white;
  font-size: 14px;
  line-height: 1.4em;
}
#msg {
  position: absolute;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  color: white;
  font-size: 18px;
  text-align: center;
}
</style>
</head>
<body>
<canvas id="game"></canvas>
<div id="ui"></div>
<div id="msg"></div>

<script>
const canvas = document.getElementById("game");
const ctx = canvas.getContext("2d");
canvas.width = window.innerWidth;
canvas.height = window.innerHeight;

let keys = {};
document.addEventListener("keydown", e => keys[e.key.toLowerCase()] = true);
document.addEventListener("keyup", e => keys[e.key.toLowerCase()] = false);

let gameState = "collect"; // collect, trap, chase, win, gameover
let keysCollected = 0;
let trapTriggered = false;
let rockTimer = 0;

let player = { x: 80, y: 80, speed: 2.5 };
let peopeo = { x: 700, y: 300, speed: 1.3, active: false };

let keysItems = [
  {x:120,y:150,collected:false},
  {x:300,y:220,collected:false},
  {x:500,y:120,collected:false},
  {x:650,y:260,collected:false},
  {x:400,y:400,collected:false}
];

let door1 = {x:750,y:500,open:false};
let fakeExit = {x:900,y:120};
let realExit = {x:1000,y:500};

function movePlayer(){
  if(gameState=="gameover" || gameState=="win") return;
  if(keys["w"]) player.y -= player.speed;
  if(keys["s"]) player.y += player.speed;
  if(keys["a"]) player.x -= player.speed;
  if(keys["d"]) player.x += player.speed;
}

function dist(a,b){ return Math.hypot(a.x-b.x,a.y-b.y); }

function collectKeys(){
  if(gameState!="collect") return;
  keysItems.forEach(k=>{
    if(!k.collected && dist(player,k)<20){
      k.collected=true;
      keysCollected++;
    }
  });
}

function openDoor1(){
  if(keysCollected===5 && dist(player,door1)<40){
    door1.open=true;
    gameState="trap";
  }
}

function triggerTrap(){
  if(gameState!="trap"||trapTriggered) return;
  trapTriggered=true;
  rockTimer=300;
}

function updateTrap(){
  if(gameState=="trap" && trapTriggered){
    rockTimer--;
    if(rockTimer<=0){
      gameState="chase";
      peopeo.active=true;
    }
  }
}

function movePeopeo(){
  if(!peopeo.active || gameState=="gameover") return;
  let dx=player.x-peopeo.x;
  let dy=player.y-peopeo.y;
  let d=Math.hypot(dx,dy);
  peopeo.x+=(dx/d)*peopeo.speed;
  peopeo.y+=(dy/d)*peopeo.speed;
}

function checkFakeExit(){
  if(gameState=="trap" && dist(player,fakeExit)<30){
    triggerTrap();
  }
}

function tryRealExit(){
  if(gameState!="chase") return;
  if(dist(player,realExit)<40){
    if(Math.random()<0.8){
      gameState="win";
    } else {
      peopeo.speed+=0.5;
      document.getElementById("msg").innerText="DOOR FAILED! THE PEOPEO IS CLOSER!";
    }
  }
}

// GAME OVER COLLISION
function checkGameOver(){
  if(gameState=="chase" && dist(player,peopeo)<18){
    gameState="gameover";
  }
}

function drawDoor(x,y,open,label){
  ctx.fillStyle=open?"#2ecc71":"#c0392b";
  ctx.fillRect(x,y,40,70);
  ctx.strokeStyle="#fff";
  ctx.strokeRect(x,y,40,70);
  ctx.fillStyle="#fff";
  ctx.font="10px Arial";
  ctx.fillText(label,x,y-5);
}

function drawPlayer(){
  ctx.fillStyle="cyan";
  ctx.beginPath();
  ctx.arc(player.x,player.y,10,0,Math.PI*2);
  ctx.fill();
  ctx.fillStyle="black";
  ctx.fillRect(player.x-3,player.y-3,2,2);
  ctx.fillRect(player.x+3,player.y-3,2,2);
}

function drawPeopeo(){
  if(!peopeo.active) return;
  ctx.fillStyle="#ff3b3b";
  ctx.beginPath();
  ctx.arc(peopeo.x,peopeo.y,16,0,Math.PI*2);
  ctx.fill();
  ctx.fillStyle="#fff";
  ctx.beginPath();
  ctx.arc(peopeo.x-5,peopeo.y-4,3,0,Math.PI*2);
  ctx.arc(peopeo.x+5,peopeo.y-4,3,0,Math.PI*2);
  ctx.fill();
}

function drawGameOver(){
  ctx.fillStyle="rgba(0,0,0,0.7)";
  ctx.fillRect(0,0,canvas.width,canvas.height);

  ctx.font="60px Arial";
  ctx.fillStyle="red";
  ctx.textAlign="center";
  ctx.fillText("👹",canvas.width/2,canvas.height/2-20);

  ctx.font="30px Arial";
  ctx.fillText("GAME OVER",canvas.width/2,canvas.height/2+40);
}

function draw(){
  ctx.clearRect(0,0,canvas.width,canvas.height);

  keysItems.forEach(k=>{
    if(!k.collected){
      ctx.fillStyle="yellow";
      ctx.fillRect(k.x,k.y,10,10);
    }
  });

  drawDoor(door1.x,door1.y,door1.open,"DOOR");
  drawDoor(fakeExit.x,fakeExit.y,false,"FAKE");
  drawDoor(realExit.x,realExit.y,true,"EXIT");

  drawPlayer();
  drawPeopeo();

  if(gameState=="trap" && trapTriggered){
    ctx.fillStyle="white";
    ctx.font="20px Arial";
    ctx.fillText("ROCK FALLING! RUN!",canvas.width/2-80,60);
  }

  if(gameState=="gameover"){
    drawGameOver();
  }
}

function updateUI(){
  document.getElementById("ui").innerHTML=`Keys: ${keysCollected}/5<br>State: ${gameState}`;

  if(gameState=="win") document.getElementById("msg").innerText="YOU ESCAPED... FOR NOW.";
  if(gameState=="gameover") document.getElementById("msg").innerText="";
}

function loop(){
  movePlayer();
  collectKeys();
  openDoor1();
  checkFakeExit();
  updateTrap();
  movePeopeo();
  tryRealExit();
  checkGameOver();

  draw();
  updateUI();

  requestAnimationFrame(loop);
}

loop();
</script>
</body>
</html>
