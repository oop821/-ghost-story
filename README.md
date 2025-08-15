<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>괴담보기</title>
  <style>
    :root {
      --bg: #0b0f14;
      --card: #121821;
      --text: #e8eef5;
      --muted: #9fb0c4;
      --accent: #56a8ff;
      --danger: #ff5c7a;
    }
    * { box-sizing: border-box; }
    body {
      margin: 0; background: radial-gradient(1200px 700px at 80% -10%, #0f1722 10%, #0b0f14 60%), var(--bg);
      color: var(--text); font-family: "Pretendard", system-ui, -apple-system, Segoe UI, Roboto, Noto Sans KR, Helvetica, Arial, sans-serif;
      min-height: 100dvh; display: grid; place-items: center;
    }
    .app {
      width: min(720px, 92vw); background: linear-gradient(180deg, #121821 0%, #0e141d 100%);
      border: 1px solid #1f2a3a; border-radius: 20px; box-shadow: 0 10px 30px rgba(0,0,0,.35), inset 0 1px 0 rgba(255,255,255,.04);
      overflow: hidden;
    }
    header { padding: 22px 24px; display: flex; align-items: center; justify-content: space-between; border-bottom: 1px solid #1c2533; }
    header h1 { font-size: 20px; margin: 0; letter-spacing: 0.5px; }

    .screen { padding: 28px 24px 32px; }

    .list {
      display: grid; grid-template-columns: repeat(auto-fit, minmax(180px,1fr)); gap: 14px;
    }
    .card-btn {
      background: #15202e; border: 1px solid #1e2a3b; color: var(--text); padding: 14px 16px; border-radius: 14px; cursor: pointer;
      display: flex; align-items: center; justify-content: space-between; transition: transform .08s ease, background .15s ease, border-color .15s ease;
    }
    .card-btn:hover { background: #192537; border-color: #27364b; transform: translateY(-1px); }
    .card-btn:active { transform: translateY(0); }
    .card-btn span { font-weight: 600; }
    .card-btn small { color: var(--muted); }

    .viewer { display: grid; gap: 16px; }
    .viewer h2 { margin: 0; font-size: 18px; }
    .img-wrap {
      border-radius: 16px; border: 1px dashed #2a3a52; background: #0b1119; padding: 12px; display: grid; place-items: center; min-height: 260px;
    }
    .img-wrap img { max-width: 100%; border-radius: 12px; height: auto; display: block; }

    .toolbar { display: flex; gap: 10px; justify-content: flex-end; }
    .btn {
      padding: 10px 14px; border-radius: 12px; border: 1px solid #253348; background: #15202e; color: var(--text); cursor: pointer; font-weight: 600;
    }
    .btn:hover { background: #19283a; border-color: #2f425b; }
    .btn.cancel { border-color: #3a2130; background: #20131b; }
    .btn.cancel:hover { background: #27161f; }

    .muted { color: var(--muted); }
    .hint { margin-top: 12px; color: var(--muted); font-size: 14px; }

    /* 작은 화면 처리 */
    @media (max-width: 420px) {
      .card-btn { padding: 12px 12px; }
      header h1 { font-size: 18px; }
    }
  </style>
</head>
<body>
  <main class="app" role="application">
    <header>
      <h1>괴담보기</h1>
      <div class="muted" aria-live="polite" id="status"></div>
    </header>

    <!-- 시작 화면 -->
    <section class="screen" id="home" aria-labelledby="home-title">
      <h2 id="home-title" class="muted" style="margin:0 0 12px 0;">보기 원하는 괴담을 선택하세요</h2>
      <div class="list" id="storyList" role="list"></div>
      <p class="hint"> <strong> STORIES</strong></p>
    </section>

    <!-- 보기 화면 -->
    <section class="screen" id="viewer" hidden aria-labelledby="viewer-title">
      <div class="viewer">
        <h2 id="viewer-title"></h2>
        <div class="img-wrap" id="imageWrap" aria-live="polite">
          <!-- 이미지 또는 안내 문구가 들어갑니다 -->
        </div>
        <div class="toolbar">
          <button class="btn cancel" id="btnCancel" type="button" aria-label="바탕화면으로">취소</button>
        </div>
      </div>
    </section>
  </main>

  <script>
    // ▼ 여기를 수정해서 각 괴담의 이미지 파일 경로를 넣으세요.
    // 빈 문자열("")이면 이미지 대신 안내 문구가 표시됩니다.
    const STORIES = [
      { name: "빨간색", img: "" },               // 예: "images/school-ghost.jpg"
      { name: "버려진 산장", img: "images/tunnel.jpg" },
      { name: "산불", img: "images/hospital.png" }
    ];

    const elHome = document.getElementById('home');
    const elViewer = document.getElementById('viewer');
    const elStoryList = document.getElementById('storyList');
    const elViewerTitle = document.getElementById('viewer-title');
    const elImageWrap = document.getElementById('imageWrap');
    const elCancel = document.getElementById('btnCancel');
    const elStatus = document.getElementById('status');

    // 홈(바탕화면)으로 이동
    function goHome(message) {
      elViewer.hidden = true;
      elHome.hidden = false;
      elStatus.textContent = message || '';
      // 포커스 접근성: 첫 카드에 포커스 이동
      const firstCard = elStoryList.querySelector('button');
      if (firstCard) firstCard.focus();
    }

    // 특정 괴담 보기 화면으로 이동
    function openStory(story) {
      elHome.hidden = true;
      elViewer.hidden = false;
      elViewerTitle.textContent = story.name;
      elImageWrap.innerHTML = '';

      if (!story.img) {
        const p = document.createElement('p');
        p.className = 'muted';
        p.textContent = '아직 괴담이 나오지 않았습니다.';
        elImageWrap.appendChild(p);
        elStatus.textContent = `${story.name} — 이미지가 아직 없습니다.`;
        return;
      }

      const img = document.createElement('img');
      img.alt = `${story.name} 이미지`;
      img.src = story.img;
      img.loading = 'lazy';
      img.addEventListener('error', () => {
        elImageWrap.innerHTML = '';
        const p = document.createElement('p');
        p.className = 'muted';
        p.textContent = '아직 괴담이 나오지 않았습니다.';
        elImageWrap.appendChild(p);
        elStatus.textContent = `${story.name} — 이미지 로드 실패`;
      });
      elImageWrap.appendChild(img);
      elStatus.textContent = `${story.name} — 열림`;
    }

    // 리스트 만들기
    function renderList() {
      elStoryList.innerHTML = '';
      STORIES.slice(0,3).forEach((story, idx) => {
        const btn = document.createElement('button');
        btn.className = 'card-btn';
        btn.setAttribute('role', 'listitem');
        btn.innerHTML = `<span>${story.name}</span><small>${story.img ? '이미지 있음' : '이미지 없음'}</small>`;
        btn.addEventListener('click', () => openStory(story));
        btn.addEventListener('keyup', (e) => { if (e.key === 'Enter' || e.key === ' ') openStory(story); });
        elStoryList.appendChild(btn);
      });
    }

    elCancel.addEventListener('click', () => goHome('바탕화면으로 돌아왔습니다.'));
    document.addEventListener('keydown', (e) => {
      // ESC로 취소
      if (!elViewer.hidden && e.key === 'Escape') {
        goHome('바탕화면으로 돌아왔습니다.');
      }
    });

    renderList();
    // 초기 상태는 홈 화면
    goHome('');
  </script>
</body>
</html>
