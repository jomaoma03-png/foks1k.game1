# foks1k.game1
<!DOCTYPE html>
<html lang="kk">
<head>
<meta charset="UTF-8">
<title>foks1k.game</title>
<style>
body{
  margin:0;
  overflow:hidden;
  font-family:Arial;
  text-align:center;
  color:white;
  background:url('https://images.unsplash.com/photo-1507525428034-b723cf961d3e?auto=format&fit=crop&w=1350&q=80') no-repeat center center fixed;
  background-size:cover;
}
button{
  padding:12px 25px;
  margin:10px;
  font-size:16px;
  border:none;
  border-radius:10px;
  background:yellow;
  cursor:pointer;
}
canvas{
  display:none;
}
#menu,#modes,#levels,#settings,#gameOver{
  position:absolute;
  width:100%;
  top:35%;
}
#inGameButtons{
  position:absolute;
  top:10px;
  left:10px;
  z-index:10;
}
</style>
</head>
<body>

<div id="menu">
  <h1>🔥 foks1k.game 🔥</h1>
  <button onclick="showModes()">▶ ОЙНАУ</button>
  <button onclick="openSettings()">⚙️ НАСТРОЙКА</button>
</div>

<div id="modes" style="display:none;">
  <h2>Режим таңда</h2>
  <button onclick="showLevels('time')">⏱ Уақытша</button>
  <button onclick="showLevels('death')">💀 Өлгенше</button>
</div>

<div id="levels" style="display:none;">
  <h2>Деңгей таңда</h2>
  <button onclick="startGame(1)">1-деңгей</button>
  <button onclick="startGame(2)">2-деңгей</button>
  <button onclick="startGame(3)">3-деңгей</button>
</div>

<div id="settings" style="display:none;">
  <h2>Настройка</h2>
  <button onclick="toggleMusic()">🎵 Музыканы қосу/өшіру</button>
  <button onclick="toggleJumpSound()">🔊 Секіру дыбысын қосу/өшіру</button><br>
  <button onclick="backMenu()">⬅ Артқа</button>
</div>

<div id="gameOver" style="display:none;">
  <h1>💀 ӨЛДІҢ!</h1>
  <button onclick="restartGame()">🔄 ҚАЙТА БАСТАУ</button>
</div>

<canvas id="gameCanvas"></canvas>

<!-- In-game buttons -->
<div id="inGameButtons" style="display:none;">
  <button onclick="stopGame()">⏹ Тоқтату / Меню</button>
</div>

<!-- Музыка -->
<audio id="bgMusic" loop>
  <source src="https://cdn.pixabay.com/audio/2022/03/15/audio_115b9b0bdf.mp3">
</audio>

<!-- Секіру дыбысы -->
<audio id="jumpSound">
  <source src="https://cdn.pixabay.com/audio/2022/03/15/audio_4c0f7b5a5f.mp3">
</audio>

<script>
let canvas=document.getElementById("gameCanvas");
let ctx=canvas.getContext("2d");

let player, obstacles, gravity, speed, ground, level, gameRunning=false, mode='death';
let musicOn=true, jumpSoundOn=true;
let music=document.getElementById("bgMusic");
let jump=document.getElementById("jumpSound");

const menu=document.getElementById("menu");
const modesDiv=document.getElementById("modes");
const levelsDiv=document.getElementById("levels");
const settingsDiv=document.getElementById("settings");
const gameOverDiv=document.getElementById("gameOver");
const inGameButtons=document.getElementById("inGameButtons");

function resize(){
  canvas.width=window.innerWidth;
  canvas.height=window.innerHeight;
  ground=canvas.height-80;
}
window.onresize=resize;
resize();

// Меню функциялары
function showModes(){menu.style.display="none"; modesDiv.style.display="block";}
function showLevels(selectedMode){modesDiv.style.display="none"; levelsDiv.style.display="block"; mode=selectedMode;}
function openSettings(){menu.style.display="none"; settingsDiv.style.display="block";}
function backMenu(){settingsDiv.style.display="none"; menu.style.display="block";}

// Музыка қосу/өшіру
function toggleMusic(){musicOn=!musicOn; if(musicOn) music.play(); else music.pause();}
function toggleJumpSound(){jumpSoundOn=!jumpSoundOn;}

// Ойын бастау
function startGame(lvl){
  level=lvl;
  levelsDiv.style.display="none";
  canvas.style.display="block";
  inGameButtons.style.display="block";
  document.body.style.background="#1e3c72"; // Ойын фоны
  initGame();
}

// Ойын қайта бастау
function restartGame(){
  gameOverDiv.style.display="none";
  canvas.style.display="block";
  inGameButtons.style.display="block";
  document.body.style.background="#1e3c72";
  initGame();
}

// Тоқтату / менюға қайту
function stopGame(){
  gameRunning=false;
  canvas.style.display="none";
  inGameButtons.style.display="none";
  menu.style.display="block";
  document.body.style.background="url('https://images.unsplash.com/photo-1507525428034-b723cf961d3e?auto=format&fit=crop&w=1350&q=80') no-repeat center center fixed";
  document.body.style.backgroundSize="cover";
  if(musicOn) music.pause();
}

// Инициализация
function initGame(){
  player={x:100,y:ground,size:30,dy:0};
  obstacles=[];
  gravity=0.9;
  speed=(level==1?6:(level==2?8:10));
  if(musicOn){music.currentTime=0; music.play();}
  gameRunning=true;
  if(mode==='time'){
    setTimeout(()=>{gameOver();},60000); // 1 минут уақытша режим
  }
  gameLoop();
}

// Пробел секіру
document.addEventListener("keydown",(e)=>{
  if(e.code==="Space" && player.y>=ground){
    player.dy=-22; // Қатты секіру
    if(jumpSoundOn) {jump.currentTime=0; jump.play();}
  }
});

// Логика
function update(){
  if(!gameRunning) return;

  player.dy+=gravity;
  player.y+=player.dy;
  if(player.y>=ground) player.y=ground, player.dy=0;

  if(Math.random()<0.02) obstacles.push({x:canvas.width,y:ground,size:30});
  obstacles.forEach(o=>o.x-=speed);

  // Collision
  for(let o of obstacles){
    if(player.x<o.x+o.size && player.x+player.size>o.x && player.y+player.size>ground-o.size){
      gameOver();
      break;
    }
  }
  obstacles=obstacles.filter(o=>o.x>-50);
}

// Drawing
function draw(){
  ctx.clearRect(0,0,canvas.width,canvas.height);

  // Ground
  ctx.fillStyle="white";
  ctx.fillRect(0,ground+30,canvas.width,3);

  // Кубик
  ctx.fillStyle="cyan";
  ctx.fillRect(player.x,player.y,player.size,player.size);

  // Кедергі - үшбұрыш
  ctx.fillStyle="red";
  obstacles.forEach(o=>{
    ctx.beginPath();
    ctx.moveTo(o.x,ground+30);
    ctx.lineTo(o.x+o.size/2,ground-o.size);
    ctx.lineTo(o.x+o.size,ground+30);
    ctx.fill();
  });

  ctx.fillStyle="white";
  ctx.fillText("Деңгей: "+level,10,30);
  ctx.fillText("Режим: "+(mode==='death'?'Өлгенше':'Уақытша'),10,50);
}

// Game loop
function gameLoop(){
  if(!gameRunning) return;
  update();
  draw();
  requestAnimationFrame(gameLoop);
}

// Game over
function gameOver(){
  gameRunning=false;
  canvas.style.display="none";
  inGameButtons.style.display="none";
  gameOverDiv.style.display="block";
  if(musicOn) music.pause();
}
</script>
</body>
</html>
