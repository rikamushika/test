/index.html
<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <title>みんなの願い事</title>
  <style>
    body {
      background: #f0f8ff;
      font-family: sans-serif;
      margin: 0;
      padding: 10px;
    }
    #gallery {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
      justify-content: center;
    }
    .tanzaku {
      width: 120px;
      height: 240px;
      background: linear-gradient(to bottom, #ffcccc, #ffe6e6);
      border: 2px solid #f66;
      border-radius: 10px;
      box-shadow: 2px 2px 6px rgba(0,0,0,0.2);
      padding: 10px;
      text-align: center;
      cursor: pointer;
      overflow: hidden;
      position: relative;
    }
    .tanzaku p {
      margin: 0;
      font-size: 14px;
      white-space: pre-wrap;
      word-wrap: break-word;
    }
    .modal {
      display: none;
      position: fixed;
      z-index: 999;
      top: 0; left: 0; right: 0; bottom: 0;
      background-color: rgba(0,0,0,0.6);
      justify-content: center;
      align-items: center;
    }
    .modal-content {
      background: white;
      padding: 20px;
      border-radius: 10px;
      width: 80%;
      max-width: 400px;
    }
  </style>
</head>
<body>

<h2>🌟 みんなの七夕の願い事 🌟</h2>
<div id="gallery"></div>

<div class="modal" id="modal">
  <div class="modal-content" id="modal-content"></div>
</div>

<script>
const layerUrl = "https://services5.arcgis.com/oISMFKlA90RsGqBJ/arcgis/rest/services/survey123_19ead63571264a7bbae5eb9c910287ec_results/FeatureServer/0/query?where=1%3D1&outFields=*&f=json";

fetch(layerUrl)
  .then(response => response.json())
  .then(data => {
    const container = document.getElementById("gallery");
    data.features.forEach(feature => {
      const wish = feature.attributes["願い事"] || "";
      const name = feature.attributes["ペンネーム"] || "匿名";

      const div = document.createElement("div");
      div.className = "tanzaku";
      div.innerHTML = `<p>${wish}</p><br><p>🖊 ${name}</p>`;
      div.addEventListener("click", () => {
        const modal = document.getElementById("modal");
        const modalContent = document.getElementById("modal-content");
        modalContent.innerHTML = `<h3>🎋 願い事</h3><p>${wish}</p><p>🖊 ${name}</p>`;
        modal.style.display = "flex";
      });
      container.appendChild(div);
    });
  });

document.getElementById("modal").addEventListener("click", () => {
  document.getElementById("modal").style.display = "none";
});
</script>

</body>
</html>
