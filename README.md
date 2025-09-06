<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Web IA Client‑Side — Keitel Mañón Ramírez</title>
  <meta name="author" content="Keitel Mañón Ramírez" />
  <meta name="description" content="Demo client‑side: detección de objetos + análisis de sentimiento. Autor: Keitel Mañón Ramírez." />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;800&display=swap" rel="stylesheet">
  <style>
  :root{
    --bg: #0b1020;
    --panel: #12172a;
    --accent: #6ea8ff;
    --text: #e5e7eb;
    --muted: #9aa0ac;
    --border: #1e2640;
  }
  *{ box-sizing: border-box; }
  html, body { height: 100%; }
  body{
    margin:0;
    font-family: 'Inter', system-ui, -apple-system, Segoe UI, Roboto, Arial, sans-serif;
    background: radial-gradient(1200px 800px at 20% -10%, #1a2240 0%, var(--bg) 45%, #080c18 100%);
    color: var(--text);
  }
  .site-header, .site-footer{ text-align:center; padding: 24px 16px; }
  .top-row{ display:flex; align-items:center; justify-content:center; gap:12px; flex-wrap: wrap; }
  .owner-badge{ border:1px solid var(--border); padding:6px 10px; border-radius:999px; font-weight:700; background: rgba(255,255,255,.05); }
  h1{ font-size: clamp(22px, 4vw, 32px); margin: 0 0 4px; }
  .subtitle{ color: var(--muted); margin: 0; }

  .container{ max-width: 1000px; margin: 0 auto; padding: 16px; display: grid; gap: 16px; }
  .card{ background: linear-gradient(180deg, rgba(18,23,42,.9), rgba(10,14,28,.9)); border: 1px solid var(--border); border-radius: 16px; padding: 16px; box-shadow: 0 10px 30px rgba(0,0,0,.25); }
  .card-title{ margin-top: 0; font-size: 18px; }

  .dropzone{ border: 2px dashed #2a355e; border-radius: 16px; padding: 24px; text-align:center; outline: none; transition: border-color .2s, background .2s; background: rgba(255,255,255,0.02); }
  .dropzone:focus, .dropzone.dragover{ border-color: var(--accent); background: rgba(110,168,255,.06); }
  .dz-inner p{ margin: 8px 0; color: var(--muted); }
  .btn{ background: linear-gradient(180deg, #7fb0ff, #5b8ef0); color: #091021; border: none; padding: 10px 14px; border-radius: 12px; font-weight: 700; cursor: pointer; transition: transform .05s ease, filter .2s ease; }
  .btn:active{ transform: translateY(1px) scale(.99); }
  .btn:disabled{ opacity: .6; cursor: not-allowed; }

  .preview-wrap{ margin-top: 16px; }
  .image-stage{ position: relative; display: inline-block; border-radius: 12px; overflow: hidden; border: 1px solid var(--border); background: #0c1122; }
  .image-stage img{ display:block; max-width: min(100%, 720px); height: auto; }
  .image-stage canvas{ position:absolute; inset:0; pointer-events:none; }
  .image-stage .watermark{ position:absolute; bottom:8px; right:8px; font-size: 12px; padding: 4px 8px; border-radius: 999px; background: rgba(0,0,0,.55); border: 1px solid rgba(255,255,255,.15); }

  .spinner{ position:absolute; top:8px; left:8px; background: rgba(0,0,0,.5); padding: 6px 10px; border-radius: 10px; font-size: 12px; border: 1px solid rgba(255,255,255,.2); }

  .results{ display:grid; gap:12px; margin-top: 12px; }
  .result-block h3{ margin: 0 0 8px; font-size: 16px; }
  .pill-list{ list-style: none; padding:0; margin: 0; display:flex; flex-wrap: wrap; gap:8px; }
  .pill-list li{ border: 1px solid var(--border); padding: 6px 10px; border-radius: 999px; background: rgba(255,255,255,.03); font-size: 13px; }

  textarea#comment{ width: 100%; min-height: 120px; background: rgba(255,255,255,.03); border: 1px solid var(--border); color: var(--text); padding: 10px 12px; border-radius: 12px; resize: vertical; }

  .actions{ display:flex; align-items:center; gap:12px; margin-top:8px; flex-wrap: wrap; }
  .muted{ color: var(--muted); }
  .sentiment-output{ margin-top: 12px; }
  .sentiment-label{ margin: 0 0 6px; color: var(--muted); }
  .sentiment-score{ display:flex; align-items:center; gap:10px; font-size: 18px; font-weight: 700; }
  .meter-wrap{ margin-top: 4px; }
  meter{ width: 100%; height: 14px; }

  .notes{ margin: 8px 0 0; }
  .notes li{ color: var(--muted); }

  .toast{ position: fixed; bottom: 16px; right: 16px; background: #0b1228; border:1px solid #22305f; padding: 10px 14px; border-radius: 10px; box-shadow: 0 6px 16px rgba(0,0,0,.35); }
  .hidden{ display:none !important; }

  .sr-only{ position:absolute !important; width:1px; height:1px; padding:0; margin:-1px; overflow:hidden; clip: rect(0,0,0,0); white-space:nowrap; border:0; }
  </style>
</head>
<body>
  <header class="site-header">
    <div class="top-row">
      <h1>Web IA (Client‑Side)</h1>
      <span class="owner-badge" title="Propietario">Keitel Mañón Ramírez</span>
    </div>
    <p class="subtitle">Sube una imagen, detecta objetos y analiza el sentimiento de tu comentario — todo en tu navegador.</p>
  </header>

  <main class="container">
    <!-- Panel de carga de imagen -->
    <section class="card">
      <h2 class="card-title">1) Sube una imagen</h2>
      <div id="dropzone" class="dropzone" tabindex="0" aria-label="Zona para soltar o seleccionar imagen">
        <div class="dz-inner">
          <svg viewBox="0 0 24 24" width="32" height="32" aria-hidden="true">
            <path d="M19 15v4a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2v-4" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
            <polyline points="16 10 12 6 8 10" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
            <line x1="12" y1="6" x2="12" y2="21" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/>
          </svg>
          <p><strong>Arrastra y suelta</strong> tu imagen aquí o</p>
          <button class="btn" id="btnSelect">Elegir archivo</button>
          <input id="fileInput" type="file" accept="image/*" hidden />
          <small>Formatos: JPG, PNG, WEBP. Todo el procesado es local.</small>
        </div>
      </div>

      <div id="previewWrap" class="preview-wrap hidden">
        <div class="image-stage">
          <img id="preview" alt="Vista previa" />
          <canvas id="overlay" aria-hidden="true"></canvas>
          <div id="spinner" class="spinner hidden" aria-live="polite">Procesando...</div>
          <div class="watermark">Keitel Mañón Ramírez</div>
        </div>
        <div class="results">
          <div class="result-block">
            <h3>Objetos detectados</h3>
            <ul id="objectsList" class="pill-list"></ul>
          </div>
        </div>
      </div>
    </section>

    <!-- Comentario y sentimiento -->
    <section class="card">
      <h2 class="card-title">2) Escribe un comentario sobre tu imagen</h2>
      <label class="sr-only" for="comment">Comentario</label>
      <textarea id="comment" placeholder="Ej.: 'Muy linda, Dios te bendiga'"></textarea>
      <div class="actions">
        <button class="btn" id="btnSentiment">Analizar sentimiento</button>
        <label class="muted" for="lang">Idioma:</label>
        <select id="lang">
          <option value="es" selected>Español</option>
          <option value="en">English</option>
        </select>
        <button class="btn" id="btnRetry" type="button" title="Reintentar carga de modelo" style="display:none">Reintentar modelo</button>
        <small id="sentimentStatus" class="muted">Modelo de sentimiento: <span id="sentimentModelState">cargando…</span></small>
      </div>
      <div id="sentimentOutput" class="sentiment-output hidden">
        <p class="sentiment-label">Resultado:</p>
        <div class="sentiment-score">
          <span id="sentimentEmoji" aria-hidden="true">😐</span>
          <span id="sentimentText">Neutral</span>
          <span id="sentimentValue" class="muted">(0.50)</span>
        </div>
        <div class="meter-wrap">
          <meter id="sentimentMeter" min="0" max="1" low="0.33" high="0.66" optimum="0.9" value="0.5"></meter>
        </div>
      </div>
    </section>

    <!-- Ayuda / Notas -->
    <section class="card">
      <h2 class="card-title">Notas</h2>
      <ul class="notes">
        <li>Todo corre 100% en el navegador (no sube nada a servidores).</li>
        <li>Visión: <code>COCO-SSD</code>. Sentimiento: <code>ml5.sentiment('movieReviews')</code> + ajuste para español.</li>
      </ul>
    </section>
  </main>

  <footer class="site-footer">
    <p>© <span id="year"></span> Keitel Mañón Ramírez • Hecho con ❤️ usando ml5.js (incluye TensorFlow.js) • Demo educativa</p>
  </footer>

  <div id="toast" class="toast hidden" role="status" aria-live="polite"></div>

  <!-- Librería ml5 (incluye una versión compatible de TensorFlow.js) -->
  <script src="https://unpkg.com/ml5@0.12.2/dist/ml5.min.js" defer></script>

  <!-- App JS inline -->
  <script defer>
  // Web IA Client-Side — Detección de objetos + Análisis de Sentimientos
  // Autor/Propietario: Keitel Mañón Ramírez — Todo se ejecuta en el navegador

  let objectDetector = null;
  let sentimentModel = null;
  let sentimentReady = false;

  const el = (id) => document.getElementById(id);

  // UI elements
  const dropzone = el('dropzone');
  const fileInput = el('fileInput');
  const btnSelect = el('btnSelect');
  const previewWrap = el('previewWrap');
  const previewImg = el('preview');
  const overlay = el('overlay');
  const spinner = el('spinner');
  const objectsList = el('objectsList');

  const commentTA = el('comment');
  const btnSentiment = el('btnSentiment');
  const btnRetry = el('btnRetry');
  const langSel = el('lang');
  const sentimentModelState = el('sentimentModelState');
  const sentimentOutput = el('sentimentOutput');
  const sentimentEmoji = el('sentimentEmoji');
  const sentimentText = el('sentimentText');
  const sentimentValue = el('sentimentValue');
  const sentimentMeter = el('sentimentMeter');
  const toast = document.getElementById('toast');

  // Año footer
  document.getElementById('year').textContent = new Date().getFullYear();

  function showToast(msg){
    if (!toast) return;
    toast.textContent = msg;
    toast.classList.remove('hidden');
    setTimeout(()=>toast.classList.add('hidden'), 3200);
  }

  // Carga de modelos
  async function loadObjectDetector(){
    objectDetector = await ml5.objectDetector('cocossd');
  }

  async function loadSentiment(){
    let tries = 0;
    const maxTries = 3;
    while (tries < maxTries){
      try{
        sentimentModelState.textContent = 'cargando…';
        sentimentModel = await ml5.sentiment('movieReviews');
        sentimentReady = true;
        sentimentModelState.textContent = 'listo ✅';
        btnRetry.style.display = 'none';
        return;
      }catch(e){
        console.warn('Fallo al cargar modelo de sentimiento, intento', tries+1, e);
        tries++;
        await new Promise(r => setTimeout(r, 1000 * tries));
      }
    }
    sentimentModelState.textContent = 'error al cargar ❌';
    btnRetry.style.display = 'inline-block';
  }

  async function loadModels(){
    try { await loadObjectDetector(); } catch(e){ console.error('coco-ssd', e); }
    try { await loadSentiment(); } catch(e){ console.error('sentiment', e); }
  }
  document.addEventListener('DOMContentLoaded', loadModels);

  btnRetry?.addEventListener('click', async () => {
    sentimentReady = false;
    await loadSentiment();
  });

  // Helpers de UI
  function setDragover(on){
    dropzone.classList.toggle('dragover', !!on);
  }

  function showPreview(){
    previewWrap.classList.remove('hidden');
  }

  function fitCanvasToImage(){
    const rect = previewImg.getBoundingClientRect();
    overlay.width = Math.round(rect.width);
    overlay.height = Math.round(rect.height);
  }

  function drawDetections(detections){
    const ctx = overlay.getContext('2d');
    fitCanvasToImage();
    ctx.clearRect(0,0,overlay.width, overlay.height);
    ctx.lineWidth = 2;
    ctx.font = '12px Inter, sans-serif';

    detections.forEach(d => {
      const {x, y, width, height, label, confidence} = d;
      ctx.strokeStyle = '#6ea8ff';
      ctx.fillStyle = 'rgba(110,168,255,.12)';
      ctx.beginPath();
      ctx.rect(x, y, width, height);
      ctx.fill();
      ctx.stroke();

      const text = `${label} ${(confidence*100).toFixed(1)}%`;
      const pad = 4;
      const tw = ctx.measureText(text).width + pad*2;
      const th = 16 + pad;
      ctx.fillStyle = 'rgba(0,0,0,.75)';
      ctx.fillRect(x, Math.max(0, y - th), tw, th);
      ctx.fillStyle = '#e5e7eb';
      ctx.fillText(text, x + pad, Math.max(12, y - 4));
    });
  }

  function listDetections(detections){
    objectsList.innerHTML = '';
    if (!detections?.length){
      const li = document.createElement('li');
      li.textContent = 'No se detectaron objetos con confianza suficiente.';
      objectsList.appendChild(li);
      return;
    }
    detections
      .sort((a,b) => b.confidence - a.confidence)
      .forEach(d => {
        const li = document.createElement('li');
        li.textContent = `${d.label} • ${(d.confidence*100).toFixed(1)}%`;
        objectsList.appendChild(li);
      });
  }

  // Carga de imagen
  function handleFiles(files){
    const file = files && files[0];
    if (!file) return;
    if (!file.type || !file.type.startsWith('image/')){
      showToast('El archivo debe ser una imagen.');
      return;
    }

    const reader = new FileReader();
    reader.onerror = () => showToast('No se pudo leer el archivo.');
    reader.onload = () => {
      previewImg.onload = async () => {
        showPreview();
        await runDetection();
      };
      previewImg.src = reader.result;
    };
    reader.readAsDataURL(file);
  }

  // Detección de objetos
  async function runDetection(){
    if (!objectDetector){
      spinner.classList.remove('hidden');
      await waitFor(() => !!objectDetector, 120000);
    }
    spinner.classList.remove('hidden');
    try {
      const results = await objectDetector.detect(previewImg);
      drawDetections(results);
      listDetections(results);
    } catch (err){
      console.error(err);
      objectsList.innerHTML = '<li>Ocurrió un error al detectar objetos.</li>';
    } finally {
      spinner.classList.add('hidden');
    }
  }

  // Sentimiento (con ajuste simple ES)
  function normalizeES(s){
    return s.toLowerCase().normalize('NFD').replace(/[\u0300-\u036f]/g,'');
  }
  const POS_ES = ['lindo','linda','bonito','bonita','hermoso','hermosa','excelente','bueno','buena','precioso','preciosa','perfecto','perfecta','feliz','me gusta','me encanto','me encantó','encanta','maravilloso','maravillosa','increible','increíble','bello','bella','chulo','nitido','nítido','brutal','bendiga','bendicion','bendición'];
  const NEG_ES = ['feo','fea','malo','mala','horrible','terrible','odio','asqueroso','asquerosa','triste','enojado','enojada','molesto','molesta','enfadado','enfadada','decepcionado','decepcionada','fatal','pesimo','pésimo','malisimo','malísimo','mierda','aburrido','aburrida','quillao','quillada','harto'];

  function lexiconAdjustES(text){
    const t = normalizeES(text);
    let pos = 0, neg = 0;
    for (const w of POS_ES){ if (t.includes(normalizeES(w))) pos++; }
    for (const w of NEG_ES){ if (t.includes(normalizeES(w))) neg++; }
    if (/\bno\s+(muy\s+)?(lind[oa]|bonit[oa]|hermos[oa]|buen[oa]|preci[oa]|perfect[oa]|increible|chulo|nitido|bendecir|bendiga)\b/.test(t)) neg += 1;
    if (/\bmuy\s+(lind[oa]|bonit[oa]|hermos[oa]|buen[oa]|preci[oa]|perfect[oa]|increible|chulo|nitido)\b/.test(t)) pos += 1;
    return 0.12 * pos - 0.12 * neg;
  }

  function analyzeSentiment(){
    if (!sentimentReady){
      showToast('El modelo de sentimiento aún se está cargando o falló. Pulsa "Reintentar modelo".');
      return;
    }
    const text = (commentTA.value || '').trim();
    if (!text){
      showToast('Escribe un comentario primero.');
      return;
    }
    const prediction = sentimentModel.predict(text);
    let score = prediction.score; // 0..1
    if (langSel.value === 'es'){
      score = Math.min(1, Math.max(0, score + lexiconAdjustES(text)));
    }

    let label = 'Neutral';
    let emoji = '😐';
    if (score > 0.66){ label = 'Positivo'; emoji = '🙂'; }
    else if (score < 0.33){ label = 'Negativo'; emoji = '🙁'; }

    sentimentOutput.classList.remove('hidden');
    sentimentEmoji.textContent = emoji;
    sentimentText.textContent = label;
    sentimentValue.textContent = `(${score.toFixed(2)})`;
    sentimentMeter.value = score;
  }

  // Utilidades
  function waitFor(cond, timeout = 10000, interval = 200){
    return new Promise((resolve, reject) => {
      const start = performance.now();
      const id = setInterval(() => {
        if (cond()){
          clearInterval(id);
          resolve(true);
        } else if (performance.now() - start > timeout){
          clearInterval(id);
          reject(new Error('timeout'));
        }
      }, interval);
    });
  }

  // Eventos
  btnSelect.addEventListener('click', () => fileInput.click());
  fileInput.addEventListener('change', (e) => handleFiles(e.target.files));

  ['dragenter','dragover'].forEach((evt) =>
    dropzone.addEventListener(evt, (e) => { e.preventDefault(); setDragover(true); })
  );
  ['dragleave','dragend','drop'].forEach((evt) =>
    dropzone.addEventListener(evt, (e) => { e.preventDefault(); setDragover(false); })
  );
  dropzone.addEventListener('drop', (e) => {
    e.preventDefault();
    handleFiles(e.dataTransfer.files);
  });
  dropzone.addEventListener('click', () => fileInput.click());

  // Permitir pegar imagen desde el portapapeles
  window.addEventListener('paste', (e) => {
    const items = e.clipboardData?.items;
    if (!items) return;
    for (const it of items){
      if (it.type?.startsWith('image/')){
        const file = it.getAsFile();
        handleFiles([file]);
        showToast('Imagen pegada desde el portapapeles.');
        break;
      }
    }
  });

  btnSentiment.addEventListener('click', analyzeSentiment);

  // Ajustar overlay al redimensionar
  window.addEventListener('resize', () => {
    if (!previewImg.src) return;
    fitCanvasToImage();
  });
  </script>
</body>
</html>
