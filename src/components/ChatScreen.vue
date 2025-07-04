<template>
  <div class="chat-wrapper">
    <!-- 모바일 전용 햄버거 버튼 -->
    <button
      class="toggle-btn mobile-toggle"
      @click="isCollapsed = !isCollapsed"
      v-show="isMobile"
      aria-label="메뉴 열기/닫기"
    >
      ☰
    </button>

    <aside :class="['history-pane', { collapsed: isCollapsed }]">
      <!-- PC 전용 좌우 화살표 버튼 -->
      <button
        class="toggle-btn pc-toggle"
        @click="toggleSidebar"
        v-show="!isMobile"
        aria-label="사이드바 접기/펼치기"
      >
        {{ isCollapsed ? '▶' : '◀' }}
      </button>

      <div v-if="!isCollapsed" class="sidebar-content">
        <h3>質問履歴</h3>

        <!-- 검색창 -->
        <input
          v-model="searchTerm"
          type="text"
          placeholder="質問を検索..."
          class="search-input"
        />

        <!-- 필터 버튼 -->
        <div class="filter-buttons">
          <button :class="{ active: filter === 'all' }" @click="filter = 'all'">全て</button>
          <button :class="{ active: filter === 'recent' }" @click="filter = 'recent'">最近の5件</button>
        </div>

        <!-- 새 질문 입력 버튼 -->
        <button class="new-question-btn" @click="startNewQuestion">新しい質問</button>

        <ul>
          <li
            v-for="(item, idx) in filteredHistory"
            :key="idx"
            @click="loadHistory(item)"
            class="history-item"
          >
            {{ item.question }}
          </li>
          <li v-if="filteredHistory.length === 0" class="no-result">該当する質問がありません。</li>
        </ul>
      </div>
    </aside>

    <main class="chat-container">
      <h2>生成型AIモデル訓練中</h2>
      <h5>社内AIシステム「IT-IS」リリース予定</h5>

      <textarea
        v-model="userInput"
        rows="4"
        placeholder="質問を入力してください..."
        @keyup.enter="sendMessage"
      ></textarea>
      <button :disabled="!userInput.trim()" @click="sendMessage">送信</button>

      <section v-if="response" class="response-section">
        <p class="response-text question"><strong>質問：</strong><br />{{ lastQuestion }}</p>
        <p class="response-text answer"><strong>応答：</strong><br />{{ response }}</p>
        <div class="feedback-buttons">
          <button class="btn-correct" @click="sendFeedback('正しい')">正しい</button>
          <button class="btn-incorrect" @click="sendFeedback('違う')">違う</button>
        </div>
      </section>
    </main>

    <footer class="footer">
      © 2025 DXPRO SOLUTIONS. All rights reserved.
    </footer>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, onBeforeUnmount } from 'vue'

const userInput = ref('')
const response = ref('')
const lastQuestion = ref('')
const historyList = ref([])

const searchTerm = ref('')
const filter = ref('all')
const isCollapsed = ref(true)
const isMobile = ref(false)

function checkIsMobile() {
  isMobile.value = window.innerWidth <= 768
}

onMounted(() => {
  checkIsMobile()
  window.addEventListener('resize', checkIsMobile)
  fetchHistory()
})

onBeforeUnmount(() => {
  window.removeEventListener('resize', checkIsMobile)
})

async function fetchHistory() {
  try {
    const res = await fetch('http://localhost:3000/api/history')
    const data = await res.json()
    historyList.value = data
  } catch (e) {
    console.error('履歴取得エラー:', e)
  }
}

function loadHistory(item) {
  lastQuestion.value = item.question
  response.value = item.answer
  userInput.value = ''
}

function startNewQuestion() {
  lastQuestion.value = ''
  response.value = ''
  userInput.value = ''
}

const filteredHistory = computed(() => {
  let list = [...historyList.value]

  if (searchTerm.value.trim() !== '') {
    list = list.filter(item =>
      item.question.toLowerCase().includes(searchTerm.value.toLowerCase())
    )
  }

  if (filter.value === 'recent') {
    list = list.slice(0, 5)
  }

  return list
})

async function sendMessage() {
  if (!userInput.value.trim()) return alert('質問を入力してください。')

  try {
    const res = await fetch('http://localhost:3000/api/chat', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ message: userInput.value }),
    })
    const data = await res.json()
    response.value = data.content || '応答がありませんでした。'
    lastQuestion.value = userInput.value
    userInput.value = ''
    fetchHistory()
  } catch (err) {
    alert('送信エラーが発生しました。')
    console.error(err)
  }
}

async function sendFeedback(feedback) {
  if (!response.value) return
  try {
    await fetch('http://localhost:3000/api/feedback', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        question: lastQuestion.value,
        answer: response.value,
        feedback,
      }),
    })
    alert('フィードバックありがとうございます。')
  } catch (err) {
    alert('フィードバック送信でエラーが発生しました。')
    console.error(err)
  }
}

function toggleSidebar() {
  isCollapsed.value = !isCollapsed.value
}
</script>

<style>
html, body, #app {
  margin: 0;
  padding: 0;
  height: 100%;
  background: #f7f9fc;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  color: #222;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.chat-wrapper {
  display: flex;
  height: 100vh;
  overflow: hidden;
  position: relative;
}

/* 사이드바 기본 스타일 */
.history-pane {
  width: 400px;
  background: #fff;
  border-right: 1px solid #ddd;
  box-shadow: 2px 0 10px rgba(0, 0, 0, 0.05);
  overflow-y: auto;
  display: flex;
  flex-direction: column;
  transition: width 0.3s ease;
  position: relative;
}

/* 사이드바 접혔을 때 스타일 */
.history-pane.collapsed {
  width: 60px;
  overflow: hidden;
}

/* 토글 버튼 기본 스타일 */
.toggle-btn {
  border: none;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.3s ease;
}

/* 모바일 햄버거 버튼 (왼쪽 상단 고정) */
.toggle-btn.mobile-toggle {
  display: none;
  position: fixed;
  top: 10px;
  left: 10px;
  width: 40px;
  height: 40px;
  font-size: 1.5rem;
  background-color: #1976d2;
  color: white;
  border-radius: 6px;
  z-index: 1101;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.3);
}

/* PC 사이드바 안쪽 좌우 버튼 */
.toggle-btn.pc-toggle {
  display: none;
  position: absolute;
  top: 0px;
  right: 10px;
  width: 20px;
  height: 45px;
  font-size: 1.3rem;
  background-color: #1976d2;
  color: white;
  border-radius: 5px;
  z-index: 10;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.2);
}

/* 반응형 */
@media (max-width: 768px) {
  .toggle-btn.mobile-toggle {
    display: block;
  }
  .toggle-btn.pc-toggle {
    display: none;
  }
  .chat-wrapper {
    flex-direction: column;
  }

  .history-pane {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100%;
    max-width: none;
    z-index: 1000;
    transform: translateX(0);
    transition: transform 0.3s ease-in-out;
    box-shadow: 4px 0 15px rgba(0, 0, 0, 0.15);
  }

  .history-pane.collapsed {
    transform: translateX(-100%);
  }

  .chat-container {
    padding: 1rem 1.2rem;
  }

  textarea {
    min-height: 100px;
    width: 100%;
    font-size: 1rem;
  }

  button {
    width: 100%;
    font-size: 1rem;
    padding: 0.75rem;
    margin-top: 1rem;
  }

  .response-section {
    padding: 1.5rem;
    margin: 1rem 0;
  }

  .footer {
    font-size: 0.85rem;
    padding: 0.6rem;
  }
}

@media (min-width: 769px) {
  .toggle-btn.mobile-toggle {
    display: none;
  }
  .toggle-btn.pc-toggle {
    display: block;
  }
}

/* 기존 스타일 유지 */

.sidebar-content {
  padding: 1rem 1.2rem;
  flex-grow: 1;
  display: flex;
  flex-direction: column;
}

.history-pane h3 {
  font-size: 1.3rem;
  margin-bottom: 1rem;
  font-weight: 600;
  color: #1976d2;
  border-bottom: 2px solid #1976d2;
  padding-bottom: 0.3rem;
}

.search-input {
  width: 100%;
  padding: 0.5rem 1rem;
  margin-bottom: 0.8rem;
  font-size: 1rem;
  border-radius: 8px;
  border: 1.5px solid #ccc;
  box-sizing: border-box;
  transition: border-color 0.3s ease;
}

.search-input:focus {
  border-color: #1976d2;
  outline: none;
  box-shadow: 0 0 8px rgba(25, 118, 210, 0.5);
}

.filter-buttons {
  display: flex;
  gap: 0.6rem;
  margin-bottom: 1rem;
}

.filter-buttons button {
  flex: 1;
  padding: 0.5rem 0;
  border-radius: 20px;
  border: none;
  font-weight: 600;
  cursor: pointer;
  background-color: #e0e0e0;
  color: #444;
  transition: background-color 0.3s ease;
  user-select: none;
}

.filter-buttons button.active,
.filter-buttons button:hover {
  background-color: #1976d2;
  color: white;
  box-shadow: 0 4px 12px rgba(25, 118, 210, 0.3);
}

.new-question-btn {
  width: 100%;
  padding: 0.7rem 1rem;
  margin-bottom: 1rem;
  background-color: #1976d2;
  color: white;
  font-weight: 700;
  border: none;
  border-radius: 30px;
  cursor: pointer;
  box-shadow: 0 6px 12px rgba(25, 118, 210, 0.4);
  user-select: none;
  transition: background-color 0.3s ease;
}

.new-question-btn:hover {
  background-color: transparent;
}

.history-pane ul {
  list-style: none;
  padding: 0;
  margin: 0;
  flex-grow: 1;
  overflow-y: auto;
}

.history-item {
  padding: 0.7rem 1rem;
  margin-bottom: 0.6rem;
  border-radius: 10px;
  cursor: pointer;
  background: #f0f5ff;
  color: #0d47a1;
  font-weight: 500;
  transition: background-color 0.25s ease, box-shadow 0.25s ease;
  box-shadow: 0 1px 2px rgba(13, 71, 161, 0.15);
}

.history-item:hover {
  background: #dbe9ff;
  box-shadow: 0 3px 8px rgba(13, 71, 161, 0.3);
}

.no-result {
  padding: 1rem;
  color: #999;
  font-style: italic;
  text-align: center;
}

.chat-container {
  flex: 1;
  flex-direction: column;
  padding: 2.5rem 3rem;
  background: #ffffff;
  box-shadow: inset 0 0 10px rgba(0, 0, 0, 0.03);
  overflow-y: auto;
}

.chat-container h2 {
  font-size: 1.8rem;
  margin-bottom: 0.3rem;
  font-weight: 700;
  color: #222;
}

.chat-container h5 {
  font-size: 1.1rem;
  color: #666;
  margin-bottom: 2rem;
  font-weight: 400;
}

textarea {
  width: 100%;
  min-height: 110px;
  padding: 1rem 1.2rem;
  border-radius: 12px;
  border: 1.8px solid #bbb;
  font-size: 1.05rem;
  resize: vertical;
  box-shadow: inset 0 2px 6px rgba(0, 0, 0, 0.07);
  transition: border-color 0.3s ease;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

textarea:focus {
  border-color: #1976d2;
  outline: none;
  box-shadow: 0 0 8px rgba(25, 118, 210, 0.4);
}

button {
  margin-top: 1.2rem;
  padding: 0.75rem 2rem;
  font-size: 1.1rem;
  border-radius: 30px;
  border: none;
  background-color: #1976d2;
  color: white;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.3s ease, box-shadow 0.3s ease;
  box-shadow: 0 6px 12px rgba(25, 118, 210, 0.3);
  align-self: flex-start;
  user-select: none;
}

button:disabled {
  background-color: #90caf9;
  cursor: not-allowed;
  box-shadow: none;
}

button:not(:disabled):hover {
  background-color: #115293;
  box-shadow: 0 8px 16px rgba(17, 82, 147, 0.5);
}

.response-section {
  margin-top: 2rem;
  margin-left: 1rem;
  margin-bottom: 1.5rem;
  background: #f5f9ff;
  border-radius: 16px;
  padding: 2rem 2.5rem;
  box-shadow: 0 4px 15px rgba(25, 118, 210, 0.15);
  max-width: 1300px;
  line-height: 1.5;
}

.response-text {
  white-space: pre-wrap;
  word-break: break-word;
  font-size: 1.1rem;
  color: #1a237e;
  margin-bottom: 1.6rem;
  font-weight: 500;
}

.response-text.question {
  border-left: 5px solid #64b5f6;
  padding-left: 1rem;
}

.response-text.answer {
  border-left: 5px solid #1976d2;
  padding-left: 1rem;
  color: #0d47a1;
}

.feedback-buttons {
  display: flex;
  gap: 1rem;
}

.feedback-buttons button {
  flex: 1;
  padding: 0.75rem 0;
  border-radius: 25px;
  font-weight: 700;
  font-size: 1rem;
  box-shadow: none;
  transition: filter 0.2s ease;
}

.feedback-buttons button:hover {
  filter: brightness(0.9);
}

.btn-correct {
  background-color: #4caf50;
  color: white;
}

.btn-incorrect {
  background-color: #e53935;
  color: white;
}

.footer {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  padding: 0.8rem;
  text-align: center;
  background: #e3f2fd;
  font-size: 0.95rem;
  color: #555;
  border-top: 1px solid #ccc;
  user-select: none;
  font-weight: 600;
}
</style>
