# 에덴 기숙학원 — SKILL.md (구현 기술 가이드)

## 1. 게임 상태 관리 구조

```javascript
// 전역 게임 상태 (localStorage 금지 → JS 변수로만 관리)
const GAME_STATE = {
  scenario: null,          // 'A' | 'B' | 'C'
  currentStage: 1,         // 1 | 2 | 3
  currentQuest: null,      // 'A-01' | 'A-02' | ...
  completedQuests: [],     // ['A-01', 'A-02', ...]
  inventory: [],           // [{id, name, emoji, desc}, ...]
  trust: { suin: 0, aran: 0 },
  hqCompleted: [],         // ['HQ-1', 'HQ-2']
  hintsUsed: {},           // {'A-01': 2, 'A-02': 0, ...}
  timerActive: false,
  timerRemaining: 0,
};
```

---

## 2. 화면 전환 패턴

```javascript
// 화면 ID 목록
// 'title-screen' | 'scenario-screen' | 'game-screen' | 'ending-screen'

function showScreen(id) {
  document.querySelectorAll('.screen').forEach(s => {
    s.classList.remove('active');
  });
  document.getElementById(id).classList.add('active');
}

// CSS
.screen { opacity: 0; pointer-events: none; transition: opacity 0.6s; }
.screen.active { opacity: 1; pointer-events: all; }
```

---

## 3. 퀘스트 데이터 구조

```javascript
const QUEST_DATA = {
  'A-01': {
    id: 'A-01',
    stage: 1,
    location: '야간 복도',
    emoji: '🏫',
    title: '경비원의 패턴',
    problem: '경비원의 순찰 패턴이 바뀌었다...',
    type: 'observe',          // 'observe'|'code'|'logic'|'caesar'|'count'|'pattern'
    answer: null,             // 코드 입력형: '5130' | 선택형: null
    choices: null,            // 선택지 배열 또는 null
    correctChoice: null,      // 선택형 정답 인덱스
    itemsRequired: [],        // 필요 아이템 ID 목록
    itemsGranted: [{id:'schedule', name:'청소 일정표', emoji:'📋'}],
    hintsMax: 2,
    timeLimit: 0,             // 0 = 없음
    trustEffect: { suin: 0, aran: 0 },
    npcLine: '수인: "조심해요. 경비 패턴을 먼저 파악해야 해."',
    hintLines: [
      '복도 벽면 어딘가에 일정이 적혀 있어.',
      '청소 일정표의 밑줄 친 숫자가 순찰 간격(분)이야.',
    ],
    nextQuest: 'A-02',
  },
  // ... 나머지 퀘스트
};
```

---

## 4. 퍼즐 유형별 구현 패턴

### 4-1. 코드 입력형 (자물쇠, 금고)
```javascript
function checkCode(questId, inputValue) {
  const quest = QUEST_DATA[questId];
  if (inputValue.trim().toUpperCase() === quest.answer) {
    onQuestSuccess(questId);
  } else {
    showFeedback('err', '틀렸어. 다시 생각해봐.');
    shakeInput();
  }
}
```

### 4-2. 카이사르 암호 퍼즐
```javascript
// 암호화: 알파벳을 N칸 오른쪽으로 이동
function caesarEncrypt(text, shift) {
  return text.toUpperCase().split('').map(ch => {
    if (ch >= 'A' && ch <= 'Z') {
      return String.fromCharCode((ch.charCodeAt(0) - 65 + shift) % 26 + 65);
    }
    return ch;
  }).join('');
}

// 해독: 반대 방향
function caesarDecrypt(text, shift) {
  return caesarEncrypt(text, 26 - shift);
}

// 에덴 적용 예시
// 규리 노트: 암호문 'HGHQ' + 이동 칸수 힌트
// shift=4 → HGHQ → DCDC (→ D=지하, C=3층 → B3)
```

### 4-3. 논리 퀴즈 (수인 vs 아란)
```javascript
const LOGIC_QUIZ = {
  premise: '셋 중 하나만 진실입니다.',
  statements: [
    { speaker: '수인', text: '이쪽 길이 안전해요. 논리적으로 맞아요.' },
    { speaker: '아란', text: '저쪽은 소각장이야! 지하는 이쪽이야!' },
    { speaker: '수인', text: '아란은 기억이 조작됐어요. 믿으면 안 돼요.' },
  ],
  // 수인 발화 2개 = 모순 → 아란 발화 1개가 진실
  correctSpeaker: 'aran',
  explanation: '수인이 두 개의 발화를 하면 둘 중 하나는 반드시 거짓. 아란의 발화만 모순 없이 진실 가능.',
};
```

### 4-4. 수색/관찰형 (클릭 인터랙션)
```javascript
// 소품 목록 정의
const ROOM_ITEMS = [
  { id: 'frame', emoji: '🖼️', pos: {x:'20%', y:'40%'}, 
    desc: '액자를 뒤집으면... 팔찌가 붙어 있다!', 
    grantsItem: 'bracelet' },
  { id: 'doll', emoji: '🪆', pos: {x:'60%', y:'55%'}, 
    desc: '인형 배 속에 쪽지가 있다.', 
    grantsItem: 'note_guri' },
];

// 클릭 핸들러
function onItemClick(itemId) {
  const item = ROOM_ITEMS.find(i => i.id === itemId);
  showModal(item.desc);
  if (item.grantsItem) grantItem(item.grantsItem);
}
```

### 4-5. 타이머 (가스실 타임어택)
```javascript
let timerInterval = null;

function startTimer(seconds, onTimeout) {
  GAME_STATE.timerActive = true;
  GAME_STATE.timerRemaining = seconds;
  updateTimerUI();
  timerInterval = setInterval(() => {
    GAME_STATE.timerRemaining--;
    updateTimerUI();
    if (GAME_STATE.timerRemaining <= 0) {
      clearInterval(timerInterval);
      onTimeout();
    }
  }, 1000);
}

function stopTimer() {
  clearInterval(timerInterval);
  GAME_STATE.timerActive = false;
}
```

---

## 5. 신뢰도 시스템

```javascript
function adjustTrust(target, amount) {
  // target: 'suin' | 'aran'
  GAME_STATE.trust[target] = Math.max(0,
    Math.min(100, GAME_STATE.trust[target] + amount)
  );
  updateTrustUI();
}

function getEndingType() {
  const { aran, suin } = GAME_STATE.trust;
  const hq = GAME_STATE.hqCompleted.length;
  if (aran >= 60 && hq >= 2) return 'liberation';
  if (aran >= 40 && hq >= 1) return 'truth';
  if (aran >= 20)            return 'escape';
  if (suin >= 50)            return 'defeat';
  return 'escape';
}
```

---

## 6. 인벤토리 시스템

```javascript
function grantItem(itemId) {
  const ITEM_DB = {
    'notebook':   { name: '규리의 암호 노트', emoji: '📓' },
    'flashlight': { name: '손전등',           emoji: '🔦' },
    'bracelet':   { name: '규리의 갈색 팔찌', emoji: '📿' },
    'hairpin':    { name: '수인의 비녀 배지', emoji: '📌' },
    'note_guri':  { name: '규리의 쪽지',      emoji: '📜' },
    'note_aran':  { name: '아란의 쪽지',      emoji: '📝' },
    'drug_code':  { name: '약품 코드 메모',   emoji: '🧪' },
    'coin':       { name: '동전',             emoji: '🪙' },
  };
  if (!GAME_STATE.inventory.find(i => i.id === itemId)) {
    GAME_STATE.inventory.push({ id: itemId, ...ITEM_DB[itemId] });
    updateInventoryUI();
    addLog(`아이템 획득: ${ITEM_DB[itemId].emoji} ${ITEM_DB[itemId].name}`, 'key');
  }
}

function hasItem(itemId) {
  return GAME_STATE.inventory.some(i => i.id === itemId);
}
```

---

## 7. 로그 시스템

```javascript
function addLog(text, type = 'normal') {
  // type: 'normal' | 'good' | 'bad' | 'key'
  const now = new Date();
  const time = `${String(now.getHours()).padStart(2,'0')}:${String(now.getMinutes()).padStart(2,'0')}`;
  const entry = { time, text, type };
  LOG_HISTORY.unshift(entry);
  renderLog();
}
```

---

## 8. 새 세션에서 이어받기 체크리스트

새 채팅 세션 시작 시 반드시 확인:
- [ ] INSTRUCTION.md 내용 숙지
- [ ] decision.log 최신 상태 확인
- [ ] 현재 Phase 확인 (Phase 1~5 중 어디까지 완료됐는지)
- [ ] 이전 세션에서 만든 HTML 파일 경로 확인
- [ ] 미결 이슈 목록 확인
