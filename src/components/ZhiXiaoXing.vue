<template>
  <div class="zxx-root">

    <!-- ══ STATE 1: FAB 悬浮机器人 ══ -->
    <Transition name="zxx-fab">
      <div v-if="!showMiniChat && !isFullscreen"
        class="zxx-fab-wrap"
        @mouseenter="robotHover = true"
        @mouseleave="robotHover = false"
        @click="openMiniChat">

        <!-- 气泡（横向，在机器人左侧） -->
        <Transition name="zxx-bubble">
          <div v-if="bubbleVisible && !robotHover" class="zxx-bubble">
            {{ currentBubble }}
            <div class="zxx-bubble-tail"></div>
          </div>
        </Transition>

        <!-- 悬浮时提示（横向） -->
        <Transition name="zxx-bubble">
          <div v-if="robotHover" class="zxx-bubble zxx-bubble--hover">
            💬 点击开始对话
            <div class="zxx-bubble-tail"></div>
          </div>
        </Transition>

        <div class="zxx-robot" :class="robotHover ? 'zxx-robot--hover' : ''">
          <img src="/robot.png" class="w-16 h-16 object-contain" />
          <span class="zxx-online-dot"></span>
        </div>
      </div>
    </Transition>

    <!-- ══ STATE 2: 迷你对话窗 ══ -->
    <Transition name="zxx-mini-chat">
      <div v-if="showMiniChat && !isFullscreen" class="zxx-mini-panel">

        <!-- 顶栏 -->
        <div class="zxx-mini-header">
          <div class="zxx-mini-header-left">
            <img src="/robot.png" class="w-7 h-7 object-contain" />
            <div>
              <p class="zxx-mini-title">智小星</p>
              <span class="zxx-mini-online">● 在线</span>
            </div>
          </div>
          <div class="zxx-mini-header-right">
            <!-- 全屏按钮 -->
            <button @click.stop="enterFullscreen" class="zxx-mini-icon-btn" title="全屏模式">
              <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.8"
                  d="M4 8V4h4M20 8V4h-4M4 16v4h4M20 16v4h-4"/>
              </svg>
            </button>
            <!-- 关闭按钮 -->
            <button @click.stop="closeMiniChat" class="zxx-mini-icon-btn zxx-mini-icon-btn--close" title="关闭">
              <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/>
              </svg>
            </button>
          </div>
        </div>

        <!-- 消息区 -->
        <div ref="miniMsgList" class="zxx-mini-messages">
          <!-- 欢迎消息 -->
          <div class="zxx-msg zxx-msg--ai">
            <div class="zxx-mini-avatar">✨</div>
            <div class="zxx-msg-bubble zxx-msg-bubble--ai">
              {{ currentPersona.welcome }}
            </div>
          </div>
          <!-- 对话记录 -->
          <template v-for="msg in messages" :key="msg.id">
            <div class="zxx-msg zxx-msg--user">
              <div class="zxx-msg-bubble zxx-msg-bubble--user">{{ msg.text }}</div>
            </div>
            <div class="zxx-msg zxx-msg--ai">
              <div class="zxx-mini-avatar">✨</div>
              <div v-if="msg.state === 'thinking'" class="zxx-msg-bubble zxx-msg-bubble--ai zxx-thinking">
                <span class="zxx-dot"></span><span class="zxx-dot"></span><span class="zxx-dot"></span>
              </div>
              <div v-else class="zxx-msg-bubble zxx-msg-bubble--ai">
                {{ msg.reply }}<span v-if="msg.state === 'typing'" class="zxx-cursor">|</span>
              </div>
            </div>
          </template>
        </div>

        <!-- 推荐气泡 -->
        <div class="zxx-chips">
          <button v-for="s in currentPersona.suggestions" :key="s"
            @click="sendSuggestion(s)" :disabled="isTyping" class="zxx-chip">
            {{ s }}
          </button>
        </div>

        <!-- 输入框 -->
        <div class="zxx-mini-input-wrap">
          <input v-model="miniInput" @keydown.enter="handleMiniSend"
            placeholder="向智小星提问…" class="zxx-mini-input" :disabled="isTyping"/>
          <button @click="handleMiniSend" class="zxx-mini-send"
            :class="miniInput.trim() ? 'zxx-mini-send--active' : ''">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                d="M12 19l9 2-9-18-9 18 9-2zm0 0v-8"/>
            </svg>
          </button>
        </div>

        <!-- Toast -->
        <Transition name="zxx-toast">
          <div v-if="showMiniToast" class="zxx-mini-toast">
            🌟 交互功能敬请期待~
          </div>
        </Transition>
      </div>
    </Transition>

    <!-- ══ STATE 3: 全屏模式 (Light Macaron) ══ -->
    <Transition name="zxx-fs">
      <div v-if="isFullscreen" class="zxx-fullscreen">

        <!-- 背景装饰光晕（浅紫，居中） -->
        <div class="zxx-center-glow"></div>

        <!-- AI 横向气泡提示 -->
        <Transition name="zxx-fs-bubble">
          <div v-if="fsAIBubble" class="zxx-fs-ai-bubble">
            <img src="/robot.png" class="w-7 h-7 object-contain flex-shrink-0" />
            <span>{{ fsAIBubble }}</span>
          </div>
        </Transition>

        <!-- 扫描框动画 -->
        <Transition name="zxx-scan">
          <div v-if="showScanOverlay" class="zxx-scan-overlay" @click="showScanOverlay = false">
            <div class="zxx-scan-box">
              <div class="zxx-scan-corner zxx-scan-corner--tl"></div>
              <div class="zxx-scan-corner zxx-scan-corner--tr"></div>
              <div class="zxx-scan-corner zxx-scan-corner--bl"></div>
              <div class="zxx-scan-corner zxx-scan-corner--br"></div>
              <div class="zxx-scan-line"></div>
              <p class="zxx-scan-hint">请录入或输入需要解析的公式</p>
            </div>
            <p class="zxx-scan-close">点击任意处关闭</p>
          </div>
        </Transition>

        <!-- ── 左侧边栏 ── -->
        <aside class="zxx-sidebar">
          <div class="zxx-sidebar-top">
            <img src="/robot.png" class="w-8 h-8 object-contain" />
            <span class="zxx-sidebar-brand">智小星</span>
          </div>

          <button class="zxx-new-chat-btn">
            <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M12 4v16m8-8H4"/>
            </svg>
            开启新问答
          </button>

          <p class="zxx-history-label">最近对话</p>
          <div class="zxx-history-list">
            <div v-for="h in historyList" :key="h.id" class="zxx-history-item">
              <svg class="w-3.5 h-3.5 flex-shrink-0 text-slate-300" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.8"
                  d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z"/>
              </svg>
              <span class="truncate">{{ h.title }}</span>
            </div>
          </div>
        </aside>

        <!-- ── 中央核心区 ── -->
        <main class="zxx-main">

          <!-- 退出按钮（扁平描边） -->
          <button @click="exitFullscreen" class="zxx-exit-btn">
            <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/>
            </svg>
            退出全屏
          </button>

          <!-- 吉祥物 + 横向气泡 -->
          <div class="zxx-main-hero">
            <div class="zxx-mascot-row">
              <!-- 横向语音气泡 -->
              <div class="zxx-mascot-bubble">
                <span>有不懂的题，随时问我 🤖</span>
                <div class="zxx-mascot-bubble-tail"></div>
              </div>
              <!-- 机器人 -->
              <img src="/robot.png" class="w-16 h-16 object-contain zxx-hero-robot" />
            </div>
            <h1 class="zxx-main-title">智小星</h1>
            <p class="zxx-main-sub">您的 AI 智能学伴，随时攻克薄弱知识点</p>
          </div>

          <!-- 模式切换 -->
          <div class="zxx-mode-row">
            <button @click="activeMode = 'study'" class="zxx-mode-btn"
              :class="activeMode === 'study' ? 'zxx-mode-btn--on' : ''">
              📚 学情模式
            </button>
            <button @click="activeMode = 'expert'" class="zxx-mode-btn"
              :class="activeMode === 'expert' ? 'zxx-mode-btn--on' : ''">
              🧠 专家模式
            </button>
          </div>

          <!-- 输入框（pill 型） -->
          <div class="zxx-fs-input-wrap" :class="{ 'zxx-fs-input-wrap--focus': fsInputFocused }">
            <svg class="zxx-fs-input-ico" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"
                d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0"/>
            </svg>
            <input v-model="fsInput"
              @keydown.enter="handleFsSend"
              @focus="fsInputFocused = true"
              @blur="fsInputFocused = false"
              placeholder="向智小星提问，攻克薄弱知识点..."
              class="zxx-fs-input"/>
            <button @click="handleFsSend" class="zxx-fs-send"
              :class="fsInput.trim() ? 'zxx-fs-send--on' : ''">
              <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                  d="M12 19l9 2-9-18-9 18 9-2zm0 0v-8"/>
              </svg>
            </button>
          </div>

          <!-- Bento 快捷卡片 -->
          <div class="zxx-bento-grid">
            <button v-for="qa in quickActions" :key="qa.label"
              @click="handleQuickAction(qa)" class="zxx-bento-card">
              <div class="zxx-bento-icon-wrap" :style="{ background: qa.bg }">
                <span class="text-xl">{{ qa.icon }}</span>
              </div>
              <div class="zxx-bento-text">
                <p class="zxx-bento-label">{{ qa.label }}</p>
                <p class="zxx-bento-desc">{{ qa.desc }}</p>
              </div>
              <svg class="zxx-bento-arrow" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5l7 7-7 7"/>
              </svg>
            </button>
          </div>

        </main>

        <!-- 全屏 Toast -->
        <Transition name="zxx-toast">
          <div v-if="showFsToast" class="zxx-fs-toast">
            🌟 交互功能敬请期待~
          </div>
        </Transition>
      </div>
    </Transition>

  </div>
</template>

<script setup>
import { ref, nextTick, onMounted, onUnmounted } from 'vue'

const emit = defineEmits(['navigate'])

// ── 三状态 ──
const showMiniChat  = ref(false)
const isFullscreen  = ref(false)
const robotHover    = ref(false)

function openMiniChat()   { showMiniChat.value = true }
function closeMiniChat()  { showMiniChat.value = false }
function enterFullscreen() { showMiniChat.value = false; isFullscreen.value = true }
function exitFullscreen()  { isFullscreen.value = false }

// ── 气泡逻辑 ──
const bubbleTexts = [
  '今天学了什么？来考考我吧 🎯',
  '你的函数专题还差一步就满分啦！',
  '有不懂的题，随时问我 🤖',
  '坚持学习 7 天，解锁新成就 ✨',
  '我帮你分析一下今天的薄弱点？',
]
const bubbleVisible  = ref(false)
const currentBubble  = ref('')
let lastBubbleIdx    = -1
let bubbleTimer      = null
let bubbleHideTimer  = null

function showNextBubble() {
  let idx
  do { idx = Math.floor(Math.random() * bubbleTexts.length) } while (idx === lastBubbleIdx)
  lastBubbleIdx       = idx
  currentBubble.value = bubbleTexts[idx]
  bubbleVisible.value = true
  bubbleHideTimer     = setTimeout(() => { bubbleVisible.value = false }, 5000)
}

// ── 性格配置 ──
const personas = [
  {
    id: 'gentle',
    welcome: '嗨～我是智小星 🌸 有什么学习困惑，尽管问我！',
    suggestions: ['今天作业好难怎么办？', '我总是记不住单词', '帮我分析薄弱点'],
    replies: {
      '今天作业好难怎么办？': '没关系的！难题说明你在挑战自己的边界 🌱 先看懂题意，再找已知条件，一步步推导。你已经很棒了，加油！',
      '我总是记不住单词': '记单词试试"联想记忆法"：把单词和有趣画面绑在一起 🎨 比如 apple 想象大苹果砸在牛顿头上。每天 10 个，坚持一个月会有惊喜！',
      '帮我分析薄弱点': '根据你近期的学习数据，数学函数模块正确率 62%，低于平均水平 📊 建议先从"二次函数图像"专题入手，打好基础再往上走！',
    },
    defaultReply: '这个问题很好！让我帮你梳理一下思路 🌸 学习是一段旅程，每一步都算数～',
  },
]
const currentPersona = ref(personas[0])

// ── 对话状态 ──
const messages      = ref([])
const miniInput     = ref('')
const isTyping      = ref(false)
const miniMsgList   = ref(null)
const showMiniToast = ref(false)
let   msgId         = 0

async function scrollBottom() {
  await nextTick()
  if (miniMsgList.value) miniMsgList.value.scrollTop = miniMsgList.value.scrollHeight
}

async function sendSuggestion(text) {
  if (isTyping.value) return
  isTyping.value = true
  const id = ++msgId
  messages.value.push({ id, text, state: 'thinking', reply: '' })
  await scrollBottom()
  await new Promise(r => setTimeout(r, 1500))
  const fullReply = currentPersona.value.replies[text] || currentPersona.value.defaultReply
  const msg = messages.value.find(m => m.id === id)
  msg.state = 'typing'
  msg.reply = ''
  for (let i = 0; i < fullReply.length; i++) {
    msg.reply += fullReply[i]
    await new Promise(r => setTimeout(r, 28))
    if (i % 6 === 0) await scrollBottom()
  }
  msg.state = 'done'
  isTyping.value = false
  await scrollBottom()
}

function handleMiniSend() {
  if (miniInput.value.trim()) {
    miniInput.value = ''
    showMiniToast.value = true
    setTimeout(() => { showMiniToast.value = false }, 2800)
  }
}

// ── 全屏状态 ──
const fsInput        = ref('')
const fsInputFocused = ref(false)
const activeMode    = ref('study')
const showFsToast   = ref(false)
const fsAIBubble    = ref('')
const showScanOverlay = ref(false)
let   fsBubbleTimer = null

function handleFsSend() {
  if (fsInput.value.trim()) {
    fsInput.value = ''
    showFsToast.value = true
    setTimeout(() => { showFsToast.value = false }, 2800)
  }
}

function showFsAIBubble(text) {
  clearTimeout(fsBubbleTimer)
  fsAIBubble.value = text
  fsBubbleTimer = setTimeout(() => { fsAIBubble.value = '' }, 4000)
}

const quickActions = [
  { icon: '📝', label: '错题本',    desc: '查看历史错题', action: 'mistakes', bg: 'linear-gradient(135deg,#fce7f3,#fbcfe8)' },
  { icon: '📊', label: '月度报告',  desc: 'AI 学情体检',  action: 'report',   bg: 'linear-gradient(135deg,#ede9fe,#ddd6fe)' },
  { icon: '🎓', label: '名师推荐',  desc: '精选课程匹配', action: 'course',   bg: 'linear-gradient(135deg,#d1fae5,#a7f3d0)' },
  { icon: '🔢', label: '公式换算',  desc: '扫描解析公式', action: 'formula',  bg: 'linear-gradient(135deg,#fef3c7,#fde68a)' },
]

function handleQuickAction(qa) {
  if (qa.action === 'mistakes') {
    showFsAIBubble("正在从您的'二次函数'专项中提取典型错题...")
  } else if (qa.action === 'report') {
    isFullscreen.value = false
    emit('navigate', { view: 'analytics', analyticsView: 'REPORT' })
  } else if (qa.action === 'course') {
    showFsAIBubble("为你匹配了最适合的'几何证明精讲'视频，点击即可跳转 🎓")
  } else if (qa.action === 'formula') {
    showScanOverlay.value = true
  }
}

// ── 历史记录 mock ──
const historyList = [
  { id: 1, title: '二次函数图像怎么画？' },
  { id: 2, title: '英语时态总结' },
  { id: 3, title: '物理力学公式推导' },
  { id: 4, title: '语文议论文写作技巧' },
  { id: 5, title: '数列通项公式求法' },
  { id: 6, title: '化学方程式配平方法' },
]

// ── 生命周期 ──
onMounted(() => {
  setTimeout(showNextBubble, 2000)
  bubbleTimer = setInterval(() => {
    bubbleVisible.value = false
    clearTimeout(bubbleHideTimer)
    setTimeout(showNextBubble, 600)
  }, 8000)
})

onUnmounted(() => {
  clearInterval(bubbleTimer)
  clearTimeout(bubbleHideTimer)
  clearTimeout(fsBubbleTimer)
})
</script>

<style scoped>
/* ── Root ── */
.zxx-root { position: fixed; bottom: 28px; right: 28px; z-index: 9999; display: flex; flex-direction: column; align-items: flex-end; gap: 8px; }

/* ══ FAB 机器人 ══ */
.zxx-fab-wrap {
  position: relative;
  display: flex; align-items: flex-end; justify-content: flex-end;
  width: 68px; height: 68px;
  cursor: pointer;
}
.zxx-robot {
  position: relative; width: 68px; height: 68px; flex-shrink: 0;
  animation: zxx-float 3s ease-in-out infinite;
  filter: drop-shadow(0 6px 18px rgba(124,58,237,.4));
  transition: filter .2s, transform .15s;
}
.zxx-robot--hover { transform: scale(1.06); filter: drop-shadow(0 10px 24px rgba(124,58,237,.6)); }
.zxx-online-dot {
  position: absolute; bottom: 6px; right: 4px;
  width: 10px; height: 10px; border-radius: 50%;
  background: #4ade80; border: 2px solid #fff;
  box-shadow: 0 0 8px rgba(74,222,128,.8);
}

/* 横向气泡：左上方 45° 位置，尾巴朝右下指向机器人 */
.zxx-bubble {
  position: absolute;
  bottom: calc(100% + 8px);
  right: calc(100% - 35px);
  background: #fff; border: 1.5px solid rgba(124,58,237,.2);
  border-radius: 14px; padding: 9px 16px;
  font-size: 12px; font-weight: 600; color: #3b0764;
  white-space: nowrap; line-height: 1.4;
  box-shadow: 0 6px 20px rgba(124,58,237,.15);
}
.zxx-bubble--hover {
  background: linear-gradient(135deg, #7c3aed, #a855f7);
  color: #fff; border-color: transparent;
}
/* 尾巴：右下角，朝右下 45° 指向机器人 */
.zxx-bubble-tail {
  position: absolute; bottom: -8px; right: 14px;
  width: 12px; height: 12px;
  background: inherit;
  border-right: 1.5px solid rgba(124,58,237,.2);
  border-bottom: 1.5px solid rgba(124,58,237,.2);
  transform: rotate(45deg);
  border-radius: 0 0 3px 0;
}
.zxx-bubble--hover .zxx-bubble-tail {
  border-color: transparent;
}

/* ══ 迷你对话窗 ══ */
.zxx-mini-panel {
  width: 320px; height: 420px;
  background: rgba(255,255,255,.92);
  backdrop-filter: blur(20px) saturate(160%);
  -webkit-backdrop-filter: blur(20px) saturate(160%);
  border: 1px solid rgba(124,58,237,.12);
  border-radius: 22px;
  box-shadow: 0 20px 60px rgba(124,58,237,.15), 0 4px 16px rgba(0,0,0,.08);
  display: flex; flex-direction: column; overflow: hidden;
  position: relative;
}
.zxx-mini-header {
  display: flex; align-items: center; justify-content: space-between;
  padding: 12px 14px 10px;
  border-bottom: 1px solid rgba(124,58,237,.08);
  background: linear-gradient(135deg, rgba(124,58,237,.06), rgba(168,85,247,.03));
  flex-shrink: 0;
}
.zxx-mini-header-left { display: flex; align-items: center; gap: 8px; }
.zxx-mini-title { font-size: 13px; font-weight: 900; color: #1e1b4b; line-height: 1.1; }
.zxx-mini-online { font-size: 10px; color: #4ade80; font-weight: 600; }
.zxx-mini-header-right { display: flex; align-items: center; gap: 4px; }
.zxx-mini-icon-btn {
  width: 26px; height: 26px; border-radius: 8px; border: none; cursor: pointer;
  background: rgba(124,58,237,.06); color: #7c3aed;
  display: flex; align-items: center; justify-content: center;
  transition: background .15s;
}
.zxx-mini-icon-btn:hover { background: rgba(124,58,237,.14); }
.zxx-mini-icon-btn--close { color: #64748b; }
.zxx-mini-icon-btn--close:hover { background: rgba(239,68,68,.1); color: #ef4444; }

/* 消息区 */
.zxx-mini-messages {
  flex: 1; overflow-y: auto; padding: 12px 12px 6px;
  display: flex; flex-direction: column; gap: 10px;
  scrollbar-width: thin; scrollbar-color: rgba(124,58,237,.15) transparent;
}
.zxx-msg { display: flex; align-items: flex-end; gap: 6px; }
.zxx-msg--user { flex-direction: row-reverse; }
.zxx-mini-avatar {
  width: 24px; height: 24px; border-radius: 8px; flex-shrink: 0;
  background: linear-gradient(135deg, rgba(124,58,237,.1), rgba(168,85,247,.07));
  display: flex; align-items: center; justify-content: center; font-size: 12px;
}
.zxx-msg-bubble {
  max-width: 82%; padding: 8px 11px;
  font-size: 12px; line-height: 1.6; border-radius: 14px;
}
.zxx-msg-bubble--ai {
  background: #fff; border: 1px solid rgba(124,58,237,.1); color: #1e293b;
  border-bottom-left-radius: 4px; box-shadow: 0 1px 6px rgba(0,0,0,.05);
}
.zxx-msg-bubble--user {
  background: linear-gradient(135deg, #7c3aed, #a855f7); color: #fff;
  font-weight: 600; border-bottom-right-radius: 4px;
  box-shadow: 0 3px 10px rgba(124,58,237,.3);
}
.zxx-thinking { display: flex; align-items: center; gap: 4px; padding: 10px 13px; }
.zxx-dot {
  width: 6px; height: 6px; border-radius: 50%;
  background: linear-gradient(135deg, #7c3aed, #a855f7);
  animation: zxx-bounce 1.2s ease-in-out infinite;
}
.zxx-dot:nth-child(2) { animation-delay: .2s; }
.zxx-dot:nth-child(3) { animation-delay: .4s; }
.zxx-cursor { color: #7c3aed; animation: zxx-blink .7s step-end infinite; margin-left: 1px; }

/* 推荐气泡 */
.zxx-chips {
  display: flex; flex-wrap: wrap; gap: 5px; padding: 6px 10px 4px; flex-shrink: 0;
}
.zxx-chip {
  padding: 4px 10px; border-radius: 16px; border: 1px solid rgba(124,58,237,.15);
  background: rgba(124,58,237,.06); color: #6d28d9;
  font-size: 10px; font-weight: 700; cursor: pointer; transition: all .15s;
}
.zxx-chip:hover:not(:disabled) { background: rgba(124,58,237,.14); transform: translateY(-1px); }
.zxx-chip:disabled { opacity: .4; cursor: not-allowed; }

/* 迷你输入框 */
.zxx-mini-input-wrap {
  display: flex; align-items: center; gap: 6px;
  padding: 8px 10px 10px; flex-shrink: 0;
}
.zxx-mini-input {
  flex: 1; padding: 8px 12px; font-size: 12px; color: #1e293b;
  background: rgba(248,250,252,.9); border: 1.5px solid rgba(124,58,237,.15);
  border-radius: 16px; outline: none; transition: border-color .2s, box-shadow .2s;
}
.zxx-mini-input:focus { border-color: rgba(124,58,237,.4); box-shadow: 0 0 0 3px rgba(124,58,237,.08); }
.zxx-mini-input::placeholder { color: #94a3b8; }
.zxx-mini-send {
  width: 32px; height: 32px; border-radius: 50%; border: none; cursor: pointer; flex-shrink: 0;
  background: rgba(124,58,237,.08); color: #94a3b8;
  display: flex; align-items: center; justify-content: center; transition: all .2s;
}
.zxx-mini-send--active { background: linear-gradient(135deg, #7c3aed, #a855f7); color: #fff; box-shadow: 0 3px 10px rgba(124,58,237,.4); }
.zxx-mini-toast {
  position: absolute; bottom: 52px; left: 50%; transform: translateX(-50%);
  background: rgba(30,27,75,.9); color: #fff; font-size: 11px; font-weight: 600;
  padding: 7px 14px; border-radius: 20px; white-space: nowrap;
  box-shadow: 0 4px 16px rgba(0,0,0,.2);
}

/* ══ 全屏 (Light Macaron) ══ */
.zxx-fullscreen {
  position: fixed; inset: 0; z-index: 10000;
  background: #f8fafc;
  display: flex; overflow: hidden;
}

/* 中心光晕（浅紫，柔和） */
.zxx-center-glow {
  position: absolute; width: 700px; height: 700px; border-radius: 50%;
  top: 50%; left: 50%; transform: translate(-50%, -50%);
  background: radial-gradient(circle, rgba(167,139,250,.1) 0%, transparent 65%);
  pointer-events: none;
}

/* AI 横向气泡提示 */
.zxx-fs-ai-bubble {
  position: absolute; top: 20px; left: 50%; transform: translateX(-50%);
  display: flex; align-items: center; gap: 10px;
  background: #fff; border: 1.5px solid rgba(124,58,237,.2);
  border-radius: 24px; padding: 10px 20px;
  color: #4c1d95; font-size: 13px; font-weight: 700;
  white-space: nowrap; z-index: 10;
  box-shadow: 0 4px 20px rgba(124,58,237,.12), 0 1px 4px rgba(0,0,0,.06);
}

/* 扫描框 */
.zxx-scan-overlay {
  position: absolute; inset: 0; z-index: 20;
  background: rgba(248,250,252,.88); backdrop-filter: blur(6px);
  display: flex; flex-direction: column; align-items: center; justify-content: center; gap: 20px;
  cursor: pointer;
}
.zxx-scan-box {
  position: relative; width: 360px; height: 240px;
  border: 1.5px solid rgba(124,58,237,.25);
  border-radius: 16px; background: rgba(124,58,237,.04);
  overflow: hidden; display: flex; align-items: center; justify-content: center;
  box-shadow: 0 8px 40px rgba(124,58,237,.1);
}
.zxx-scan-corner { position: absolute; width: 20px; height: 20px; border-color: #7c3aed; border-style: solid; }
.zxx-scan-corner--tl { top: 8px; left: 8px; border-width: 2px 0 0 2px; border-radius: 3px 0 0 0; }
.zxx-scan-corner--tr { top: 8px; right: 8px; border-width: 2px 2px 0 0; border-radius: 0 3px 0 0; }
.zxx-scan-corner--bl { bottom: 8px; left: 8px; border-width: 0 0 2px 2px; border-radius: 0 0 0 3px; }
.zxx-scan-corner--br { bottom: 8px; right: 8px; border-width: 0 2px 2px 0; border-radius: 0 0 3px 0; }
.zxx-scan-line {
  position: absolute; left: 0; right: 0; height: 2px;
  background: linear-gradient(90deg, transparent, #7c3aed, transparent);
  animation: zxx-scan 2s ease-in-out infinite;
}
.zxx-scan-hint { font-size: 13px; color: #6d28d9; font-weight: 700; }
.zxx-scan-close { font-size: 12px; color: #94a3b8; }

/* ── 侧边栏 (Light) ── */
.zxx-sidebar {
  width: 260px; flex-shrink: 0;
  background: #fcfaff;
  border-right: 1px solid #e8e0f7;
  display: flex; flex-direction: column; padding: 24px 14px; gap: 0; overflow: hidden;
}
.zxx-sidebar-top { display: flex; align-items: center; gap: 10px; margin-bottom: 20px; }
.zxx-sidebar-brand {
  font-size: 17px; font-weight: 900;
  background: linear-gradient(135deg, #7c3aed, #a855f7);
  -webkit-background-clip: text; -webkit-text-fill-color: transparent;
}
.zxx-new-chat-btn {
  display: flex; align-items: center; gap: 8px; width: 100%;
  padding: 9px 14px; margin-bottom: 18px;
  background: linear-gradient(135deg, #ede9fe, #ddd6fe);
  border: 1px solid #c4b5fd; border-radius: 10px;
  color: #6d28d9; font-size: 13px; font-weight: 700; cursor: pointer; transition: all .2s;
}
.zxx-new-chat-btn:hover { background: linear-gradient(135deg, #ddd6fe, #c4b5fd); box-shadow: 0 4px 14px rgba(124,58,237,.15); }
.zxx-history-label {
  font-size: 10px; font-weight: 700; color: #cbd5e1;
  letter-spacing: 1.2px; text-transform: uppercase; padding: 0 4px; margin-bottom: 6px;
}
.zxx-history-list { flex: 1; overflow-y: auto; display: flex; flex-direction: column; gap: 1px; }
.zxx-history-item {
  display: flex; align-items: center; gap: 8px;
  padding: 8px 10px; border-radius: 8px;
  color: #64748b; font-size: 12px; cursor: pointer; transition: all .15s;
}
.zxx-history-item:hover { background: #f3f0ff; color: #4c1d95; }

/* ── 中央 ── */
.zxx-main {
  flex: 1; display: flex; flex-direction: column; align-items: center; justify-content: center;
  padding: 40px 60px; position: relative; gap: 0;
}

/* 退出按钮（扁平描边） */
.zxx-exit-btn {
  position: absolute; top: 20px; right: 22px;
  display: flex; align-items: center; gap: 5px;
  padding: 6px 14px; border-radius: 8px; cursor: pointer;
  background: transparent; border: 1px solid #e2d9f8;
  color: #a78bfa; font-size: 12px; font-weight: 600; transition: all .2s;
}
.zxx-exit-btn:hover { border-color: #7c3aed; color: #7c3aed; background: #faf5ff; }

/* 吉祥物区（横向气泡） */
.zxx-main-hero { display: flex; flex-direction: column; align-items: center; gap: 10px; margin-bottom: 24px; }
.zxx-mascot-row {
  display: flex; flex-direction: row; align-items: center; gap: 14px; margin-bottom: 6px;
}
/* 横向气泡 */
.zxx-mascot-bubble {
  position: relative;
  background: #fff; border: 1.5px solid #e8d9fb;
  border-radius: 20px; padding: 9px 18px;
  font-size: 13px; font-weight: 700; color: #4c1d95;
  white-space: nowrap;
  box-shadow: 0 4px 16px rgba(124,58,237,.1);
  animation: zxx-breathe 3s ease-in-out infinite;
}
.zxx-mascot-bubble-tail {
  position: absolute; right: -9px; top: 50%; transform: translateY(-50%);
  width: 10px; height: 16px; background: #fff;
  clip-path: polygon(0 20%, 100% 50%, 0 80%);
  filter: drop-shadow(1px 0 1px rgba(124,58,237,.15));
}
.zxx-hero-robot { animation: zxx-float 3s ease-in-out infinite; filter: drop-shadow(0 6px 16px rgba(124,58,237,.25)); }

.zxx-main-title {
  font-size: 38px; font-weight: 900; line-height: 1;
  background: linear-gradient(135deg, #4c1d95 0%, #7c3aed 60%, #a855f7 100%);
  -webkit-background-clip: text; -webkit-text-fill-color: transparent;
}
.zxx-main-sub { font-size: 14px; color: #94a3b8; font-weight: 500; text-align: center; }

/* 模式切换 */
.zxx-mode-row { display: flex; gap: 10px; margin-bottom: 24px; margin-top: 18px; }
.zxx-mode-btn {
  padding: 9px 22px; border-radius: 22px;
  border: 1.5px solid #e2d9f8; background: #fff;
  color: #94a3b8; font-size: 13px; font-weight: 700; cursor: pointer; transition: all .2s;
}
.zxx-mode-btn:hover { border-color: #c4b5fd; color: #7c3aed; background: #faf5ff; }
.zxx-mode-btn--on {
  background: linear-gradient(135deg, #ede9fe, #ddd6fe);
  border-color: #a78bfa; color: #5b21b6;
  box-shadow: 0 4px 16px rgba(124,58,237,.15);
}

/* 全屏输入框（pill 型，光影） */
.zxx-fs-input-wrap {
  width: 100%; max-width: 640px; display: flex; align-items: center; gap: 12px;
  background: #fff; border: 1.5px solid #ddd6fe;
  border-radius: 999px; padding: 12px 16px 12px 22px;
  margin-bottom: 28px;
  box-shadow: 0 4px 24px rgba(124,58,237,.1), 0 1px 4px rgba(0,0,0,.04);
  transition: border-color .25s, box-shadow .25s;
}
.zxx-fs-input-wrap--focus {
  border-color: #7c3aed;
  box-shadow: 0 0 0 4px rgba(124,58,237,.1), 0 6px 32px rgba(124,58,237,.18);
}
.zxx-fs-input-ico { width: 18px; height: 18px; flex-shrink: 0; color: #a78bfa; }
.zxx-fs-input {
  flex: 1; background: transparent; border: none; outline: none;
  font-size: 15px; color: #1e1b4b; font-weight: 500;
}
.zxx-fs-input::placeholder { color: #c4b5fd; }
.zxx-fs-send {
  width: 40px; height: 40px; border-radius: 50%; border: none; cursor: pointer; flex-shrink: 0;
  background: #ede9fe; color: #a78bfa;
  display: flex; align-items: center; justify-content: center; transition: all .2s;
}
.zxx-fs-send--on {
  background: linear-gradient(135deg, #7c3aed, #a855f7); color: #fff;
  box-shadow: 0 4px 14px rgba(124,58,237,.4);
}
.zxx-fs-send--on:hover { transform: scale(1.08); }

/* Bento 快捷卡片 */
.zxx-bento-grid {
  display: grid; grid-template-columns: repeat(4, 1fr); gap: 12px;
  width: 100%; max-width: 640px;
}
.zxx-bento-card {
  display: flex; align-items: center; gap: 12px;
  padding: 14px 16px; border-radius: 16px;
  background: #fff; border: 1px solid #f1f0fb;
  cursor: pointer; transition: all .2s; text-align: left;
  box-shadow: 0 1px 4px rgba(0,0,0,.04);
}
.zxx-bento-card:hover {
  border-color: #c4b5fd; transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(124,58,237,.12);
}
.zxx-bento-icon-wrap {
  width: 38px; height: 38px; border-radius: 12px; flex-shrink: 0;
  display: flex; align-items: center; justify-content: center;
}
.zxx-bento-text { flex: 1; min-width: 0; }
.zxx-bento-label { font-size: 12px; font-weight: 800; color: #1e1b4b; }
.zxx-bento-desc  { font-size: 10px; color: #94a3b8; margin-top: 1px; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; }
.zxx-bento-arrow { width: 14px; height: 14px; flex-shrink: 0; color: #c4b5fd; transition: transform .2s; }
.zxx-bento-card:hover .zxx-bento-arrow { transform: translateX(2px); color: #7c3aed; }

/* Toast */
.zxx-fs-toast {
  position: absolute; bottom: 28px; left: 50%; transform: translateX(-50%);
  background: #4c1d95; color: #fff; font-size: 13px; font-weight: 600;
  padding: 11px 22px; border-radius: 20px; white-space: nowrap;
  box-shadow: 0 6px 24px rgba(124,58,237,.3);
}

/* ── Keyframes ── */
@keyframes zxx-float    { 0%,100%{transform:translateY(0)} 50%{transform:translateY(-8px)} }
@keyframes zxx-bounce   { 0%,80%,100%{transform:translateY(0);opacity:.6} 40%{transform:translateY(-6px);opacity:1} }
@keyframes zxx-blink    { 0%,100%{opacity:1} 50%{opacity:0} }
@keyframes zxx-scan     { 0%{top:0} 50%{top:calc(100% - 2px)} 100%{top:0} }
@keyframes zxx-pulse-dot { 0%,100%{opacity:1;transform:scale(1)} 50%{opacity:.5;transform:scale(.75)} }
@keyframes zxx-breathe  { 0%,100%{transform:translateY(0) scale(1)} 50%{transform:translateY(-3px) scale(1.01)} }

/* ── Transitions ── */
.zxx-fab-enter-active,.zxx-fab-leave-active { transition: all .25s cubic-bezier(.34,1.56,.64,1); }
.zxx-fab-enter-from,.zxx-fab-leave-to { opacity:0; transform:scale(.6); }

.zxx-mini-chat-enter-active { transition: all .32s cubic-bezier(.34,1.56,.64,1); }
.zxx-mini-chat-leave-active { transition: all .2s ease-in; }
.zxx-mini-chat-enter-from { opacity:0; transform:scale(.85) translateY(16px); transform-origin:bottom right; }
.zxx-mini-chat-leave-to  { opacity:0; transform:scale(.9)  translateY(10px); transform-origin:bottom right; }

.zxx-fs-enter-active { transition: all .45s cubic-bezier(.16,1,.3,1); }
.zxx-fs-leave-active { transition: all .3s cubic-bezier(.7,0,.84,0); }
.zxx-fs-enter-from { opacity:0; clip-path:circle(48px at calc(100% - 60px) calc(100% - 60px)); }
.zxx-fs-enter-to   { clip-path:circle(150% at 50% 50%); }
.zxx-fs-leave-from { clip-path:circle(150% at 50% 50%); }
.zxx-fs-leave-to   { opacity:0; clip-path:circle(48px at calc(100% - 60px) calc(100% - 60px)); }

.zxx-bubble-enter-active { transition: all .32s cubic-bezier(.34,1.56,.64,1); }
.zxx-bubble-leave-active { transition: all .18s ease-in; }
.zxx-bubble-enter-from,.zxx-bubble-leave-to { opacity:0; transform:translate(8px, 8px) scale(.88); }
.zxx-bubble-enter-to { opacity:1; transform:translate(0,0) scale(1); }

.zxx-fs-bubble-enter-active { transition: all .3s cubic-bezier(.34,1.56,.64,1); }
.zxx-fs-bubble-leave-active { transition: all .2s ease-in; }
.zxx-fs-bubble-enter-from,.zxx-fs-bubble-leave-to { opacity:0; transform:translateX(-50%) translateY(-8px) scale(.92); }

.zxx-scan-enter-active { transition: all .25s ease-out; }
.zxx-scan-leave-active { transition: all .2s ease-in; }
.zxx-scan-enter-from,.zxx-scan-leave-to { opacity:0; }

.zxx-toast-enter-active { transition: all .28s cubic-bezier(.34,1.56,.64,1); }
.zxx-toast-leave-active { transition: all .2s ease-in; }
.zxx-toast-enter-from,.zxx-toast-leave-to { opacity:0; transform:translateX(-50%) translateY(10px) scale(.94); }
</style>
