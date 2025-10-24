# TCM-Guide-Image-Editor
image editor
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <title>Canva-Lite — Hosted Editor</title>
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <!-- Google fonts -->
  <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600;700&family=Roboto:wght@400;700&display=swap" rel="stylesheet">
  <!-- Fabric.js CDN -->
  <script src="https://cdnjs.cloudflare.com/ajax/libs/fabric.js/5.3.0/fabric.min.js"></script>

  <style>
    :root{
      --bg:#f4f6fb; --panel:#fff; --muted:#6b7280; --accent:#2563eb;
    }
    *{box-sizing:border-box}
    body{margin:0;font-family:'Poppins',Inter,system-ui,Arial;background:var(--bg);color:#111}
    .topbar{display:flex;align-items:center;justify-content:space-between;gap:12px;padding:12px 16px;background:#fff;border-bottom:1px solid #e8edf3}
    .brand{font-weight:700}
    .container{max-width:1200px;margin:18px auto;display:flex;gap:12px;padding:12px}
    .left{flex:1;background:var(--panel);border-radius:10px;padding:12px;box-shadow:0 6px 18px rgba(15,23,42,0.06)}
    .sidebar{width:320px;background:var(--panel);border-radius:10px;padding:12px;box-shadow:0 6px 18px rgba(15,23,42,0.06);height:calc(100vh - 140px);overflow:auto}
    .toolbar{display:flex;flex-wrap:wrap;gap:8px;margin-bottom:10px}
    button,input,select{padding:8px 10px;border:1px solid #dfe6f0;border-radius:8px;background:#fff}
    button:hover{background:#f3f8ff}
    #canvas-wrapper{border:1px solid #e6e9ef;border-radius:8px;overflow:hidden;background:#fff}
    #c{width:100%;height:620px;display:block}
    label.small{font-size:12px;color:var(--muted);display:block;margin-bottom:6px}
    .section{margin-top:10px}
    .layers {max-height:220px;overflow:auto;border:1px solid #f0f2f6;border-radius:6px;padding:6px}
    .layer-item{display:flex;justify-content:space-between;align-items:center;padding:6px;border-bottom:1px solid #f3f4f7}
    .layer-item:last-child{border-bottom:0}
    .small-muted{font-size:12px;color:var(--muted)}
    .footer{padding:10px;text-align:center;color:var(--muted);font-size:13px}
    @media (max-width:980px){
      .container{flex-direction:column}
      .sidebar{width:100%;height:auto}
    }
  </style>
</head>
<body>
  <header class="topbar">
    <div class="brand">Canva-Lite</div>
    <div class="small-muted">Host, embed, customize — starter editor</div>
  </header>

  <div class="container">
    <div class="left">
      <div class="toolbar">
        <input id="fileInput" type="file" accept="image/*" />
        <button id="addTextBtn">Add Text</button>
        <select id="fontSelect">
          <option value="Poppins">Poppins</option>
          <option value="Roboto">Roboto</option>
          <option value="Arial">Arial</option>
          <option value="Courier New">Courier New</option>
          <option value="Georgia">Georgia</option>
        </select>
        <input id="fontSize" type="number" value="40" min="8" max="200" style="width:90px" />
        <input id="colorPicker" type="color" value="#111111" />
        <button id="boldBtn"><b>B</b></button>
        <button id="italicBtn"><i>I</i></button>
        <button id="underlineBtn">U</button>
        <button id="bringForward">Bring ↑</button>
        <button id="sendBackward">Send ↓</button>
        <button id="duplicateBtn">Duplicate</button>
        <button id="deleteBtn">Delete</button>
      </div>

      <div style="display:flex;gap:8px;align-items:center;margin-bottom:8px">
        <button id="saveBtn">💾 Save</button>
        <button id="loadBtn">📂 Load</button>
        <button id="exportBtn">⬇️ Export PNG</button>
        <label style="display:flex;align-items:center;gap:6px;margin-left:6px" class="small-muted">
          <input type="checkbox" id="autosaveChk" /> Auto-save
        </label>
      </div>

      <div id="canvas-wrapper">
        <canvas id="c" width="1000" height="620"></canvas>
      </div>

      <div class="section" style="display:flex;gap:8px;align-items:center;margin-top:10px">
        <button id="templateBtn">Apply Template</button>
        <button id="clearBtn">Clear</button>
      </div>
    </div>

    <aside class="sidebar">
      <div>
        <div style="display:flex;justify-content:space-between;align-items:center">
          <strong>Layers</strong>
          <span class="small-muted">Top → Bottom</span>
        </div>
        <div class="layers" id="layersBox"></div>
      </div>

      <div class="section">
        <div class="small-muted">Selected Object</div>
        <div id="propPanel" style="margin-top:8px"></div>
      </div>

      <div class="section">
        <div class="small-muted">Tips</div>
        <ul style="padding-left:18px;margin:8px 0;color:var(--muted);font-size:13px">
          <li>Double-click a text object to edit its content.</li>
          <li>Use transformer handles to scale and rotate.</li>
          <li>Save to localStorage and load later.</li>
        </ul>
      </div>

    </aside>
  </div>

  <div class="footer">Built with Fabric.js • Copy & host anywhere • Embed using an iframe</div>

  <script>
    // Bootstrap canvas with Fabric.js
    const canvas = new fabric.Canvas('c', { preserveObjectStacking: true, backgroundColor: '#ffffff' });

    // Helpers
    function uid(prefix='id'){ return prefix + '_' + Date.now() + '_' + Math.floor(Math.random()*9999); }

    // Elements
    const fileInput = document.getElementById('fileInput');
    const addTextBtn = document.getElementById('addTextBtn');
    const fontSelect = document.getElementById('fontSelect');
    const fontSizeInput = document.getElementById('fontSize');
    const colorPicker = document.getElementById('colorPicker');
    const boldBtn = document.getElementById('boldBtn');
    const italicBtn = document.getElementById('italicBtn');
    const underlineBtn = document.getElementById('underlineBtn');
    const bringForwardBtn = document.getElementById('bringForward');
    const sendBackwardBtn = document.getElementById('sendBackward');
    const duplicateBtn = document.getElementById('duplicateBtn');
    const deleteBtn = document.getElementById('deleteBtn');
    const saveBtn = document.getElementById('saveBtn');
    const loadBtn = document.getElementById('loadBtn');
    const exportBtn = document.getElementById('exportBtn');
    const clearBtn = document.getElementById('clearBtn');
    const templateBtn = document.getElementById('templateBtn');
    const layersBox = document.getElementById('layersBox');
    const propPanel = document.getElementById('propPanel');
    const autosaveChk = document.getElementById('autosaveChk');

    // Autosave interval
    let autosaveTimer = null;
    autosaveChk.checked = true;
    function scheduleAutosave(){
      if(autosaveTimer) clearTimeout(autosaveTimer);
      if(autosaveChk.checked){
        autosaveTimer = setTimeout(()=> saveProject(true), 1000);
      }
    }

    // Upload image file
    fileInput.addEventListener('change', (e) => {
      const f = e.target.files[0];
      if (!f) return;
      const reader = new FileReader();
      reader.onload = (evt) => {
        fabric.Image.fromURL(evt.target.result, function(img){
          const maxW = 800, maxH = 500;
          const scale = Math.min(maxW / img.width, maxH / img.height, 1);
          img.scale(scale);
          img.set({ left: 80, top: 60, selectable:true, objectCaching:true });
          img.setControlsVisibility({ mt:true, mb:true, ml:true, mr:true, bl:true, br:true, tl:true, tr:true, mtr:true });
          canvas.add(img);
          canvas.setActiveObject(img);
          canvas.requestRenderAll();
          updateLayersPanel();
          scheduleAutosave();
        }, { crossOrigin: 'anonymous' });
      };
      reader.readAsDataURL(f);
      e.target.value = '';
    });

    // Add text
    addTextBtn.addEventListener('click', () => {
      const txt = new fabric.IText('Double-click to edit', {
        left: 220, top: 160, fontFamily: fontSelect.value, fontSize: parseInt(fontSizeInput.value,10) || 40, fill: colorPicker.value, editable: true
      });
      canvas.add(txt).setActiveObject(txt);
      updateLayersPanel();
      scheduleAutosave();
    });

    // Object property changes when selection changes
    canvas.on('selection:updated', updatePanel);
    canvas.on('selection:created', updatePanel);
    canvas.on('selection:cleared', () => { propPanel.innerHTML = ''; updateLayersPanel(); });

    canvas.on('object:modified', () => { updateLayersPanel(); scheduleAutosave(); });
    canvas.on('object:added', () => { updateLayersPanel(); scheduleAutosave(); });
    canvas.on('object:removed', () => { updateLayersPanel(); scheduleAutosave(); });

    function updatePanel(e){
      const o = canvas.getActiveObject();
      renderPropPanel(o);
      updateLayersPanel();
    }

    function renderPropPanel(obj){
      if(!obj){ propPanel.innerHTML = ''; return; }
      // Basic controls
      const type = obj.type || 'object';
      let html = `<div style="display:flex;flex-direction:column;gap:8px">`;
      html += `<div class="small-muted">Type: ${type}</div>`;
      if(type === 'i-text' || type === 'textbox' || obj.isType && obj.isType === 'i-text'){
        html += `<label class="small">Text</label><textarea id="prop_text" style="width:100%;min-height:72px">${obj.text}</textarea>`;
        html += `<div style="display:flex;gap:8px"><label>Font <select id="prop_font">${['Poppins','Roboto','Arial','Georgia','Courier New'].map(f=>`<option ${obj.fontFamily===f?'selected':''}>${f}</option>`).join('')}</select></label><label>Size <input id="prop_size" type="number" value="${obj.fontSize}" style="width:80px" /></label></div>`;
        html += `<div style="display:flex;gap:8px;align-items:center"><label>Color <input id="prop_color" type="color" value="${obj.fill || '#111'}" /></label><button id="prop_bold">${obj.fontWeight==='700'?'Unbold':'Bold'}</button><button id="prop_italic">${obj.fontStyle==='italic'?'Unitalic':'Italic'}</button><button id="prop_underline">${obj.underline?'Ununderline':'Underline'}</button></div>`;
      } else if(obj.type === 'image' || obj.type === 'group'){
        html += `<div class="small-muted">Image object</div>`;
      } else {
        html += `<div class="small-muted">Object</div>`;
      }

      html += `<div style="display:flex;gap:8px;margin-top:6px"><label>Opacity <input id="prop_op" type="range" min="0" max="1" step="0.01" value="${obj.opacity || 1}" /></label></div>`;
      html += `</div>`;
      propPanel.innerHTML = html;

      // Wire events
      const prop_text = document.getElementById('prop_text');
      if(prop_text){ prop_text.addEventListener('input', (ev)=>{ obj.text = ev.target.value; canvas.requestRenderAll(); scheduleAutosave(); }); }
      const prop_font = document.getElementById('prop_font');
      if(prop_font){ prop_font.addEventListener('change', (ev)=>{ obj.fontFamily = ev.target.value; canvas.requestRenderAll(); scheduleAutosave(); }); }
      const prop_size = document.getElementById('prop_size');
      if(prop_size){ prop_size.addEventListener('change', (ev)=>{ obj.fontSize = parseInt(ev.target.value || 12); canvas.requestRenderAll(); scheduleAutosave(); }); }
      const prop_color = document.getElementById('prop_color');
      if(prop_color){ prop_color.addEventListener('change', (ev)=>{ obj.set('fill', ev.target.value); canvas.requestRenderAll(); scheduleAutosave(); }); }
      const prop_bold = document.getElementById('prop_bold');
      if(prop_bold){ prop_bold.addEventListener('click', ()=>{ obj.fontWeight = (obj.fontWeight === '700') ? 'normal' : '700'; canvas.requestRenderAll(); scheduleAutosave(); renderPropPanel(obj); }); }
      const prop_italic = document.getElementById('prop_italic');
      if(prop_italic){ prop_italic.addEventListener('click', ()=>{ obj.fontStyle = (obj.fontStyle === 'italic') ? 'normal' : 'italic'; canvas.requestRenderAll(); scheduleAutosave(); renderPropPanel(obj); }); }
      const prop_underline = document.getElementById('prop_underline');
      if(prop_underline){ prop_underline.addEventListener('click', ()=>{ obj.underline = !obj.underline; canvas.requestRenderAll(); scheduleAutosave(); renderPropPanel(obj); }); }
      const prop_op = document.getElementById('prop_op');
      if(prop_op){ prop_op.addEventListener('input', (ev)=>{ obj.opacity = parseFloat(ev.target.value); canvas.requestRenderAll(); scheduleAutosave(); }); }
    }

    // Toolbar actions
    boldBtn.addEventListener('click', () => {
      const o = canvas.getActiveObject(); if(!o || (o.type!=='i-text' && o.type!=='textbox')) return;
      o.fontWeight = (o.fontWeight === '700') ? 'normal' : '700';
      canvas.requestRenderAll(); scheduleAutosave(); renderPropPanel(o);
    });

    italicBtn.addEventListener('click', () => {
      const o = canvas.getActiveObject(); if(!o || (o.type!=='i-text' && o.type!=='textbox')) return;
      o.fontStyle = (o.fontStyle === 'italic') ? 'normal' : 'italic';
      canvas.requestRenderAll(); scheduleAutosave(); renderPropPanel(o);
    });

    underlineBtn.addEventListener('click', () => {
      const o = canvas.getActiveObject(); if(!o || (o.type!=='i-text' && o.type!=='textbox')) return;
      o.underline = !o.underline;
      canvas.requestRenderAll(); scheduleAutosave(); renderPropPanel(o);
    });

    fontSelect.addEventListener('change', () => {
      const o = canvas.getActiveObject(); if(!o) return;
      if(o.type==='i-text' || o.type==='textbox'){ o.fontFamily = fontSelect.value; canvas.requestRenderAll(); scheduleAutosave(); renderPropPanel(o); }
    });

    fontSizeInput.addEventListener('change', () => {
      const o = canvas.getActiveObject(); if(!o) return;
      const size = parseInt(fontSizeInput.value,10) || 12;
      if(o.type==='i-text' || o.type==='textbox'){ o.fontSize = size; canvas.requestRenderAll(); scheduleAutosave(); renderPropPanel(o); }
    });

    colorPicker.addEventListener('change', () => {
      const o = canvas.getActiveObject(); if(!o) return;
      if(o.type==='i-text' || o.type==='textbox'){ o.set('fill', colorPicker.value); } else { o.set('tint', colorPicker.value); }
      canvas.requestRenderAll(); scheduleAutosave();
    });

    bringForwardBtn.addEventListener('click', () => {
      const o = canvas.getActiveObject(); if(!o) return;
      canvas.bringForward(o); canvas.requestRenderAll(); updateLayersPanel(); scheduleAutosave();
    });

    sendBackwardBtn.addEventListener('click', () => {
      const o = canvas.getActiveObject(); if(!o) return;
      canvas.sendBackwards(o); canvas.requestRenderAll(); updateLayersPanel(); scheduleAutosave();
    });

    duplicateBtn.addEventListener('click', () => {
      const o = canvas.getActiveObject(); if(!o) return;
      o.clone(function(cloned){
        cloned.set({ left: o.left + 20, top: o.top + 20 });
        canvas.add(cloned);
        canvas.setActiveObject(cloned);
        canvas.requestRenderAll();
        updateLayersPanel();
        scheduleAutosave();
      });
    });

    deleteBtn.addEventListener('click', () => {
      const o = canvas.getActiveObject(); if(!o) return;
      canvas.remove(o); canvas.discardActiveObject(); canvas.requestRenderAll(); updateLayersPanel(); scheduleAutosave();
    });

    // Templates
    templateBtn.addEventListener('click', () => {
      applyTemplate();
      scheduleAutosave();
    });

    function applyTemplate(){
      canvas.clear(); canvas.setBackgroundColor('#fff', canvas.renderAll.bind(canvas));
      // Add colored background rectangle
      const bgRect = new fabric.Rect({ left:0, top:0, width:1000, height:620, fill:'#fffbeb', selectable:false });
      canvas.add(bgRect);
      // Add an image (using a placeholder data URL or remote image)
      fabric.Image.fromURL('https://images.unsplash.com/photo-1547658719-da2b51169166?q=80&w=1400&auto=format&fit=crop&ixlib=rb-4.0.3&s=1f0c81c9a3a6dbd6d6b2d1b3e9b3b0b8', function(img){
        img.set({ left:50, top:80, scaleX:0.26, scaleY:0.26, selectable:true });
        canvas.add(img);
      }, { crossOrigin:'anonymous' });

      // Add text headline
      const headline = new fabric.IText('Summer Sale\nBig Savings', {
        left: 520, top: 160, fontFamily:'Poppins', fontSize:56, fontWeight:'700', fill:'#111827', textAlign:'left'
      });
      canvas.add(headline);

      // Add CTA box
      const cta = new fabric.Rect({ left:520, top:340, width:220, height:60, rx:8, fill:'#2563eb' , selectable:false });
      const ctaText = new fabric.IText('Shop Now', { left:630, top:370, fontFamily:'Poppins', fontSize:20, fill:'#fff', originX:'center', originY:'center' });
      ctaText.set({ selectable:true });
      canvas.add(cta); canvas.add(ctaText);
      canvas.requestRenderAll();
      updateLayersPanel();
    }

    // Save / Load
    function saveProject(silent=false){
      // Fabric canvas to JSON (includes images src)
      const json = canvas.toJSON(['selectable','id']);
      // Because fabric serializes images with src, it will persist base64 or remote URLs
      localStorage.setItem('canva_lite_project_v1', JSON.stringify(json));
      if(!silent) alert('Project saved to localStorage.');
    }

    function loadProject(){
      const raw = localStorage.getItem('canva_lite_project_v1');
      if(!raw){ alert('No saved project found'); return; }
      try{
        const obj = JSON.parse(raw);
        // Use loadFromJSON to restore images as well
        canvas.loadFromJSON(obj, canvas.renderAll.bind(canvas), function(o, object){
          // optional per-object callback
        });
        setTimeout(()=> { updateLayersPanel(); }, 300);
      }catch(e){ alert('Failed to load project'); console.error(e); }
    }

    saveBtn.addEventListener('click', ()=> saveProject(false));
    loadBtn.addEventListener('click', ()=> loadProject());

    // Export PNG
    exportBtn.addEventListener('click', ()=> {
      // export at 2x pixel ratio for better quality
      const dataURL = canvas.toDataURL({ format: 'png', multiplier: 2 });
      const a = document.createElement('a'); a.href = dataURL; a.download = 'design.png'; a.click();
    });

    // Clear
    clearBtn.addEventListener('click', ()=> {
      if(!confirm('Clear canvas?')) return;
      canvas.clear(); canvas.setBackgroundColor('#fff', canvas.renderAll.bind(canvas));
      updateLayersPanel();
      scheduleAutosave();
    });

    // Layers panel
    function updateLayersPanel(){
      const objs = canvas.getObjects().slice().reverse(); // top-first
      layersBox.innerHTML = objs.map((o, idx) => {
        const id = o.__uid || (o.__uid = uid('o'));
        const type = (o.type || 'object').toUpperCase();
        const label = (o.type && (o.type.indexOf('text')>-1) && o.text) ? (String(o.text).slice(0,30)) : type;
        return `<div class="layer-item" data-uid="${id}" data-index="${objs.length-1-idx}">
                  <div style="cursor:pointer">${label}</div>
                  <div style="display:flex;gap:4px">
                    <button class="layer-select" data-uid="${id}" data-index="${objs.length-1-idx}">Select</button>
                  </div>
                </div>`;
      }).join('');
      // bind selects
      Array.from(layersBox.querySelectorAll('.layer-select')).forEach(btn => {
        btn.addEventListener('click', (ev) => {
          const idx = parseInt(btn.dataset.index,10);
          const obj = canvas.getObjects()[idx];
          if(obj){ canvas.setActiveObject(obj); canvas.requestRenderAll(); renderPropPanel(obj); }
        });
      });
    }

    // Initialize canvas default background
    canvas.setBackgroundColor('#fff', canvas.renderAll.bind(canvas));
    updateLayersPanel();

    // Double-click to edit text (Fabric IText supports editing by default)
    // Already supported for IText; nothing extra needed.

    // Load saved automatically on start if present
    window.addEventListener('load', ()=> {
      try{
        const raw = localStorage.getItem('canva_lite_project_v1');
        if(raw){ loadProject(); }
      } catch(e){}
    });

    // Update layers when objects change
    canvas.on('object:modified', updateLayersPanel);
    canvas.on('object:added', updateLayersPanel);
    canvas.on('object:removed', updateLayersPanel);

    // Autosave toggle
    autosaveChk.addEventListener('change', ()=> {
      if(autosaveChk.checked) scheduleAutosave();
      else if(autosaveTimer) clearTimeout(autosaveTimer);
    });

    // Keyboard shortcuts
    window.addEventListener('keydown', (e) => {
      if((e.ctrlKey || e.metaKey) && e.key === 's'){ e.preventDefault(); saveProject(); }
      if(e.key === 'Delete' || e.key === 'Backspace'){ const o = canvas.getActiveObject(); if(o){ canvas.remove(o); updateLayersPanel(); scheduleAutosave(); } }
      if((e.ctrlKey || e.metaKey) && e.key.toLowerCase() === 'd'){ e.preventDefault(); const o = canvas.getActiveObject(); if(o){ o.clone(function(cloned){ cloned.set({ left: o.left + 20, top: o.top + 20 }); canvas.add(cloned); canvas.setActiveObject(cloned); updateLayersPanel(); scheduleAutosave(); }); } }
    });
  </script>
</body>
</html>
