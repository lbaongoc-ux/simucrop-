<!DOCTYPE html>

<html lang="vi">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Nông Trại Thông Minh</title>
<style>
:root {
  --green:#217346; --light:#edf7ee; --text:#20372a;
}
\* { box-sizing:border-box; }
body {
  margin:0; background:var(--light); color:var(--text);
  font-family:Arial,sans-serif;
}
header {
  padding:25px 16px; text-align:center;
  color:white; background:linear-gradient(120deg,#155d36,#40a15b);
}
header h1 { margin:0 0 8px; }
main { max-width:1000px; margin:auto; padding:16px; }
.grid {
  display:grid; grid-template-columns:repeat(auto-fit,minmax(230px,1fr));
  gap:14px;
}
.card {
  background:white; border-radius:15px; padding:18px;
  margin-bottom:15px; box-shadow:0 3px 12px #183c2010;
  min-width:0;
}
h2 { font-size:19px; margin:0 0 15px; }
label { display:block; margin-top:10px; font-size:14px; }
button,select,input {
  width:100%; padding:12px; border-radius:9px;
  border:1px solid #d1dfd3; margin:6px 0; font-size:15px;
}
button {
  background:var(--green); color:white; border:0;
  cursor:pointer; font-weight:bold;
}
button.secondary { background:#e4efe5; color:#205c37; }
button:disabled { opacity:.6; cursor:wait; }
.stat { font-size:28px; color:var(--green); font-weight:bold; }
.muted { font-size:13px; color:#657568; line-height:1.5; }
.plant {
  background:#f1f8ed; text-align:center; font-size:75px;
  padding:15px; border-radius:12px;
}
.bar { height:10px; background:#e5eae4; border-radius:9px; overflow:hidden; }
.fill { height:100%; background:#45a45e; transition:width .3s; }
.weather-icon { font-size:32px; }
.note {
  padding:13px; background:#fff6d9; border-radius:10px; line-height:1.6;
  overflow-wrap:anywhere;
}
.note.good { background:#e8f6e9; }
.note.warn { background:#fff0df; }
.metrics { display:grid; grid-template-columns:1fr 1fr; gap:10px; }
.metric { background:#f3f8f1; padding:12px; border-radius:10px; min-width:0; }
.metric b { display:block; font-size:21px; margin-top:6px; }
ul { padding-left:22px; }
li { margin:9px 0; overflow-wrap:anywhere; }
footer { text-align:center; padding:20px; }
</style>
</head>
<body>
<header>
  <h1>🌱 Nông Trại Thông Minh</h1>
  <div>Mô phỏng cây trồng và theo dõi thời tiết</div>
</header>

<main>
  <section class="card">
    <h2>📍 Vị trí trang trại</h2>
    <label for="lat">Vĩ độ</label>
    <input id="lat" type="number" step="any" value="21.45">
    <label for="lon">Kinh độ</label>
    <input id="lon" type="number" step="any" value="105.22">
    <div class="grid">
      <button onclick="useMyLocation()">📍 Lấy vị trí thiết bị</button>
      <button onclick="loadWeather()">🌦️ Cập nhật thời tiết</button>
    </div>
    <p id="weatherStatus" class="muted" aria-live="polite">
      Nhấn cập nhật để tải dự báo thời tiết.
    </p>
    <p id="weatherUpdated" class="muted"></p>
  </section>

<div class="grid">
    <section class="card">
      <h2>🌦️ Dự báo thời tiết</h2>
      <div class="weather-icon" id="weatherIcon">🌤️</div>
      <p id="weatherDescription">Chưa có dữ liệu</p>
      <div class="metrics">
        <div class="metric">🌡️ Nhiệt độ<b id="temperature">-- °C</b></div>
        <div class="metric">💨 Độ ẩm không khí<b id="airHumidity">-- %</b></div>
        <div class="metric">🌧️ Lượng mưa giờ tới<b id="rainForecast">-- mm</b></div>
        <div class="metric">☔ Xác suất mưa<b id="rainChance">-- %</b></div>
      </div>
      <p class="muted">Dữ liệu thời tiết theo giờ tại tọa độ đã chọn.</p>
    </section>

&#x20;   <section class="card">
      <h2>🌱 Môi trường đất</h2>
      <p class="muted">Độ ẩm đất từ mô hình thời tiết, lớp 0–7 cm</p>
      <div class="stat" id="soilMoisture">--</div>
      <p id="soilDescription">Chưa tải dữ liệu đất</p>
      <div class="bar"><div id="soilBar" class="fill" style="width:0%"></div></div>
      <p>Nhiệt độ đất: <b id="soilTemperature">--</b></p>
      <hr>
      <p class="muted">Độ ẩm đất không phải độ ẩm không khí. Nếu API không có dữ liệu, app sẽ báo rõ thay vì tự tạo số đo.</p>
    </section>

</div>

<section class="card">
    <h2>🩺 Khuyến nghị theo thời tiết</h2>
    <div id="advice" class="note">Hãy cập nhật thời tiết để nhận khuyến nghị.</div>
  </section>

<div class="grid">
    <section class="card">
      <h2>🌾 Mô phỏng mùa vụ</h2>
      <label for="crop">Loại cây trồng</label>
      <select id="crop">
        <option value="cai">Rau cải</option>
        <option value="tomato">Cà chua</option>
        <option value="rice">Lúa</option>
      </select>
      <button onclick="newSeason()">🌱 Bắt đầu vụ mới</button>
      <div class="plant" id="plant">🌱</div>
      <p>Giai đoạn: <b id="stage">Gieo hạt</b></p>
      <p>Ngày mô phỏng: <b id="day">0</b></p>
      <p>Sức khỏe mô phỏng: <b id="health">100</b>/100</p>
      <div class="bar"><div id="healthBar" class="fill" style="width:100%"></div></div>
      <button onclick="nextDay()">⏩ Mô phỏng thêm một ngày</button>
    </section>

&#x20;   <section class="card">
      <h2>💧 Chăm sóc cây</h2>
      <label for="waterAmount">Lượng nước mô phỏng (đơn vị)</label>
      <input id="waterAmount" type="number" min="1" max="40" value="10">
      <button onclick="waterPlant()">Tưới nước</button>
      <label for="fertilizer">Lượng phân mô phỏng (đơn vị)</label>
      <input id="fertilizer" type="number" min="1" max="10" value="2">
      <button onclick="fertilizePlant()">Bón phân</button>
      <p>Độ ẩm mô phỏng riêng: <b id="simMoisture">55%</b></p>
      <p>Dinh dưỡng mô phỏng: <b id="nutrients">50</b></p>
      <p class="muted">Các thao tác tưới và bón ở đây chỉ tác động lên mô phỏng, không làm thay đổi dữ liệu thời tiết.</p>
    </section>

</div>

<section class="card">
    <h2>📒 Nhật ký mùa vụ</h2>
    <button class="secondary" onclick="exportLog()">📥 Tải nhật ký CSV</button>
    <button class="secondary" onclick="clearLog()">Xóa nhật ký</button>
    <ul id="log"></ul>
  </section>
</main>

<footer class="muted">
  Nông Trại Thông Minh · Dữ liệu thời tiết: Open-Meteo
</footer>

<script>
const $ = id => document.getElementById(id);

const crops = {
  cai: {
    name:"Rau cải", days:40, icons:\["🌱","🌿","🥬"],
    stages:\["Gieo hạt","Nảy mầm","Phát triển lá","Thu hoạch"]
  },
  tomato: {
    name:"Cà chua", days:90, icons:\["🌱","🌿","🌼","🍅"],
    stages:\["Gieo hạt","Cây con","Ra hoa","Đậu quả","Thu hoạch"]
  },
  rice: {
    name:"Lúa", days:110, icons:\["🌱","🌿","🌾"],
    stages:\["Gieo/cấy","Đẻ nhánh","Làm đòng","Trỗ bông","Chín"]
  }
};

const initialState = () => ({
  crop:"cai", day:0, health:100, moisture:55,
  nutrients:50, totalWater:0, log:\[]
});

let state;
try {
  state = JSON.parse(localStorage.getItem("farmState")) || initialState();
  if (!crops\[state.crop]) state = initialState();
} catch {
  state = initialState();
}

let weatherData = null;
let requestNumber = 0;

function save() {
  try {
    localStorage.setItem("farmState", JSON.stringify(state));
  } catch (e) {
    console.warn("Không thể lưu dữ liệu trình duyệt.", e);
  }
}

function record(message) {
  state.log.unshift(
    new Date().toLocaleString("vi-VN") + " — " +
    "Ngày mô phỏng " + state.day + ": " + message
  );
  state.log = state.log.slice(0,100);
}

function cropInfo() { return crops\[state.crop]; }

function stageIndex() {
  const c = cropInfo();
  return Math.min(c.stages.length - 1,
    Math.floor(state.day / c.days \* c.stages.length));
}

function newSeason() {
  state = {...initialState(), crop:$("crop").value};
  record("Bắt đầu vụ trồng " + cropInfo().name);
  render();
}

function checkedAmount(id, max) {
  const n = Number($(id).value);
  if (!Number.isFinite(n) || n < 1 || n > max) {
    alert("Vui lòng nhập số từ 1 đến " + max + ".");
    return null;
  }
  return n;
}

function waterPlant() {
  const n = checkedAmount("waterAmount",40);
  if (n === null) return;
  state.moisture = Math.min(100,state.moisture+n);
  state.totalWater = (state.totalWater || 0)+n;
  record("Tưới " + n + " đơn vị nước mô phỏng.");
  render();
}

function fertilizePlant() {
  const n = checkedAmount("fertilizer",10);
  if (n === null) return;
  state.nutrients = Math.min(100,state.nutrients+n\*5);
  record("Bón " + n + " đơn vị phân mô phỏng.");
  render();
}

function nextDay() {
  if (state.day >= cropInfo().days) {
    alert("Đã kết thúc chu kỳ mô phỏng. Hãy bắt đầu vụ mới.");
    return;
  }

  state.day++;
  state.moisture = Math.max(0,state.moisture-5);
  state.nutrients = Math.max(0,state.nutrients-0.5);

  if (state.moisture < 40) state.health -= 7;
  else if (state.moisture > 85) state.health -= 4;
  else state.health += 2;

  if (state.nutrients < 15) state.health -= 3;
  state.health = Math.max(0,Math.min(100,state.health));

  record("Cập nhật cây trồng; sức khỏe mô phỏng " + state.health + "/100.");
  render();
}

function render() {
  const c = cropInfo();
  $("crop").value = state.crop;
  $("day").textContent = state.day;
  $("stage").textContent = c.stages\[stageIndex()];
  $("health").textContent = state.health;
  $("healthBar").style.width = state.health + "%";
  $("plant").textContent = c.icons\[Math.min(
    c.icons.length-1,
    Math.floor(state.day/c.days\*c.icons.length)
  )];
  $("simMoisture").textContent = state.moisture + "%";
  $("nutrients").textContent = Math.round(state.nutrients);
  renderLog();
  save();
  renderAdvice();
}

function renderLog() {
  const list = $("log");
  list.replaceChildren();
  state.log.forEach(message => {
    const li = document.createElement("li");
    li.textContent = message;
    list.appendChild(li);
  });
}

function clearLog() {
  if (!confirm("Bạn muốn xóa toàn bộ nhật ký?")) return;
  state.log = \[];
  record("Nhật ký được tạo lại.");
  renderLog();
  save();
}

function exportLog() {
  const csv = "\\uFEFFNhật ký\\n" +
    state.log.map(s => '"' + s.replace(/"/g,'""') + '"').join("\\n");
  const blob = new Blob(\[csv],{type:"text/csv;charset=utf-8;"});
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href=url;
  a.download="nhat-ky-nong-trai.csv";
  a.click();
  setTimeout(() => URL.revokeObjectURL(url),1000);
}

function weatherCodeDescription(code) {
  if (code === 0) return \["☀️","Trời quang"];
  if (\[1,2].includes(code)) return \["🌤️","Ít mây hoặc có mây"];
  if (code === 3) return \["☁️","Nhiều mây"];
  if (\[45,48].includes(code)) return \["🌫️","Sương mù"];
  if (\[51,53,55,56,57].includes(code)) return \["🌦️","Mưa phùn"];
  if (\[61,63,65,66,67,80,81,82].includes(code)) return \["🌧️","Có mưa"];
  if (\[71,73,75,77,85,86].includes(code)) return \["❄️","Tuyết"];
  if (\[95,96,99].includes(code)) return \["⛈️","Dông"];
  return \["🌦️","Thời tiết thay đổi"];
}

async function loadWeather() {
  const id = ++requestNumber;
  const button = document.querySelector('button\[onclick="loadWeather()"]');
  const lat = Number($("lat").value);
  const lon = Number($("lon").value);

  if (!Number.isFinite(lat) || !Number.isFinite(lon) ||
      lat < -90 || lat > 90 || lon < -180 || lon > 180) {
    alert("Vĩ độ hoặc kinh độ không hợp lệ.");
    return;
  }

  button.disabled = true;
  $("weatherStatus").textContent = "Đang tải dữ liệu thời tiết...";
  weatherData = null;
  renderAdvice();

  try {
    const params = new URLSearchParams({
      latitude:String(lat),
      longitude:String(lon),
      hourly:\[
        "temperature\_2m",
        "relative\_humidity\_2m",
        "precipitation",
        "precipitation\_probability",
        "weather\_code",
        "soil\_moisture\_0\_to\_7cm",
        "soil\_temperature\_0\_to\_7cm"
      ].join(","),
      timezone:"auto",
      forecast\_days:"3"
    });

    const response = await fetch(
      "https://api.open-meteo.com/v1/forecast?" + params
    );
    if (!response.ok) throw new Error("API trả về lỗi " + response.status);

    const data = await response.json();
    if (id !== requestNumber) return;

    const h = data.hourly;
    if (!h || !h.time || !h.time.length) {
      throw new Error("Không có dữ liệu theo giờ.");
    }

    const current = new Date();
    const localParts = new Intl.DateTimeFormat("en-CA", {
      timeZone:data.timezone || "Asia/Bangkok",
      year:"numeric",month:"2-digit",day:"2-digit",hour:"2-digit",
      hourCycle:"h23"
    }).formatToParts(current);
    const parts = Object.fromEntries(localParts.map(p=>\[p.type,p.value]));
    const target = `${parts.year}-${parts.month}-${parts.day}T${parts.hour}:00`;

    let i = h.time.findIndex(t => t >= target);
    if (i < 0) i = 0;

    const get = key => h\[key] \&\& h\[key]\[i] != null ? h\[key]\[i] : null;

    weatherData = {
      time:h.time\[i],
      temperature:get("temperature\_2m"),
      humidity:get("relative\_humidity\_2m"),
      rain:get("precipitation"),
      rainChance:get("precipitation\_probability"),
      code:get("weather\_code"),
      soil:get("soil\_moisture\_0\_to\_7cm"),
      soilTemp:get("soil\_temperature\_0\_to\_7cm")
    };

    $("temperature").textContent = fmt(weatherData.temperature," °C");
    $("airHumidity").textContent = fmt(weatherData.humidity," %");
    $("rainForecast").textContent = fmt(weatherData.rain," mm");
    $("rainChance").textContent = fmt(weatherData.rainChance," %");

    const desc = weatherCodeDescription(weatherData.code);
    $("weatherIcon").textContent = desc\[0];
    $("weatherDescription").textContent = desc\[1];

    if (weatherData.soil != null) {
      // API trả độ ẩm đất theo m³/m³, không phải phần trăm.
      $("soilMoisture").textContent =
        weatherData.soil.toFixed(3) + " m³/m³";
      $("soilDescription").textContent =
        "Độ ẩm đất mô hình ở lớp 0–7 cm";
      $("soilBar").style.width =
        Math.min(100,Math.max(0,weatherData.soil\*100)) + "%";
    } else {
      $("soilMoisture").textContent = "Chưa có dữ liệu";
      $("soilDescription").textContent =
        "Không có số liệu độ ẩm đất từ API";
      $("soilBar").style.width = "0%";
    }

    $("soilTemperature").textContent = fmt(weatherData.soilTemp," °C");
    $("weatherStatus").textContent =
      "Đã tải dữ liệu thành công. Vị trí: " +
      Number(lat).toFixed(3) + ", " + Number(lon).toFixed(3) + ".";
    $("weatherUpdated").textContent =
      "Giờ dự báo: " + weatherData.time +
      " · Múi giờ: " + (data.timezone || "theo API");

    record("Cập nhật dự báo thời tiết tại tọa độ trang trại.");
    renderLog();
    save();
    renderAdvice();
  } catch (err) {
    if (id === requestNumber) {
      $("weatherStatus").textContent =
        "Không tải được thời tiết. Hãy kiểm tra kết nối mạng và thử lại.";
      console.error(err);
    }
  } finally {
    if (id === requestNumber) button.disabled = false;
  }
}

function fmt(value, suffix) {
  return value == null ? "Chưa có" : value + suffix;
}

function useMyLocation() {
  if (!navigator.geolocation) {
    alert("Trình duyệt không hỗ trợ định vị.");
    return;
  }
  navigator.geolocation.getCurrentPosition(
    pos => {
      $("lat").value = pos.coords.latitude.toFixed(5);
      $("lon").value = pos.coords.longitude.toFixed(5);
      loadWeather();
    },
    () => alert(
      "Không lấy được vị trí. Hãy cấp quyền định vị hoặc nhập tọa độ."
    ),
    {enableHighAccuracy:true,timeout:10000}
  );
}

function renderAdvice() {
  const box = $("advice");
  if (!weatherData) {
    box.className = "note";
    box.textContent =
      "Chưa có dự báo. Hãy cập nhật thời tiết để nhận gợi ý chăm sóc.";
    return;
  }

  const tips = \[];
  if (weatherData.rainChance != null \&\& weatherData.rainChance >= 60) {
    tips.push("Khả năng mưa cao. Kiểm tra lượng mưa và hệ thống thoát nước trước khi tưới.");
  } else if (weatherData.temperature != null \&\& weatherData.temperature >= 35) {
    tips.push("Nhiệt độ cao. Theo dõi dấu hiệu héo và kiểm tra độ ẩm đất.");
  } else {
    tips.push("Kiểm tra đất thực tế trước khi quyết định tưới.");
  }

  if (weatherData.soil != null) {
    if (weatherData.soil >= 0.35) {
      tips.push("Mô hình cho thấy đất khá ẩm. Kiểm tra tại ruộng trước khi tưới thêm.");
    } else if (weatherData.soil <= 0.15) {
      tips.push("Độ ẩm mô hình ở mức thấp. Kiểm tra đất và nhu cầu nước của cây.");
    } else {
      tips.push("Theo dõi độ ẩm đất cùng giai đoạn sinh trưởng của cây.");
    }
  } else {
    tips.push("Chưa có dữ liệu độ ẩm đất mô hình; không suy ra độ ẩm đất từ độ ẩm không khí.");
  }

  tips.push("Khuyến nghị chỉ mang tính tham khảo; điều chỉnh theo loại đất, cây trồng và điều kiện ruộng.");

  box.className = "note " +
    (weatherData.rainChance >= 60 ? "warn" : "good");
  box.textContent = tips.join(" ");
}

$("crop").addEventListener("change", () => {
  state.crop = $("crop").value;
  render();
});

// Khởi tạo giao diện; không tự động gửi yêu cầu vị trí.
render();

</body>
</html>

