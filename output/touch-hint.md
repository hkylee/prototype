# 터치 힌트 (Touch Area Hint)

빈 영역을 2번 연속 탭하면 유효한 터치 영역을 은은하게 표시합니다.

---

## 1. CSS — `<style>` 블록에 추가

```css
/* ── 터치 힌트 오버레이 ── */
.tap-hint-overlay {
  position: absolute;
  pointer-events: none;
  z-index: 99999;
  border-radius: 999px;
  border: 2px solid rgba(89, 64, 223, 0.65);
  background: rgba(89, 64, 223, 0.07);
  animation: tap-ring 1.4s cubic-bezier(0.16, 1, 0.3, 1) 1 forwards;
}
/* 넓은 영역(키보드·패널 등) — pill 대신 둥근 직사각형 */
.tap-hint-overlay--rect { border-radius: 14px; }
@keyframes tap-ring {
  0%   { opacity: 0; transform: scale(0.94); }
  15%  { opacity: 1; transform: scale(1);    }
  70%  { opacity: 1; transform: scale(1);    }
  100% { opacity: 0; transform: scale(1.03); }
}
```

---

## 2. HTML — 터치 대상 요소에 `id` 부여

힌트를 표시할 요소에 고유 `id`를 달아둡니다.

```html
<!-- 예시: Screen 1 검색바 -->
<div id="s1-gnb-tap" onclick="showKeyboard()"> ... </div>

<!-- 예시: Screen 4 선물하기 버튼 -->
<button id="s4-gift-btn" onclick="goToScreen5()">선물하기</button>

<!-- 예시: Screen 6 옵션 -->
<div id="s6-opt1" onclick="selectGiftYes()">네, 선물할게요</div>
```

---

## 3. JS — `<script>` 블록 끝에 추가

```js
(function() {
  // 화면 ID → 힌트 대상 요소 ID 배열
  var HINTS = {
    'screen-1': ['s1-gnb-tap'],
    'screen-4': ['s4-gift-btn'],
    'screen-6': ['s6-opt1'],
    // 'screen-N': ['요소id1', '요소id2'],  ← 화면 추가 시 여기에
  };
  var missCount = {};

  // 현재 활성 화면 ID 반환
  function activeScreenId() {
    var s = document.querySelector('.screen.active');
    return s ? s.id : null;
  }

  // 타겟 위치에 오버레이 생성
  function showHints(ids) {
    var stage = document.getElementById('ut-stage');
    var stageRect = stage.getBoundingClientRect();
    ids.forEach(function(id) {
      var el = document.getElementById(id);
      if (!el) return;
      var r = el.getBoundingClientRect();
      var pad = 8;
      var ov = document.createElement('div');
      ov.className = 'tap-hint-overlay';
      ov.style.left   = (r.left - stageRect.left - pad) + 'px';
      ov.style.top    = (r.top  - stageRect.top  - pad) + 'px';
      ov.style.width  = (r.width  + pad * 2) + 'px';
      ov.style.height = (r.height + pad * 2) + 'px';
      stage.appendChild(ov);
      setTimeout(function() { ov.remove(); }, 1400);
    });
  }

  // 키보드/모달 열림 시 대체 힌트
  // guard 요소에 class 'open'이 붙으면 HINTS 대신 hints 배열을 사용
  var KEYBOARD_GUARDS = {
    'screen-1': { guard: 's1-keyboard', hints: ['s1-return-btn'] },
    // 'screen-N': { guard: 'sN-modal-id', hints: ['sN-cta-id'] },
  };

  // 빈 영역 탭 감지 — 2번 연속 빗나가면 힌트 표시
  document.getElementById('ut-stage').addEventListener('pointerdown', function(e) {
    var el = e.target;
    while (el && el !== this) {
      if (el.tagName === 'BUTTON' || el.tagName === 'INPUT' ||
          el.onclick || el.getAttribute('onclick') ||
          el.hasAttribute('onmousedown') || el.hasAttribute('ontouchstart')) {
        return; // 인터랙티브 요소 클릭 → 무시
      }
      el = el.parentElement;
    }
    var sid = activeScreenId();
    if (!HINTS[sid]) return;
    var hints = HINTS[sid];
    var kg = KEYBOARD_GUARDS[sid];
    if (kg) {
      var guardEl = document.getElementById(kg.guard);
      if (guardEl && guardEl.classList.contains('open')) {
        // 텍스트 입력 완료 상태면 return 버튼 힌트, 미입력이면 키보드 전체 힌트
        var typedEl = document.getElementById('s1-typed');
        var hasText = typedEl && typedEl.style.display !== 'none';
        hints = hasText ? kg.hints : ['s1-keyboard'];
      }
    }
    missCount[sid] = (missCount[sid] || 0) + 1;
    if (missCount[sid] >= 2) {
      showHints(hints);
      missCount[sid] = 0;
    }
  });

  // 화면 전환 시 카운트 리셋
  document.getElementById('ut-stage').addEventListener('screenchange', function() {
    missCount = {};
  });
})();
```

---

## 적용 방법 요약

| 단계 | 작업 |
|------|------|
| CSS | `<style>` 블록에 `.tap-hint-overlay` + `@keyframes tap-ring` 추가 |
| HTML | 힌트 대상 요소에 `id` 부여 |
| JS | `HINTS` 맵에 `'screen-ID': ['요소id']` 추가 후 스크립트 삽입 |

> **주의:** `ut-stage` 컨테이너가 `position: relative` 여야 오버레이 위치가 정확합니다.
