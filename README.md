Coding by Phuluan
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Sky Master - CEO Edition</title>
    <style>
        body { margin: 0; overflow: hidden; background: #4facfe; font-family: 'Segoe UI', Arial, sans-serif; }
        #game-container { position: relative; width: 100vw; height: 100vh; background: linear-gradient(#4facfe, #00f2fe); }
        
        
</head>
<body>

<div id="game-container">
    <div id="hud">
        TOP: <span id="topVal">0</span>m | DISTANCE: <span id="distVal">0</span>m | GOLD: <span id="goldVal">7595</span>
    </div>
    <div id="countdown">5</div>

    <audio id="bgMusic" loop><source src="https://www.soundhelix.com/examples/mp3/SoundHelix-Song-8.mp3" type="audio/mpeg"></audio>
    <audio id="tingSfx"><source src="https://assets.mixkit.co/active_storage/sfx/2571/2571-preview.mp3" type="audio/mpeg"></audio>

    <div id="ui-layer">
        <div id="startMenu" class="panel">
            <h1>SKY MASTER</h1>
            <div class="gold-info">Vàng hiện có: <span class="currentGold">7595</span></div>
            <button class="btn-start" onclick="pressStart()">BẮT ĐẦU 🚀</button>
            <button class="btn-shop" onclick="showShop()">SHOP 🛒</button>
            <button class="btn-inventory" onclick="alert('Kho đồ đang cập nhật...')">KHO ĐỒ 📦</button>
        </div>

        <div id="shopMenu" class="panel" style="display:none">
            <h1>SHOP VẬT PHẨM</h1>
            <p style="font-size:12px; color:red">Thuế bán vật phẩm: 10%</p>
            <button class="btn-shop" style="background:#42a5f5; color:white" onclick="buy('Shield', 500)">MUA KHIÊN (500G)</button>
            <button class="btn-shop" style="background:#ffa726; color:white" onclick="buy('Turbo', 1000)">MUA TURBO (1000G)</button>
            <button class="btn-inventory" style="background:#78909c; color:white" onclick="hideShop()">QUAY LẠI</button>
        </div>

        <div id="deathMenu" class="panel" style="display:none">
            <h2 style="color:#e53935">GAME OVER!</h2>
            <p>Quãng đường: <span id="finalDist">0</span>m</p>
            <button class="btn-start" style="background:#42a5f5" onclick="pressRestart()">CHƠI LẠI 🔄</button>
            <button class="btn-shop" onclick="showShop()">VÀO SHOP 🛒</button>
        </div>
    </div>

    <canvas id="gameCanvas"></canvas>
</div>

<script>
    const canvas = document.getElementById('gameCanvas');
    const ctx = canvas.getContext('2d');
    canvas.width = window.innerWidth; canvas.height = window.innerHeight;

    let bird = { x: 90, y: canvas.height/2, w: 45, h: 32, dy: 0, gravity: 0.5, jump: -8.5 };
    let pipes = [], clouds = [], dist = 0, frame = 0;
    let gold = 7595, highscore = localStorage.getItem('skyTop') || 0;
    let isPlaying = false;

    const music = document.getElementById('bgMusic');
    const ting = document.getElementById('tingSfx');
    music.volume = 0.5;

    // Cập nhật HUD ban đầu
    function updateHUD() {
        document.getElementById('topVal').innerText = highscore;
        document.getElementById('goldVal').innerText = gold;
        document.querySelectorAll('.currentGold').forEach(el => el.innerText = gold);
    }
    updateHUD();

    function showShop() { document.getElementById('startMenu').style.display='none'; document.getElementById('deathMenu').style.display='none'; document.getElementById('shopMenu').style.display='block'; }
    function hideShop() { document.getElementById('shopMenu').style.display='none'; document.getElementById('startMenu').style.display='block'; }
    
    function buy(item, price) {
        if(gold >= price) { gold -= price; alert("Đã mua " + item + "!"); updateHUD(); } 
        else { alert("Sếp không đủ vàng!"); }
    }

    function pressStart() {
        document.getElementById('startMenu').style.display = 'none';
        music.play();
        runCountdown();
    }

    function runCountdown() {
        let time = 5;
        const cd = document.getElementById('countdown');
        cd.style.display = 'block';
        let timer = setInterval(() => {
            cd.innerText = time; time--;
            if(time < 0) { clearInterval(timer); cd.style.display = 'none'; isPlaying = true; gameLoop(); }
        }, 1000);
    }

    function gameLoop() {
        if (!isPlaying) return;
        ctx.clearRect(0, 0, canvas.width, canvas.height);

        // 1. MÂY CHILL BỒNG BỀNH
        if (frame % 80 === 0) clouds.push({ x: canvas.width, y: Math.random()*200+50, w: 100, h: 40, s: Math.random()*1+0.5 });
        ctx.fillStyle = "rgba(255, 255, 255, 0.7)";
        clouds.forEach(c => {
            c.x -= c.s;
            ctx.beginPath(); ctx.ellipse(c.x, c.y, c.w/2, c.h/2, 0, 0, Math.PI*2); ctx.fill();
        });

        // 2. NHÂN VẬT (CHIM VÀNG)
        bird.dy += bird.gravity;
        bird.y += bird.dy;
        ctx.fillStyle = "#fbc02d"; ctx.fillRect(bird.x, bird.y, bird.w, bird.h);
        ctx.strokeStyle = "#fff"; ctx.lineWidth = 2; ctx.strokeRect(bird.x, bird.y, bird.w, bird.h);

        // 3. CỘT & TIẾNG TING
        if (frame % 100 === 0) {
            let gap = 200, h = Math.random()*(canvas.height-400)+100;
            pipes.push({ x: canvas.width, h: h, gap: gap, passed: false });
        }
        ctx.fillStyle = "#43a047";
        pipes.forEach(p => {
            p.x -= 6;
            ctx.fillRect(p.x, 0, 60, p.h); // Cột trên
            ctx.fillRect(p.x, p.h + p.gap, 60, canvas.height); // Cột dưới
            
            if (!p.passed && bird.x > p.x + 60) {
                p.passed = true; ting.currentTime = 0; ting.play();
            }
            if (bird.x < p.x + 60 && bird.x + bird.w > p.x && (bird.y < p.h || bird.y + bird.h > p.h + p.gap)) endGame();
        });

        if (bird.y > canvas.height || bird.y < 0) endGame();

        dist += 0.1;
        document.getElementById('distVal').innerText = Math.floor(dist);
        frame++;
        requestAnimationFrame(gameLoop);
    }

    function endGame() {
        isPlaying = false;
        if(dist > highscore) { highscore = Math.floor(dist); localStorage.setItem('skyTop', highscore); }
        document.getElementById('deathMenu').style.display = 'block';
        document.getElementById('finalDist').innerText = Math.floor(dist);
        updateHUD();
    }

    function pressRestart() {
        bird.y = canvas.height/2; bird.dy = 0; dist = 0; pipes = []; clouds = []; frame = 0;
        document.getElementById('deathMenu').style.display = 'none';
        runCountdown();
    }

    window.onmousedown = () => { if(isPlaying) bird.dy = bird.jump; };
    window.onkeydown = (e) => { if(e.code === 'Space' && isPlaying) bird.dy = bird.jump; };
</script>
</body>
</html>
