<!DOCTYPE html>
<html lang="zh-HK">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>眼睛調焦機制模型（睫狀體與懸韌帶）</title>
  <style>
    :root {
      --bg-color: #f4f7f6;
      --card-bg: #ffffff;
      --primary: #2b5c8f;
      --accent: #e76f51;
      --text: #2d3748;
    }

    body {
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
      background-color: var(--bg-color);
      color: var(--text);
      margin: 0;
      padding: 20px;
      display: flex;
      justify-content: center;
    }

    .container {
      max-width: 900px;
      width: 100%;
      background: var(--card-bg);
      padding: 25px;
      border-radius: 12px;
      box-shadow: 0 4px 20px rgba(0,0,0,0.08);
    }

    h1 {
      text-align: center;
      color: var(--primary);
      margin-bottom: 8px;
      font-size: 1.5rem;
    }

    p.subtitle {
      text-align: center;
      color: #666;
      font-size: 0.95rem;
      margin-top: 0;
      margin-bottom: 20px;
    }

    .simulation-area {
      display: flex;
      flex-direction: column;
      align-items: center;
      background: #fafafa;
      border: 1px solid #e1e8ed;
      border-radius: 8px;
      padding: 15px;
      position: relative;
    }

    svg {
      width: 100%;
      max-width: 500px;
      height: auto;
      overflow: visible;
    }

    .controls {
      margin-top: 20px;
      width: 100%;
      max-width: 500px;
      display: flex;
      flex-direction: column;
      gap: 12px;
    }

    .btn-group {
      display: flex;
      gap: 10px;
    }

    button {
      flex: 1;
      padding: 10px 15px;
      border: 1px solid var(--primary);
      background: #fff;
      color: var(--primary);
      font-weight: bold;
      border-radius: 6px;
      cursor: pointer;
      transition: all 0.2s ease;
    }

    button.active, button:hover {
      background: var(--primary);
      color: #fff;
    }

    .slider-container {
      display: flex;
      align-items: center;
      gap: 10px;
    }

    input[type="range"] {
      flex: 1;
      accent-color: var(--primary);
    }

    .status-board {
      margin-top: 25px;
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
      gap: 15px;
    }

    .status-card {
      background: #f8fafc;
      border-left: 4px solid var(--primary);
      padding: 12px 15px;
      border-radius: 0 6px 6px 0;
    }

    .status-card h3 {
      margin: 0 0 5px 0;
      font-size: 0.9rem;
      color: #64748b;
    }

    .status-card p {
      margin: 0;
      font-size: 1.05rem;
      font-weight: bold;
      color: var(--text);
    }

    .misconception-box {
      margin-top: 20px;
      background: #fff3ed;
      border: 1px solid #ffd8c2;
      border-radius: 6px;
      padding: 12px 15px;
      font-size: 0.9rem;
      color: #9a3412;
    }

    .misconception-box strong {
      display: block;
      margin-bottom: 3px;
    }
  </style>
</head>
<body>

<div class="container">
  <h1>眼球「睫狀體—懸韌帶—晶狀體」調焦模擬器</h1>
  <p class="subtitle">拉動滑桿或切換模式，觀察橡筋模擬實驗的物理變化</p>

  <div class="simulation-area">
    <svg viewBox="0 0 400 400">
      <!-- 網膜焦點提示線 -->
      <line x1="370" y1="50" x2="370" y2="350" stroke="#cbd5e1" stroke-width="2" stroke-dasharray="4" />
      <text x="370" y="40" font-size="10" fill="#94a3b8" text-anchor="middle">視網膜位置</text>

      <!-- 光線模擬 -->
      <path id="light-ray-top" stroke="#f59e0b" stroke-width="1.5" fill="none" opacity="0.6" />
      <path id="light-ray-bottom" stroke="#f59e0b" stroke-width="1.5" fill="none" opacity="0.6" />

      <!-- 睫狀體（外環/刺繡圈） -->
      <circle id="ciliary-body" cx="200" cy="200" r="140" fill="none" stroke="#2b5c8f" stroke-width="12" stroke-dasharray="8 4" />
      <text id="label-ciliary" x="200" y="45" font-size="12" fill="#2b5c8f" font-weight="bold" text-anchor="middle">睫狀體（可變直徑外環）</text>

      <!-- 懸韌帶（橡筋群） -->
      <g id="ligaments"></g>

      <!-- 晶狀體（水氣球） -->
      <ellipse id="lens" cx="200" cy="200" rx="45" ry="30" fill="rgba(56, 189, 248, 0.4)" stroke="#0284c7" stroke-width="3" />
      <text x="200" y="204" font-size="12" fill="#0369a1" font-weight="bold" text-anchor="middle">晶狀體</text>
    </svg>

    <div class="controls">
      <div class="btn-group">
        <button id="btn-far" onclick="setPreset('far')">看遠物（遠焦）</button>
        <button id="btn-near" onclick="setPreset('near')">看近物（近焦）</button>
      </div>

      <div class="slider-container">
        <span style="font-size:0.85rem;">看近 (肌肉收縮)</span>
        <input type="range" id="state-slider" min="0" max="100" value="0" oninput="updateSimulation(this.value)">
        <span style="font-size:0.85rem;">看遠 (肌肉放鬆)</span>
      </div>
    </div>
  </div>

  <div class="status-board">
    <div class="status-card">
      <h3>視物目標</h3>
      <p id="status-target">看遠物 (Far)</p>
    </div>
    <div class="status-card">
      <h3>睫狀肌狀態 (Ciliary Muscle)</h3>
      <p id="status-ciliary">舒張 / 放鬆（外環直徑變大）</p>
    </div>
    <div class="status-card">
      <h3>懸韌帶狀態 (橡筋)</h3>
      <p id="status-ligament">被拉緊 (Tense)</p>
    </div>
    <div class="status-card">
      <h3>晶狀體形態 (水氣球)</h3>
      <p id="status-lens">拉平 / 變薄（焦距變長）</p>
    </div>
  </div>

  <div class="misconception-box">
    <strong>💡 關鍵物理概念直覺破除：</strong>
    當看近物時，眼睛睫狀肌其實是呈<b>「收縮」</b>狀態，使得環狀直徑變小；直徑縮小會釋放懸韌帶（橡筋）的拉力，晶狀體才得以靠自身的彈性<b>「變厚/變凸」</b>。
  </div>
</div>

<script>
  const numLigaments = 8;
  const ligamentsGroup = document.getElementById('ligaments');
  const slider = document.getElementById('state-slider');
  const ciliaryBody = document.getElementById('ciliary-body');
  const lens = document.getElementById('lens');
  const lightRayTop = document.getElementById('light-ray-top');
  const lightRayBottom = document.getElementById('light-ray-bottom');

  // 初始化懸韌帶 DOM
  const ligamentElements = [];
  for (let i = 0; i < numLigaments; i++) {
    const path = document.createElementNS("http://www.w3.org/2000/svg", "path");
    path.setAttribute("stroke", "#e76f51");
    path.setAttribute("stroke-width", "2.5");
    path.setAttribute("fill", "none");
    ligamentsGroup.appendChild(path);
    ligamentElements.push(path);
  }

  function updateSimulation(val) {
    // val: 0 (Near/近) -> 100 (Far/遠)
    const factor = val / 100; // 0 to 1

    // 1. 睫狀體半徑 (看遠時環變大，看近時環變小)
    const ringRadius = 110 + factor * 30; // 110 (Near) ~ 140 (Far)
    ciliaryBody.setAttribute("r", ringRadius);

    // 2. 晶狀體形狀 (看遠時扁平，看近時圓凸)
    const rx = 38 + factor * 12; // 38 (Near) ~ 50 (Far)
    const ry = 42 - factor * 18; // 42 (Near) ~ 24 (Far)
    lens.setAttribute("rx", rx);
    lens.setAttribute("ry", ry);

    // 3. 更新懸韌帶 (橡筋) 繪製
    for (let i = 0; i < numLigaments; i++) {
      const angle = (i * 2 * Math.PI) / numLigaments;
      
      // 晶狀體邊緣點
      const lx = 200 + rx * Math.cos(angle);
      const ly = 200 + ry * Math.sin(angle);
      
      // 睫狀體邊緣點
      const cx = 200 + ringRadius * Math.cos(angle);
      const cy = 200 + ringRadius * Math.sin(angle);

      if (factor > 0.3) {
        // 拉緊狀態：直線
        ligamentElements[i].setAttribute("d", `M ${lx} ${ly} L ${cx} ${cy}`);
        ligamentElements[i].setAttribute("stroke-dasharray", "none");
      } else {
        // 鬆弛狀態：微彎曲或虛線感 (模擬鬆弛的橡筋)
        const midX = (lx + cx) / 2 + (1 - factor) * 4 * Math.sin(angle);
        const midY = (ly + cy) / 2 + (1 - factor) * 4 * Math.cos(angle);
        ligamentElements[i].setAttribute("d", `M ${lx} ${ly} Q ${midX} ${midY} ${cx} ${cy}`);
      }
    }

    // 4. 光線折射示意
    const lightTargetX = 370;
    lightRayTop.setAttribute("d", `M 30 160 L ${200 - rx} 160 L 200 160 L ${lightTargetX} 200`);
    lightRayBottom.setAttribute("d", `M 30 240 L ${200 - rx} 240 L 200 240 L ${lightTargetX} 200`);

    // 5. 更新狀態文字與 UI
    updateLabels(factor);
  }

  function updateLabels(factor) {
    const statusTarget = document.getElementById('status-target');
    const statusCiliary = document.getElementById('status-ciliary');
    const statusLigament = document.getElementById('status-ligament');
    const statusLens = document.getElementById('status-lens');
    const btnFar = document.getElementById('btn-far');
    const btnNear = document.getElementById('btn-near');

    if (factor > 0.5) {
      statusTarget.innerText = "看遠物 (Far)";
      statusCiliary.innerText = "舒張 / 放鬆（外環直徑變大）";
      statusLigament.innerText = "被拉緊 (Tense)";
      statusLens.innerText = "被拉平 / 變薄（折射力低，焦距長）";
      btnFar.classList.add('active');
      btnNear.classList.remove('active');
    } else {
      statusTarget.innerText = "看近物 (Near)";
      statusCiliary.innerText = "收縮（外環直徑變小）";
      statusLigament.innerText = "鬆弛 (Slack)";
      statusLens.innerText = "靠彈性復原 / 變厚凸起（折射力高，焦距短）";
      btnNear.classList.add('active');
      btnFar.classList.remove('active');
    }
  }

  function setPreset(mode) {
    if (mode === 'far') {
      slider.value = 100;
    } else {
      slider.value = 0;
    }
    updateSimulation(slider.value);
  }

  // 初始載入看遠模式
  setPreset('far');
</script>

</body>
</html>
