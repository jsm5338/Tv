# Tv[index.html](https://github.com/user-attachments/files/33409061/index.html)
<!doctype html>
<html lang="ko">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>MIDI 반주기 · 다층 재생 진행바</title>
<style>
  :root{
    --bg:#0E1014;
    --panel:#16191F;
    --stage:#2B111A;
    --stroke:#242830;
    --track:#2B3038;
    --buffered:#4D545F;
    --accent:#FF7A3C;
    --accent2:#FF5A18;
    --text:#F2F3F5;
    --muted:#9AA0AA;
    --bar-h:4px;
    --bar-h-active:9px;
    --thumb:13px;
  }
  *{box-sizing:border-box;-webkit-tap-highlight-color:transparent}
  body{
    margin:0;padding:24px 16px 48px;
    background:radial-gradient(120% 80% at 50% 0%,#171B22 0%,var(--bg) 60%);
    color:var(--text);
    font-family:-apple-system,BlinkMacSystemFont,"Segoe UI","Noto Sans KR",Roboto,sans-serif;
    display:flex;justify-content:center;
  }
  .phone{
    width:min(430px,100%);
    background:var(--panel);
    border:1px solid var(--stroke);
    border-radius:26px;
    padding:16px 14px 18px;
    box-shadow:0 24px 60px rgba(0,0,0,.6);
  }

  /* ---------- 상단 헤더 ---------- */
  .hdr{display:flex;align-items:center;gap:10px}
  .mic{
    width:42px;height:42px;flex:0 0 42px;border-radius:12px;
    background:linear-gradient(180deg,var(--accent),var(--accent2));
    display:grid;place-items:center;font-size:20px;
    box-shadow:0 6px 16px rgba(255,90,24,.35);
  }
  .hdr h1{margin:0;font-size:19px;letter-spacing:-.4px;display:flex;align-items:center;gap:6px}
  .badge{font-size:10px;font-weight:700;border:1px solid #6b7480;color:#c9ced6;border-radius:5px;padding:1px 5px}
  .sub{margin:2px 0 0;font-size:11px;color:var(--muted)}
  .gear{margin-left:auto;font-size:18px;color:#8d939d}

  /* ---------- 검색 + 아이콘 ---------- */
  .row{display:flex;gap:8px;margin-top:14px}
  .search{
    flex:1;display:flex;align-items:center;gap:8px;
    background:#20242B;border:1px solid var(--stroke);border-radius:14px;
    padding:10px 12px;color:#7f868f;font-size:13px;min-width:0;
  }
  .search span{overflow:hidden;white-space:nowrap;text-overflow:ellipsis}
  .ico{width:44px;flex:0 0 44px;height:40px;border-radius:12px;background:#20242B;border:1px solid var(--stroke);display:grid;place-items:center;font-size:17px}

  /* ---------- 탭 ---------- */
  .tabs{display:flex;gap:10px;margin-top:12px}
  .tab{
    flex:1;height:40px;border-radius:12px;display:grid;place-items:center;
    font-size:13px;font-weight:700;border:1px solid var(--stroke);
    background:#1B1F26;color:#aeb4bd;
  }
  .tab.on{background:linear-gradient(180deg,var(--accent),var(--accent2));color:#fff;border-color:transparent;box-shadow:0 6px 18px rgba(255,90,24,.3)}
  .chips{display:flex;gap:8px;margin-top:10px;overflow-x:auto;scrollbar-width:none}
  .chips::-webkit-scrollbar{display:none}
  .chip{flex:0 0 auto;padding:7px 14px;border-radius:999px;background:#1B1F26;border:1px solid var(--stroke);font-size:12px;color:#b6bcc4}
  .chip.on{background:linear-gradient(180deg,var(--accent),var(--accent2));color:#fff;border-color:transparent}

  /* ---------- 무대(스테이지) ---------- */
  .stage{
    margin-top:14px;border-radius:16px;background:
      radial-gradient(90% 60% at 50% 62%,rgba(255,90,24,.10) 0%,transparent 70%),
      linear-gradient(180deg,#2C121B,#1A0D13);
    border:1px solid #33161f;
    padding:18px 14px 12px;
    min-height:360px;
    display:flex;flex-direction:column;
  }
  .song{text-align:center;font-size:24px;font-weight:800;color:#F5E3E8;letter-spacing:-.5px;line-height:1.25;padding:6px 0 2px}
  .wave{flex:1;display:grid;place-items:center;padding:6px 0}
  .wave svg{width:100%;height:auto;overflow:visible}

  /* ================= 다층 재생 진행바 ================= */
  .seek-wrap{position:relative;margin-top:6px;padding:16px 0 2px}   /* 넉넉한 터치 영역 */
  .seek{
    position:relative;
    height:var(--bar-h);
    border-radius:999px;
    background:var(--track);                 /* 1층 트랙 */
    cursor:pointer;
    touch-action:none;                        /* 모바일 스크롤/줌 차단 */
    outline:none;
    transition:height .18s cubic-bezier(.2,.8,.2,1), box-shadow .18s ease;
    box-shadow:inset 0 0 0 1px rgba(255,255,255,.03);
  }
  .seek:hover,.seek:focus-visible,.seek.scrubbing{height:var(--bar-h-active)}
  .seek:focus-visible{box-shadow:0 0 0 3px rgba(255,122,60,.35)}

  /* 시각 두께는 4px이지만 실제 터치 타깃은 28px (투명 박스도 히트 테스트됨) */
  .seek::before{
    content:'';position:absolute;left:0;right:0;top:50%;
    height:28px;transform:translateY(-50%);
  }

  /* 모든 층은 같은 컨테이너 안에서 절대배치로 겹친다(=다층) */
  .layer{
    position:absolute;left:0;top:0;bottom:0;
    width:0%;
    border-radius:999px;
    will-change:width;
  }
  /* 2층: 버퍼(미리 받아둔 구간) */
  .layer.buffered{background:var(--buffered)}
  /* 3층: 재생 진행 구간 */
  .layer.played{
    background:linear-gradient(90deg,var(--accent) 0%,var(--accent2) 100%);
    box-shadow:0 0 14px rgba(255,90,24,.55);
  }
  /* 4층: 호버/드래그 지점까지의 미리보기 */
  .layer.hover{background:rgba(255,122,60,.24)}

  /* 5층: 썸(노브) */
  .thumb{
    position:absolute;top:50%;left:0;
    width:var(--thumb);height:var(--thumb);
    margin-left:calc(var(--thumb) / -2);
    transform:translateY(-50%) scale(1);   /* 모바일: 항상 표시 */
    border-radius:50%;
    background:#fff;
    border:3px solid var(--accent);
    box-shadow:0 0 0 4px rgba(255,122,60,.20),0 2px 8px rgba(0,0,0,.55);
    transition:transform .16s ease;
    pointer-events:none;
  }
  .seek:hover .thumb,.seek:focus-visible .thumb{transform:translateY(-50%) scale(1.15)}
  .seek.scrubbing .thumb{transform:translateY(-50%) scale(1.35)}

  /* 6층(부가): 시간 툴팁 */
  .tip{
    position:absolute;bottom:calc(100% + 10px);left:0;
    transform:translateX(-50%);
    padding:4px 8px;border-radius:7px;
    background:rgba(8,9,12,.92);border:1px solid rgba(255,255,255,.12);
    color:#fff;font-size:11px;font-weight:700;font-variant-numeric:tabular-nums;
    white-space:nowrap;opacity:0;pointer-events:none;
    transition:opacity .14s ease;
  }
  .seek:hover .tip,.seek.scrubbing .tip,.seek:focus-visible .tip{opacity:1}
  .tip[data-side="left"]{transform:translateX(-92%)}
  .tip[data-side="right"]{transform:translateX(-8%)}

  /* ---------- 시간 + 컨트롤 ---------- */
  .times{display:flex;justify-content:space-between;margin-top:8px;font-size:12px;color:#c3c8d0;font-variant-numeric:tabular-nums}
  .times .mid{color:#8d939d}
  .ctrl{display:flex;align-items:center;justify-content:center;gap:12px;margin-top:14px}
  .btn{width:40px;height:40px;border-radius:10px;display:grid;place-items:center;background:#23272E;border:1px solid #2E333B;font-size:16px;color:#E6E8EC;cursor:pointer}
  .btn.acc{background:linear-gradient(180deg,var(--accent),var(--accent2));border-color:transparent;color:#fff;box-shadow:0 6px 16px rgba(255,90,24,.35)}
  .play{width:60px;height:60px;border-radius:50%;display:grid;place-items:center;font-size:24px;border:none;cursor:pointer;
        background:linear-gradient(180deg,var(--accent),var(--accent2));color:#fff;box-shadow:0 10px 26px rgba(255,90,24,.45)}
  .hint{margin-top:14px;font-size:11.5px;line-height:1.75;color:#868d97}
  .hint b{color:#c9ced6}
  .legend{margin-top:12px;display:grid;gap:6px;font-size:11.5px;color:#9aa0aa}
  .lg{display:flex;align-items:center;gap:8px}
  .sw{width:22px;height:10px;border-radius:99px;flex:0 0 22px}
</style>
</head>
<body>
<div class="phone">

  <!-- 헤더 -->
  <div class="hdr">
    <div class="mic">🎙️</div>
    <div>
      <h1>MIDI 반주기 <span class="badge">V2.0</span></h1>
      <p class="sub">사운드폰트 직접 재생 · 파일처럼 편리한 UI</p>
    </div>
    <div class="gear">⚙️</div>
  </div>

  <!-- 검색 / 아이콘 -->
  <div class="row">
    <div class="search">🔍 <span>제목, 가수, 가...</span> <span style="margin-left:auto">🎤</span></div>
    <div class="ico">📁</div>
    <div class="ico">🎵</div>
    <div class="ico">🎛️</div>
    <div class="ico">🎚️</div>
    <div class="ico">📝</div>
  </div>

  <!-- 탭 / 카테고리 -->
  <div class="tabs">
    <div class="tab on">🎹 MIDI 목록</div>
    <div class="tab">🎧 MP3 목록</div>
  </div>
  <div class="chips">
    <div class="chip on">전체</div><div class="chip">가요</div><div class="chip">게임</div>
    <div class="chip">기타</div><div class="chip">만화</div><div class="chip">복음성가</div>
  </div>

  <!-- 스테이지 -->
  <div class="stage">
    <div class="song" id="songTitle">광섬유 파동 (탭하여 전환)</div>

    <div class="wave">
      <svg viewBox="0 0 400 120" xmlns="http://www.w3.org/2000/svg">
        <defs>
          <linearGradient id="gst" x1="0" x2="1">
            <stop offset="0" stop-color="#ff9a5c"/><stop offset="0.5" stop-color="#ffd7a8"/><stop offset="1" stop-color="#7ce0c0"/>
          </linearGradient>
          <filter id="glow"><feGaussianBlur stdDeviation="3" result="b"/><feMerge><feMergeNode in="b"/><feMergeNode in="SourceGraphic"/></feMerge></filter>
        </defs>
        <g fill="none" stroke="url(#gst)" stroke-width="1.6" filter="url(#glow)" opacity=".95">
          <path d="M0,62 C20,30 40,95 60,62 C80,29 100,96 120,62 C140,28 160,97 180,62 C200,30 220,94 240,62 C260,29 280,96 300,62 C320,30 340,94 360,62 C380,30 400,95 420,62"/>
          <path d="M0,62 C25,88 45,34 70,62 C95,90 115,33 140,62 C165,89 185,34 210,62 C235,90 255,33 280,62 C305,89 325,34 350,62 C375,90 395,33 420,62" opacity=".75"/>
          <path d="M0,62 C30,44 55,80 85,62 C115,44 140,80 170,62 C200,44 225,80 255,62 C285,44 310,80 340,62 C370,44 395,82 420,62" opacity=".5"/>
        </g>
      </svg>
    </div>

    <!-- ===== 다층 재생 진행바 ===== -->
    <div class="seek-wrap">
      <div class="seek" id="seek" tabindex="0" role="slider"
           aria-label="재생 진행바" aria-valuemin="0" aria-valuemax="195" aria-valuenow="0">
        <div class="layer buffered" id="lBuffered"></div>   <!-- 2층 -->
        <div class="layer played"   id="lPlayed"></div>     <!-- 3층 -->
        <div class="layer hover"    id="lHover"></div>      <!-- 4층 -->
        <div class="thumb"          id="lThumb"></div>      <!-- 5층 -->
        <div class="tip"            id="tip">00:00</div>     <!-- 시간 툴팁 -->
      </div>
    </div>

    <div class="times">
      <span id="tCur">00:00</span>
      <span class="mid" id="tMid">재생 중인 곡 없음</span>
      <span id="tDur">00:00</span>
    </div>

    <div class="ctrl">
      <button class="btn acc" title="한 곡 반복">🔁</button>
      <button class="btn acc" title="셔플">🔀</button>
      <button class="btn" id="btnPrev" title="이전 곡">⏮</button>
      <button class="play" id="btnPlay" title="재생/일시정지">▶</button>
      <button class="btn" id="btnNext" title="다음 곡">⏭</button>
      <button class="btn acc" title="전체 반복">🔁</button>
    </div>
  </div>

  <p class="hint">
    <b>조작</b> · 바를 클릭하면 즉시 이동, 누른 채 좌우로 끌면 스크럽(모바일 터치 동일),
    키보드 ←/→ 는 ±5초, Space 는 재생·정지. 마우스를 올리면 바 높이(4px→9px)와
    썸·시간 툴팁이 함께 살아납니다.
  </p>

  <div class="legend">
    <div class="lg"><i class="sw" style="background:#2B3038"></i>1층 트랙 · 전체 길이</div>
    <div class="lg"><i class="sw" style="background:#4D545F"></i>2층 버퍼 · 미리 받아둔 구간</div>
    <div class="lg"><i class="sw" style="background:linear-gradient(90deg,#FF7A3C,#FF5A18)"></i>3층 재생 · 진행 구간</div>
    <div class="lg"><i class="sw" style="background:rgba(255,122,60,.24)"></i>4층 호버 · 이동 예정 지점</div>
    <div class="lg"><i class="sw" style="background:#fff;box-shadow:0 0 0 2px #FF7A3C inset"></i>5층 썸 + 시간 툴팁</div>
  </div>
</div>

<script>
/* =========================================================
   0) 상태 + 헬퍼
   ========================================================= */
const seek   = document.getElementById('seek');
const lBuf   = document.getElementById('lBuffered');
const lPlay  = document.getElementById('lPlayed');
const lHover = document.getElementById('lHover');
const thumb  = document.getElementById('lThumb');
const tip    = document.getElementById('tip');
const tCur   = document.getElementById('tCur');
const tDur   = document.getElementById('tDur');
const tMid   = document.getElementById('tMid');
const btnPlay= document.getElementById('btnPlay');
const songTitle = document.getElementById('songTitle');

let media = null;                       // 실제 <video>/<audio> 를 꽂으면 여기에 보관
const state = {
  duration: 195,        // 데모: 3분 15초
  current:  0,
  buffered: 22,
  playing:  false,
  scrubbing:false
};

const clamp01 = v => Math.max(0, Math.min(1, v));
const fmt = s => {
  if (!isFinite(s) || s < 0) s = 0;
  const m = Math.floor(s / 60), ss = Math.floor(s % 60);
  return String(m).padStart(2,'0') + ':' + String(ss).padStart(2,'0');
};
const ratioAt = clientX => {
  const r = seek.getBoundingClientRect();
  return clamp01((clientX - r.left) / r.width);
};
const timeAt = clientX => ratioAt(clientX) * state.duration;

/* =========================================================
   1) 렌더 : 층별 width 를 한 번에 갱신
   ========================================================= */
function paint(){
  const d = state.duration || 1;
  const pPlayed = clamp01(state.current  / d) * 100;
  const pBuf    = clamp01(state.buffered / d) * 100;

  lPlay.style.width  = pPlayed + '%';
  lBuf.style.width   = pBuf + '%';

  // 썸 위치는 재생 지점에 고정 (호버 중이 아니어도 값은 유지)
  thumb.style.left = pPlayed + '%';

  tCur.textContent = fmt(state.current);
  tDur.textContent = fmt(state.duration);
  seek.setAttribute('aria-valuenow', Math.round(state.current));
  seek.setAttribute('aria-valuemax', Math.round(state.duration));
  tMid.textContent = state.playing ? '재생 중' : (state.current > 0 ? '일시정지' : '재생 중인 곡 없음');
}

/* 호버/드래그 미리보기 층 + 툴팁 */
function showPreview(clientX){
  const r = ratioAt(clientX);
  lHover.style.width = (r * 100) + '%';
  tip.style.left = (r * 100) + '%';
  tip.textContent = fmt(r * state.duration);
  tip.setAttribute('data-side', r > .85 ? 'left' : (r < .15 ? 'right' : ''));
}
function hidePreview(){
  lHover.style.width = '0%';
}

/* =========================================================
   2) 실제 이동 (데모와 실제 미디어 모두 지원)
   ========================================================= */
function seekTo(t){
  state.current = Math.max(0, Math.min(state.duration, t));
  if (media) media.currentTime = state.current;
  // 데모: 버퍼가 재생 지점을 뒤따라오게
  state.buffered = Math.max(state.buffered, Math.min(state.duration, state.current + (state.playing ? 22 : 0)));
  paint();
}

/* =========================================================
   3) 포인터 이벤트 : 클릭 + 드래그(마우스·터치·펜 공용)
   ========================================================= */
seek.addEventListener('pointerdown', e => {
  seek.setPointerCapture(e.pointerId);       // 밖으로 나가도 계속 추적
  state.scrubbing = true;
  seek.classList.add('scrubbing');
  showPreview(e.clientX);
  seekTo(timeAt(e.clientX));
  updateDemoClock();                          // 드래그 중에도 시계 정렬
});
seek.addEventListener('pointermove', e => {
  if (state.scrubbing){
    showPreview(e.clientX);
    seekTo(timeAt(e.clientX));
  } else if (e.pointerType === 'mouse'){
    showPreview(e.clientX);                   // 데스크톱: 호버 미리보기
  }
});
function endScrub(e){
  if (!state.scrubbing) return;
  state.scrubbing = false;
  seek.classList.remove('scrubbing');
  showPreview(e.clientX);                     // 값은 남기고
  hidePreview();                              // 호버 층은 정리
  tip.style.opacity = '';
}
seek.addEventListener('pointerup', endScrub);
seek.addEventListener('pointercancel', endScrub);
seek.addEventListener('pointerleave', e => { if (!state.scrubbing) hidePreview(); });

/* 키보드 접근성 */
seek.addEventListener('keydown', e => {
  if (e.key === 'ArrowRight'){ seekTo(state.current + 5); e.preventDefault(); }
  if (e.key === 'ArrowLeft' ){ seekTo(state.current - 5); e.preventDefault(); }
  if (e.key === ' ' || e.key === 'Enter'){ togglePlay(); e.preventDefault(); }
});

/* =========================================================
   4) 재생/정지 + 데모 시계 (rAF)
   ========================================================= */
let rafId = null, lastTs = 0;
function tick(ts){
  if (!state.playing) return;
  if (!lastTs) lastTs = ts;
  const dt = (ts - lastTs) / 1000;
  lastTs = ts;

  if (media){
    state.current  = media.currentTime;                                   // 실제 미디어가 진실
    state.duration = media.duration || state.duration;
    if (media.buffered.length){
      state.buffered = media.buffered.end(media.buffered.length - 1);
    }
  } else {
    state.current = Math.min(state.duration, state.current + dt);          // 데모 시계
    state.buffered = Math.min(state.duration, Math.max(state.buffered, state.current + 22));
  }
  if (state.current >= state.duration - .05 && !media){
    state.current = state.duration; paint(); setPlaying(false); return;
  }
  paint();
  rafId = requestAnimationFrame(tick);
}
function updateDemoClock(){ lastTs = 0; }
function setPlaying(on){
  state.playing = on;
  btnPlay.textContent = on ? '❚❚' : '▶';
  if (on){
    if (media && media.play) media.play().catch(()=>{});
    lastTs = 0;
    rafId = requestAnimationFrame(tick);
  } else {
    if (media && media.pause) media.pause();
    cancelAnimationFrame(rafId);
  }
  paint();
}
function togglePlay(){
  // 데모에 곡이 없으면 첫 재생 시 곡 제목을 채워 준다
  if (!state.playing && state.current === 0 && tMid.dataset.empty !== '0'){
    songTitle.textContent = '광섬유 파동 (재생 중)';
  }
  setPlaying(!state.playing);
}
btnPlay.addEventListener('click', togglePlay);
document.getElementById('btnPrev').addEventListener('click', () => seekTo(0));
document.getElementById('btnNext').addEventListener('click', () => { seekTo(0); paint(); });

/* =========================================================
   5) 실제 <video>/<audio> 연결용 어댑터
      (아래 한 줄만 호출하면 위 UI가 그대로 실제 미디어를 제어합니다)
   ========================================================= */
function bindMediaElement(el){
  media = el;
  const syncDur = () => { state.duration = isFinite(el.duration) ? el.duration : state.duration; paint(); };
  el.addEventListener('loadedmetadata', syncDur);
  el.addEventListener('durationchange', syncDur);
  el.addEventListener('progress', () => {
    if (el.buffered.length) state.buffered = el.buffered.end(el.buffered.length - 1);
    paint();
  });
  el.addEventListener('play',  () => { if (!state.playing) setPlaying(true); });
  el.addEventListener('pause', () => { if (state.playing)  setPlaying(false); });
  el.addEventListener('ended', () => { setPlaying(false); seekTo(0); });
  syncDur();
  return el;
}
// 사용 예:
//   <video id="mv" src="a.mp4"></video>
//   bindMediaElement(document.getElementById('mv'));
//   <audio id="ma" src="a.mp3"></audio>
//   bindMediaElement(document.getElementById('ma'));

/* 시작 */
paint();
</script>
<script type="module" src="https://static.cloudflareinsights.com/beacon.min.js/v4bc70e2c01a94c73b74392e4234840661791215815920" integrity="sha512-L0ha0OXavK/8okipN9F8BtP84dg9DUhPERbBXzwI6dgTA55d2+yweo3pn5CSFYs45/r8md2+xvUPtTdvTNRfjA==" data-cf-beacon='{"version":"2024.11.0","token":"4edd5f8ec12a48cfa682ab8261b80a79","spa":2}' crossorigin="anonymous"></script>
</body>
</html>
