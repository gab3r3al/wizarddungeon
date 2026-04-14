const canvas = document.getElementById("gameCanvas");
const ctx = canvas.getContext("2d");

let keys = {};
let bullets = [];
let enemies = [];
let powerUps = [];
let gameOver = false;
let bossSpawned = false;

const player = {
  x: 400,
  y: 300,
  size: 16,
  speed: 3,
  hp: 5,
  color: "#6dbf6d", // wizard robe
  shootCooldown: 0,
  power: 1
};

// Controls
document.addEventListener("keydown", e => keys[e.key] = true);
document.addEventListener("keyup", e => keys[e.key] = false);

// Shoot
document.addEventListener("click", e => {
  if (player.shootCooldown <= 0) {
    bullets.push({
      x: player.x,
      y: player.y,
      dx: (e.offsetX - player.x) / 10,
      dy: (e.offsetY - player.y) / 10
    });
    player.shootCooldown = 20;
  }
});

// Spawn skeletons
function spawnEnemy() {
  const side = Math.floor(Math.random() * 4);
  let x, y;

  if (side === 0) { x = 0; y = Math.random() * 600; }
  if (side === 1) { x = 800; y = Math.random() * 600; }
  if (side === 2) { x = Math.random() * 800; y = 0; }
  if (side === 3) { x = Math.random() * 800; y = 600; }

  enemies.push({
    x, y,
    size: 14,
    speed: 1 + Math.random(),
    hp: 2
  });
}

// Boss
function spawnBoss() {
  enemies.push({
    x: 400,
    y: 100,
    size: 40,
    speed: 0.6,
    hp: 50,
    boss: true
  });
}

// Power-up drop
function dropPowerUp(x, y) {
  if (Math.random() < 0.3) {
    powerUps.push({ x, y, type: "power" });
  }
}

// Update
function update() {
  if (gameOver) return;

  // Movement
  if (keys["w"]) player.y -= player.speed;
  if (keys["s"]) player.y += player.speed;
  if (keys["a"]) player.x -= player.speed;
  if (keys["d"]) player.x += player.speed;

  player.shootCooldown--;

  // Bullets
  bullets.forEach((b, i) => {
    b.x += b.dx;
    b.y += b.dy;

    enemies.forEach((e, ei) => {
      if (Math.hypot(b.x - e.x, b.y - e.y) < e.size) {
        e.hp--;
        bullets.splice(i, 1);
        if (e.hp <= 0) {
          dropPowerUp(e.x, e.y);
          enemies.splice(ei, 1);
        }
      }
    });
  });

  // Enemies
  enemies.forEach(e => {
    let dx = player.x - e.x;
    let dy = player.y - e.y;
    let dist = Math.hypot(dx, dy);

    e.x += (dx / dist) * e.speed;
    e.y += (dy / dist) * e.speed;

    // Damage player
    if (dist < e.size + player.size) {
      player.hp -= 0.02;
      if (player.hp <= 0) gameOver = true;
    }
  });

  // Power-ups
  powerUps.forEach((p, i) => {
    if (Math.hypot(player.x - p.x, player.y - p.y) < 20) {
      player.power++;
      powerUps.splice(i, 1);
    }
  });

  // Spawn enemies
  if (Math.random() < 0.02 && !bossSpawned) spawnEnemy();

  // Spawn boss
  if (!bossSpawned && player.power > 5) {
    spawnBoss();
    bossSpawned = true;
  }
}

// Draw pixel-style wizard
function drawPlayer() {
  ctx.fillStyle = "#6dbf6d";
  ctx.fillRect(player.x - 10, player.y - 10, 20, 20);

  // Hat
  ctx.fillStyle = "#4a8f4a";
  ctx.fillRect(player.x - 8, player.y - 20, 16, 10);

  // Face
  ctx.fillStyle = "#ffffff";
  ctx.fillRect(player.x - 6, player.y - 4, 12, 8);
}

// Draw
function draw() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  // Player
  drawPlayer();

  // Bullets
  ctx.fillStyle = "#ffd966";
  bullets.forEach(b => ctx.fillRect(b.x, b.y, 4, 4));

  // Enemies (skeletons)
  enemies.forEach(e => {
    ctx.fillStyle = e.boss ? "#cccccc" : "#ffffff";
    ctx.fillRect(e.x - e.size/2, e.y - e.size/2, e.size, e.size);
  });

  // Power-ups
  ctx.fillStyle = "#ffcc00";
  powerUps.forEach(p => ctx.fillRect(p.x - 5, p.y - 5, 10, 10));

  // UI
  ctx.fillStyle = "white";
  ctx.fillText("HP: " + Math.floor(player.hp), 10, 20);
  ctx.fillText("Power: " + player.power, 10, 40);

  if (gameOver) {
    ctx.fillText("GAME OVER", 350, 300);
  }
}

// Game loop
function loop() {
  update();
  draw();
  requestAnimationFrame(loop);
}

loop();
