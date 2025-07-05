/index.html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>短冊ビューア アプリ版</title>
  <style>
    body {
      margin: 0;
      padding: 0;
      background-color: #002244;
      font-family: "Noto Sans JP", "Yu Gothic", sans-serif;
      background-image: url('https://iccarcgis.maps.arcgis.com/sharing/rest/content/items/eb1fe275b5e84701b7f804c8d8a0704b/data');
      background-size: cover;
      background-position: center;
      background-repeat: no-repeat;
      overflow-x: hidden;
    }

    #tanzaku-container {
      position: relative;
      width: 100%;
      min-height: 100vh;
      overflow-y: auto;
      padding-bottom: 200px;
    }

    @keyframes sway {
      0% { transform: rotate(-3deg); }
      50% { transform: rotate(3deg); }
      100% { transform: rotate(-3deg); }
    }

    .tanzaku {
      position: absolute;
      width: 100px;
      text-align: center;
      cursor: pointer;
      animation: sway 3s infinite ease-in-out;
      transform-origin: top center;
    }

    .tanzaku img {
      width: 100%;
    }

    .text {
      position: absolute;
      top: 20%;
      left: 20%;
      transform: translate(-50%, 0%);
      width: 80%;
      max-height: 70%;
      padding: 2px;
      color: black;
      font-size: 10px;
      background-color: transparent;
      writing-mode: vertical-rl;
      text-align: center;
      overflow: hidden;
      word-break: break-word;
      line-height: 1.2;
      box-sizing: border-box;
    }

    #popup {
      display: none;
      position: fixed;
      top: 0;
      left: 0;
      width: 100vw;
      height: 100vh;
      background-color: rgba(0, 0, 0, 0.5);
      justify-content: center;
      align-items: center;
      z-index: 1000;
    }

    #popup-content {
      padding: 20px;
      border-radius: 10px;
      max-width: 90vw;
      font-size: 18px;
      font-weight: bold;
      line-height: 1.4;
    }

    #age-buttons {
      position: fixed;
      top: 10px;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      background: rgba(255, 255, 255, 0.9);
      padding: 10px;
      border-radius: 10px;
      z-index: 999;
      gap: 5px;
      width: 95vw;
    }

    #age-buttons button {
      font-size: 12px;
      padding: 6px 8px;
      border: 1px solid #888;
      border-radius: 5px;
      background-color: #ffffff;
      flex-grow: 1;
    }

    #age-buttons button:hover {
      background-color: #d0eaff;
    }

    #count-display {
      position: fixed;
      top: 110px;
      left: 50%;
      transform: translateX(-50%);
      background: #fff8c6;
      color: #333;
      padding: 8px 16px;
      border-radius: 12px;
      font-size: 16px;
      font-weight: bold;
      border: 2px solid #ffd700;
      box-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
      z-index: 998;
    }

    .shooting-star {
      position: fixed;
      top: -50px;
      font-size: 30px;
      color: gold;
      z-index: 2000;
      animation: starFall 3s ease-out forwards;
      pointer-events: none;
    }

    @keyframes starFall {
      0% { top: -50px; opacity: 1; }
      100% { top: 100%; opacity: 0; }
    }
  </style>
</head>
<body>

<div id="age-buttons">
  <button onclick="filterByAge('10代以下')">10代以下</button>
  <button onclick="filterByAge('20代')">20代</button>
  <button onclick="filterByAge('30代')">30代</button>
  <button onclick="filterByAge('40代')">40代</button>
  <button onclick="filterByAge('50代')">50代</button>
  <button onclick="filterByAge('60代')">60代</button>
  <button onclick="filterByAge('70代')">70代</button>
  <button onclick="filterByAge('80代以上')">80代以上</button>
  <button onclick="filterByAge()">全て表示</button>
</div>

<div id="count-display">願いの数：0 枚</div>

<div id="tanzaku-container"></div>

<div id="popup" onclick="this.style.display = 'none';">
  <div id="popup-content"></div>
</div>

<script>
const container = document.getElementById("tanzaku-container");
const popup = document.getElementById("popup");
const popupContent = document.getElementById("popup-content");
const countDisplay = document.getElementById("count-display");
const imgURL = "https://iccarcgis.maps.arcgis.com/sharing/rest/content/items/63b21d4621df4c6eb4ffc05ecad72bd6/data";

const confirmField = "確認";
const ageField = "field_4";
let currentAgeWhere = "";

function buildWhereClause() {
  return `${confirmField}='済'` + (currentAgeWhere ? ` AND ${ageField}='${currentAgeWhere}'` : "");
}

function fetchAndDisplay() {
  const whereClause = buildWhereClause();
  const url = `https://services5.arcgis.com/oISMFKlA90RsGqBJ/arcgis/rest/services/survey123_428ee531d6f64629ad1fb10db7e23778_results/FeatureServer/0/query?where=${encodeURIComponent(whereClause)}&outFields=field_1,field_2,${ageField},${confirmField}&orderByFields=objectid desc&f=json`;

  fetch(url)
    .then(res => res.json())
    .then(data => {
      const features = data.features || [];
      displayTanzaku(features);
    });
}

function filterByAge(ageGroup) {
  currentAgeWhere = ageGroup || "";
  fetchAndDisplay();
}

function createShootingStar() {
  const star = document.createElement("div");
  star.className = "shooting-star";
  star.textContent = "★";
  star.style.left = `${Math.random() * 80 + 10}vw`;
  document.body.appendChild(star);
  setTimeout(() => star.remove(), 3000);
}

function displayTanzaku(features) {
  container.innerHTML = "";
  countDisplay.textContent = `願いの数：${features.length} 枚`;

  const spacingX = 120;
  const spacingY = 250;
  const columns = 3;

  features.forEach((feature, index) => {
    const attr = feature.attributes;
    const penname = attr["field_1"] || "";
    const wish = attr["field_2"] || "";
    const div = document.createElement("div");
    div.className = "tanzaku";

    const col = index % columns;
    const row = Math.floor(index / columns);
    const x = 10 + col * spacingX;
    const y = 160 + row * spacingY;

    div.style.left = `${x}px`;
    div.style.top = `${y}px`;
    div.style.animationDelay = `${Math.random() * 3}s`;

    div.innerHTML = `<img src="${imgURL}" alt="短冊">
                     <div class="text">${wish}<br>★${penname}</div>`;

    div.onclick = () => {
      for (let i = 0; i < 5; i++) {
        setTimeout(createShootingStar, i * 400);
      }

      const isOldGalaxy = /Android\s([1-9]|10)/.test(navigator.userAgent) && /SM-/.test(navigator.userAgent);

      if (isOldGalaxy) {
        popupContent.innerHTML = `
          <div style="
            width: 90vw;
            max-width: 400px;
            background-color: #fff8dc;
            border: 4px solid #ff69b4;
            padding: 20px;
            border-radius: 12px;
            text-align: center;
            font-size: 18px;
            font-weight: bold;
            font-family: 'Noto Sans JP', 'Yu Gothic', sans-serif;
            line-height: 1.8;
            box-sizing: border-box;
          ">
            ${wish}<br><br>★${penname}
          </div>`;
      } else {
        popupContent.innerHTML = `
          <div style="position: relative; width: 100%; max-width: 400px; margin: 0 auto;">
           <img src="${imgURL}" alt="短冊" style="width: 100%; background-color: transparent;">

            <div style="
              position: absolute;
              top: 18%;
              left: 50%;
              transform: translateX(-50%);
              width: 50%;
              max-height: 70%;
              padding: 5px;
              color: black;
              font-size: 18px;
              font-weight: bold;
              background-color: transparent;
              writing-mode: vertical-rl;
              text-orientation: upright;
              line-height: 2.4;
              font-family: 'Noto Sans JP', 'Yu Gothic', sans-serif;
              text-align: center;
              overflow: hidden;
              word-break: break-word;
              box-sizing: border-box;
            ">
              ${wish}<br><br>★${penname}
            </div>
          </div>`;
      }

      popup.style.display = "flex";
    };

    container.appendChild(div);
  });
}

fetchAndDisplay();
</script>
</body>
</html>

