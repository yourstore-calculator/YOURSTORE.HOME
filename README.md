# YOURSTORE.HOME
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <title>حاسبة الأسعار</title>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background-color: #1e1e1e;
      color: white;
      direction: rtl;
      padding: 30px;
    }
    .logo {
      text-align: center;
      margin-bottom: 20px;
    }
    .logo img {
      max-height: 100px;
    }
    select, input {
      padding: 8px;
      margin: 5px 0;
      width: 100%;
      border: none;
      border-radius: 4px;
      box-sizing: border-box;
    }
    input:disabled {
      background-color: #555;
      cursor: not-allowed;
      color: #aaa;
    }
    .section {
      margin-bottom: 20px;
    }
    button {
      background-color: #444;
      color: white;
      padding: 10px 20px;
      border: none;
      margin-top: 10px;
      border-radius: 5px;
      cursor: pointer;
    }
    button:hover {
      background-color: #666;
    }
    .results {
      background: #2b2b2b;
      padding: 20px;
      margin-top: 20px;
      border-radius: 8px;
    }
    .result-item {
      border-bottom: 1px solid #555;
      padding: 10px 0;
    }
    .result-item:last-child {
      border-bottom: none;
    }
    .summary {
      font-weight: bold;
      color: #00e676;
      padding-top: 10px;
      border-top: 1px solid #555;
      margin-top: 10px;
    }
  </style>
</head>
<body>

<div class="logo">
  <!-- تم استبدال الشعار هنا -->
  <img src="https://i.imgur.com/ih5zjVj.png" alt="شعار الشركة">
</div>

<div class="section">
  <label>الفئة الرئيسية:</label>
  <select id="mainCategory" onchange="loadSubTypes()">
    <option value="">-- اختر الفئة --</option>
    <option value="نوافذ">نوافذ</option>
    <option value="أبواب">أبواب</option>
    <option value="أبواب سحب">أبواب سحب</option>
  </select>
</div>

<div class="section">
  <label>النوع الفرعي:</label>
  <select id="subType"></select>
</div>

<div class="section">
  <label>الطول (متر):</label>
  <input type="number" id="height" step="0.01" value="1" />
</div>

<div class="section">
  <label>العرض (متر):</label>
  <input type="number" id="width" step="0.01" value="1" />
</div>

<div class="section">
  <label>الكمية:</label>
  <input type="number" id="quantity" value="1" />
</div>

<div class="section" id="curtainSection" style="display:none">
  <label>إضافة ستارة داخلية؟</label>
  <input type="checkbox" id="addCurtain" />
</div>

<div class="section" id="netSection" style="display:none">
  <label>اختر نوع الشبك:</label>
  <select id="netType">
    <option value="">-- بدون --</option>
    <option value="ثابت">ثابت</option>
    <option value="سلايد">سلايد</option>
    <option value="فولدينج">فولدينج</option>
    <option value="باب">باب</option>
  </select>
</div>

<button onclick="calculate()">➕ أضف للنتائج</button>
<button onclick="clearResults()">🗑️ مسح النتائج</button>
<button onclick="saveAsWord()">💾 حفظ كـ Word</button>

<div class="results" id="results"></div>

<script>
const prices = {
  // نوافذ بأنواعها و(ثابت، حركة، حركتين)
  "دبل جلاس دبل فريم ثابتة": 34,
  "دبل جلاس دبل فريم حركة": 73,
  "دبل جلاس دبل فريم حركتين": 92,
  "دبل جلاس سنجل فريم ثابتة": 26,
  "دبل جلاس سنجل فريم حركة": 46,
  "دبل جلاس سنجل فريم حركتين": 58,
  "سنجل جلاس سنجل فريم ثابتة": 20,
  "سنجل جلاس سنجل فريم حركة": 43,
  "سنجل جلاس سنجل فريم حركتين": 47,
  "نوافذ السلايدنج": 10,
  "النوافذ الكهربائية": 102,
  "سكاي لايت بدون مكينة": 56,
  "سكاي لايت مع مكينة": 145,
  "كارتن وول ثقيل": 56,
  "كارتن وول خفيف": 45,
  "باب المدخل زينك": 66,
  "باب المدخل ستينلس ستيل": 120,
  "باب المدخل كاست المنيوم": 168,
  "WPC فارغ": 45,
  "WPC مع خشب": 50,
  "WPC مع حشوة ضد الصوت": 60,
  "سلايدنج WPC": 65,
  "ألمنيوم فارغ": 65,
  "ألمنيوم مع خشب": 75,
  "ألمنيوم فل": 85,
  "ألمنيوم مخفي": 110,
  "ألمنيوم خارجي": 61,
  "دورات المياه الجديد": 55,
  "دورات المياه القديم": 45,
  "دورات المياه مخفي زجاجي": 65,
  "داخلي زجاج": 38,
  "داخلي متين": 41,
  "خارجي جزء مفتوح": 55,
  "خارجي جزئين مفتوحات": 58,
  "WPC سلايد": 61,
};

const factors = {
  "default": 0.13,
  "WPC": 0.11,
  "ألمنيوم": 0.11,
  "دورات المياه": 0.11,
  "باب المدخل": 0.2,
};

let resultsList = [];

function loadSubTypes() {
  const cat = document.getElementById("mainCategory").value;
  const sub = document.getElementById("subType");
  sub.innerHTML = "";

  let opts = [];
  if (cat === "نوافذ") {
    opts = [
      "دبل جلاس دبل فريم ثابتة","دبل جلاس دبل فريم حركة","دبل جلاس دبل فريم حركتين",
      "دبل جلاس سنجل فريم ثابتة","دبل جلاس سنجل فريم حركة","دبل جلاس سنجل فريم حركتين",
      "سنجل جلاس سنجل فريم ثابتة","سنجل جلاس سنجل فريم حركة","سنجل جلاس سنجل فريم حركتين",
      "نوافذ السلايدنج","النوافذ الكهربائية","سكاي لايت بدون مكينة","سكاي لايت مع مكينة","كارتن وول ثقيل","كارتن وول خفيف"
    ];
  } else if (cat === "أبواب") {
    opts = [
      "باب المدخل زينك","باب المدخل ستينلس ستيل","باب المدخل كاست المنيوم",
      "WPC فارغ","WPC مع خشب","WPC مع حشوة ضد الصوت","سلايدنج WPC",
      "ألمنيوم فارغ","ألمنيوم مع خشب","ألمنيوم فل","ألمنيوم مخفي","ألمنيوم خارجي",
      "دورات المياه الجديد","دورات المياه القديم","دورات المياه مخفي زجاجي"
    ];
  } else if (cat === "أبواب سحب") {
    opts = ["داخلي زجاج","داخلي متين","خارجي جزء مفتوح","خارجي جزئين مفتوحات","WPC سلايد"];
  }

  opts.forEach(v => {
    const o = document.createElement("option");
    o.value = v;
    o.text = v;
    sub.appendChild(o);
  });
  
  updateUIBasedOnType();
}

function updateUIBasedOnType(){
    setTimeout(() => {
        const val = document.getElementById("subType").value;
        const cs = document.getElementById("curtainSection");
        const ns = document.getElementById("netSection");
        const heightInput = document.getElementById("height");
        const widthInput = document.getElementById("width");

        const showCurtain = ["دبل جلاس دبل فريم","دبل جلاس سنجل فريم","نوافذ السلايدنج","داخلي زجاج","داخلي متين","خارجي جزء مفتوح","خارجي جزئين مفتوحات","WPC سلايد"]
                            .some(pref => val.startsWith(pref));
        const showNet = ["دبل جلاس دبل فريم","دبل جلاس سنجل فريم","سنجل جلاس سنجل فريم"]
                        .some(pref => val.startsWith(pref));

        cs.style.display = showCurtain ? "block" : "none";
        ns.style.display = showNet ? "block" : "none";

        if (val.includes("WPC") || val.includes("ألمنيوم")) {
            heightInput.value = 2.2;
            widthInput.value = 1.0;
            heightInput.disabled = true;
            widthInput.disabled = true;
        } else if (val.includes("دورات المياه")) {
            heightInput.value = 2.2;
            widthInput.value = 0.8;
            heightInput.disabled = true;
            widthInput.disabled = true;
        } else {
            heightInput.disabled = false;
            widthInput.disabled = false;
        }
    }, 50);
}

document.getElementById("subType").addEventListener("change", updateUIBasedOnType);

function calculate(){
  const sub = document.getElementById("subType").value;
  const qty = parseInt(document.getElementById("quantity").value) || 1;
  const h = parseFloat(document.getElementById("height").value) || 0;
  const w = parseFloat(document.getElementById("width").value) || 0;
  
  if (!sub || h === 0 || w === 0 || qty === 0) {
    alert("يرجى التأكد من إدخال كافة البيانات بشكل صحيح.");
    return;
  }
  
  const area = h * w;
  const pricePer = prices[sub] || 0;
  let price = pricePer * area;
  
  const factorKey = Object.keys(factors).find(k => k !== 'default' && sub.includes(k)) || 'default';
  const factor = factors[factorKey];
  const shippingPerItem = area * factor * 48;
  
  let totalPerItem = price + shippingPerItem;

  if(document.getElementById("addCurtain").checked && document.getElementById("curtainSection").style.display !== 'none'){
    totalPerItem += area * 26;
  }
  if(document.getElementById("netType").value && document.getElementById("netSection").style.display !== 'none'){
    totalPerItem += 30;
  }

  const finalPrice = totalPerItem * qty;
  const totalShipping = shippingPerItem * qty;

  resultsList.push({sub, h, w, qty, shipping: totalShipping, final: finalPrice});
  renderResults();
}

function renderResults(){
  const container = document.getElementById("results");
  container.innerHTML = "";
  let sum = 0;
  
  resultsList.forEach(r=>{
    sum += r.final;
    container.innerHTML += `
      <div class="result-item">
        <b>النوع:</b> ${r.sub}<br>
        <b>المقاس:</b> ${r.h} × ${r.w} متر<br>
        <b>الكمية:</b> ${r.qty}<br>
        <b>🚚 إجمالي الشحن:</b> ${r.shipping.toFixed(2)}<br>
        <b>💵 السعر الإجمالي للبند:</b> ${r.final.toFixed(2)}
      </div>
    `;
  });

  if (resultsList.length > 0) {
    const comm = sum * 0.04;
    const total = sum + comm;
    container.innerHTML += `
      <div class="summary">
        ✅ المجموع الفرعي: ${sum.toFixed(2)}<br>
        💼 عمولة المكتب (4%): ${comm.toFixed(2)}<br>
        💰 <b>الناتج النهائي: ${total.toFixed(2)}</b><br><br>
        <span style="color:#ccc; font-size: 0.9em;">* الأسعار شاملة الشحن<br>** الأسعار لا تشمل التركيب</span>
      </div>
    `;
  }
}

function clearResults(){
  resultsList = [];
  document.getElementById("results").innerHTML = "";
  document.getElementById("mainCategory").value = "";
  document.getElementById("subType").innerHTML = "";
  document.getElementById("height").value = "1";
  document.getElementById("width").value = "1";
  document.getElementById("height").disabled = false;
  document.getElementById("width").disabled = false;
  document.getElementById("quantity").value = "1";
  document.getElementById("addCurtain").checked = false;
  document.getElementById("netType").value = "";
  document.getElementById("curtainSection").style.display = "none";
  document.getElementById("netSection").style.display = "none";
}

function saveAsWord(){
    if (resultsList.length === 0) {
        alert("لا توجد نتائج لحفظها.");
        return;
    }
    const header = "<html xmlns:o='urn:schemas-microsoft-com:office:office' " +
            "xmlns:w='urn:schemas-microsoft-com:office:word' " +
            "xmlns='http://www.w3.org/TR/REC-html40'>" +
            "<head><meta charset='utf-8'><title>عرض سعر</title></head><body>";
    const footer = "</body></html>";
    let content = document.getElementById("results").innerHTML;
    
    const sourceHTML = header + content.replace(/style="[^"]*"/g, '') + footer;

    const source = 'data:application/vnd.ms-word;charset=utf-8,' + encodeURIComponent(sourceHTML);
    const fileDownload = document.createElement("a");
    document.body.appendChild(fileDownload);
    fileDownload.href = source;
    fileDownload.download = 'عرض_سعر.doc';
    fileDownload.click();
    document.body.removeChild(fileDownload);
}

loadSubTypes();
</script>

</body>
</html>
