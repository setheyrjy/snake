<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Snake+</title>
  <style>
    body { background: #222; color: #fff; font-family: sans-serif; text-align: center; }
    #gameCanvas { background: #111; margin: 20px auto; display: block; }
    .hidden { display: none; }
    #ad { margin: 20px auto; }
    #adsImg { max-width: 400px; max-height: 300px; }
    #menu, #quests, #features { margin: 20px auto; }
    .btn { padding: 10px 20px; margin: 10px; font-size: 16px; }
  </style>
</head>
<body>
  <h1>Snake+</h1>
  <div id="ad" class="hidden">
    <img id="adsImg" src="" alt="Ad poster">
    <div>Ad will close in <span id="adTimer">8</span> seconds...</div>
  </div>
  <div id="menu">
    <div>
      <button class="btn" onclick="startGame('Easy')">Play Easy</button>
      <button class="btn" onclick="startGame('Medium')">Play Medium</button>
      <button class="btn" onclick="startGame('Hard')">Play Hard</button>
    </div>
    <div id="features"></div>
    <div id="quests"></div>
    <div>Coins: <span id="coins">0</span></div>
    <div id="highscores"></div>
  </div>
  <canvas id="gameCanvas" width="400" height="400" class="hidden"></canvas>
  <div id="gameOver" class="hidden">
    <h2>Game Over!</h2>
    <button class="btn" onclick="backToMenu()">Back to Menu</button>
  </div>
<script>
// --- Persistent Data ---
function getSave() {
  let s = localStorage.getItem('snakeplus_save');
  if (s) return JSON.parse(s);
  return {
    coins: 0,
    highscores: {Easy: 0, Medium: 0, Hard: 0},
    custom_snake: false,
    player_name: 'Player',
    ads_disabled: false,
    leaderboard_unlocked: false,
    name_unlocked: false,
    quests: [],
    quest_refresh: '',
  };
}
function setSave(s) { localStorage.setItem('snakeplus_save', JSON.stringify(s)); }
let save = getSave();

// --- Ads ---
const adImages = ['ad1.png', 'ad2.png', 'ad3.png'];
function showAd(callback) {
  if (save.ads_disabled) { callback(); return; }
  document.getElementById('menu').classList.add('hidden');
  document.getElementById('ad').classList.remove('hidden');
  let img = adImages[Math.floor(Math.random()*adImages.length)];
  document.getElementById('adsImg').src = img;
  let timer = 8;
  document.getElementById('adTimer').innerText = timer;
  let intv = setInterval(() => {
    timer--;
    document.getElementById('adTimer').innerText = timer;
    if (timer === 0) {
      clearInterval(intv);
      document.getElementById('ad').classList.add('hidden');
      callback();
    }
  }, 1000);
}

// --- Menu & Features ---
function updateMenu() {
  document.getElementById('coins').innerText = save.coins;
  let f = [];
  f.push(save.custom_snake ? "Snake Customization (Unlocked!)" : "Unlock Customization: 200 coins");
  f.push(save.name_unlocked ? `Change Name (${save.player_name}) (Unlocked!)` : "Unlock Name Change: 400 coins");
  f.push(save.leaderboard_unlocked ? "Leaderboard (Unlocked!)" : "Unlock Leaderboard: 800 coins");
  f.push(save.ads_disabled ? "Ads Disabled!" : "Disable Ads: 1600 coins");
  document.getElementById('features').innerHTML = f.map(x => `<div>${x}</div>`).join('');
  let hs = [];
  for (let k of Object.keys(save.highscores)) hs.push(`${k}: ${save.highscores[k]}`);
  document.getElementById('highscores').innerText = "Highscores: " + hs.join(", ");
  document.getElementById('quests').innerHTML = "<b>Daily Quests:</b><br>(Coming soon!)";
}
updateMenu();

// --- Game Logic ---
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
let snake, apple, dir, nextDir, score, interval, applesEaten, running, difficulty, gameOverFlag;

function startGame(diff) {
  showAd(() => {
    document.getElementById('menu').classList.add('hidden');
    canvas.classList.remove('hidden');
    document.getElementById('gameOver').classList.add('hidden');
    difficulty = diff;
    let speed = {Easy:120, Medium:80, Hard:50}[diff];
    snake = [{x:200, y:200}];
    dir = {x:20, y:0};
    nextDir = dir;
    score = 0;
    applesEaten = 0;
    gameOverFlag = false;
    placeApple();
    running = true;
    interval = setInterval(gameLoop, speed);
  });
}

function placeApple() {
  while (true) {
    let x = Math.floor(Math.random()*20)*20, y = Math.floor(Math.random()*20)*20;
    if (!snake.some(s => s.x===x && s.y===y)) {
      apple = {x, y};
      return;
    }
  }
}

function gameLoop() {
  // Update direction
  dir = nextDir;
  // Move head
  let nx = snake[0].x + dir.x, ny = snake[0].y + dir.y;
  if (nx<0||ny<0||nx>=400||ny>=400||snake.some(s=>s.x===nx&&s.y===ny)) {
    gameOver();
    return;
  }
  snake.unshift({x:nx, y:ny});
  // Eat apple
  if (nx === apple.x && ny === apple.y) {
    score++;
    applesEaten++;
    placeApple();
  } else {
    snake.pop();
  }
  render();
}

function render() {
  ctx.fillStyle = "#111"; ctx.fillRect(0,0,400,400);
  ctx.fillStyle = "red"; ctx.fillRect(apple.x, apple.y, 20, 20);
  ctx.fillStyle = save.custom_snake ? "#"+((1<<24)*Math.random()|0).toString(16) : "lime";
  for (let s of snake) ctx.fillRect(s.x, s.y, 20, 20);
  ctx.fillStyle = "#fff";
  ctx.fillText("Score: " + score, 10, 20);
}

document.addEventListener("keydown", e => {
  if (!running) return;
  let d = dir;
  if (e.key==="ArrowUp" && d.y===0) nextDir = {x:0, y:-20};
  else if (e.key==="ArrowDown" && d.y===0) nextDir = {x:0, y:20};
  else if (e.key==="ArrowLeft" && d.x===0) nextDir = {x:-20, y:0};
  else if (e.key==="ArrowRight" && d.x===0) nextDir = {x:20, y:0};
});

function gameOver() {
  running = false;
  clearInterval(interval);
  // Highscore logic
  if (score > (save.highscores[difficulty]||0)) {
    save.highscores[difficulty] = score;
    let coinsAdd = difficulty==="Easy"?30:difficulty==="Medium"?40:50;
    save.coins += coinsAdd;
    alert(`New Highscore! +${coinsAdd} coins`);
    checkUnlocks();
  }
  setSave(save);
  updateMenu();
  canvas.classList.add('hidden');
  document.getElementById('gameOver').classList.remove('hidden');
}

function backToMenu() {
  document.getElementById('gameOver').classList.add('hidden');
  document.getElementById('menu').classList.remove('hidden');
  updateMenu();
}

// --- Unlocks ---
function checkUnlocks() {
  if (!save.custom_snake && save.coins >= 200) save.custom_snake = true;
  if (!save.name_unlocked && save.coins >= 400) save.name_unlocked = true;
  if (!save.leaderboard_unlocked && save.coins >= 800) save.leaderboard_unlocked = true;
  if (!save.ads_disabled && save.coins >= 1600) save.ads_disabled = true;
  setSave(save);
}
</script>
</body>
</html>
