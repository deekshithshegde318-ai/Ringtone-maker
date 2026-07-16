<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Splice — Ringtone Cutter</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Bebas+Neue&family=IBM+Plex+Mono:wght@400;500;600&family=IBM+Plex+Sans:wght@400;500;600&display=swap" rel="stylesheet">
<style>
  :root{
    --bg: #1c1815;
    --bg-raised: #241f1a;
    --tape: #2c241c;
    --amber: #e8873a;
    --amber-dim: #a85f28;
    --cream: #f2e9dd;
    --cream-dim: #b8ac9a;
    --moss: #6b7355;
    --cut: #c1432a;
    --line: #3a3128;
  }
  *{box-sizing:border-box; -webkit-tap-highlight-color: transparent;}
  html,body{margin:0; padding:0;}
  body{
    background: var(--bg);
    color: var(--cream);
    font-family: 'IBM Plex Sans', sans-serif;
    min-height:100vh;
    display:flex;
    justify-content:center;
    padding: 32px 16px 80px;
  }
  .app{ width:100%; max-width: 640px; }

  header{ margin-bottom: 28px; }
  .eyebrow{
    font-family:'IBM Plex Mono', monospace;
    font-size:11px;
    letter-spacing: 0.18em;
    text-transform:uppercase;
    color: var(--amber);
    margin: 0 0 6px;
  }
  h1{
    font-family:'Bebas Neue', sans-serif;
    font-size: 52px;
    letter-spacing: 0.02em;
    margin: 0 0 8px;
    line-height: 0.95;
    color: var(--cream);
  }
  .sub{
    color: var(--cream-dim);
    font-size: 14.5px;
    line-height: 1.5;
    max-width: 46ch;
    margin: 0;
  }

  .panel{
    background: var(--bg-raised);
    border: 1px solid var(--line);
    border-radius: 6px;
    padding: 24px;
    margin-top: 22px;
  }

  /* Drop zone */
  .dropzone{
    border: 2px dashed var(--line);
    border-radius: 6px;
    padding: 40px 20px;
    text-align:center;
    cursor:pointer;
    transition: border-color .15s ease, background .15s ease;
    position:relative;
  }
  .dropzone.drag{ border-color: var(--amber); background: rgba(232,135,58,0.06); }
  .dropzone .icon{
    width:40px; height:40px; margin:0 auto 14px;
    border:2px solid var(--cream-dim);
    border-radius:3px;
    position:relative;
  }
  .dropzone .icon::before{
    content:'';
    position:absolute; left:50%; top:50%;
    width:14px; height:14px;
    border-left:2px solid var(--cream-dim);
    border-bottom:2px solid var(--cream-dim);
    transform: translate(-50%,-65%) rotate(45deg);
  }
  .dropzone p{ margin:4px 0; color:var(--cream); font-size:15px; }
  .dropzone .hint{ color: var(--cream-dim); font-size: 12.5px; }
  input[type=file]{ display:none; }

  .disclaimer{
    font-family:'IBM Plex Mono', monospace;
    font-size: 11px;
    color: var(--cream-dim);
    line-height: 1.6;
    border-left: 2px solid var(--moss);
    padding-left: 10px;
    margin-top: 16px;
  }

  /* Editor */
  #editor{ display:none; }
  .file-row{
    display:flex; justify-content:space-between; align-items:baseline;
    margin-bottom: 16px; flex-wrap: wrap; gap: 6px;
  }
  .filename{ font-size: 15px; color: var(--cream); font-weight:500; }
  .filemeta{ font-family:'IBM Plex Mono', monospace; font-size: 11.5px; color: var(--cream-dim); }
  .swap-btn{
    background:none; border:none; color: var(--amber); font-size:12px;
    font-family:'IBM Plex Mono', monospace; cursor:pointer; padding:0;
    text-decoration: underline; text-underline-offset:3px;
  }

  /* Tape strip */
  .tape-strip{
    position:relative;
    background: var(--tape);
    border-radius: 4px;
    padding: 10px 0;
    user-select:none;
    touch-action: none;
  }
  .sprockets{
    display:flex; justify-content:space-between;
    padding: 0 6px;
  }
  .sprockets span{
    width:6px; height:6px; border-radius:50%;
    background: var(--bg);
    box-shadow: inset 0 0 0 1px rgba(0,0,0,0.4);
  }
  .waveform-wrap{
    position:relative;
    height: 110px;
    margin: 8px 6px;
    border-radius: 3px;
    overflow:hidden;
    background: #171310;
  }
  canvas#wave{ position:absolute; inset:0; width:100%; height:100%; display:block; }
  .dim-region{
    position:absolute; top:0; bottom:0;
    background: rgba(0,0,0,0.55);
    pointer-events:none;
  }
  .select-region{
    position:absolute; top:0; bottom:0;
    background: rgba(232,135,58,0.10);
    border-left: 1px solid var(--amber-dim);
    border-right: 1px solid var(--amber-dim);
    pointer-events:none;
  }
  .playhead{
    position:absolute; top:0; bottom:0; width:1px;
    background: var(--cream);
    box-shadow: 0 0 6px var(--cream);
    pointer-events:none;
    display:none;
  }
  .handle{
    position:absolute; top:-2px; bottom:-2px;
    width: 14px; margin-left:-7px;
    cursor: ew-resize;
    display:flex; align-items:center; justify-content:center;
  }
  .handle .blade{
    width: 3px; height: 100%;
    background: linear-gradient(180deg, var(--amber), var(--cut));
    border-radius: 2px;
    box-shadow: 0 0 0 1px rgba(0,0,0,0.3);
  }
  .handle::after{
    content:'';
    position:absolute; top:-5px; left:50%; transform:translateX(-50%);
    width:0; height:0;
    border-left:5px solid transparent;
    border-right:5px solid transparent;
    border-top:6px solid var(--amber);
  }
  .handle-label{
    position:absolute; top:-24px; left:50%; transform:translateX(-50%);
    font-family:'IBM Plex Mono', monospace;
    font-size: 10.5px;
    color: var(--bg);
    background: var(--amber);
    padding: 2px 5px;
    border-radius: 3px;
    white-space:nowrap;
  }

  /* Timecodes */
  .timecodes{
    display:flex; gap: 14px; margin-top: 14px;
    font-family:'IBM Plex Mono', monospace;
  }
  .tc-box{ flex:1; }
  .tc-box label{
    display:block; font-size:10px; letter-spacing:0.1em; text-transform:uppercase;
    color: var(--cream-dim); margin-bottom:4px;
  }
  .tc-box .val{
    background: var(--tape);
    border: 1px solid var(--line);
    border-radius: 4px;
    padding: 8px 10px;
    font-size: 14px;
    color: var(--amber);
  }
  .tc-box.length .val{ color: var(--cream); }

  .warn{
    font-family:'IBM Plex Mono', monospace;
    font-size: 11.5px;
    color: var(--cut);
    margin-top: 10px;
    display:none;
  }

  /* Controls */
  .controls-row{
    display:flex; align-items:center; justify-content:space-between;
    margin-top: 20px; flex-wrap:wrap; gap:12px;
  }
  .fade-toggle{
    display:flex; align-items:center; gap:8px;
    font-size: 13px; color: var(--cream-dim);
    font-family:'IBM Plex Mono', monospace;
  }
  .fade-toggle input{ accent-color: var(--amber); width:14px; height:14px; }

  .btn{
    font-family: 'IBM Plex Sans', sans-serif;
    font-weight:600;
    font-size: 14px;
    border-radius: 5px;
    padding: 11px 18px;
    border: none;
    cursor:pointer;
    transition: transform .08s ease, opacity .15s ease;
  }
  .btn:active{ transform: scale(0.97); }
  .btn-play{
    background: var(--bg);
    border: 1px solid var(--amber-dim);
    color: var(--amber);
    display:flex; align-items:center; gap:8px;
  }
  .btn-play:hover{ border-color: var(--amber); }
  .btn-export{
    background: var(--amber);
    color: var(--bg);
  }
  .btn-export:hover{ background:#f2984d; }
  .btn-export:disabled{ opacity:0.5; cursor:not-allowed; }

  .name-row{ margin-top: 18px; }
  .name-row label{
    display:block; font-size:10px; letter-spacing:0.1em; text-transform:uppercase;
    color: var(--cream-dim); margin-bottom:6px;
    font-family:'IBM Plex Mono', monospace;
  }
  .name-row input{
    width:100%;
    background: var(--tape);
    border: 1px solid var(--line);
    color: var(--cream);
    border-radius:5px;
    padding:10px 12px;
    font-size:14px;
    font-family:'IBM Plex Sans', sans-serif;
  }
  .name-row input:focus{ outline:1px solid var(--amber); }

  .icon-play{ width:0; height:0; border-top:6px solid transparent; border-bottom:6px solid transparent; border-left:9px solid var(--amber); }
  .icon-pause{ width:9px; height:12px; display:flex; gap:3px; }
  .icon-pause span{ width:3px; background:var(--amber); }

  footer{
    margin-top: 26px;
    font-family:'IBM Plex Mono', monospace;
    font-size: 11px;
    color: var(--cream-dim);
    text-align:center;
  }

  :focus-visible{ outline: 2px solid var(--amber); outline-offset: 2px; }

  @media (prefers-reduced-motion: reduce){
    .btn{ transition:none; }
  }
</style>
</head>
<body>
<div class="app">
  <header>
    <p class="eyebrow">Reel 001 · Splice bay</p>
    <h1>SPLICE</h1>
    <p class="sub">Load a track you already own, mark your cut with the blade, and print a ringtone. No streaming lookups — just you, the tape, and a razor.</p>
  </header>

  <div class="panel" id="uploadPanel">
    <div class="dropzone" id="dropzone" tabindex="0" role="button" aria-label="Upload audio file">
      <div class="icon"></div>
      <p>Drop an audio file here, or tap to browse</p>
      <p class="hint">MP3, WAV, M4A, OGG — your own files only</p>
      <input type="file" id="fileInput" accept="audio/*">
    </div>
    <p class="disclaimer">Splice works on audio you already have rights to use. It can't fetch songs from streaming services — that would mean distributing copyrighted recordings without permission, which we don't build.</p>
  </div>

  <div class="panel" id="editor">
    <div class="file-row">
      <div>
        <div class="filename" id="filename">—</div>
        <div class="filemeta" id="filemeta">—</div>
      </div>
      <button class="swap-btn" id="swapBtn">Load a different file</button>
    </div>

    <div class="tape-strip">
      <div class="sprockets" id="sprocketsTop"></div>
      <div class="waveform-wrap" id="waveWrap">
        <canvas id="wave"></canvas>
        <div class="dim-region" id="dimLeft"></div>
        <div class="dim-region" id="dimRight"></div>
        <div class="select-region" id="selectRegion"></div>
        <div class="playhead" id="playhead"></div>
        <div class="handle" id="handleStart"><div class="handle-label" id="labelStart">0:00.00</div><div class="blade"></div></div>
        <div class="handle" id="handleEnd"><div class="handle-label" id="labelEnd">0:00.00</div><div class="blade"></div></div>
      </div>
      <div class="sprockets" id="sprocketsBottom"></div>
    </div>

    <div class="timecodes">
      <div class="tc-box"><label>In point</label><div class="val" id="tcStart">0:00.00</div></div>
      <div class="tc-box"><label>Out point</label><div class="val" id="tcEnd">0:00.00</div></div>
      <div class="tc-box length"><label>Clip length</label><div class="val" id="tcLength">0:00.00</div></div>
    </div>
    <p class="warn" id="lengthWarn">Clips over 40s may get truncated as a ringtone by some phones.</p>

    <div class="controls-row">
      <label class="fade-toggle"><input type="checkbox" id="fadeToggle" checked> Fade in/out (120ms)</label>
      <button class="btn btn-play" id="playBtn">
        <span class="icon-play" id="playIcon"></span>
        <span id="playLabel">Preview cut</span>
      </button>
    </div>

    <div class="name-row">
      <label for="ringName">Ringtone name</label>
      <input type="text" id="ringName" placeholder="e.g. midnight-drive-chorus">
    </div>

    <div class="controls-row" style="justify-content:flex-end; margin-top:16px;">
      <button class="btn btn-export" id="exportBtn">Export ringtone (.wav)</button>
    </div>
  </div>

  <footer>Everything happens in your browser — nothing you upload leaves this tab.</footer>
</div>

<script>
(function(){
  const dropzone = document.getElementById('dropzone');
  const fileInput = document.getElementById('fileInput');
  const uploadPanel = document.getElementById('uploadPanel');
  const editor = document.getElementById('editor');
  const swapBtn = document.getElementById('swapBtn');
  const filenameEl = document.getElementById('filename');
  const filemetaEl = document.getElementById('filemeta');
  const waveWrap = document.getElementById('waveWrap');
  const canvas = document.getElementById('wave');
  const ctx2d = canvas.getContext('2d');
  const dimLeft = document.getElementById('dimLeft');
  const dimRight = document.getElementById('dimRight');
  const selectRegion = document.getElementById('selectRegion');
  const playhead = document.getElementById('playhead');
  const handleStart = document.getElementById('handleStart');
  const handleEnd = document.getElementById('handleEnd');
  const labelStart = document.getElementById('labelStart');
  const labelEnd = document.getElementById('labelEnd');
  const tcStart = document.getElementById('tcStart');
  const tcEnd = document.getElementById('tcEnd');
  const tcLength = document.getElementById('tcLength');
  const lengthWarn = document.getElementById('lengthWarn');
  const playBtn = document.getElementById('playBtn');
  const playIcon = document.getElementById('playIcon');
  const playLabel = document.getElementById('playLabel');
  const exportBtn = document.getElementById('exportBtn');
  const fadeToggle = document.getElementById('fadeToggle');
  const ringName = document.getElementById('ringName');
  const sprocketsTop = document.getElementById('sprocketsTop');
  const sprocketsBottom = document.getElementById('sprocketsBottom');

  let audioCtx = null;
  let audioBuffer = null;
  let startPct = 0, endPct = 1;
  let dragging = null;
  let sourceNode = null;
  let isPlaying = false;
  let playStartTime = 0;
  let rafId = null;

  // Build sprocket holes
  for(let i=0;i<28;i++){
    const s1 = document.createElement('span');
    const s2 = document.createElement('span');
    sprocketsTop.appendChild(s1);
    sprocketsBottom.appendChild(s2);
  }

  function getCtx(){
    if(!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    return audioCtx;
  }

  // ---- File loading ----
  dropzone.addEventListener('click', () => fileInput.click());
  dropzone.addEventListener('keydown', e => { if(e.key === 'Enter' || e.key === ' ') fileInput.click(); });
  dropzone.addEventListener('dragover', e => { e.preventDefault(); dropzone.classList.add('drag'); });
  dropzone.addEventListener('dragleave', () => dropzone.classList.remove('drag'));
  dropzone.addEventListener('drop', e => {
    e.preventDefault();
    dropzone.classList.remove('drag');
    if(e.dataTransfer.files.length) loadFile(e.dataTransfer.files[0]);
  });
  fileInput.addEventListener('change', e => { if(e.target.files.length) loadFile(e.target.files[0]); });
  swapBtn.addEventListener('click', () => {
    editor.style.display = 'none';
    uploadPanel.style.display = 'block';
    stopPlayback();
  });

  function loadFile(file){
    const reader = new FileReader();
    reader.onload = async (e) => {
      try{
        const ctx = getCtx();
        const decoded = await ctx.decodeAudioData(e.target.result.slice(0));
        audioBuffer = decoded;
        startPct = 0;
        endPct = Math.min(1, (30 / decoded.duration)) || 1;
        if(decoded.duration <= 30) endPct = 1;
        filenameEl.textContent = file.name;
        const mins = Math.floor(decoded.duration/60);
        const secs = Math.round(decoded.duration%60);
        filemetaEl.textContent = `${mins}:${secs.toString().padStart(2,'0')} · ${decoded.sampleRate/1000}kHz · ${decoded.numberOfChannels === 2 ? 'stereo' : 'mono'}`;
        ringName.value = file.name.replace(/\.[^/.]+$/, '');
        uploadPanel.style.display = 'none';
        editor.style.display = 'block';
        requestAnimationFrame(() => {
          resizeCanvas();
          drawWaveform();
          updateHandles();
        });
      }catch(err){
        alert('Could not decode that audio file. Try an MP3 or WAV.');
      }
    };
    reader.readAsArrayBuffer(file);
  }

  // ---- Waveform drawing ----
  function resizeCanvas(){
    const rect = waveWrap.getBoundingClientRect();
    const dpr = window.devicePixelRatio || 1;
    canvas.width = rect.width * dpr;
    canvas.height = rect.height * dpr;
    ctx2d.setTransform(dpr,0,0,dpr,0,0);
  }

  function drawWaveform(){
    if(!audioBuffer) return;
    const rect = waveWrap.getBoundingClientRect();
    const w = rect.width, h = rect.height;
    ctx2d.clearRect(0,0,w,h);
    const ch0 = audioBuffer.getChannelData(0);
    const ch1 = audioBuffer.numberOfChannels > 1 ? audioBuffer.getChannelData(1) : null;
    const len = ch0.length;
    const samplesPerPx = Math.max(1, Math.floor(len / w));
    const mid = h/2;
    ctx2d.strokeStyle = '#e8873a';
    ctx2d.lineWidth = 1;
    ctx2d.beginPath();
    for(let x=0; x<w; x++){
      const start = x * samplesPerPx;
      let min = 1.0, max = -1.0;
      for(let i=0; i<samplesPerPx; i++){
        const idx = start + i;
        if(idx >= len) break;
        let v = ch0[idx];
        if(ch1) v = (v + ch1[idx]) / 2;
        if(v < min) min = v;
        if(v > max) max = v;
      }
      const y1 = mid + min * mid * 0.9;
      const y2 = mid + max * mid * 0.9;
      ctx2d.moveTo(x + 0.5, y1);
      ctx2d.lineTo(x + 0.5, y2);
    }
    ctx2d.stroke();
  }

  window.addEventListener('resize', () => {
    if(audioBuffer){ resizeCanvas(); drawWaveform(); }
  });

  // ---- Handles / selection ----
  function fmtTime(sec){
    const m = Math.floor(sec/60);
    const s = sec - m*60;
    return `${m}:${s.toFixed(2).padStart(5,'0')}`;
  }

  function updateHandles(){
    const sPct = startPct * 100;
    const ePct = endPct * 100;
    handleStart.style.left = sPct + '%';
    handleEnd.style.left = ePct + '%';
    dimLeft.style.left = '0'; dimLeft.style.width = sPct + '%';
    dimRight.style.left = ePct + '%'; dimRight.style.width = (100-ePct) + '%';
    selectRegion.style.left = sPct + '%'; selectRegion.style.width = (ePct-sPct) + '%';

    const startSec = startPct * audioBuffer.duration;
    const endSec = endPct * audioBuffer.duration;
    const lenSec = endSec - startSec;

    labelStart.textContent = fmtTime(startSec);
    labelEnd.textContent = fmtTime(endSec);
    tcStart.textContent = fmtTime(startSec);
    tcEnd.textContent = fmtTime(endSec);
    tcLength.textContent = fmtTime(lenSec);

    lengthWarn.style.display = lenSec > 40 ? 'block' : 'none';
  }

  function pctFromClientX(clientX){
    const rect = waveWrap.getBoundingClientRect();
    let pct = (clientX - rect.left) / rect.width;
    return Math.min(1, Math.max(0, pct));
  }

  function attachDrag(handle, which){
    handle.addEventListener('pointerdown', e => {
      dragging = which;
      handle.setPointerCapture(e.pointerId);
      e.preventDefault();
    });
  }
  attachDrag(handleStart, 'start');
  attachDrag(handleEnd, 'end');

  window.addEventListener('pointermove', e => {
    if(!dragging || !audioBuffer) return;
    const pct = pctFromClientX(e.clientX);
    const minGapPct = Math.min(0.02, 0.5/audioBuffer.duration);
    if(dragging === 'start'){
      startPct = Math.min(pct, endPct - minGapPct);
      startPct = Math.max(0, startPct);
    } else {
      endPct = Math.max(pct, startPct + minGapPct);
      endPct = Math.min(1, endPct);
    }
    updateHandles();
  });
  window.addEventListener('pointerup', () => { dragging = null; });

  // ---- Playback preview ----
  function stopPlayback(){
    if(sourceNode){
      try{ sourceNode.stop(); }catch(e){}
      sourceNode.disconnect();
      sourceNode = null;
    }
    isPlaying = false;
    playIcon.className = 'icon-play';
    playLabel.textContent = 'Preview cut';
    playhead.style.display = 'none';
    if(rafId) cancelAnimationFrame(rafId);
  }

  playBtn.addEventListener('click', () => {
    if(!audioBuffer) return;
    if(isPlaying){ stopPlayback(); return; }
    const ctx = getCtx();
    if(ctx.state === 'suspended') ctx.resume();
    const startSec = startPct * audioBuffer.duration;
    const endSec = endPct * audioBuffer.duration;
    const dur = endSec - startSec;
    sourceNode = ctx.createBufferSource();
    sourceNode.buffer = audioBuffer;
    const gain = ctx.createGain();
    sourceNode.connect(gain);
    gain.connect(ctx.destination);
    sourceNode.start(0, startSec, dur);
    playStartTime = ctx.currentTime;
    isPlaying = true;
    playIcon.outerHTML = '<span class;
