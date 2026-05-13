# Handoff prompt for claude.ai

**For dad:** Open this file, copy everything from the `---` line below to the end of the file, paste it as one message into a fresh chat at https://claude.ai. Claude.ai supports Artifacts, which renders HTML inline so Livy can play directly in chat — that solves the hosting problem.

---

# 🥪 Take over building my daughter's game (Sandwich Shop Tycoon)

Hi Claude! I'm Olivia's dad. My 8-year-old daughter ("Livy") has been building her first-ever video game with another Claude session. We're moving over here so she can actually PLAY her game inline.

## 🚨 CRITICAL — please use an Artifact

Put the game in a single HTML **Artifact** so Livy can play it right in this chat by tapping it on her iPhone. Update the same artifact when she asks for changes — don't make a new one each time.

## 👧 About Livy

- Age 8, zero coding experience.
- She is the **game designer / boss**. You are her helper.
- Talk like a friendly buddy. Simple kid words. Emojis 🎉.
- Ask **ONE question at a time**.
- Break things into TINY steps. Celebrate every small win.
- No jargon ("instantiate", "framework", etc.) without a simple kid explanation.
- She's on an iPhone — assume her dad is sometimes relaying for her.

## 🥪 The Game — Sandwich Shop Tycoon

- **Hero:** a sandwich-with-chef-hat character who runs the shop.
- **Setting:** busy city street with a cozy sandwich shop.
- **Goal:** serve customers, earn money, upgrade, become the Sandwich Tycoon.
- **Challenge:** timer — customers get grumpy if they wait too long, so she taps them fast.
- **Colors:** warm/cozy — bread browns 🍞, cheese yellows 🧀, tomato reds 🍅.
- **Platform:** mobile-first iPhone & iPad portrait.

## 🛠 Tech constraints

- ONE self-contained HTML file. No external libs, no fetch, no modules, no servers.
- `<canvas>` + plain JS in `<script>` tags + inline CSS in `<style>`.
- Mobile-first: 500×900 design canvas that scales to any iPhone/iPad with `devicePixelRatio` handling for crisp retina rendering.
- Touch via `pointerdown`. Prevent pinch-zoom, double-tap-zoom, scroll bounce.

## ⚠️ Design decisions Livy has already made — keep these

- **Game does NOT get harder over time.** She specifically asked to remove the speedup. Keep the customer spawn rate steady at 2.5s.
- **Shop name:** "LIVY'S SANDWICHES".
- **Chef face:** simple friendly dots + smile — NOT googly tracking eyes (she tried, decided they were creepy).

## ✅ What's already built (don't rebuild — use the code below)

Mobile-first canvas · cozy shop with red/cream awning + sign + window · sandwich-chef hero with waving arms that bounces on sale · sidewalk + dashed-line street · emoji customers walking in from the right and queuing at 3 spots · 🥪? speech bubbles + green/yellow/red wait bars · tap a waiting customer to serve (+$3, happy leave) or timeout (grumpy leave) · HUD with 💰 / 😊 / 😡 counters · steady 2.5s customer spawn rate.

## 💡 Likely next steps (let HER pick — don't decide for her)

- 🎵 Sound effects when she serves a customer.
- 👨‍🍳 Helpers she can buy with money that auto-serve customers.
- 🏪 A second shop location.
- 🎨 Custom colors, hat color, or rename the shop.
- 🥪 Build-the-sandwich mini-game by tapping ingredients.

When ready, greet Livy warmly and ask her what she wants to add next. 🥪🎉

## 📂 Current `game.html` — start your artifact from this exact code

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, minimum-scale=1, user-scalable=no, viewport-fit=cover">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  <meta name="theme-color" content="#fff4d6">
  <title>Sandwich Shop Tycoon</title>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; -webkit-tap-highlight-color: transparent; -webkit-touch-callout: none; }
    html, body {
      width: 100%; height: 100%; overflow: hidden; background: #fff4d6;
      font-family: -apple-system, BlinkMacSystemFont, "Helvetica Neue", Arial, sans-serif;
      touch-action: none; -webkit-user-select: none; user-select: none;
      overscroll-behavior: none; position: fixed;
    }
    #game { display: block; position: absolute; top: 0; left: 0; width: 100%; height: 100%; touch-action: none; }
  </style>
</head>
<body>
  <canvas id="game"></canvas>
  <script>
    const canvas = document.getElementById("game");
    const ctx = canvas.getContext("2d");

    const DESIGN_W = 500;
    const DESIGN_H = 900;
    let scale = 1, offsetX = 0, offsetY = 0, dpr = 1;

    function resize() {
      dpr = window.devicePixelRatio || 1;
      const cssW = window.innerWidth, cssH = window.innerHeight;
      canvas.width = Math.round(cssW * dpr);
      canvas.height = Math.round(cssH * dpr);
      canvas.style.width = cssW + "px";
      canvas.style.height = cssH + "px";
      scale = Math.min(cssW / DESIGN_W, cssH / DESIGN_H);
      offsetX = (cssW - DESIGN_W * scale) / 2;
      offsetY = (cssH - DESIGN_H * scale) / 2;
    }
    function applyTransform() {
      ctx.setTransform(scale * dpr, 0, 0, scale * dpr, offsetX * dpr, offsetY * dpr);
    }
    function screenToDesign(x, y) {
      return { x: (x - offsetX) / scale, y: (y - offsetY) / scale };
    }

    let money = 0, happyCount = 0, grumpyCount = 0;
    let lastTime = 0, spawnTimer = 0, spawnEvery = 2.5, chefBounce = 0;

    const SPOTS = [
      { x: 110, y: 660, taken: false },
      { x: 250, y: 660, taken: false },
      { x: 390, y: 660, taken: false }
    ];
    const customers = [];
    const CUSTOMER_EMOJIS = ["🧑", "👩", "🧒", "👨", "👵", "👦", "🧓", "👱‍♀️", "👨‍🦰", "🧑‍🦱"];

    function spawnCustomer() {
      const free = SPOTS.findIndex(s => !s.taken);
      if (free === -1) return;
      SPOTS[free].taken = true;
      customers.push({
        emoji: CUSTOMER_EMOJIS[Math.floor(Math.random() * CUSTOMER_EMOJIS.length)],
        x: DESIGN_W + 40, y: SPOTS[free].y, spotIndex: free,
        state: "walking", wait: 8, maxWait: 8, speed: 90, pay: 3,
        bob: Math.random() * Math.PI * 2
      });
    }

    function tick(now) {
      const dt = Math.min(0.05, (now - lastTime) / 1000 || 0);
      lastTime = now;
      spawnTimer += dt;
      if (spawnTimer >= spawnEvery) { spawnTimer = 0; spawnCustomer(); }

      for (const c of customers) {
        c.bob += dt * 4;
        if (c.state === "walking") {
          const target = SPOTS[c.spotIndex];
          c.x -= c.speed * dt;
          if (c.x <= target.x) { c.x = target.x; c.state = "waiting"; }
        } else if (c.state === "waiting") {
          c.wait -= dt;
          if (c.wait <= 0) {
            c.state = "leavingGrumpy"; grumpyCount++;
            SPOTS[c.spotIndex].taken = false;
          }
        } else if (c.state === "leavingHappy" || c.state === "leavingGrumpy") {
          c.x -= c.speed * 1.4 * dt;
        }
      }
      for (let i = customers.length - 1; i >= 0; i--) {
        if (customers[i].x < -60) customers.splice(i, 1);
      }
      chefBounce *= 0.9;
      draw();
      requestAnimationFrame(tick);
    }

    function draw() {
      ctx.setTransform(1, 0, 0, 1, 0, 0);
      ctx.fillStyle = "#fff4d6";
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      applyTransform();

      ctx.fillStyle = "#ffe4c4"; ctx.fillRect(0, 0, DESIGN_W, 500);
      ctx.fillStyle = "#cfa97a"; ctx.fillRect(0, 500, DESIGN_W, 200);
      ctx.fillStyle = "#b8915f";
      for (let x = 0; x < DESIGN_W; x += 50) ctx.fillRect(x, 500, 2, 200);
      ctx.fillStyle = "#555"; ctx.fillRect(0, 700, DESIGN_W, 160);
      ctx.fillStyle = "#ffd54f";
      for (let x = 10; x < DESIGN_W; x += 40) ctx.fillRect(x, 778, 20, 4);

      drawShop();
      drawChef();
      for (const c of customers) drawCustomer(c);
      drawTopBar();

      ctx.fillStyle = "#fff";
      ctx.font = "bold 16px -apple-system, Arial, sans-serif";
      ctx.textAlign = "center";
      ctx.fillText("Tap a customer to give them a 🥪!", DESIGN_W / 2, 875);
    }

    function drawTopBar() {
      ctx.fillStyle = "#5d3a1a"; ctx.fillRect(0, 0, DESIGN_W, 60);
      ctx.fillStyle = "#fff8dc";
      ctx.font = "bold 22px -apple-system, Arial, sans-serif";
      ctx.textAlign = "left";   ctx.fillText("💰 $" + money, 18, 38);
      ctx.textAlign = "center"; ctx.fillText("🥪 Tycoon", DESIGN_W / 2, 38);
      ctx.textAlign = "right";  ctx.fillText("😊 " + happyCount + "   😡 " + grumpyCount, DESIGN_W - 18, 38);
    }

    function drawShop() {
      const awningY = 70, awningH = 50;
      for (let i = 0; i < 10; i++) {
        ctx.fillStyle = i % 2 === 0 ? "#e53935" : "#fff8dc";
        ctx.fillRect(i * (DESIGN_W / 10), awningY, DESIGN_W / 10, awningH);
      }
      ctx.fillStyle = "#8b5a2b"; ctx.fillRect(0, awningY + awningH, DESIGN_W, 6);
      ctx.fillStyle = "#fff1d6"; ctx.fillRect(20, 130, DESIGN_W - 40, 370);
      ctx.strokeStyle = "#8b5a2b"; ctx.lineWidth = 6;
      ctx.strokeRect(20, 130, DESIGN_W - 40, 370);
      ctx.fillStyle = "#8b5a2b"; ctx.fillRect(60, 140, DESIGN_W - 120, 50);
      ctx.fillStyle = "#fff8dc";
      ctx.font = "bold 26px -apple-system, Arial, sans-serif";
      ctx.textAlign = "center";
      ctx.fillText("LIVY'S SANDWICHES", DESIGN_W / 2, 174);
      ctx.fillStyle = "#cde9ff"; ctx.fillRect(50, 210, DESIGN_W - 100, 220);
      ctx.strokeStyle = "#8b5a2b"; ctx.lineWidth = 5;
      ctx.strokeRect(50, 210, DESIGN_W - 100, 220);
      ctx.beginPath();
      ctx.moveTo(DESIGN_W / 2, 210); ctx.lineTo(DESIGN_W / 2, 430);
      ctx.moveTo(50, 320); ctx.lineTo(DESIGN_W - 50, 320);
      ctx.stroke();
      ctx.fillStyle = "#a06a3c"; ctx.fillRect(20, 440, DESIGN_W - 40, 60);
      ctx.fillStyle = "#7a4a25"; ctx.fillRect(20, 490, DESIGN_W - 40, 10);
      drawTinySandwich(110, 470);
      drawTinySandwich(250, 470);
      drawTinySandwich(390, 470);
    }

    function drawTinySandwich(cx, cy) {
      ctx.fillStyle = "#d9a066"; ctx.fillRect(cx - 22, cy + 6, 44, 8);
      ctx.fillStyle = "#66bb6a"; ctx.fillRect(cx - 22, cy + 2, 44, 4);
      ctx.fillStyle = "#e53935"; ctx.fillRect(cx - 22, cy - 2, 44, 4);
      ctx.fillStyle = "#ffd54f"; ctx.fillRect(cx - 22, cy - 6, 44, 4);
      ctx.fillStyle = "#d9a066";
      ctx.beginPath();
      ctx.moveTo(cx - 22, cy - 6);
      ctx.quadraticCurveTo(cx, cy - 22, cx + 22, cy - 6);
      ctx.closePath(); ctx.fill();
    }

    function drawChef() {
      const cx = DESIGN_W / 2;
      const cy = 340 - chefBounce * 14;
      ctx.save();
      ctx.translate(cx, cy); ctx.scale(0.55, 0.55);
      drawSandwichBig(0, 0); drawFace(0, 0);
      ctx.restore();
      drawChefHat(cx, cy - 55);
      ctx.strokeStyle = "#3a1d0a"; ctx.lineWidth = 4; ctx.lineCap = "round";
      const wave = Math.sin(performance.now() / 300) * 6;
      ctx.beginPath();
      ctx.moveTo(cx - 60, cy + 10); ctx.lineTo(cx - 85, cy + 30 + wave);
      ctx.moveTo(cx + 60, cy + 10); ctx.lineTo(cx + 85, cy + 30 - wave);
      ctx.stroke();
    }

    function drawSandwichBig(cx, cy) {
      ctx.fillStyle = "rgba(0,0,0,0.15)";
      ctx.beginPath(); ctx.ellipse(cx, cy + 140, 200, 18, 0, 0, Math.PI * 2); ctx.fill();
      ctx.fillStyle = "#d9a066"; ctx.fillRect(cx - 180, cy + 70, 360, 50);
      ctx.fillStyle = "#b8843f"; ctx.fillRect(cx - 180, cy + 115, 360, 8);
      ctx.fillStyle = "#e57373"; ctx.fillRect(cx - 190, cy + 50, 380, 25);
      ctx.fillStyle = "#ffd54f";
      ctx.beginPath();
      ctx.moveTo(cx - 195, cy + 35); ctx.lineTo(cx + 195, cy + 35);
      ctx.lineTo(cx + 180, cy + 55); ctx.lineTo(cx - 180, cy + 55);
      ctx.closePath(); ctx.fill();
      ctx.fillStyle = "#e53935";
      ctx.beginPath();
      ctx.arc(cx - 110, cy + 25, 26, 0, Math.PI * 2);
      ctx.arc(cx, cy + 25, 26, 0, Math.PI * 2);
      ctx.arc(cx + 110, cy + 25, 26, 0, Math.PI * 2);
      ctx.fill();
      ctx.fillStyle = "#66bb6a";
      ctx.beginPath();
      ctx.moveTo(cx - 190, cy + 10);
      for (let x = -190; x <= 190; x += 30) {
        ctx.quadraticCurveTo(cx + x + 15, cy - 15, cx + x + 30, cy + 10);
      }
      ctx.lineTo(cx + 190, cy + 25); ctx.lineTo(cx - 190, cy + 25);
      ctx.closePath(); ctx.fill();
      ctx.fillStyle = "#d9a066";
      ctx.beginPath();
      ctx.moveTo(cx - 180, cy + 10);
      ctx.quadraticCurveTo(cx, cy - 100, cx + 180, cy + 10);
      ctx.closePath(); ctx.fill();
      ctx.fillStyle = "#fff8dc";
      const seeds = [[-110,-35],[-55,-58],[0,-68],[55,-58],[110,-35],[-80,-22],[25,-45],[80,-22]];
      for (const [dx, dy] of seeds) {
        ctx.beginPath();
        ctx.ellipse(cx + dx, cy + dy, 4.5, 2.8, 0.3, 0, Math.PI * 2);
        ctx.fill();
      }
    }

    function drawFace(cx, cy) {
      ctx.fillStyle = "#222";
      ctx.beginPath();
      ctx.arc(cx - 45, cy - 45, 7, 0, Math.PI * 2);
      ctx.arc(cx + 45, cy - 45, 7, 0, Math.PI * 2);
      ctx.fill();
      ctx.strokeStyle = "#3a1d0a"; ctx.lineWidth = 4; ctx.lineCap = "round";
      ctx.beginPath();
      ctx.arc(cx, cy - 25, 24, 0.15 * Math.PI, 0.85 * Math.PI);
      ctx.stroke();
    }

    function drawChefHat(cx, cy) {
      ctx.fillStyle = "#ffffff"; ctx.fillRect(cx - 40, cy - 6, 80, 14);
      ctx.beginPath();
      ctx.arc(cx - 30, cy - 18, 26, 0, Math.PI * 2);
      ctx.arc(cx + 30, cy - 18, 26, 0, Math.PI * 2);
      ctx.arc(cx, cy - 32, 30, 0, Math.PI * 2);
      ctx.fill();
      ctx.strokeStyle = "#dcdcdc"; ctx.lineWidth = 2;
      ctx.beginPath(); ctx.arc(cx - 30, cy - 18, 26, Math.PI, Math.PI * 2); ctx.stroke();
      ctx.beginPath(); ctx.arc(cx + 30, cy - 18, 26, Math.PI, Math.PI * 2); ctx.stroke();
      ctx.beginPath(); ctx.arc(cx, cy - 32, 30, Math.PI, Math.PI * 2); ctx.stroke();
    }

    function drawCustomer(c) {
      const bob = Math.sin(c.bob) * 3;
      ctx.fillStyle = "rgba(0,0,0,0.18)";
      ctx.beginPath();
      ctx.ellipse(c.x, c.y + 40, 22, 5, 0, 0, Math.PI * 2);
      ctx.fill();
      ctx.font = "56px -apple-system, Arial, sans-serif";
      ctx.textAlign = "center"; ctx.textBaseline = "middle";
      ctx.fillText(c.emoji, c.x, c.y + bob);

      if (c.state === "waiting") {
        const bx = c.x, by = c.y - 60 + bob;
        ctx.fillStyle = "#fff"; ctx.strokeStyle = "#333"; ctx.lineWidth = 2;
        ctx.beginPath();
        ctx.ellipse(bx, by, 30, 20, 0, 0, Math.PI * 2);
        ctx.fill(); ctx.stroke();
        ctx.beginPath();
        ctx.moveTo(bx - 6, by + 15);
        ctx.lineTo(bx, by + 28);
        ctx.lineTo(bx + 8, by + 15);
        ctx.closePath();
        ctx.fillStyle = "#fff"; ctx.fill(); ctx.stroke();
        ctx.font = "22px -apple-system, Arial, sans-serif";
        ctx.fillStyle = "#222";
        ctx.fillText("🥪?", bx, by);
        const w = 60;
        const frac = Math.max(0, c.wait / c.maxWait);
        ctx.fillStyle = "#444"; ctx.fillRect(c.x - w / 2 - 2, c.y + 32, w + 4, 10);
        ctx.fillStyle = frac > 0.5 ? "#66bb6a" : frac > 0.25 ? "#ffb300" : "#e53935";
        ctx.fillRect(c.x - w / 2, c.y + 34, w * frac, 6);
      } else if (c.state === "leavingHappy") {
        ctx.font = "30px -apple-system, Arial, sans-serif";
        ctx.fillText("🥪", c.x, c.y - 50);
        ctx.font = "bold 22px -apple-system, Arial, sans-serif";
        ctx.fillStyle = "#2e7d32";
        ctx.fillText("+$" + c.pay, c.x + 30, c.y - 70);
      } else if (c.state === "leavingGrumpy") {
        ctx.font = "30px -apple-system, Arial, sans-serif";
        ctx.fillText("😡", c.x, c.y - 50);
      }
    }

    canvas.addEventListener("pointerdown", e => {
      const p = screenToDesign(e.clientX, e.clientY);
      let best = null, bestDist = 9999;
      for (const c of customers) {
        if (c.state !== "waiting") continue;
        const dx = p.x - c.x, dy = p.y - c.y;
        const d = Math.hypot(dx, dy);
        if (d < 60 && d < bestDist) { best = c; bestDist = d; }
      }
      if (best) {
        money += best.pay; happyCount += 1;
        best.state = "leavingHappy";
        SPOTS[best.spotIndex].taken = false;
        chefBounce = 1;
      }
    });

    window.addEventListener("resize", resize);
    window.addEventListener("orientationchange", resize);
    document.addEventListener("gesturestart", e => e.preventDefault());
    document.addEventListener("dblclick", e => e.preventDefault());
    canvas.addEventListener("touchmove", e => e.preventDefault(), { passive: false });

    resize();
    requestAnimationFrame(tick);
  </script>
</body>
</html>
```
