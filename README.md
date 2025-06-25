/index.html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>短冊ビューア</title>
  <style>
    body {
      margin: 0;
      padding: 0;
      background: #f0f8ff;
      font-family: sans-serif;
      background-image: url('https://iccarcgis.maps.arcgis.com/sharing/rest/content/items/eb1fe275b5e84701b7f804c8d8a0704b/data');
      background-size: cover;
      background-position: center;
      background-repeat: no-repeat;
    }

    #tanzaku-container {
      position: relative;
      width: 100%;
      height: 100vh;
      overflow: hidden;
    }

    @keyframes sway {
      0% { transform: rotate(-3deg); }
      50% { transform: rotate(3deg); }
      100% { transform: rotate(-3deg); }
    }

    .tanzaku {
      position: absolute;
      width: 130px;
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
      top: 15%;
      left: 50%;
      transform: translateX(-50%);
      width: auto;
      height: 73%;
      max-height: 80%;
      padding: 3px;
      color: black;
      font-size: 14px;
      background-color: transparent;
      writing-mode: vertical-rl;
      text-align: center;
      overflow: hidden;
      word-break: break-word;
      flex-direction: column;
      font-size: 13px;
      line-height: 1.2;
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
    }

    #popup-content {
      background-color: white;
      padding: 20px;
      border-radius: 10px;
      max-width: 400px;
    }

    #age-buttons {
      position: fixed;
      bottom: 20px;
      left: 50%;
      transform: translateX(-50%);
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      background: rgba(255, 255, 255, 0.8);
      padding: 10px;
      border-radius: 10px;
      z-index: 999;
      gap: 10px;
      max-width: 800px;
    }

    #age-buttons button {
      font-size: 14px;
      padding: 5px 10px;
      cursor: pointer;
      border: 1px solid #888;
      border-radius: 5px;
      background-color: #ffffff;
    }

    #age-buttons button:hover {
      background-color: #d0eaff;
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

<div id="tanzaku-container"></div>

<div id="popup" onclick="popup.style.display = 'none';">
  <div id="popup-content" onclick="event.stopPropagation();"></div>
</div>

<script>
const container = document.getElementById("tanzaku-container");
const popup = document.getElementById("popup");
const popupContent = document.getElementById("popup-content");
const imgURL = "https://iccarcgis.maps.arcgis.com/sharing/rest/content/items/63b21d4621df4c6eb4ffc05ecad72bd6/data";

const ageField = "field_4";
let currentWhere = "1=1";

function fetchAndDisplay() {
  const url = https://services5.arcgis.com/oISMFKlA90RsGqBJ/arcgis/rest/services/survey123_428ee531d6f64629ad1fb10db7e23778_results/FeatureServer/0/query?where=${encodeURIComponent(currentWhere)}&outFields=field_1,field_2,${ageField}&orderByFields=objectid desc&resultRecordCount=16&f=json;

  fetch(url)
    .then(res => res.json())
    .then(data => {
      const features = data.features || [];
      displayTanzaku(features);
    });
}

function displayTanzaku(features) {
  container.innerHTML = "";
  const spacingX = 150;
  const startY = 100;
  const secondRowY = 350;

  features.forEach((feature, index) => {
    const attr = feature.attributes;
    const penname = attr["field_1"] || "";
    const wish = attr["field_2"] || "";
    const div = document.createElement("div");
    div.className = "tanzaku";

    const x = 50 + (index % 8) * spacingX;
    const y = index < 8 ? startY : secondRowY;

    div.style.left = ${x}px;
    div.style.top = ${y}px;

    div.innerHTML = <img src="${imgURL}" alt="短冊">
                     <div class="text">${wish}<br><br>★${penname}</div>;

    div.onclick = () => {
      popupContent.innerHTML = <p><strong>願い事</strong>：${wish}</p>
                                <p><strong>ペンネーム</strong>：${penname}</p>;
      popup.style.display = "flex";
    };

    container.appendChild(div);
  });
}

function filterByAge(ageGroup) {
  currentWhere = ageGroup ? field_4='${ageGroup}' : "1=1";
  fetchAndDisplay();
}

fetchAndDisplay();
</script>
</body>
</html>
