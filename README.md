<!DOCTYPE html>
<html lang="ml">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Standard ID Card Hub</title>

<script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>

<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Amiri:wght@400;700&family=Inter:wght@400;700;800&display=swap" rel="stylesheet">

<style>
*{box-sizing:border-box;margin:0;padding:0}

body{
  font-family:'Inter',sans-serif;
  background:#f4f7f6;
  padding:20px;
  display:flex;
  flex-direction:column;
  align-items:center;
}

.panel{
  background:#fff;
  border-radius:12px;
  padding:25px;
  width:100%;
  max-width:900px;
  margin-bottom:30px;
  box-shadow:0 4px 15px rgba(0,0,0,0.05);
}

.input-grid{
  display:grid;
  grid-template-columns:repeat(auto-fit, minmax(200px, 1fr));
  gap:15px;
  margin-bottom:20px;
}

.fi label{
  display:block;
  font-size:11px;
  font-weight:700;
  margin-bottom:5px;
  color:#444;
}

.fi input, .fi textarea{
  width:100%;
  padding:10px;
  border:1px solid #ddd;
  border-radius:6px;
}

.fi textarea{
  height:80px;
  resize:none;
}

.btn-row{
  display:flex;
  gap:10px;
}

button{
  padding:12px 25px;
  border:none;
  border-radius:6px;
  cursor:pointer;
  font-weight:700;
}

.btn-prev{background:#1e3a5f;color:#fff;}
.btn-pdf{background:#27ae60;color:#fff;display:none;}

/* ---- CARD ---- */
.id-card-base{
  width:323.5px;
  height:204px;
  background:#fff;
  position:relative;
  overflow:hidden;
  border:0.5px solid #ccc;
  display:flex;
  flex-direction:column;
}

.card-header{
  flex:1;
  padding:12px 18px 8px 18px;
  color:#fff;
  display:flex;
  flex-direction:column;
  justify-content:space-between;
}

.c-name{
  font-weight:800;
  text-transform:uppercase;
  border-bottom:1px solid rgba(255,255,255,0.3);
  padding-bottom:4px;
  margin-bottom:4px;
  white-space:nowrap;
  overflow:hidden;
  text-overflow:ellipsis;
}

.c-label{
  font-family: 'Amiri', serif;
  font-size:24px;
  font-weight:700;
  opacity:0.95;
  text-align:center;
  direction:rtl;
  unicode-bidi:embed;
  letter-spacing:0;
  line-height:1.2;
}

.c-details{
  line-height:1.5;
  margin-top:4px;
}

.c-details .tok{
  font-size:15px;
  font-weight:800;
  color:#fff;
}

.c-details .meta{
  font-size:11px;
  font-weight:600;
  color:#fff;
  opacity:0.9;
}

.card-footer{
  height:68px;
  flex-shrink:0;
  background:#fff;
  display:flex;
  align-items:center;
  padding:0 12px;
  justify-content:space-between;
  border-top:1px solid #eee;
}

.c-qr{
  width:52px;
  height:52px;
  flex-shrink:0;
}

.c-org-info{
  text-align:right;
  flex:1;
  padding-right:10px;
}

.c-org-name{
  font-size:11px;
  font-weight:800;
  color:#1e3a5f;
}

.c-org-sub{
  font-size:9px;
  color:#777;
}

.c-logo{
  width:40px;
  height:40px;
  flex-shrink:0;
  border-radius:50%;
  overflow:hidden;
  display:flex;
  align-items:center;
  justify-content:center;
  border:1px solid #f0f0f0;
}

.c-logo img{
  width:100%;
  height:100%;
  object-fit:contain;
}

#print-area{background:#fff; display:none;}

.print-page{
  width:210mm;
  height:297mm;
  padding:15mm;
  display:grid;
  grid-template-columns:85.6mm 85.6mm;
  grid-template-rows:repeat(4,53.98mm);
  gap:4mm 6mm;
  justify-content:center;
  align-content:start;
  page-break-after:always;
}

.print-item{
  width:85.6mm;
  height:53.98mm;
  overflow:hidden;
  border:0.1mm solid #ccc;
}

@media screen{
  #preview-grid{
    display:grid;
    grid-template-columns:repeat(auto-fill, 323px);
    gap:20px;
    justify-content:center;
    margin-top:20px;
  }
}
</style>
</head>

<body>

<div class="panel">
  <h2>ID Generator</h2>

  <div class="input-grid">
    <div class="fi"><label>Organisation</label><input id="org" value="THARBIYATHUL ULOOM"></div>
    <div class="fi"><label>Branch</label><input id="sub" value="WEST CHALIPPARAMBA"></div>
    <div class="fi"><label>Arabic Label</label><input id="arabicLabel" value="مسجد توبة"></div>
    <div class="fi"><label>Theme Color</label><input id="clr" type="color" value="#1e3a5f"></div>
    <div class="fi"><label>Logo</label><input type="file" id="logoInp" accept="image/*"></div>
  </div>

  <div class="input-grid">
    <div class="fi"><label>Names (comma)</label><textarea id="names"></textarea></div>
    <div class="fi"><label>Tokens</label><textarea id="tokens"></textarea></div>
    <div class="fi"><label>MHL Nos</label><textarea id="mhls"></textarea></div>
    <div class="fi"><label>Ward Nos</label><textarea id="wards"></textarea></div>
  </div>

  <div class="btn-row">
    <button class="btn-prev" onclick="generate()">&#9654; Preview Cards</button>
    <button id="pdfBtn" class="btn-pdf" onclick="downloadPDF()">&#8681; Download PDF</button>
  </div>
</div>

<div id="preview-grid"></div>
<div id="print-area"></div>

<script>
let logoBase64 = '';

document.getElementById('logoInp').onchange = function(e){
  let reader = new FileReader();
  reader.onload = (ev) => logoBase64 = ev.target.result;
  reader.readAsDataURL(e.target.files[0]);
};

function parse(id){
  return document.getElementById(id).value.split(',').map(s => s.trim());
}

function nameFontSize(name){
  const len = name.length;
  if(len <= 14) return '18px';
  if(len <= 20) return '15px';
  if(len <= 26) return '13px';
  return '11px';
}

function buildCard(name, token, mhl, ward, color, org, sub, id, arabicLabel){
  const logo = logoBase64 ? '<img src="' + logoBase64 + '">' : '';
  const fs = nameFontSize(name);
  return '<div class="id-card-base">' +
    '<div class="card-header" style="background:' + color + '">' +
      '<div class="c-name" style="font-size:' + fs + '">' + name + '</div>' +
      '<div class="c-label">' + arabicLabel + '</div>' +
      '<div class="c-details">' +
        '<span class="tok">TOKEN: ' + token + '</span>' +
        '<span class="meta"> &nbsp;| MHL: ' + mhl + ' | WARD: ' + ward + '</span>' +
      '</div>' +
    '</div>' +
    '<div class="card-footer">' +
      '<div class="c-qr" id="' + id + '"></div>' +
      '<div class="c-org-info">' +
        '<div class="c-org-name">' + org + '</div>' +
        '<div class="c-org-sub">' + sub + '</div>' +
      '</div>' +
      '<div class="c-logo">' + logo + '</div>' +
    '</div>' +
  '</div>';
}

function generateQR(elId, name, token, mhl, ward, org, sub){
  const el = document.getElementById(elId);
  if(!el) return;
  el.innerHTML = "";
  new QRCode(el, {
    text: 'TOKEN: ' + token + '\n' + name + '\nMHL: ' + mhl + '  WARD: ' + ward + '\n' + org + ' - ' + sub,
    width: 52,
    height: 52
  });
}

function generate(){
  const names = parse('names').filter(n => n !== "");
  if(names.length === 0) return alert("Enter names");

  const tokens = parse('tokens');
  const mhls   = parse('mhls');
  const wards  = parse('wards');
  const color  = document.getElementById('clr').value;
  const org    = document.getElementById('org').value;
  const sub    = document.getElementById('sub').value;
  const arabicLabel = document.getElementById('arabicLabel').value;

  const preview = document.getElementById('preview-grid');
  preview.innerHTML = '';

  names.forEach(function(n,i){
    const qid = 'qrv'+i;
    const div = document.createElement('div');
    div.innerHTML = buildCard(n, tokens[i]||'-', mhls[i]||'-', wards[i]||'-', color, org, sub, qid, arabicLabel);
    preview.appendChild(div);
    generateQR(qid, n, tokens[i]||'-', mhls[i]||'-', wards[i]||'-', org, sub);
  });

  document.getElementById('pdfBtn').style.display = 'block';
}

function downloadPDF(){
  const names  = parse('names').filter(n => n !== "");
  const tokens = parse('tokens');
  const mhls   = parse('mhls');
  const wards  = parse('wards');
  const color  = document.getElementById('clr').value;
  const org    = document.getElementById('org').value;
  const sub    = document.getElementById('sub').value;
  const arabicLabel = document.getElementById('arabicLabel').value;

  const printArea = document.getElementById('print-area');
  printArea.innerHTML = '';
  printArea.style.display = 'block';

  const styleTag = document.createElement('style');
  styleTag.textContent = [
    "@import url('https://fonts.googleapis.com/css2?family=Amiri:wght@400;700&display=swap');",
    "*{box-sizing:border-box;margin:0;padding:0}",
    ".id-card-base{width:323.5px;height:204px;background:#fff;overflow:hidden;border:0.5px solid #ccc;display:flex;flex-direction:column;}",
    ".card-header{flex:1;padding:12px 18px 8px 18px;color:#fff;display:flex;flex-direction:column;justify-content:space-between;}",
    ".c-name{font-family:Inter,sans-serif;font-weight:800;text-transform:uppercase;border-bottom:1px solid rgba(255,255,255,0.3);padding-bottom:4px;margin-bottom:4px;white-space:nowrap;overflow:hidden;text-overflow:ellipsis;}",
    ".c-label{font-family:'Amiri',serif;font-size:24px;font-weight:700;opacity:0.95;text-align:center;direction:rtl;unicode-bidi:embed;letter-spacing:0;line-height:1.2;}",
    ".c-details{line-height:1.5;margin-top:4px;}",
    ".c-details .tok{font-family:Inter,sans-serif;font-size:15px;font-weight:800;color:#fff;}",
    ".c-details .meta{font-family:Inter,sans-serif;font-size:11px;font-weight:600;color:#fff;opacity:0.9;}",
    ".card-footer{height:68px;flex-shrink:0;background:#fff;display:flex;align-items:center;padding:0 12px;justify-content:space-between;border-top:1px solid #eee;}",
    ".c-qr{width:52px;height:52px;flex-shrink:0;}",
    ".c-org-info{text-align:right;flex:1;padding-right:10px;}",
    ".c-org-name{font-family:Inter,sans-serif;font-size:11px;font-weight:800;color:#1e3a5f;}",
    ".c-org-sub{font-family:Inter,sans-serif;font-size:9px;color:#777;}",
    ".c-logo{width:40px;height:40px;flex-shrink:0;border-radius:50%;overflow:hidden;display:flex;align-items:center;justify-content:center;border:1px solid #f0f0f0;}",
    ".c-logo img{width:100%;height:100%;object-fit:contain;}",
    ".print-page{width:210mm;min-height:297mm;padding:15mm;display:grid;grid-template-columns:85.6mm 85.6mm;grid-template-rows:repeat(4,53.98mm);gap:4mm 6mm;justify-content:center;align-content:start;page-break-after:always;}",
    ".print-item{width:85.6mm;height:53.98mm;overflow:hidden;border:0.1mm solid #ccc;}"
  ].join('\n');
  printArea.appendChild(styleTag);

  let page = document.createElement('div');
  page.className = 'print-page';

  names.forEach(function(n,i){
    if(i !== 0 && i % 8 === 0){
      printArea.appendChild(page);
      page = document.createElement('div');
      page.className = 'print-page';
    }

    const qid = 'qrp'+i;
    const item = document.createElement('div');
    item.className = 'print-item';
    item.innerHTML = buildCard(n, tokens[i]||'-', mhls[i]||'-', wards[i]||'-', color, org, sub, qid, arabicLabel);
    page.appendChild(item);

    setTimeout(function(){
      generateQR(qid, n, tokens[i]||'-', mhls[i]||'-', wards[i]||'-', org, sub);
    }, 150);
  });

  printArea.appendChild(page);

  document.fonts.load('700 26px Amiri').then(function(){
    const opt = {
      margin: 0,
      filename: 'ID_Cards.pdf',
      image: { type: 'jpeg', quality: 1 },
      html2canvas: {
        scale: 3,
        useCORS: true,
        allowTaint: true,
        logging: false,
        onclone: function(clonedDoc){
          const s = clonedDoc.createElement('style');
          s.textContent = [
            "@import url('https://fonts.googleapis.com/css2?family=Amiri:wght@400;700&display=swap');",
            ".id-card-base{display:flex!important;flex-direction:column!important;}",
            ".card-header{flex:1!important;display:flex!important;flex-direction:column!important;justify-content:space-between!important;padding:12px 18px 8px 18px!important;}",
            ".card-footer{height:68px!important;flex-shrink:0!important;background:#fff!important;}",
            ".c-label{font-family:'Amiri',serif!important;direction:rtl!important;unicode-bidi:embed!important;letter-spacing:0!important;}",
            ".c-details{margin-top:4px!important;line-height:1.5!important;}",
            ".c-details .tok{font-size:15px!important;font-weight:800!important;color:#fff!important;}",
            ".c-details .meta{font-size:11px!important;font-weight:600!important;color:#fff!important;}"
          ].join('\n');
          clonedDoc.head.appendChild(s);
        }
      },
      jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' }
    };

    setTimeout(function(){
      html2pdf().set(opt).from(printArea).save().then(function(){
        printArea.style.display = 'none';
      });
    }, 800);
  });
}
</script>

</body>
</html>
