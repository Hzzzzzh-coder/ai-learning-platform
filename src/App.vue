<template>
  <div class="min-h-screen bg-slate-50" style="font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif">

    <!-- ── AI Modal ── -->
    <Transition name="modal">
      <div v-if="showAIModal" class="fixed inset-0 z-50 flex items-center justify-center p-4">
        <div class="absolute inset-0 bg-black/40 backdrop-blur-sm" @click="closeAIModal"></div>
        <div class="relative z-10 w-full max-w-2xl bg-white/80 backdrop-blur-2xl border border-white/60 rounded-3xl shadow-2xl overflow-hidden">
          <!-- Header -->
          <div class="bg-gradient-to-r from-violet-500 via-purple-500 to-fuchsia-500 p-6 text-white">
            <div class="flex items-center justify-between">
              <div class="flex items-center gap-3">
                <span class="text-3xl">{{ currentAIContent?.icon }}</span>
                <div>
                  <p class="text-xs font-medium text-white/70 uppercase tracking-widest">AI 智能助手</p>
                  <h3 class="text-xl font-black">{{ currentAIContent?.title }}</h3>
                </div>
              </div>
              <button @click="closeAIModal" class="w-9 h-9 rounded-full bg-white/20 hover:bg-white/30 flex items-center justify-center transition-colors">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/></svg>
              </button>
            </div>
          </div>

          <!-- Body -->
          <div class="p-6 max-h-[58vh] overflow-y-auto space-y-4">
            <div v-for="block in currentAIContent?.content" :key="block.type">
              <!-- summary -->
              <div v-if="block.type === 'summary'" class="bg-violet-50 border border-violet-100 rounded-2xl p-4">
                <p class="text-slate-700 text-sm leading-relaxed">{{ block.text }}</p>
              </div>
              <!-- keyPoints -->
              <div v-else-if="block.type === 'keyPoints'">
                <p class="text-xs font-semibold text-slate-400 uppercase tracking-widest mb-3">核心考点</p>
                <div class="grid grid-cols-2 gap-2">
                  <div v-for="(item, i) in block.items" :key="i" class="flex items-center gap-2.5 bg-white border border-slate-100 rounded-xl p-3 shadow-sm">
                    <span class="w-5 h-5 rounded-full bg-violet-100 text-violet-600 text-xs font-bold flex items-center justify-center flex-shrink-0">{{ i + 1 }}</span>
                    <span class="text-sm text-slate-700">{{ item }}</span>
                  </div>
                </div>
              </div>
              <!-- analysis -->
              <div v-else-if="block.type === 'analysis'">
                <p class="text-xs font-semibold text-slate-400 uppercase tracking-widest mb-3">深度解析</p>
                <div class="space-y-3">
                  <div v-for="item in block.items" :key="item.title" class="rounded-2xl p-4 border"
                    :class="item.level==='hard' ? 'bg-red-50 border-red-100' : item.level==='medium' ? 'bg-amber-50 border-amber-100' : 'bg-blue-50 border-blue-100'">
                    <div class="flex items-start gap-3">
                      <span class="text-xs px-2 py-0.5 rounded-full font-semibold mt-0.5 flex-shrink-0"
                        :class="item.level==='hard' ? 'bg-red-100 text-red-600' : item.level==='medium' ? 'bg-amber-100 text-amber-600' : 'bg-blue-100 text-blue-600'">
                        {{ item.level==='hard' ? '难点' : item.level==='medium' ? '注意' : '考点' }}
                      </span>
                      <div>
                        <p class="text-sm font-bold text-slate-800 mb-1">{{ item.title }}</p>
                        <p class="text-sm text-slate-600">{{ item.desc }}</p>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
              <!-- tip -->
              <div v-else-if="block.type === 'tip'" class="flex items-start gap-3 bg-emerald-50 border border-emerald-100 rounded-2xl p-4">
                <span class="text-xl flex-shrink-0">💡</span>
                <p class="text-sm text-emerald-700">{{ block.text }}</p>
              </div>
            </div>
          </div>

          <!-- Footer -->
          <div class="border-t border-slate-100 p-4 flex items-center justify-between bg-white/60">
            <button class="text-sm text-slate-400 hover:text-slate-600 transition-colors">加入复习计划</button>
            <div class="flex gap-2">
              <button @click="closeAIModal" class="px-4 py-2 text-sm rounded-xl border border-slate-200 hover:bg-slate-50 transition-colors">关闭</button>
              <button class="px-5 py-2 text-sm font-bold rounded-xl bg-gradient-to-r from-violet-500 to-purple-600 text-white hover:shadow-lg hover:scale-[1.02] transition-all">立即学习</button>
            </div>
          </div>
        </div>
      </div>
    </Transition>

    <!-- ── Focus Mode Overlay ── -->
    <Transition name="focus">
      <div v-if="focusMode" class="fixed inset-0 z-50 bg-[#060912] flex overflow-hidden">
        <!-- Star field bg -->
        <div class="absolute inset-0 overflow-hidden pointer-events-none">
          <div v-for="i in 60" :key="i" class="star" :style="{
            left: (Math.sin(i * 137.5 * Math.PI / 180) * 50 + 50) + '%',
            top:  (Math.cos(i * 97.3  * Math.PI / 180) * 50 + 50) + '%',
            animationDelay: (i * 0.15) + 's',
            width:  (i % 3 === 0 ? 2 : 1) + 'px',
            height: (i % 3 === 0 ? 2 : 1) + 'px',
          }"></div>
        </div>

        <!-- Main panel: video -->
        <div class="flex-1 flex flex-col p-8 gap-5">
          <!-- Top bar -->
          <div class="flex items-center justify-between">
            <div class="flex items-center gap-3">
              <span class="text-white/50 text-sm font-mono">🌌 黑洞专注模式</span>
              <span class="w-1.5 h-1.5 rounded-full bg-emerald-400 animate-pulse"></span>
              <span class="text-emerald-400 text-xs font-semibold">AI 正在专注度监测中... 专注时长：{{ focusMinutes }} 分钟</span>
            </div>
            <button @click="toggleFocusMode"
              class="flex items-center gap-2 px-4 py-2 rounded-xl bg-white/5 hover:bg-white/10 text-white/60 hover:text-white text-sm transition-all border border-white/10">
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/></svg>
              退出专注
            </button>
          </div>

          <!-- Video area -->
          <div class="flex-1 rounded-3xl bg-black/50 border border-white/5 flex flex-col items-center justify-center relative overflow-hidden">
            <div class="absolute inset-0 bg-gradient-to-br from-violet-900/20 to-blue-900/20"></div>
            <div class="relative z-10 text-center">
              <div class="w-20 h-20 rounded-full border-4 border-violet-500/40 flex items-center justify-center mx-auto mb-4 hover:border-violet-400/60 cursor-pointer transition-colors">
                <svg class="w-8 h-8 text-white ml-1 opacity-80" fill="currentColor" viewBox="0 0 20 20">
                  <path d="M6.3 2.841A1.5 1.5 0 004 4.11v11.78a1.5 1.5 0 002.3 1.269l9.344-5.89a1.5 1.5 0 000-2.538L6.3 2.84z"/>
                </svg>
              </div>
              <p class="text-white/50 text-sm">{{ selectedCourse?.title || '初中数学-函数专题精讲' }}</p>
              <p class="text-violet-400/60 text-xs mt-1 font-mono">10:23 / 32:45</p>
            </div>
            <!-- Progress bar -->
            <div class="absolute bottom-5 left-8 right-8">
              <div class="h-0.5 bg-white/10 rounded-full">
                <div class="h-full w-1/3 bg-violet-500 rounded-full"></div>
              </div>
            </div>
          </div>
        </div>

        <!-- Notes panel -->
        <div class="w-72 flex-shrink-0 border-l border-white/5 flex flex-col p-5 gap-4">
          <div class="flex items-center gap-2">
            <span class="text-white/60 text-sm font-semibold">📝 专注笔记</span>
            <span class="text-xs text-white/30">自动保存</span>
          </div>
          <textarea v-model="focusNotes" placeholder="在这里记录灵感与要点..." rows="10"
            class="flex-1 bg-white/5 border border-white/10 rounded-2xl p-4 text-white/80 text-sm placeholder:text-white/20 focus:outline-none focus:border-violet-500/40 resize-none leading-relaxed"></textarea>
          <!-- AI tips in focus mode -->
          <div class="bg-violet-500/10 border border-violet-500/20 rounded-2xl p-4">
            <p class="text-xs text-violet-300/80 font-semibold mb-2">👁️ AI 专注洞察</p>
            <p class="text-xs text-white/40 leading-relaxed">检测到你已持续专注 {{ focusMinutes }} 分钟，处于高效学习状态。当前知识点：二次函数图像变换。</p>
          </div>
          <button @click="takeSnapshot"
            class="w-full py-2.5 rounded-xl bg-white/5 hover:bg-white/10 border border-white/10 text-white/60 hover:text-white text-sm font-semibold transition-all">
            📸 AI 快照当前时刻
          </button>
        </div>
      </div>
    </Transition>

    <!-- ── Focus Score Card (exit popup) ── -->
    <Transition name="slide-up">
      <div v-if="showFocusScore" class="fixed bottom-6 right-6 z-50 w-72 bg-white rounded-3xl shadow-2xl border border-slate-100 p-5 overflow-hidden">
        <div class="absolute inset-0 bg-gradient-to-br from-violet-50 to-purple-50 opacity-60"></div>
        <div class="relative z-10">
          <div class="flex items-center gap-3 mb-4">
            <div class="w-10 h-10 rounded-2xl bg-violet-100 flex items-center justify-center text-xl">🎯</div>
            <div>
              <p class="text-xs font-black text-violet-500 uppercase tracking-widest">专注度评分</p>
              <p class="text-sm font-black text-slate-800">本次学习报告</p>
            </div>
          </div>
          <div class="grid grid-cols-3 gap-2 mb-4">
            <div class="bg-white rounded-xl p-2.5 text-center shadow-sm">
              <p class="text-lg font-black text-violet-600">{{ focusMinutes }}</p>
              <p class="text-xs text-slate-400">分钟</p>
            </div>
            <div class="bg-white rounded-xl p-2.5 text-center shadow-sm">
              <p class="text-lg font-black text-emerald-500">92</p>
              <p class="text-xs text-slate-400">专注分</p>
            </div>
            <div class="bg-white rounded-xl p-2.5 text-center shadow-sm">
              <p class="text-lg font-black text-amber-500">+80</p>
              <p class="text-xs text-slate-400">XP 奖励</p>
            </div>
          </div>
          <div class="h-2 bg-slate-100 rounded-full overflow-hidden mb-3">
            <div class="h-full w-[92%] bg-gradient-to-r from-violet-400 to-purple-500 rounded-full"></div>
          </div>
          <p class="text-xs text-slate-500 text-center">优秀！你的专注度超过了班级 87% 的同学 🚀</p>
        </div>
      </div>
    </Transition>

    <!-- ── Class Auth Modal ── -->
    <Transition name="modal">
      <div v-if="showClassModal" class="fixed inset-0 z-50 flex items-center justify-center p-4">
        <div class="absolute inset-0 bg-black/40 backdrop-blur-sm" @click="showClassModal = false"></div>
        <div class="relative z-10 w-full max-w-sm bg-white rounded-3xl shadow-2xl overflow-hidden">
          <!-- Animated success overlay -->
          <Transition name="modal">
            <div v-if="showJoinSuccess" class="absolute inset-0 z-20 bg-emerald-500 flex flex-col items-center justify-center gap-4">
              <div class="w-16 h-16 rounded-full bg-white/20 flex items-center justify-center text-4xl animate-bounce">🎉</div>
              <p class="text-white font-black text-lg">入班成功！</p>
              <p class="text-white/80 text-sm">欢迎加入班级</p>
            </div>
          </Transition>
          <div class="bg-gradient-to-r from-violet-500 to-purple-600 p-6">
            <div class="flex items-center justify-between">
              <div>
                <p class="text-xs text-white/70 uppercase tracking-widest mb-1">班级认证</p>
                <h3 class="text-lg font-black text-white">加入班级</h3>
              </div>
              <button @click="showClassModal = false" class="w-8 h-8 rounded-full bg-white/20 hover:bg-white/30 flex items-center justify-center text-white transition-colors">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12"/></svg>
              </button>
            </div>
          </div>
          <div class="p-6 space-y-4">
            <p class="text-sm text-slate-500">请输入老师发放的班级码，加入后即可与班级同学同步课程进度。</p>
            <div>
              <label class="text-xs font-bold text-slate-500 uppercase tracking-widest mb-2 block">班级码</label>
              <input v-model="classCodeInput" @keydown.enter="joinClass"
                placeholder="例如：CLASS-802"
                class="w-full bg-slate-50 border-2 border-slate-200 focus:border-violet-400 rounded-2xl px-4 py-3 text-sm font-bold text-slate-800 placeholder:font-normal placeholder:text-slate-300 focus:outline-none transition-colors uppercase tracking-widest">
            </div>
            <div class="flex gap-2 text-xs text-slate-400">
              <span class="bg-violet-50 text-violet-500 px-2 py-1 rounded-lg font-semibold cursor-pointer hover:bg-violet-100" @click="classCodeInput='CLASS-802'">CLASS-802</span>
              <span class="bg-violet-50 text-violet-500 px-2 py-1 rounded-lg font-semibold cursor-pointer hover:bg-violet-100" @click="classCodeInput='CLASS-901'">CLASS-901</span>
              <span class="text-slate-300 leading-6">（快速填入示例码）</span>
            </div>
            <button @click="joinClass"
              class="w-full py-3 bg-gradient-to-r from-violet-500 to-purple-600 text-white font-black rounded-2xl hover:shadow-lg hover:scale-[1.02] transition-all text-sm">
              确认加入 →
            </button>
          </div>
        </div>
      </div>
    </Transition>

    <!-- ── Top Nav ── -->
    <header class="h-16 bg-white/90 backdrop-blur-sm border-b border-slate-100 flex items-center px-5 gap-4 sticky top-0 z-40">
      <!-- Logo + Grade Switcher -->
      <div class="flex items-center gap-2.5 w-56 flex-shrink-0">
        <div class="w-8 h-8 rounded-xl bg-gradient-to-br from-violet-500 to-purple-600 flex items-center justify-center shadow-sm shadow-violet-200">
          <svg class="w-4 h-4 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M12 6.253v13m0-13C10.832 5.477 9.246 5 7.5 5S4.168 5.477 3 6.253v13C4.168 18.477 5.754 18 7.5 18s3.332.477 4.5 1.253m0-13C13.168 5.477 14.754 5 16.5 5c1.747 0 3.332.477 4.5 1.253v13C19.832 18.477 18.247 18 16.5 18c-1.746 0-3.332.477-4.5 1.253"/>
          </svg>
        </div>
        <span class="font-black text-slate-800 text-sm tracking-tight">智学星球</span>
        <select v-model="currentGrade" class="ml-1 text-xs bg-slate-100 border-0 rounded-lg px-2 py-1.5 text-slate-600 font-semibold cursor-pointer focus:ring-2 focus:ring-violet-300 focus:outline-none">
          <option v-for="g in grades" :key="g" :value="g">{{ g }}</option>
        </select>
      </div>

      <!-- AI Search -->
      <div class="flex-1 max-w-lg mx-auto">
        <div class="relative">
          <svg class="absolute left-3 top-1/2 -translate-y-1/2 w-4 h-4 text-violet-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"/>
          </svg>
          <input type="text" placeholder="AI 智能搜索课程、题目、知识点..." v-model="searchQuery"
            class="w-full pl-10 pr-14 py-2.5 bg-slate-50 border border-slate-200 rounded-2xl text-sm focus:outline-none focus:ring-2 focus:ring-violet-300 focus:border-transparent placeholder:text-slate-400 transition-all">
          <span class="absolute right-3 top-1/2 -translate-y-1/2 text-xs bg-violet-100 text-violet-600 px-2 py-0.5 rounded-lg font-bold">AI</span>
        </div>
      </div>

      <!-- Right Controls -->
      <div class="flex items-center gap-3 ml-auto">
        <!-- Focus Mode Toggle -->
        <button @click="toggleFocusMode"
          class="flex items-center gap-2 px-3 py-2 rounded-xl text-sm font-bold transition-all duration-300 border"
          :class="focusMode ? 'bg-violet-900 border-violet-700 text-violet-300 shadow-lg shadow-violet-900/30' : 'bg-slate-50 border-slate-200 text-slate-600 hover:border-violet-300 hover:text-violet-600 hover:bg-violet-50'">
          <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11.049 2.927c.3-.921 1.603-.921 1.902 0l1.519 4.674a1 1 0 00.95.69h4.915c.969 0 1.371 1.24.588 1.81l-3.976 2.888a1 1 0 00-.363 1.118l1.518 4.674c.3.922-.755 1.688-1.538 1.118l-3.976-2.888a1 1 0 00-1.176 0l-3.976 2.888c-.783.57-1.838-.197-1.538-1.118l1.518-4.674a1 1 0 00-.363-1.118l-3.976-2.888c-.784-.57-.38-1.81.588-1.81h4.914a1 1 0 00.951-.69l1.519-4.674z"/>
          </svg>
          <span>{{ focusMode ? '专注中' : '专注模式' }}</span>
          <span v-if="focusMode" class="w-1.5 h-1.5 rounded-full bg-emerald-400 animate-pulse"></span>
        </button>
        <!-- Streak -->
        <div class="flex items-center gap-1.5 bg-orange-50 border border-orange-100 rounded-xl px-3 py-1.5 cursor-default">
          <span class="text-base leading-none">🔥</span>
          <span class="text-sm font-black text-orange-600">{{ user.streak }}</span>
        </div>
        <!-- XP Bar -->
        <div class="flex items-center gap-2">
          <div class="w-28 bg-slate-100 rounded-full h-2 overflow-hidden">
            <div class="h-full bg-gradient-to-r from-violet-400 to-purple-500 rounded-full transition-all duration-700"
              :style="{ width: xpPercent + '%' }"></div>
          </div>
          <div class="bg-violet-100 rounded-xl px-2.5 py-1 flex items-center gap-1">
            <span class="text-xs text-violet-500 font-medium">Lv</span>
            <span class="text-sm font-black text-violet-700">{{ user.level }}</span>
          </div>
        </div>
        <!-- Notification -->
        <button class="relative w-9 h-9 rounded-xl hover:bg-slate-100 flex items-center justify-center transition-colors">
          <svg class="w-5 h-5 text-slate-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 17h5l-1.405-1.405A2.032 2.032 0 0118 14.158V11a6.002 6.002 0 00-4-5.659V5a2 2 0 10-4 0v.341C7.67 6.165 6 8.388 6 11v3.159c0 .538-.214 1.055-.595 1.436L4 17h5m6 0v1a3 3 0 11-6 0v-1m6 0H9"/>
          </svg>
          <span class="absolute top-1.5 right-1.5 w-2 h-2 bg-red-400 rounded-full ring-2 ring-white"></span>
        </button>
        <!-- Avatar -->
        <div class="w-9 h-9 rounded-xl bg-gradient-to-br from-violet-400 to-purple-500 flex items-center justify-center text-white font-black text-sm cursor-pointer hover:shadow-md hover:shadow-violet-200 transition-all">
          {{ user.avatar }}
        </div>
      </div>
    </header>

    <!-- ── Main Layout ── -->
    <div class="flex h-[calc(100vh-64px)]">

      <!-- ── Left Sidebar ── -->
      <aside class="w-56 flex-shrink-0 bg-white border-r border-slate-100 flex flex-col py-4 px-3">
        <nav class="space-y-1">
          <button v-for="item in navItems" :key="item.view"
            @click="switchView(item.view)"
            class="w-full flex items-center gap-3 px-3 py-2.5 rounded-xl text-sm font-semibold transition-all duration-150"
            :class="currentView === item.view ? 'bg-violet-50 text-violet-700' : 'text-slate-500 hover:bg-slate-50 hover:text-slate-800'">
            <span class="w-8 h-8 rounded-lg flex items-center justify-center flex-shrink-0 transition-colors"
              :class="currentView === item.view ? 'bg-violet-100' : 'bg-slate-100'">
              <svg class="w-4 h-4" :class="currentView === item.view ? 'text-violet-600' : 'text-slate-400'"
                fill="none" stroke="currentColor" viewBox="0 0 24 24" v-html="item.icon"></svg>
            </span>
            {{ item.label }}
          </button>
        </nav>

        <!-- Profile card -->
        <div class="mt-auto pt-4 border-t border-slate-100">
          <div class="bg-gradient-to-br from-violet-50 to-purple-50 border border-violet-100 rounded-2xl p-3">
            <div class="flex items-center gap-2 mb-2.5">
              <div class="relative flex-shrink-0">
                <div class="w-8 h-8 rounded-xl bg-gradient-to-br from-violet-400 to-purple-500 flex items-center justify-center text-white font-black text-xs">{{ user.avatar }}</div>
                <Transition name="badge">
                  <span v-if="className" class="absolute -top-1.5 -right-1.5 bg-blue-500 text-white text-[9px] font-black px-1 py-0.5 rounded-md leading-tight whitespace-nowrap shadow-sm">{{ className }}</span>
                </Transition>
              </div>
              <div class="min-w-0">
                <p class="text-xs font-bold text-slate-800 truncate">{{ user.name }}</p>
                <p class="text-xs text-violet-500 font-medium">Lv{{ user.level }} 学习达人</p>
              </div>
            </div>
            <div class="flex justify-between text-xs text-slate-400 mb-1">
              <span>{{ user.xp }} / {{ user.xpToNext }} XP</span>
              <span class="font-semibold">{{ xpPercent }}%</span>
            </div>
            <div class="h-1.5 bg-white rounded-full overflow-hidden mb-2.5">
              <div class="h-full bg-gradient-to-r from-violet-400 to-purple-500 rounded-full transition-all duration-700"
                :style="{ width: xpPercent + '%' }"></div>
            </div>
            <!-- Class section -->
            <div v-if="className" class="flex items-center gap-1.5 bg-blue-50 border border-blue-100 rounded-xl px-2.5 py-1.5">
              <span class="text-base leading-none">🏫</span>
              <span class="text-xs font-bold text-blue-700">{{ className }}</span>
              <button @click="className=''" class="ml-auto text-blue-300 hover:text-blue-500 text-xs">✕</button>
            </div>
            <button v-else @click="showClassModal = true"
              class="w-full flex items-center justify-center gap-1.5 py-1.5 bg-white/70 hover:bg-white border border-dashed border-violet-200 hover:border-violet-400 rounded-xl text-xs font-bold text-violet-500 hover:text-violet-700 transition-all">
              <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M12 4v16m8-8H4"/></svg>
              加入班级
            </button>
          </div>
        </div>
      </aside>

      <!-- ── Main Content ── -->
      <main :class="currentView === 'schedule'
        ? 'flex-1 overflow-hidden flex flex-col'
        : 'flex-1 overflow-y-auto'"
        style="scrollbar-width: none; -ms-overflow-style: none;">

        <!-- ══ VIEW 1: Home Dashboard ══ -->
        <div v-if="currentView === 'home'" class="p-6 space-y-5">

          <!-- Hero Banner -->
          <div class="relative bg-gradient-to-r from-violet-600 via-purple-600 to-indigo-600 rounded-3xl p-8 overflow-hidden">
            <div class="absolute -top-10 -right-10 w-56 h-56 bg-white/5 rounded-full"></div>
            <div class="absolute -bottom-8 left-1/3 w-40 h-40 bg-white/5 rounded-full"></div>
            <div class="absolute top-4 right-32 w-20 h-20 bg-white/5 rounded-full"></div>
            <div class="relative z-10 flex items-center justify-between gap-8">
              <div>
                <p class="text-violet-200 text-sm font-semibold mb-3">🌟 本周名师推荐</p>
                <h1 class="text-3xl font-black text-white mb-5 leading-tight">
                  学习不是冲刺，<br>而是一场马拉松。
                </h1>
                <p class="text-violet-200 text-sm mb-6 max-w-sm leading-relaxed">
                  张华老师《初中数学·函数精讲》正在热播，已有 <strong class="text-white">3.2万</strong>名同学加入。现在开始，不让知识成为盲区。
                </p>
                <button @click="switchView('courses')"
                  class="bg-white text-violet-700 font-black px-6 py-3 rounded-2xl hover:shadow-xl hover:scale-105 transition-all text-sm">
                  立即开始学习 →
                </button>
              </div>
              <div class="hidden lg:flex flex-col items-center gap-3 flex-shrink-0">
                <div class="w-36 h-36 rounded-3xl bg-white/10 border border-white/20 flex flex-col items-center justify-center">
                  <p class="text-5xl font-black text-white">{{ user.streak }}</p>
                  <p class="text-violet-200 text-xs font-semibold mt-1">连续打卡天数</p>
                </div>
                <div class="flex gap-1">
                  <div v-for="i in 7" :key="i"
                    class="w-4 h-4 rounded-full text-xs flex items-center justify-center"
                    :class="i <= user.streak ? 'bg-orange-400' : 'bg-white/10'">
                    <span v-if="i <= user.streak" class="text-white text-xs">🔥</span>
                  </div>
                </div>
              </div>
            </div>
          </div>

          
          <!-- Stats Row -->
          <div class="grid grid-cols-4 gap-8">
            <div v-for="stat in homeStats" :key="stat.label"
              class="bg-white rounded-2xl p-4 shadow-sm border border-slate-100 text-center hover:shadow-md transition-shadow group relative overflow-hidden">
              <!-- bg accent strip -->
              <div class="absolute bottom-0 left-0 right-0 h-0.5 rounded-full transition-all duration-500"
                :class="stat.color.replace('text-', 'bg-')"
                :style="{ opacity: 0.3 }"></div>
              <p class="text-2xl font-black mb-0.5" :class="stat.color">{{ stat.value }}</p>
              <p class="text-xs text-slate-400 font-medium mb-2">{{ stat.label }}</p>
              <!-- Class percentile pill -->
              <div class="inline-flex items-center gap-1 rounded-full px-2 py-0.5 border"
                :class="stat.pct <= 20 ? 'bg-emerald-50 border-emerald-100' : stat.pct <= 60 ? 'bg-amber-50 border-amber-100' : 'bg-slate-50 border-slate-100'">
                <span class="text-xs font-black"
                  :class="stat.pct <= 20 ? 'text-emerald-600' : stat.pct <= 60 ? 'text-amber-500' : 'text-slate-400'">
                  班级前 {{ stat.pct }}%
                </span>
              </div>
            </div>
          </div>
          <!-- Bento Grid -->
          <div class="grid grid-cols-3 gap-5">
            <!-- Learning Path -->
            <div class="bg-white rounded-3xl p-5 shadow-sm border border-slate-100">
              <div class="flex items-center justify-between mb-5">
                <h2 class="text-sm font-black text-slate-800">学习闯关图</h2>
                <select v-model="activeSeriesId"
                  class="text-xs text-violet-600 bg-violet-50 border border-violet-100 rounded-full
                         px-3 py-1 font-semibold outline-none cursor-pointer
                         hover:bg-violet-100 transition-colors appearance-none pr-7"
                  :style="seriesSelectStyle">
                  <option v-for="s in myEnrolledCourses.filter(c => c.type === 'series')"
                    :key="s.id" :value="s.id">
                    {{ s.title }}
                  </option>
                </select>
              </div>
              <div class="space-y-0">
                <div v-for="(node, i) in activeSeriesData.nodes" :key="node.id" class="relative">
                  <!-- 虚线连接线 -->
                  <div v-if="i < activeSeriesData.nodes.length - 1"
                    class="absolute left-[19px] top-10 w-px h-6 z-0"
                    :style="{ borderLeft: '1.5px dashed', borderColor: node.status==='completed' ? '#c4b5fd' : '#e2e8f0' }">
                  </div>
                  <div class="relative z-10 flex items-center gap-3 py-1.5">
                    <!-- completed -->
                    <div v-if="node.status==='completed'"
                      class="w-10 h-10 rounded-2xl flex items-center justify-center flex-shrink-0
                             bg-violet-500 shadow-lg shadow-violet-200 ring-4 ring-violet-100">
                      <svg class="w-4 h-4 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M5 13l4 4L19 7"/>
                      </svg>
                    </div>
                    <!-- in-progress -->
                    <div v-else-if="node.status==='in-progress'"
                      class="w-10 h-10 rounded-2xl flex items-center justify-center flex-shrink-0
                             bg-gradient-to-br from-amber-400 to-orange-500 shadow-lg shadow-orange-200 ring-4 ring-orange-100">
                      <svg class="w-4 h-4 text-white" fill="currentColor" viewBox="0 0 24 24">
                        <path d="M8 5v14l11-7z"/>
                      </svg>
                    </div>
                    <!-- locked -->
                    <div v-else
                      class="w-10 h-10 rounded-2xl flex items-center justify-center flex-shrink-0 bg-slate-100">
                      <svg class="w-4 h-4 text-amber-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                          d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z"/>
                      </svg>
                    </div>
                    <!-- 文字 -->
                    <div class="flex-1 min-w-0">
                      <p class="text-sm font-semibold leading-snug"
                        :class="node.status==='locked' ? 'text-slate-400' : 'text-slate-800'">
                        {{ node.title }}
                      </p>
                      <p class="text-xs font-medium mt-0.5"
                        :class="node.status==='locked' ? 'text-slate-300' : 'text-violet-400'">
                        +{{ node.xp }} XP · {{ node.duration }}
                      </p>
                    </div>
                    <!-- 进行中胶囊 -->
                    <span v-if="node.status==='in-progress'"
                      class="flex-shrink-0 text-xs bg-amber-100 text-amber-600 px-2 py-0.5 rounded-full font-semibold">
                      进行中
                    </span>
                  </div>
                </div>
              </div>
            </div>

            <!-- Ongoing Courses -->
            <div class="col-span-2 bg-white rounded-3xl p-5 shadow-sm border border-slate-100">
              <div class="flex items-center justify-between mb-4">
                <h2 class="text-sm font-black text-slate-800">在学课程</h2>
                <button @click="goToProfileBasic()"
                  class="text-xs text-violet-500 font-semibold hover:text-violet-700 transition-colors">
                  查看全部 →
                </button>
              </div>

              <!-- 空状态 -->
              <div v-if="homeEnrolledCourses.length === 0"
                class="flex flex-col items-center justify-center py-8 text-center">
                <p class="text-3xl mb-2">📭</p>
                <p class="text-sm font-bold text-slate-500">暂无在学课程</p>
                <p class="text-xs text-slate-400 mt-1 mb-4">快去课程中心选一门吧</p>
                <button @click="switchView('courses')"
                  class="px-4 py-2 bg-violet-600 text-white text-xs font-black rounded-xl hover:bg-violet-700 transition-colors">
                  去选课 →
                </button>
              </div>

              <!-- 课程列表（基础课 + 进行中的系列子节点，最多 4 条） -->
              <div v-else class="space-y-1">
                <div v-for="c in homeEnrolledCourses.slice(0, 3)" :key="c.id"
                  class="flex items-center gap-3 px-3 py-3 rounded-2xl transition-all group cursor-pointer hover:bg-slate-50"
                  @click="goToProfileBasic()">
                  <!-- 左侧科目图标 -->
                  <div class="w-11 h-11 rounded-2xl flex items-center justify-center flex-shrink-0 text-xl"
                    :class="c.color==='purple' ? 'bg-violet-50'
                          : c.color==='blue'   ? 'bg-blue-50'
                          : c.color==='green'  ? 'bg-emerald-50'
                          : c.color==='orange' ? 'bg-orange-50'
                                               : 'bg-teal-50'">
                    {{ getCourseEmoji(c.subject) }}
                  </div>
                  <!-- 标题 + 进度条 -->
                  <div class="flex-1 min-w-0">
                    <div class="flex items-center gap-1.5 leading-snug">
                      <p class="text-sm font-bold text-slate-800 truncate">{{ c.title }}</p>
                      <span v-if="c.type === 'series-node'"
                        class="flex-shrink-0 text-xs bg-amber-100 text-amber-600 px-1.5 py-0.5 rounded-full font-bold whitespace-nowrap">
                        进行中
                      </span>
                    </div>
                    <p class="text-xs text-slate-400 mt-0.5 truncate">
                      <span v-if="c.type === 'series-node'">{{ c.seriesTitle }}</span>
                      <span v-else>{{ c.teacher }}</span>
                    </p>
                    <div class="flex items-center gap-2 mt-2">
                      <div class="flex-1 h-1.5 bg-slate-100 rounded-full overflow-hidden">
                        <div class="h-full rounded-full transition-all duration-700"
                          :class="c.color==='purple' ? 'bg-gradient-to-r from-violet-400 to-purple-500'
                                : c.color==='blue'   ? 'bg-gradient-to-r from-blue-400 to-cyan-400'
                                : c.color==='orange' ? 'bg-gradient-to-r from-orange-400 to-amber-400'
                                : c.color==='teal'   ? 'bg-gradient-to-r from-teal-400 to-cyan-400'
                                :                     'bg-gradient-to-r from-emerald-400 to-teal-500'"
                          :style="{ width: seriesProgress(c) + '%' }">
                        </div>
                      </div>
                      <span class="text-xs font-bold text-slate-400 flex-shrink-0 w-8 text-right">{{ seriesProgress(c) }}%</span>
                    </div>
                  </div>
                  <!-- SVG 环形进度 -->
                  <svg class="w-10 h-10 -rotate-90 flex-shrink-0" viewBox="0 0 36 36">
                    <circle cx="18" cy="18" r="15" fill="none" stroke="#f1f5f9" stroke-width="3"/>
                    <circle cx="18" cy="18" r="15" fill="none"
                      :stroke="c.color==='purple' ? '#8b5cf6'
                             : c.color==='blue'   ? '#3b82f6'
                             : c.color==='orange' ? '#f97316'
                             : c.color==='teal'   ? '#14b8a6'
                                                  : '#10b981'"
                      stroke-width="3" stroke-linecap="round"
                      :stroke-dasharray="`${seriesProgress(c) * 0.942} 94.2`"/>
                  </svg>
                  <!-- Hover 淡入继续按钮 -->
                  <button @click.stop="continueSeriesNode(c)"
                    class="px-3 py-1.5 text-xs font-black rounded-xl bg-violet-100 text-violet-700
                           opacity-0 group-hover:opacity-100 translate-x-2 group-hover:translate-x-0
                           transition-all duration-200 hover:bg-violet-200 flex-shrink-0 whitespace-nowrap">
                    继续 →
                  </button>
                </div>
              </div>
            </div>
          </div>

        </div>

        <!-- ══ VIEW 2: Course Center ══ -->
        <div v-else-if="currentView === 'courses'" class="p-6 space-y-5">
          <div class="flex items-center justify-between">
            <div>
              <h1 class="text-xl font-black text-slate-800">课程中心</h1>
              <p class="text-sm text-slate-400 mt-0.5">为 <span class="text-violet-500 font-semibold">{{ currentGrade }}</span> 学段精选优质内容</p>
            </div>
          </div>

          <!-- Tabs -->
          <div class="flex gap-1 bg-slate-100 p-1 rounded-2xl w-fit">
            <button v-for="tab in courseTabs" :key="tab.value"
              @click="courseTab = tab.value"
              class="px-5 py-2 text-sm font-semibold rounded-xl transition-all"
              :class="courseTab === tab.value ? 'bg-white text-slate-800 shadow-sm' : 'text-slate-500 hover:text-slate-700'">
              {{ tab.label }}
            </button>
          </div>

          <!-- Course Grid (basic / interest) -->
          <div v-if="courseTab !== 'series'" class="grid grid-cols-3 gap-5">
            <div v-for="course in currentCourses[courseTab]" :key="course.id"
              class="bg-white rounded-3xl overflow-hidden shadow-sm border border-slate-100 hover:shadow-lg hover:-translate-y-1 transition-all duration-200 group">
              <!-- Thumbnail -->
              <div class="relative cursor-pointer" @click="openVideoPlayer(course)">
                <div class="h-36 flex items-center justify-center"
                  :class="{
                    'bg-gradient-to-br from-violet-100 to-purple-100': course.color==='purple',
                    'bg-gradient-to-br from-blue-100 to-cyan-100': course.color==='blue',
                    'bg-gradient-to-br from-emerald-100 to-teal-100': course.color==='green',
                    'bg-gradient-to-br from-orange-100 to-amber-100': course.color==='orange',
                    'bg-gradient-to-br from-pink-100 to-rose-100': course.color==='pink',
                    'bg-gradient-to-br from-teal-100 to-cyan-100': course.color==='teal',
                  }">
                  <div class="text-center">
                    <p class="text-4xl">{{ getCourseEmoji(course.subject) }}</p>
                    <p class="text-xs text-slate-400 mt-1 font-semibold">{{ course.subject }}</p>
                  </div>
                  <!-- Play overlay -->
                  <div class="absolute inset-0 bg-black/0 group-hover:bg-black/10 transition-all flex items-center justify-center">
                    <div class="w-12 h-12 rounded-full bg-white/90 flex items-center justify-center opacity-0 group-hover:opacity-100 transition-all shadow-xl scale-90 group-hover:scale-100">
                      <svg class="w-5 h-5 text-slate-700 ml-0.5" fill="currentColor" viewBox="0 0 20 20">
                        <path d="M6.3 2.841A1.5 1.5 0 004 4.11v11.78a1.5 1.5 0 002.3 1.269l9.344-5.89a1.5 1.5 0 000-2.538L6.3 2.84z"/>
                      </svg>
                    </div>
                  </div>
                </div>
                <!-- Overlays -->
                <div class="absolute bottom-2 left-2 right-2 flex items-center justify-between">
                  <div class="flex items-center gap-1 bg-black/50 backdrop-blur-sm text-white text-xs px-2 py-0.5 rounded-full">
                    <svg class="w-3 h-3" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"/><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z"/></svg>
                    {{ formatNumber(course.views) }}
                  </div>
                  <div class="bg-black/50 backdrop-blur-sm text-yellow-300 text-xs px-2 py-0.5 rounded-full font-semibold">★ {{ course.rating }}</div>
                </div>
              </div>

              <!-- Body -->
              <div class="p-4">
                <h3 class="text-sm font-black text-slate-800 leading-tight line-clamp-2 mb-1">{{ course.title }}</h3>
                <p class="text-xs text-slate-400 mb-3">{{ course.teacher }}</p>
                <!-- Tags -->
                <div class="flex flex-wrap gap-1 mb-3">
                  <span v-for="tag in course.tags" :key="tag" class="text-xs bg-slate-100 text-slate-500 px-2 py-0.5 rounded-full font-medium">{{ tag }}</span>
                </div>
                <!-- Avatar Stack -->
                <div class="flex items-center gap-2 mb-3">
                  <div class="flex -space-x-1.5">
                    <div v-for="(av, i) in course.avatar" :key="i"
                      class="w-6 h-6 rounded-full border-2 border-white flex items-center justify-center text-white text-xs font-bold"
                      :class="['bg-violet-400','bg-blue-400','bg-emerald-400','bg-orange-400'][i % 4]">
                      {{ av }}
                    </div>
                  </div>
                  <span class="text-xs text-slate-400">+{{ Math.floor(course.views / 100) }} 在学</span>
                </div>
                <!-- AI Buttons -->
                <div v-if="courseTab === 'basic'" class="grid grid-cols-3 gap-1.5 mb-2">
                  <button @click.stop="openAIModal('preview', course)"
                    class="py-1.5 text-xs font-bold rounded-xl bg-violet-50 text-violet-600 hover:bg-violet-100 border border-violet-100 transition-colors">
                    🔮 预习
                  </button>
                  <button @click.stop="openAIModal('summary', course)"
                    class="py-1.5 text-xs font-bold rounded-xl bg-emerald-50 text-emerald-600 hover:bg-emerald-100 border border-emerald-100 transition-colors">
                    ✨ 总结
                  </button>
                  <button @click.stop="openAIModal('deepread', course)"
                    class="py-1.5 text-xs font-bold rounded-xl bg-blue-50 text-blue-600 hover:bg-blue-100 border border-blue-100 transition-colors">
                    🧠 精读
                  </button>
                </div>
                <!-- 加入学习按钮 -->
                <button @click.stop="enrollBasicCourse(course)"
                  class="w-full py-2 text-xs font-black rounded-2xl transition-all"
                  :class="isBasicEnrolled(course.id)
                    ? 'bg-emerald-50 text-emerald-600 cursor-default'
                    : 'bg-violet-600 text-white hover:bg-violet-700 active:scale-95'">
                  {{ isBasicEnrolled(course.id) ? '✓ 已加入学习计划' : '＋ 加入学习计划' }}
                </button>
              </div>
            </div>
          </div>

          <!-- 套系课 Grid -->
          <div v-if="courseTab === 'series'" class="grid grid-cols-3 gap-5">
            <div v-for="series in courseSeries" :key="series.id"
              class="rounded-3xl overflow-hidden cursor-pointer group transition-all duration-200 hover:-translate-y-1 bg-white"
              style="box-shadow: 0 1px 3px rgba(0,0,0,.08), 6px 6px 0 #f1f5f9, 10px 10px 0 #e2e8f0"
              @click="openSeriesModal(series)">
              <!-- 封面 -->
              <div class="h-32 flex flex-col items-center justify-center relative"
                :class="{
                  'bg-gradient-to-br from-violet-100 to-purple-100': series.color==='purple',
                  'bg-gradient-to-br from-blue-100 to-cyan-100':    series.color==='blue',
                  'bg-gradient-to-br from-emerald-100 to-teal-100': series.color==='green',
                  'bg-gradient-to-br from-orange-100 to-amber-100': series.color==='orange',
                  'bg-gradient-to-br from-teal-100 to-cyan-100':    series.color==='teal',
                }">
                <p class="text-4xl">{{ getCourseEmoji(series.subject) }}</p>
                <div class="flex items-center gap-1 mt-2 bg-white/70 backdrop-blur-sm px-2.5 py-0.5 rounded-full">
                  <span class="w-1.5 h-1.5 rounded-full bg-violet-400 flex-shrink-0"></span>
                  <span class="text-xs font-black text-slate-600">{{ series.nodes.length }} 个闯关节点</span>
                </div>
                <div class="absolute inset-0 bg-black/0 group-hover:bg-black/5 transition-all flex items-center justify-center">
                  <div class="w-10 h-10 rounded-full bg-white/90 flex items-center justify-center opacity-0 group-hover:opacity-100 transition-all shadow-xl scale-90 group-hover:scale-100">
                    <svg class="w-4 h-4 text-slate-700 ml-0.5" fill="currentColor" viewBox="0 0 24 24">
                      <path d="M8 5v14l11-7z"/>
                    </svg>
                  </div>
                </div>
              </div>
              <!-- 卡片内容 -->
              <div class="p-4">
                <div class="flex items-center gap-1.5 mb-1.5">
                  <span class="text-xs bg-violet-100 text-violet-600 px-2 py-0.5 rounded-full font-bold">{{ series.tag }}</span>
                  <span class="text-xs bg-slate-100 text-slate-500 px-2 py-0.5 rounded-full font-medium">★ {{ series.rating }}</span>
                </div>
                <h3 class="text-sm font-black text-slate-800 leading-tight mb-1">{{ series.title }}</h3>
                <p class="text-xs text-slate-400 mb-2">{{ series.teacher }}</p>
                <p class="text-xs text-slate-500 line-clamp-2 mb-3">{{ series.description }}</p>
                <!-- 已加入进度条 -->
                <div v-if="isEnrolled(series.id)" class="mb-3">
                  <div class="flex items-center justify-between mb-1">
                    <span class="text-xs text-slate-400">学习进度</span>
                    <span class="text-xs font-black text-violet-600">{{ seriesProgress(series) }}%</span>
                  </div>
                  <div class="h-1.5 bg-slate-100 rounded-full overflow-hidden">
                    <div class="h-full rounded-full bg-gradient-to-r from-violet-400 to-purple-500 transition-all duration-700"
                      :style="{ width: seriesProgress(series) + '%' }"></div>
                  </div>
                </div>
                <!-- 加入 / 已加入 按钮 -->
                <button @click.stop="isEnrolled(series.id) ? openSeriesModal(series) : enrollSeries(series)"
                  class="w-full py-2 text-xs font-black rounded-2xl transition-all"
                  :class="isEnrolled(series.id)
                    ? 'bg-emerald-50 text-emerald-600 hover:bg-emerald-100'
                    : 'bg-violet-600 text-white hover:bg-violet-700 active:scale-95'">
                  {{ isEnrolled(series.id) ? '✓ 已加入 · 查看路线' : '＋ 加入学习计划' }}
                </button>
              </div>
            </div>
          </div>

        </div>

        <!-- ══ VIEW 3: Analytics ══ -->
        <div v-else-if="currentView === 'analytics'" class="p-6 space-y-5">
          <div class="flex items-center justify-between">
            <div>
              <h1 class="text-xl font-black text-slate-800">AI 成绩看板</h1>
              <p class="text-sm text-slate-400 mt-0.5">智能分析学习数据，精准定位突破口</p>
            </div>
            <button class="flex items-center gap-2 bg-violet-600 text-white px-5 py-2.5 rounded-2xl text-sm font-black hover:bg-violet-700 transition-colors shadow-lg shadow-violet-200">
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-8l-4-4m0 0L8 8m4-4v12"/></svg>
              上传最新成绩单
            </button>
          </div>

          <div class="grid grid-cols-3 gap-5">
            <!-- Line Chart -->
            <div class="col-span-2 bg-white rounded-3xl p-5 shadow-sm border border-slate-100">
              <div class="flex items-center justify-between mb-4">
                <h2 class="text-sm font-black text-slate-800">近期成绩趋势</h2>
                <div class="flex items-center gap-4">
                  <div class="flex items-center gap-1.5"><div class="w-3 h-0.5 bg-violet-500 rounded-full"></div><span class="text-xs text-slate-400">数学</span></div>
                  <div class="flex items-center gap-1.5"><div class="w-3 h-0.5 bg-emerald-500 rounded-full"></div><span class="text-xs text-slate-400">语文</span></div>
                  <div class="flex items-center gap-1.5"><div class="w-3 h-0.5 bg-blue-500 rounded-full"></div><span class="text-xs text-slate-400">英语</span></div>
                </div>
              </div>
              <svg viewBox="0 0 500 200" class="w-full h-auto">
                <defs>
                  <linearGradient id="mathArea" x1="0" y1="0" x2="0" y2="1">
                    <stop offset="0%" stop-color="#8b5cf6" stop-opacity="0.15"/>
                    <stop offset="100%" stop-color="#8b5cf6" stop-opacity="0"/>
                  </linearGradient>
                </defs>
                <!-- Grid -->
                <line x1="48" y1="20" x2="48" y2="165" stroke="#e2e8f0" stroke-width="1"/>
                <line x1="48" y1="165" x2="492" y2="165" stroke="#e2e8f0" stroke-width="1"/>
                <line x1="48" y1="115" x2="492" y2="115" stroke="#f1f5f9" stroke-width="1" stroke-dasharray="5,5"/>
                <line x1="48" y1="65" x2="492" y2="65" stroke="#f1f5f9" stroke-width="1" stroke-dasharray="5,5"/>
                <line x1="48" y1="15" x2="492" y2="15" stroke="#f8fafc" stroke-width="1" stroke-dasharray="5,5"/>
                <!-- Y labels -->
                <text x="40" y="169" text-anchor="end" font-size="10" fill="#94a3b8">60</text>
                <text x="40" y="119" text-anchor="end" font-size="10" fill="#94a3b8">75</text>
                <text x="40" y="69" text-anchor="end" font-size="10" fill="#94a3b8">90</text>
                <!-- X labels: 11月~4月, 6 points, x from 48 to 492, step ~88.8 -->
                <text x="48" y="180" text-anchor="middle" font-size="10" fill="#94a3b8">11月</text>
                <text x="137" y="180" text-anchor="middle" font-size="10" fill="#94a3b8">12月</text>
                <text x="226" y="180" text-anchor="middle" font-size="10" fill="#94a3b8">1月</text>
                <text x="315" y="180" text-anchor="middle" font-size="10" fill="#94a3b8">2月</text>
                <text x="403" y="180" text-anchor="middle" font-size="10" fill="#94a3b8">3月</text>
                <text x="492" y="180" text-anchor="middle" font-size="10" fill="#94a3b8">4月</text>
                <!--
                  Score → y: y = 165 - (score-60)/35*150
                  Math:  78→88,  82→71,  75→106, 88→52,  85→62,  91→39
                  Chi:   85→62,  83→71,  88→52,  86→58,  90→45,  87→55
                  Eng:   72→115, 75→106, 78→88,  82→71,  85→62,  88→52
                -->
                <!-- Math area fill -->
                <polygon points="48,88 137,71 226,106 315,52 403,62 492,39 492,165 48,165"
                  fill="url(#mathArea)"/>
                <!-- Math line -->
                <polyline points="48,88 137,71 226,106 315,52 403,62 492,39"
                  fill="none" stroke="#8b5cf6" stroke-width="2.5" stroke-linejoin="round" stroke-linecap="round"/>
                <!-- Chinese line -->
                <polyline points="48,62 137,71 226,52 315,58 403,45 492,55"
                  fill="none" stroke="#10b981" stroke-width="2.5" stroke-linejoin="round" stroke-linecap="round"/>
                <!-- English line -->
                <polyline points="48,115 137,106 226,88 315,71 403,62 492,52"
                  fill="none" stroke="#3b82f6" stroke-width="2.5" stroke-linejoin="round" stroke-linecap="round"/>
                <!-- Latest dots -->
                <circle cx="492" cy="39" r="4.5" fill="#8b5cf6"/>
                <circle cx="492" cy="55" r="4.5" fill="#10b981"/>
                <circle cx="492" cy="52" r="4.5" fill="#3b82f6"/>
                <!-- Value labels -->
                <text x="500" y="43" font-size="10" fill="#8b5cf6" font-weight="bold">91</text>
                <text x="500" y="59" font-size="10" fill="#10b981" font-weight="bold">87</text>
                <text x="500" y="56" font-size="10" fill="#3b82f6" font-weight="bold">88</text>
              </svg>
            </div>

            <!-- Radar Chart -->
            <div class="bg-white rounded-3xl p-5 shadow-sm border border-slate-100">
              <h2 class="text-sm font-black text-slate-800 mb-2">知识雷达图</h2>
              <svg viewBox="0 0 240 240" class="w-full h-auto">
                <!-- Background pentagons (radii: 75, 56, 37) -->
                <polygon points="120,45 191,97 164,181 76,181 49,97" fill="none" stroke="#f1f5f9" stroke-width="1.5"/>
                <polygon points="120,64 168,105 148,162 92,162 72,105" fill="none" stroke="#f1f5f9" stroke-width="1.5"/>
                <polygon points="120,83 144,113 131,143 109,143 96,113" fill="none" stroke="#f1f5f9" stroke-width="1.5"/>
                <!-- Axis lines -->
                <line x1="120" y1="120" x2="120" y2="45" stroke="#e2e8f0" stroke-width="1"/>
                <line x1="120" y1="120" x2="191" y2="97" stroke="#e2e8f0" stroke-width="1"/>
                <line x1="120" y1="120" x2="164" y2="181" stroke="#e2e8f0" stroke-width="1"/>
                <line x1="120" y1="120" x2="76" y2="181" stroke="#e2e8f0" stroke-width="1"/>
                <line x1="120" y1="120" x2="49" y2="97" stroke="#e2e8f0" stroke-width="1"/>
                <!--
                  Scores: 代数85, 几何62, 统计78, 函数90, 方程75
                  Point = center + score/100 * (axis - center)
                  center=(120,120)
                  Axis0(120,45): 代数85→(120,120+0.85*(45-120))=(120,56.25)
                  Axis1(191,97): 几何62→(120+0.62*(71),120+0.62*(-23))=(164,105.7)
                  Axis2(164,181): 统计78→(120+0.78*44,120+0.78*61)=(154.3,167.6)
                  Axis3(76,181):  函数90→(120+0.90*(-44),120+0.90*61)=(80.4,174.9)
                  Axis4(49,97):   方程75→(120+0.75*(-71),120+0.75*(-23))=(66.75,102.75)
                -->
                <polygon points="120,56 164,106 154,168 80,175 67,103"
                  fill="rgba(139,92,246,0.18)" stroke="#8b5cf6" stroke-width="2" stroke-linejoin="round"/>
                <!-- Dots -->
                <circle cx="120" cy="56" r="4" fill="#8b5cf6"/>
                <circle cx="164" cy="106" r="4" fill="#ef4444"/>
                <circle cx="154" cy="168" r="4" fill="#8b5cf6"/>
                <circle cx="80" cy="175" r="4" fill="#8b5cf6"/>
                <circle cx="67" cy="103" r="4" fill="#8b5cf6"/>
                <!-- Labels -->
                <text x="120" y="36" text-anchor="middle" font-size="11" fill="#64748b" font-weight="700">代数</text>
                <text x="203" y="99" text-anchor="start" font-size="11" fill="#64748b" font-weight="700">几何</text>
                <text x="170" y="193" text-anchor="middle" font-size="11" fill="#64748b" font-weight="700">统计</text>
                <text x="70" y="193" text-anchor="middle" font-size="11" fill="#64748b" font-weight="700">函数</text>
                <text x="37" y="99" text-anchor="end" font-size="11" fill="#64748b" font-weight="700">方程</text>
                <!-- Scores -->
                <text x="132" y="55" font-size="9" fill="#8b5cf6" font-weight="800">85</text>
                <text x="167" y="103" font-size="9" fill="#ef4444" font-weight="800">62</text>
              </svg>
              <!-- Legend -->
              <div class="flex items-center justify-center gap-3 text-xs -mt-2">
                <div class="flex items-center gap-1"><div class="w-2 h-2 rounded-full bg-violet-500"></div><span class="text-slate-400">良好</span></div>
                <div class="flex items-center gap-1"><div class="w-2 h-2 rounded-full bg-red-400"></div><span class="text-slate-400">薄弱</span></div>
              </div>
            </div>
          </div>

          <!-- AI Insight Row -->
          <div class="grid grid-cols-3 gap-5">
            <div class="col-span-2 bg-gradient-to-br from-violet-50 to-purple-50 border border-violet-100 rounded-3xl p-5">
              <div class="flex items-start gap-4">
                <div class="w-10 h-10 rounded-2xl bg-white border border-violet-100 flex items-center justify-center flex-shrink-0 shadow-sm">
                  <span class="text-xl">🤖</span>
                </div>
                <div>
                  <p class="text-xs font-black text-violet-500 uppercase tracking-widest mb-2">AI 智能诊断报告</p>
                  <p class="text-sm text-slate-700 leading-relaxed">
                    根据近 6 个月成绩轨迹分析，你的英语呈稳步提升态势（<span class="text-emerald-600 font-bold">+16分</span>），数学本月出现明显突破峰值（91分）。然而雷达图显示，<strong class="text-red-500">几何板块（62分）存在显著知识断层</strong>，与整体均分差距达 20 分。建议优先攻克几何证明与空间想象题型，预计专项练习 2 周可提升 10~15 分。
                  </p>
                </div>
              </div>
            </div>

            <div class="bg-white border border-slate-100 rounded-3xl p-5 shadow-sm flex flex-col">
              <p class="text-xs font-black text-slate-400 uppercase tracking-widest mb-3">AI 针对性推荐</p>
              <div class="flex-1 bg-gradient-to-br from-red-50 to-orange-50 border border-red-100 rounded-2xl p-4 cursor-pointer hover:shadow-md transition-all group"
                @click="switchView('courses')">
                <div class="flex items-center gap-2 mb-2">
                  <span class="text-xl">📐</span>
                  <span class="text-xs bg-red-100 text-red-600 px-2 py-0.5 rounded-full font-bold">弱点突破</span>
                </div>
                <p class="text-sm font-black text-slate-800 mb-0.5">几何证明专题精讲</p>
                <p class="text-xs text-slate-400">张华老师 · 共 12 讲</p>
                <div class="mt-3 flex items-center gap-1 text-xs text-violet-600 font-bold group-hover:gap-2 transition-all">
                  <span>立即补强</span><span>→</span>
                </div>
              </div>
            </div>
          </div>

          <!-- ── 深度诊断报告卡 ── -->
          <div class="bg-white rounded-3xl shadow-sm border border-slate-100 overflow-hidden">
            <!-- Header strip -->
            <div class="bg-gradient-to-r from-slate-800 to-slate-900 px-6 py-4 flex items-center justify-between">
              <div class="flex items-center gap-3">
                <div class="w-8 h-8 rounded-xl bg-white/10 flex items-center justify-center text-lg">🩺</div>
                <div>
                  <p class="text-xs text-white/50 uppercase tracking-widest font-semibold">AI 学情诊断报告</p>
                  <p class="text-sm font-black text-white">2026 年 04 月 · 月度智能体检</p>
                </div>
              </div>
              <div class="flex items-center gap-2">
                <span class="w-2 h-2 rounded-full bg-emerald-400 animate-pulse"></span>
                <span class="text-emerald-400 text-xs font-semibold">实时分析</span>
              </div>
            </div>
            <!-- Metrics row -->
            <div class="grid grid-cols-4 divide-x divide-slate-100 border-b border-slate-100">
              <div v-for="m in diagMetrics" :key="m.label" class="p-4 text-center">
                <p class="text-xs text-slate-400 font-medium mb-1">{{ m.label }}</p>
                <p class="text-xl font-black" :class="m.color">{{ m.value }}</p>
                <p class="text-xs mt-1" :class="m.trend > 0 ? 'text-emerald-500' : 'text-red-400'">
                  {{ m.trend > 0 ? '↑' : '↓' }} {{ Math.abs(m.trend) }}{{ m.unit }}
                </p>
              </div>
            </div>
            <!-- Detailed text -->
            <div class="p-6 space-y-4">
              <div class="flex gap-4">
                <div class="w-1 rounded-full bg-gradient-to-b from-violet-500 to-purple-600 flex-shrink-0"></div>
                <div>
                  <p class="text-xs font-black text-violet-600 uppercase tracking-wider mb-2">整体评估</p>
                  <p class="text-sm text-slate-700 leading-relaxed">近一周知识留存率 <strong class="text-violet-600">78%</strong>，高于班级平均水平 12 个百分点。代数方程模块掌握极佳，错题率仅 6%；但<strong class="text-red-500">空间几何（立体几何）单元连续两次考核失分严重</strong>，主要集中在"辅助线构造"与"体积公式推导"两个子考点，建议本周优先攻克。</p>
                </div>
              </div>
              <!-- Breakdown bars -->
              <div class="space-y-2.5 pt-2">
                <div v-for="item in diagDetails" :key="item.name" class="flex items-center gap-3">
                  <span class="text-xs font-semibold text-slate-500 w-24 flex-shrink-0">{{ item.name }}</span>
                  <div class="flex-1 h-2 bg-slate-100 rounded-full overflow-hidden">
                    <div class="h-full rounded-full transition-all duration-700"
                      :class="item.score >= 80 ? 'bg-emerald-400' : item.score >= 65 ? 'bg-amber-400' : 'bg-red-400'"
                      :style="{ width: item.score + '%' }"></div>
                  </div>
                  <span class="text-xs font-black w-8 text-right"
                    :class="item.score >= 80 ? 'text-emerald-600' : item.score >= 65 ? 'text-amber-600' : 'text-red-500'">
                    {{ item.score }}
                  </span>
                  <span class="text-xs px-1.5 py-0.5 rounded-lg font-semibold w-12 text-center"
                    :class="item.score >= 80 ? 'bg-emerald-50 text-emerald-600' : item.score >= 65 ? 'bg-amber-50 text-amber-600' : 'bg-red-50 text-red-500'">
                    {{ item.score >= 80 ? '良好' : item.score >= 65 ? '一般' : '薄弱' }}
                  </span>
                </div>
              </div>
            </div>
            <!-- Targeted micro-course strip -->
            <div class="border-t border-slate-100 px-6 pt-4 pb-5">
              <p class="text-xs font-black text-slate-400 uppercase tracking-widest mb-3">🎯 靶向补弱 · AI 推荐微课</p>
              <div class="flex gap-3 overflow-x-auto pb-1 -mx-1 px-1">
                <div v-for="mc in microCourses" :key="mc.id"
                  class="flex-shrink-0 w-52 rounded-2xl border p-4 cursor-pointer hover:shadow-md transition-all duration-200 hover:-translate-y-0.5 group"
                  :class="mc.urgent ? 'border-red-100 bg-red-50/50' : 'border-slate-100 bg-white'">
                  <div class="flex items-center gap-2 mb-2">
                    <span class="text-xl">{{ mc.emoji }}</span>
                    <span class="text-xs px-2 py-0.5 rounded-full font-bold"
                      :class="mc.urgent ? 'bg-red-100 text-red-600' : 'bg-violet-100 text-violet-600'">
                      {{ mc.tag }}
                    </span>
                  </div>
                  <p class="text-xs font-black text-slate-800 leading-snug mb-1">{{ mc.title }}</p>
                  <p class="text-xs text-slate-400">{{ mc.teacher }} · {{ mc.duration }}</p>
                  <div class="mt-3 flex items-center gap-1 text-xs font-bold text-violet-500 group-hover:gap-2 transition-all">
                    <span>立即学习</span><span>→</span>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <!-- Sankey -->
          <div class="bg-white rounded-3xl p-5 shadow-sm border border-slate-100">
            <h2 class="text-sm font-black text-slate-800 mb-4">知识流向图（桑基图模拟）</h2>
            <div class="flex items-center gap-6 h-28">
              <div class="flex flex-col gap-2 flex-shrink-0">
                <div class="bg-violet-100 text-violet-700 text-xs font-bold px-3 py-2 rounded-xl text-center">代数 85%</div>
                <div class="bg-blue-100 text-blue-700 text-xs font-bold px-3 py-2 rounded-xl text-center">函数 90%</div>
                <div class="bg-amber-100 text-amber-700 text-xs font-bold px-3 py-2 rounded-xl text-center">方程 75%</div>
              </div>
              <svg class="flex-1 h-full" viewBox="0 0 300 112" preserveAspectRatio="none">
                <path d="M0,14 C150,14 150,35 300,30" fill="none" stroke="#8b5cf6" stroke-width="20" stroke-opacity="0.4"/>
                <path d="M0,56 C150,56 150,35 300,30" fill="none" stroke="#3b82f6" stroke-width="24" stroke-opacity="0.4"/>
                <path d="M0,98 C150,98 150,85 300,88" fill="none" stroke="#f59e0b" stroke-width="14" stroke-opacity="0.4"/>
                <path d="M0,56 C150,56 150,85 300,88" fill="none" stroke="#10b981" stroke-width="10" stroke-opacity="0.35"/>
              </svg>
              <div class="flex flex-col gap-2 flex-shrink-0">
                <div class="bg-emerald-100 text-emerald-700 text-xs font-bold px-3 py-2 rounded-xl flex items-center gap-1.5">
                  <span class="w-2 h-2 rounded-full bg-emerald-400 flex-shrink-0"></span>掌握 82%
                </div>
                <div class="bg-slate-100 text-slate-500 text-xs font-bold px-3 py-2 rounded-xl flex items-center gap-1.5">
                  <span class="w-2 h-2 rounded-full bg-slate-400 flex-shrink-0"></span>模糊 10%
                </div>
                <div class="bg-red-100 text-red-600 text-xs font-bold px-3 py-2 rounded-xl flex items-center gap-1.5">
                  <span class="w-2 h-2 rounded-full bg-red-400 flex-shrink-0"></span>薄弱 8%
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- ══ VIEW 4: Review Center ══ -->
        <div v-else-if="currentView === 'review'" class="p-6 space-y-5">
          <div>
            <h1 class="text-xl font-black text-slate-800">复习题库</h1>
            <p class="text-sm text-slate-400 mt-0.5">科学记忆，战胜遗忘曲线</p>
          </div>

          <!-- Tabs -->
          <div class="flex gap-1 bg-slate-100 p-1 rounded-2xl w-fit">
            <button v-for="tab in reviewTabs" :key="tab.value"
              @click="reviewTab = tab.value; resetQuiz()"
              class="px-5 py-2 text-sm font-semibold rounded-xl transition-all"
              :class="reviewTab === tab.value ? 'bg-white text-slate-800 shadow-sm' : 'text-slate-500 hover:text-slate-700'">
              {{ tab.label }}
            </button>
          </div>

          <!-- Targeted Review -->
          <div v-if="reviewTab === 'targeted'" class="bg-white rounded-3xl p-6 shadow-sm border border-slate-100 space-y-5">
            <h2 class="text-sm font-black text-slate-800">选择课程，生成定向试卷</h2>
            <div class="flex gap-3">
              <select class="flex-1 bg-slate-50 border border-slate-200 rounded-2xl px-4 py-3 text-sm text-slate-700 focus:outline-none focus:ring-2 focus:ring-violet-300 font-medium">
                <option>初中数学-函数专题精讲</option>
                <option>英语语法精讲</option>
                <option>语文现代文阅读</option>
              </select>
              <button @click="startQuiz" class="px-6 py-3 bg-violet-600 text-white text-sm font-black rounded-2xl hover:bg-violet-700 transition-colors shadow-lg shadow-violet-200 whitespace-nowrap">
                生成试卷 →
              </button>
            </div>
            <!-- Quiz or placeholder -->
            <div v-if="quizStarted">
              <QuizBlock :questions="quizQuestions" :current-index="currentQuizIndex"
                :selected-answer="selectedAnswer" :show-explanation="showExplanation"
                @select="selectAnswer" @next="nextQuestion"/>
            </div>
            <div v-else class="text-center py-10 text-slate-300">
              <p class="text-5xl mb-3">📝</p>
              <p class="text-sm text-slate-400">选择课程后点击「生成试卷」开始练习</p>
            </div>
          </div>

          <!-- Smart Review -->
          <div v-else class="space-y-4">
            <div class="bg-white rounded-3xl p-6 shadow-sm border border-slate-100">
              <div class="flex items-start justify-between mb-5">
                <div>
                  <div class="flex items-center gap-2 mb-2">
                    <h2 class="text-sm font-black text-slate-800">艾宾浩斯智能抽查</h2>
                    <span class="text-xs bg-violet-100 text-violet-600 px-2 py-0.5 rounded-full font-bold">🧠 AI 驱动</span>
                  </div>
                  <p class="text-xs text-slate-400 leading-relaxed max-w-sm">
                    基于遗忘曲线模型，AI 已从你过去 30 天的历史薄弱点中智能筛选了 <strong class="text-slate-600">8 道</strong>待复习题目。
                  </p>
                </div>
                <button @click="startRandomReview" class="px-5 py-2.5 bg-gradient-to-r from-violet-500 to-purple-600 text-white text-sm font-black rounded-2xl hover:shadow-xl hover:scale-105 transition-all whitespace-nowrap">
                  一键抽查
                </button>
              </div>

              <!-- Forgetting Curve Bars -->
              <div class="bg-slate-50 rounded-2xl p-4 mb-5">
                <div class="flex items-end gap-2 h-20 mb-2">
                  <div v-for="(bar, i) in [90,75,60,48,38,30,25,22]" :key="i"
                    class="flex-1 rounded-t-xl transition-all"
                    :class="bar > 60 ? 'bg-emerald-300' : bar > 40 ? 'bg-amber-300' : 'bg-red-300'"
                    :style="{ height: bar + '%' }"></div>
                </div>
                <div class="flex justify-between text-xs text-slate-400 font-medium">
                  <span>1天</span><span>2天</span><span>4天</span><span>7天</span><span>14天</span><span>21天</span><span>30天</span><span>60天</span>
                </div>
                <p class="text-xs text-center text-slate-400 mt-2">记忆保留率曲线 · 红色区域已触发复习提醒</p>
              </div>

              <!-- Quiz if started -->
              <div v-if="randomReviewStarted" class="border-t border-slate-100 pt-5">
                <p class="text-xs font-black text-slate-400 mb-4">🔀 从薄弱点随机抽取 · 第 {{ currentQuizIndex + 1 }}/{{ quizQuestions.length }}</p>
                <div class="bg-violet-50 rounded-2xl p-4 mb-4">
                  <p class="text-sm font-semibold text-slate-800">{{ quizQuestions[currentQuizIndex].question }}</p>
                </div>
                <div class="grid grid-cols-2 gap-3 mb-4">
                  <button v-for="(opt, i) in quizQuestions[currentQuizIndex].options" :key="i"
                    @click="selectAnswer(i)"
                    class="p-3.5 rounded-2xl border text-sm font-medium text-left transition-all"
                    :class="getOptionClass(i)">
                    <span class="font-black mr-2 text-slate-300">{{ ['A','B','C','D'][i] }}.</span>{{ opt }}
                  </button>
                </div>
                <div v-if="showExplanation" class="bg-amber-50 border border-amber-100 rounded-2xl p-4 mb-4">
                  <p class="text-xs font-black text-amber-600 mb-1">💡 解析</p>
                  <p class="text-sm text-slate-700">{{ quizQuestions[currentQuizIndex].explanation }}</p>
                </div>
                <div class="flex justify-end">
                  <button v-if="showExplanation && currentQuizIndex < quizQuestions.length - 1" @click="nextQuestion"
                    class="px-6 py-2.5 bg-violet-600 text-white text-sm font-black rounded-2xl hover:bg-violet-700 transition-colors">
                    下一题 →
                  </button>
                  <p v-else-if="showExplanation" class="text-sm text-emerald-600 font-black">🎉 本轮抽查完成！获得 +60 XP</p>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- ══ VIEW 5: Video Player ══ -->
        <div v-else-if="currentView === 'player'" class="p-6 space-y-5">
          <!-- Back -->
          <div class="flex items-center gap-3">
            <button @click="goBack" class="w-9 h-9 rounded-xl border border-slate-200 flex items-center justify-center hover:bg-slate-100 transition-colors">
              <svg class="w-4 h-4 text-slate-500" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 19l-7-7 7-7"/></svg>
            </button>
            <div class="flex-1">
              <h1 class="text-lg font-black text-slate-800">{{ selectedCourse?.title }}</h1>
              <p class="text-sm text-slate-400">{{ selectedCourse?.teacher }}</p>
            </div>
            <!-- Snapshot button -->
            <button @click="takeSnapshot"
              class="flex items-center gap-2 px-4 py-2 rounded-xl
                     bg-gradient-to-r from-violet-500 to-purple-600
                     hover:from-violet-600 hover:to-purple-700
                     text-white text-sm font-black shadow-sm
                     hover:shadow-lg hover:shadow-violet-200/60
                     transition-all active:scale-95">
              📸 一键 AI 快照笔记
            </button>
            <!-- Focus button in player -->
            <button @click="toggleFocusMode"
              class="flex items-center gap-2 px-4 py-2 rounded-xl bg-slate-900 hover:bg-slate-800 text-white text-sm font-bold transition-all">
              <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11.049 2.927c.3-.921 1.603-.921 1.902 0l1.519 4.674a1 1 0 00.95.69h4.915c.969 0 1.371 1.24.588 1.81l-3.976 2.888a1 1 0 00-.363 1.118l1.518 4.674c.3.922-.755 1.688-1.538 1.118l-3.976-2.888a1 1 0 00-1.176 0l-3.976 2.888c-.783.57-1.838-.197-1.538-1.118l1.518-4.674a1 1 0 00-.363-1.118l-3.976-2.888c-.784-.57-.38-1.81.588-1.81h4.914a1 1 0 00.951-.69l1.519-4.674z"/></svg>
              专注模式
            </button>
          </div>

          <!-- Video Player -->
          <div class="bg-black rounded-3xl overflow-hidden relative" style="aspect-ratio:16/9">
            <div class="absolute inset-0 bg-gradient-to-br from-slate-900 via-slate-800 to-black flex flex-col items-center justify-center">
              <div class="w-20 h-20 rounded-full border-4 border-white/20 hover:border-white/40 transition-colors cursor-pointer flex items-center justify-center mb-4">
                <svg class="w-8 h-8 text-white ml-1" fill="currentColor" viewBox="0 0 20 20">
                  <path d="M6.3 2.841A1.5 1.5 0 004 4.11v11.78a1.5 1.5 0 002.3 1.269l9.344-5.89a1.5 1.5 0 000-2.538L6.3 2.84z"/>
                </svg>
              </div>
              <p class="text-white/50 text-sm">{{ selectedCourse?.title }}</p>
              <p class="text-white/30 text-xs mt-1">点击播放</p>
            </div>
            <!-- Controls -->
            <div class="absolute bottom-0 left-0 right-0 bg-gradient-to-t from-black/80 to-transparent p-5">
              <div class="h-1 bg-white/20 rounded-full mb-3 cursor-pointer">
                <div class="h-full w-1/3 bg-white rounded-full"></div>
              </div>
              <div class="flex items-center justify-between text-white/60 text-xs">
                <div class="flex items-center gap-4">
                  <span class="font-mono">10:23 / 32:45</span>
                  <span class="bg-white/10 px-2 py-0.5 rounded">1.0×</span>
                </div>
                <div class="flex items-center gap-3">
                  <span class="bg-white/10 px-2 py-0.5 rounded">CC</span>
                  <span>⛶</span>
                </div>
              </div>
            </div>
          </div>

          <!-- Timer Banner -->
          <div class="bg-gradient-to-r from-amber-50 to-orange-50 border border-amber-100 rounded-3xl p-5">
            <div class="flex items-center justify-between">
              <div class="flex items-center gap-4">
                <span class="text-3xl">⏰</span>
                <div>
                  <p class="text-sm font-black text-amber-800">课后限时自测</p>
                  <p class="text-xs text-amber-600 mt-0.5">趁热打铁！AI 已生成 5 道课后练习，限时 10 分钟作答</p>
                </div>
              </div>
              <div class="text-right">
                <p class="text-3xl font-black text-amber-700 font-mono">09:48</p>
                <p class="text-xs text-amber-400 font-medium">剩余时间</p>
              </div>
            </div>
          </div>

          <!-- Quiz -->
          <div class="bg-white rounded-3xl p-5 shadow-sm border border-slate-100">
            <div v-if="!quizStarted" class="text-center py-8">
              <p class="text-4xl mb-4">✍️</p>
              <p class="text-sm font-bold text-slate-600 mb-5">完成课程后立即巩固所学知识，记忆留存率提升 3 倍</p>
              <button @click="startQuiz" class="px-8 py-3 bg-gradient-to-r from-violet-500 to-purple-600 text-white font-black rounded-2xl hover:shadow-xl hover:scale-105 transition-all text-sm">
                开始自测 &nbsp;+45 XP
              </button>
            </div>
            <div v-else>
              <div class="flex items-center justify-between mb-4">
                <h3 class="text-sm font-black text-slate-800">课后自测</h3>
                <div class="flex items-center gap-2">
                  <div v-for="(q, i) in quizQuestions" :key="i"
                    class="w-6 h-1.5 rounded-full transition-all"
                    :class="i < currentQuizIndex ? 'bg-emerald-400' : i===currentQuizIndex ? 'bg-violet-500' : 'bg-slate-200'"></div>
                </div>
              </div>
              <div class="bg-violet-50 rounded-2xl p-4 mb-4">
                <p class="text-sm font-semibold text-slate-800">{{ quizQuestions[currentQuizIndex].question }}</p>
              </div>
              <div class="grid grid-cols-2 gap-3 mb-4">
                <button v-for="(opt, i) in quizQuestions[currentQuizIndex].options" :key="i"
                  @click="selectAnswer(i)"
                  class="p-3.5 rounded-2xl border text-sm font-medium text-left transition-all"
                  :class="getOptionClass(i)">
                  <span class="font-black mr-2 text-slate-300">{{ ['A','B','C','D'][i] }}.</span>{{ opt }}
                </button>
              </div>
              <Transition name="slide-up">
                <div v-if="showExplanation && selectedAnswer !== null && selectedAnswer !== quizQuestions[currentQuizIndex].correct"
                  class="flex items-center gap-2 mb-3 px-3.5 py-2.5 bg-red-50 border border-red-100 rounded-xl text-xs font-semibold text-red-600">
                  <span>⚠️ 知识点未掌握，已自动收录至</span>
                  <button @click="profileTab='mistakes'; switchView('profile')"
                    class="text-violet-600 underline hover:text-violet-800 font-black transition-colors">
                    个人中心 · 错题本
                  </button>
                </div>
              </Transition>
              <div v-if="showExplanation" class="bg-amber-50 border border-amber-100 rounded-2xl p-4 mb-4">
                <p class="text-xs font-black text-amber-600 mb-1">💡 解析</p>
                <p class="text-sm text-slate-700">{{ quizQuestions[currentQuizIndex].explanation }}</p>
              </div>
              <div class="flex justify-end">
                <button v-if="showExplanation && currentQuizIndex < quizQuestions.length - 1" @click="nextQuestion"
                  class="px-6 py-2.5 bg-violet-600 text-white text-sm font-black rounded-2xl hover:bg-violet-700 transition-colors">
                  下一题 →
                </button>
                <p v-else-if="showExplanation" class="text-sm text-emerald-600 font-black">🎉 自测完成！获得 +45 XP</p>
              </div>
            </div>
          </div>
          <!-- ── AI 快照拍立得区 ── -->
          <Transition name="slide-up">
            <div v-if="snapshots.length > 0" class="space-y-3">
              <div class="flex items-center gap-2">
                <h2 class="text-sm font-black text-slate-800">📸 AI 伴读快照</h2>
                <span class="text-xs bg-violet-100 text-violet-600 px-2 py-0.5 rounded-full font-bold">{{ snapshots.length }} 张</span>
                <button @click="snapshots = []" class="ml-auto text-xs text-slate-400 hover:text-slate-600 transition-colors">清空</button>
              </div>
              <div class="flex gap-4 overflow-x-auto pb-2 -mx-1 px-1">
                <div v-for="(snap, si) in snapshots" :key="snap.id"
                  class="flex-shrink-0 w-52 bg-white rounded-2xl shadow-lg border border-slate-100 overflow-hidden transition-all duration-300 hover:shadow-xl hover:-translate-y-1"
                  :style="{ transform: `rotate(${(si % 2 === 0 ? -1.5 : 1.5)}deg)` }">
                  <!-- Polaroid frame top: video frame placeholder -->
                  <div class="h-28 flex items-center justify-center relative"
                    :class="{
                      'bg-gradient-to-br from-violet-800 to-purple-900': snap.tint==='violet',
                      'bg-gradient-to-br from-blue-800 to-indigo-900': snap.tint==='blue',
                      'bg-gradient-to-br from-emerald-800 to-teal-900': snap.tint==='emerald',
                    }">
                    <!-- Simulated blackboard content -->
                    <div class="absolute inset-3 rounded-xl border border-white/10 flex items-center justify-center">
                      <div class="text-center">
                        <p class="text-white/60 text-xs font-mono">{{ snap.time }}</p>
                        <p class="text-white/30 text-xs mt-0.5">帧截图</p>
                      </div>
                    </div>
                    <!-- Timestamp badge -->
                    <div class="absolute top-2 right-2 bg-black/40 text-white/80 text-xs px-2 py-0.5 rounded-lg font-mono">{{ snap.time }}</div>
                  </div>
                  <!-- Polaroid bottom: AI notes -->
                  <div class="p-3 bg-white">
                    <p class="text-xs font-black text-slate-600 mb-1.5 truncate">{{ snap.course }}</p>
                    <p class="text-xs text-slate-500 leading-relaxed whitespace-pre-line font-mono">{{ snap.note }}</p>
                    <div class="flex items-center justify-between mt-2 pt-2 border-t border-slate-50">
                      <span class="text-xs text-slate-300 font-medium">AI 提取板书</span>
                      <span class="text-xs text-violet-400 font-bold cursor-pointer hover:text-violet-600">加入笔记</span>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </Transition>
        </div>

        <!-- ══ VIEW 6: Schedule (Google Calendar) ══ -->
        <div v-else-if="currentView === 'schedule'" class="flex-1 flex flex-col overflow-hidden">

          <!-- Page header -->
          <div class="flex-shrink-0 bg-white border-b border-slate-100 px-6 py-4">
            <div class="flex items-center justify-between">
              <div>
                <h1 class="text-xl font-black text-slate-800">我的课表</h1>
                <p class="text-sm text-slate-400 mt-0.5">{{ className ? className + ' · ' : '' }}本周 4/21 – 4/27</p>
              </div>
              <button @click="openAddEvent(0, 9, 0)"
                class="flex items-center gap-2 px-4 py-2.5 bg-violet-600 text-white text-sm font-black rounded-2xl hover:bg-violet-700 active:scale-95 transition-all shadow-sm">
                <svg class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.5" d="M12 4v16m8-8H4"/>
                </svg>
                添加课外课程
              </button>
            </div>
          </div>

          <!-- Day label row (sticky) -->
          <div class="flex-shrink-0 bg-white border-b border-slate-100">
            <div class="flex">
              <div class="w-14 flex-shrink-0 border-r border-slate-100"></div>
              <div v-for="(label, di) in calDayLabels" :key="di"
                class="flex-1 py-2.5 text-center border-l border-slate-100 first:border-l-0">
                <p class="text-xs font-black text-slate-600">{{ label }}</p>
                <p class="text-xs text-slate-300 font-mono mt-0.5">{{ calDates[di] }}</p>
              </div>
            </div>
          </div>

          <!-- Scrollable time grid -->
          <div class="flex-1 overflow-y-auto">
            <div class="flex" :style="{ minHeight: (calHours.length * CAL_PX_PER_HOUR) + 'px' }">

              <!-- Hour gutter -->
              <div class="w-14 flex-shrink-0 border-r border-slate-100 relative"
                :style="{ height: (calHours.length * CAL_PX_PER_HOUR) + 'px' }">
                <div v-for="h in calHours" :key="h"
                  class="absolute right-2 text-xs text-slate-300 font-mono select-none"
                  :style="{ top: ((h - CAL_START_HOUR) * CAL_PX_PER_HOUR - 8) + 'px' }">
                  {{ String(h).padStart(2,'0') }}
                </div>
              </div>

              <!-- Day columns -->
              <div v-for="(label, di) in calDayLabels" :key="di"
                class="flex-1 border-l border-slate-100 relative select-none"
                :style="{ height: (calHours.length * CAL_PX_PER_HOUR) + 'px' }"
                @click="handleColClick(di, $event)">

                <!-- Horizontal hour lines -->
                <div v-for="h in calHours" :key="h"
                  class="absolute left-0 right-0 pointer-events-none"
                  :style="{ top: ((h - CAL_START_HOUR) * CAL_PX_PER_HOUR) + 'px',
                             borderTop: h % 2 === 0 ? '1px solid #f1f5f9' : '1px dashed #f8fafc' }">
                </div>

                <!-- Event cards -->
                <div v-for="ev in dayEvents(di)" :key="ev.id"
                  class="absolute left-1 right-1 rounded-xl overflow-hidden cursor-pointer hover:brightness-95 transition-all"
                  :style="{
                    top:    evTop(ev) + 'px',
                    height: evHeight(ev) + 'px',
                    backgroundColor: EVENT_COLOR_MAP[ev.colorKey]?.light || '#ede9fe',
                    borderLeft: '3px solid ' + (EVENT_COLOR_MAP[ev.colorKey]?.hex || '#8b5cf6'),
                  }"
                  @click.stop="openEventDetail(ev)">
                  <div class="px-2 py-1 h-full flex flex-col justify-start overflow-hidden">
                    <p class="text-xs font-black leading-snug truncate"
                      :style="{ color: EVENT_COLOR_MAP[ev.colorKey]?.text }">{{ ev.subject }}</p>
                    <p v-if="evHeight(ev) >= 38" class="text-xs leading-snug truncate opacity-70"
                      :style="{ color: EVENT_COLOR_MAP[ev.colorKey]?.text }">
                      {{ String(ev.startHour).padStart(2,'0') }}:{{ String(ev.startMin).padStart(2,'0') }}
                      <span v-if="ev.teacher"> · {{ ev.teacher }}</span>
                    </p>
                    <span v-if="ev.type === 'extra' && evHeight(ev) >= 50"
                      class="mt-auto text-xs font-bold opacity-60"
                      :style="{ color: EVENT_COLOR_MAP[ev.colorKey]?.text }">课外</span>
                  </div>
                </div>

                <!-- Click-to-add hint overlay (transparent) -->
                <div class="absolute inset-0 hover:bg-violet-500/5 transition-colors pointer-events-none rounded-sm"></div>
              </div>

            </div>
          </div>

          <!-- Add / View event modal -->
          <Transition name="modal">
            <div v-if="showAddEventModal"
              class="fixed inset-0 z-50 flex items-center justify-center bg-black/40 backdrop-blur-sm"
              @click.self="showAddEventModal = false">
              <div class="bg-white rounded-3xl p-6 w-80 shadow-2xl">

                <!-- Viewing an existing event -->
                <template v-if="selectedEvent">
                  <div class="flex items-start justify-between mb-4">
                    <div class="flex items-center gap-3">
                      <div class="w-3 h-3 rounded-full flex-shrink-0"
                        :style="{ backgroundColor: EVENT_COLOR_MAP[selectedEvent.colorKey]?.hex }"></div>
                      <h3 class="text-base font-black text-slate-800">{{ selectedEvent.subject }}</h3>
                      <span v-if="selectedEvent.type === 'extra'"
                        class="text-xs bg-violet-100 text-violet-600 px-2 py-0.5 rounded-full font-bold">课外</span>
                    </div>
                    <button @click="showAddEventModal = false" class="text-slate-400 hover:text-slate-600 text-lg leading-none">×</button>
                  </div>
                  <div class="space-y-2 mb-5 text-sm text-slate-600">
                    <p>🕐 {{ calDayLabels[selectedEvent.day] }} {{ String(selectedEvent.startHour).padStart(2,'0') }}:{{ String(selectedEvent.startMin).padStart(2,'0') }} · {{ selectedEvent.duration }} 分钟</p>
                    <p v-if="selectedEvent.teacher">👩‍🏫 {{ selectedEvent.teacher }}</p>
                  </div>
                  <div class="flex gap-2">
                    <button v-if="selectedEvent.type === 'extra'"
                      @click="deleteEvent(selectedEvent.id)"
                      class="flex-1 py-2.5 bg-red-50 text-red-500 font-black rounded-2xl text-sm hover:bg-red-100 transition-colors">
                      删除
                    </button>
                    <button @click="showAddEventModal = false"
                      class="flex-1 py-2.5 bg-slate-100 text-slate-600 font-black rounded-2xl text-sm hover:bg-slate-200 transition-colors">
                      关闭
                    </button>
                  </div>
                </template>

                <!-- Adding a new event -->
                <template v-else>
                  <div class="flex items-center justify-between mb-5">
                    <h3 class="text-base font-black text-slate-800">添加课外课程</h3>
                    <button @click="showAddEventModal = false" class="text-slate-400 hover:text-slate-600 text-lg leading-none">×</button>
                  </div>
                  <div class="space-y-3">
                    <input v-model="addEventForm.subject" placeholder="课程名称（如：钢琴、编程）"
                      class="w-full px-4 py-2.5 bg-slate-50 border border-slate-200 rounded-2xl text-sm outline-none focus:border-violet-400 focus:bg-white transition-colors" />
                    <input v-model="addEventForm.teacher" placeholder="老师（可选）"
                      class="w-full px-4 py-2.5 bg-slate-50 border border-slate-200 rounded-2xl text-sm outline-none focus:border-violet-400 focus:bg-white transition-colors" />
                    <div class="grid grid-cols-2 gap-2">
                      <select v-model.number="addEventForm.day"
                        class="px-3 py-2.5 bg-slate-50 border border-slate-200 rounded-2xl text-sm outline-none focus:border-violet-400 transition-colors">
                        <option v-for="(l, i) in calDayLabels" :key="i" :value="i">{{ l }}</option>
                      </select>
                      <select v-model.number="addEventForm.duration"
                        class="px-3 py-2.5 bg-slate-50 border border-slate-200 rounded-2xl text-sm outline-none focus:border-violet-400 transition-colors">
                        <option :value="30">30 分钟</option>
                        <option :value="45">45 分钟</option>
                        <option :value="60">1 小时</option>
                        <option :value="90">1.5 小时</option>
                        <option :value="120">2 小时</option>
                      </select>
                    </div>
                    <div class="grid grid-cols-2 gap-2">
                      <select v-model.number="addEventForm.startHour"
                        class="px-3 py-2.5 bg-slate-50 border border-slate-200 rounded-2xl text-sm outline-none focus:border-violet-400 transition-colors">
                        <option v-for="h in calHours" :key="h" :value="h">{{ String(h).padStart(2,'0') }}:00</option>
                      </select>
                      <select v-model.number="addEventForm.startMin"
                        class="px-3 py-2.5 bg-slate-50 border border-slate-200 rounded-2xl text-sm outline-none focus:border-violet-400 transition-colors">
                        <option :value="0">整点</option>
                        <option :value="15">:15</option>
                        <option :value="30">:30</option>
                        <option :value="45">:45</option>
                      </select>
                    </div>
                    <!-- Color picker -->
                    <div>
                      <p class="text-xs text-slate-400 font-semibold mb-2">颜色标签</p>
                      <div class="flex gap-2 flex-wrap">
                        <button v-for="(val, key) in EVENT_COLOR_MAP" :key="key"
                          @click="addEventForm.colorKey = key"
                          class="w-7 h-7 rounded-full border-2 transition-all hover:scale-110"
                          :style="{ backgroundColor: val.hex }"
                          :class="addEventForm.colorKey === key ? 'border-slate-700 scale-110 shadow-md' : 'border-transparent'">
                        </button>
                      </div>
                    </div>
                  </div>
                  <div class="flex gap-2 mt-5">
                    <button @click="showAddEventModal = false"
                      class="flex-1 py-2.5 bg-slate-100 text-slate-600 font-black rounded-2xl text-sm hover:bg-slate-200 transition-colors">
                      取消
                    </button>
                    <button @click="addEvent"
                      class="flex-1 py-2.5 bg-violet-600 text-white font-black rounded-2xl text-sm hover:bg-violet-700 active:scale-95 transition-all">
                      添加
                    </button>
                  </div>
                </template>

              </div>
            </div>
          </Transition>

        </div>

        <!-- ══ VIEW 7: 个人中心 ══ -->
        <div v-else-if="currentView === 'profile'" class="p-6 space-y-5">

          <!-- 用户信息横幅 -->
          <div class="bg-gradient-to-r from-violet-600 via-purple-600 to-indigo-600 rounded-3xl p-6 text-white relative overflow-hidden">
            <div class="absolute -top-8 -right-8 w-40 h-40 bg-white/5 rounded-full"></div>
            <div class="absolute -bottom-6 left-1/4 w-28 h-28 bg-white/5 rounded-full"></div>
            <div class="relative flex items-center gap-5">
              <div class="w-16 h-16 rounded-2xl bg-white/20 backdrop-blur-sm flex items-center justify-center text-2xl font-black border border-white/20 flex-shrink-0">
                {{ user.avatar }}
              </div>
              <div class="flex-1 min-w-0">
                <h1 class="text-xl font-black">{{ user.name }}</h1>
                <p class="text-white/70 text-sm mt-0.5">{{ className || '未加入班级' }} · Lv {{ user.level }}</p>
                <div class="flex items-center gap-5 mt-3 text-sm flex-wrap">
                  <span class="text-white/80">📚 在学 <b class="text-white">{{ myEnrolledCourses.length }}</b> 门</span>
                  <span class="text-white/80">🔥 连续 <b class="text-white">{{ user.streak }}</b> 天</span>
                  <span class="text-white/80">📅 累计 <b class="text-white">{{ user.totalStudyDays }}</b> 天</span>
                </div>
              </div>
            </div>
          </div>

          <!-- ── 统计 + 热力图 左右并排 ── -->
          <div class="grid grid-cols-12 gap-9 items-stretch">

            <!-- 左侧：2×2 统计四宫格 (5/12) -->
            <div class="col-span-5 grid grid-cols-2 gap-3">
              <div v-for="stat in homeStats" :key="stat.label"
                class="bg-white rounded-2xl p-4 shadow-sm border border-slate-100 text-center
                       flex flex-col items-center justify-center">
                <p class="text-2xl font-black leading-none" :class="stat.color">{{ stat.value }}</p>
                <p class="text-xs text-slate-400 mt-1.5 font-medium leading-snug">{{ stat.label }}</p>
                <div class="inline-flex items-center gap-1 rounded-full px-2 py-0.5 border mt-2.5"
                  :class="stat.pct <= 20 ? 'bg-emerald-50 border-emerald-100'
                        : stat.pct <= 60 ? 'bg-amber-50 border-amber-100'
                        :                  'bg-slate-50 border-slate-100'">
                  <span class="text-xs font-black"
                    :class="stat.pct <= 20 ? 'text-emerald-600'
                          : stat.pct <= 60 ? 'text-amber-500'
                          :                  'text-slate-400'">
                    前 {{ stat.pct }}%
                  </span>
                </div>
              </div>
            </div>

            <!-- 右侧：学情热力图 (7/12) -->
            <div class="col-span-7 bg-white rounded-3xl p-5 shadow-sm border border-slate-100 flex flex-col justify-between">

              <!-- 卡片头 -->
              <div class="flex items-start justify-between mb-3">
                <div>
                  <h2 class="text-sm font-black text-slate-800">学情热力图</h2>
                  <p class="text-xs text-slate-400 mt-0.5">最近 12 周学习打卡记录</p>
                </div>
                <div class="flex items-center gap-2.5 flex-shrink-0">
                  <span class="flex items-center gap-1.5 text-xs text-slate-400">
                    <span class="w-3 h-3 rounded-sm bg-rose-400 flex-shrink-0"></span>有错题
                  </span>
                  <span class="flex items-center gap-1.5 text-xs text-slate-400">
                    <span class="w-3 h-3 rounded-sm bg-emerald-400 flex-shrink-0"></span>完美通关
                  </span>
                  <span class="flex items-center gap-1.5 text-xs text-slate-400">
                    <span class="w-3 h-3 rounded-sm bg-slate-100 border border-slate-200 flex-shrink-0"></span>未做题
                  </span>
                </div>
              </div>

              <!-- 月份标签行 -->
              <div class="flex mb-1" style="padding-left: 22px;">
                <div v-for="(label, wi) in heatmapWeekLabels" :key="wi"
                  class="text-[10px] text-slate-400 font-medium leading-none"
                  style="width: 19px; margin-right: 4px; overflow: visible; white-space: nowrap;">
                  {{ label }}
                </div>
              </div>

              <!-- 热力图主体（居中） -->
              <div class="flex items-start gap-2 flex-1">
                <!-- 星期标签 -->
                <div class="flex flex-col gap-[4px] text-[10px] text-slate-300 font-medium leading-none flex-shrink-0" style="padding-top: 1px;">
                  <div style="height:15px" class="flex items-center">一</div>
                  <div style="height:15px" class="flex items-center">二</div>
                  <div style="height:15px" class="flex items-center">三</div>
                  <div style="height:15px" class="flex items-center">四</div>
                  <div style="height:15px" class="flex items-center">五</div>
                  <div style="height:15px" class="flex items-center">六</div>
                  <div style="height:15px" class="flex items-center">日</div>
                </div>

                <!-- 方格矩阵（7行 × 12列，列优先） -->
                <div class="grid grid-rows-7 grid-flow-col gap-[4px]" style="overflow: visible;">
                  <div v-for="(cell, idx) in heatmapData" :key="idx"
                    @click="clickCell(cell)"
                    class="w-[15px] h-[15px] rounded-sm cursor-pointer transition-all duration-150 hover:scale-125 relative group"
                    :class="cell.status==='mistake' ? 'bg-rose-400 hover:bg-rose-500'
                           : cell.status==='perfect' ? 'bg-emerald-400 hover:bg-emerald-500'
                           :                           'bg-slate-100 hover:bg-slate-200 border border-slate-200/60'">
                    <!-- Tooltip -->
                    <div class="absolute bottom-full left-1/2 -translate-x-1/2 mb-2 z-50
                                opacity-0 group-hover:opacity-100 pointer-events-none
                                transition-opacity duration-150 whitespace-nowrap
                                bg-slate-800 text-white text-[10px] font-semibold
                                px-2 py-1 rounded-lg shadow-lg">
                      {{ cell.fullDate }}：{{ cell.status==='mistake' ? '有错题 🔴' : cell.status==='perfect' ? '完美通关 🟢' : '未做题 ⬜' }}
                      <div class="absolute top-full left-1/2 -translate-x-1/2 border-4 border-transparent border-t-slate-800"></div>
                    </div>
                  </div>
                </div>
              </div>

              <!-- 统计摘要行 -->
              <div class="flex items-center gap-4 mt-3 pt-3 border-t border-slate-50">
                <div class="flex items-center gap-1.5">
                  <span class="w-2 h-2 rounded-full bg-rose-400 flex-shrink-0"></span>
                  <span class="text-xs text-slate-500">错题日 <b class="text-slate-700">{{ heatmapData.filter(c=>c.status==='mistake').length }}</b> 天</span>
                </div>
                <div class="flex items-center gap-1.5">
                  <span class="w-2 h-2 rounded-full bg-emerald-400 flex-shrink-0"></span>
                  <span class="text-xs text-slate-500">完美日 <b class="text-slate-700">{{ heatmapData.filter(c=>c.status==='perfect').length }}</b> 天</span>
                </div>
                <div class="flex items-center gap-1.5">
                  <span class="w-2 h-2 rounded-full bg-slate-200 flex-shrink-0"></span>
                  <span class="text-xs text-slate-500">未学习 <b class="text-slate-700">{{ heatmapData.filter(c=>c.status==='empty').length }}</b> 天</span>
                </div>
                <span class="ml-auto text-xs text-slate-300 whitespace-nowrap">点击红格查看当日错题</span>
              </div>
            </div>

          </div>

          <!-- ── 内容 Tab 区 ── -->
          <div class="bg-white rounded-3xl shadow-sm border border-slate-100 overflow-hidden">
            <!-- Tab 导航栏（LeetCode 风格下划线式） -->
            <div class="flex items-center border-b border-slate-100 px-5">
              <button v-for="tab in [{v:'courses',l:'我的课程'},{v:'notes',l:'我的笔记'},{v:'mistakes',l:'全部错题'}]" :key="tab.v"
                @click="profileTab = tab.v; if(tab.v !== 'mistakes') heatmapFilterDate = null"
                class="px-4 py-3.5 text-sm font-bold transition-all border-b-2 -mb-px mr-2 whitespace-nowrap"
                :class="profileTab === tab.v
                  ? 'border-violet-500 text-violet-700'
                  : 'border-transparent text-slate-400 hover:text-slate-700'">
                {{ tab.l }}
                <span v-if="tab.v === 'courses'" class="ml-1.5 text-xs bg-violet-100 text-violet-500 px-1.5 py-0.5 rounded-full font-black">
                  {{ myEnrolledCourses.length }}
                </span>
                <span v-if="tab.v === 'notes'" class="ml-1.5 text-xs bg-blue-100 text-blue-500 px-1.5 py-0.5 rounded-full font-black">
                  {{ myNotes.length }}
                </span>
                <span v-if="tab.v === 'mistakes'" class="ml-1.5 text-xs bg-rose-100 text-rose-500 px-1.5 py-0.5 rounded-full font-black">
                  {{ myMistakes.length }}
                </span>
              </button>
              <!-- 日期过滤徽章 -->
              <Transition name="slide-up">
                <div v-if="heatmapFilterDate && profileTab === 'mistakes'"
                  class="ml-auto flex items-center gap-1.5 text-xs bg-rose-50 text-rose-600 border border-rose-100 px-2.5 py-1 rounded-full font-semibold">
                  <span class="w-2 h-2 rounded-full bg-rose-400"></span>
                  {{ heatmapFilterDate }} 错题
                  <button @click="heatmapFilterDate = null" class="ml-1 text-rose-400 hover:text-rose-600 font-black">✕</button>
                </div>
              </Transition>
            </div>

            <!-- 内容区 -->
            <div class="p-5">

              <!-- ▸ 我的课程 -->
              <template v-if="profileTab === 'courses'">
                <div v-if="myEnrolledCourses.length === 0" class="text-center py-10">
                  <p class="text-4xl mb-3">📭</p>
                  <p class="text-sm font-bold text-slate-500 mb-1">暂无已加入的课程</p>
                  <button @click="switchView('courses')"
                    class="mt-3 px-5 py-2 bg-violet-600 text-white text-xs font-black rounded-xl hover:bg-violet-700 transition-colors">
                    去课程中心选课
                  </button>
                </div>
                <div v-else class="space-y-1">
                  <div v-for="c in myEnrolledCourses" :key="c.id"
                    class="flex items-center gap-4 px-3 py-3.5 rounded-2xl transition-all group cursor-pointer hover:bg-slate-50">
                    <div class="w-11 h-11 rounded-2xl flex items-center justify-center flex-shrink-0 text-xl"
                      :class="c.color==='purple' ? 'bg-violet-50'
                            : c.color==='blue'   ? 'bg-blue-50'
                            : c.color==='green'  ? 'bg-emerald-50'
                            : c.color==='orange' ? 'bg-orange-50'
                                                 : 'bg-teal-50'">
                      {{ getCourseEmoji(c.subject) }}
                    </div>
                    <div class="flex-1 min-w-0">
                      <div class="flex items-center gap-2">
                        <p class="text-sm font-bold text-slate-800 truncate">{{ c.title }}</p>
                        <span v-if="c.type==='series'" class="flex-shrink-0 text-[10px] bg-violet-100 text-violet-600 px-1.5 py-0.5 rounded-full font-bold">套系</span>
                      </div>
                      <p class="text-xs text-slate-400 mt-0.5 truncate">{{ c.teacher }}</p>
                      <div class="flex items-center gap-2 mt-2">
                        <div class="flex-1 h-1.5 bg-slate-100 rounded-full overflow-hidden">
                          <div class="h-full rounded-full transition-all duration-700"
                            :class="c.color==='purple' ? 'bg-gradient-to-r from-violet-400 to-purple-500'
                                  : c.color==='blue'   ? 'bg-gradient-to-r from-blue-400 to-cyan-400'
                                  : c.color==='orange' ? 'bg-gradient-to-r from-orange-400 to-amber-400'
                                  : c.color==='teal'   ? 'bg-gradient-to-r from-teal-400 to-cyan-400'
                                  :                     'bg-gradient-to-r from-emerald-400 to-teal-500'"
                            :style="{ width: seriesProgress(c) + '%' }"></div>
                        </div>
                        <span class="text-xs font-bold text-slate-400 w-8 text-right flex-shrink-0">{{ seriesProgress(c) }}%</span>
                      </div>
                    </div>
                    <svg class="w-10 h-10 -rotate-90 flex-shrink-0" viewBox="0 0 36 36">
                      <circle cx="18" cy="18" r="15" fill="none" stroke="#f1f5f9" stroke-width="3"/>
                      <circle cx="18" cy="18" r="15" fill="none"
                        :stroke="c.color==='purple' ? '#8b5cf6' : c.color==='blue' ? '#3b82f6' : c.color==='orange' ? '#f97316' : c.color==='teal' ? '#14b8a6' : '#10b981'"
                        stroke-width="3" stroke-linecap="round"
                        :stroke-dasharray="`${seriesProgress(c) * 0.942} 94.2`"/>
                    </svg>
                    <button @click.stop="continueSeriesNode(c)"
                      class="px-3 py-1.5 text-xs font-black rounded-xl bg-violet-100 text-violet-700
                             opacity-0 group-hover:opacity-100 translate-x-2 group-hover:translate-x-0
                             transition-all duration-200 hover:bg-violet-200 flex-shrink-0 whitespace-nowrap">
                      继续 →
                    </button>
                  </div>
                </div>
              </template>

              <!-- ▸ 我的笔记 -->
              <template v-else-if="profileTab === 'notes'">
                <div v-if="myNotes.length === 0" class="text-center py-10">
                  <p class="text-4xl mb-3">📒</p>
                  <p class="text-sm font-bold text-slate-500 mb-1">还没有 AI 笔记</p>
                  <p class="text-xs text-slate-400">在视频播放器中点击「📸 一键 AI 快照笔记」即可生成</p>
                </div>
                <div v-else class="grid grid-cols-2 gap-3">
                  <div v-for="note in myNotes" :key="note.id"
                    @click="openNote(note)"
                    class="rounded-2xl overflow-hidden cursor-pointer group transition-all hover:-translate-y-0.5"
                    style="box-shadow: 0 1px 3px rgba(0,0,0,.07)">
                    <div class="px-3.5 pt-3 pb-2.5"
                      :class="note.tint==='violet' ? 'bg-violet-50' : note.tint==='blue' ? 'bg-blue-50' : 'bg-emerald-50'">
                      <div class="flex items-center justify-between mb-1">
                        <span class="text-xs font-black"
                          :class="note.tint==='violet' ? 'text-violet-600' : note.tint==='blue' ? 'text-blue-600' : 'text-emerald-600'">
                          🤖 AI 笔记
                        </span>
                        <span class="text-xs text-slate-400">{{ note.createdAt }}</span>
                      </div>
                      <p class="text-xs font-black text-slate-800 leading-snug line-clamp-2">{{ note.title }}</p>
                      <p class="text-xs text-slate-400 mt-0.5 truncate">{{ note.course }}</p>
                    </div>
                    <div class="px-3.5 py-3 bg-white">
                      <p class="text-xs text-slate-500 leading-relaxed line-clamp-3">{{ note.summary }}</p>
                    </div>
                  </div>
                </div>
              </template>

              <!-- ▸ 全部错题（支持热力图日期过滤） -->
              <template v-else-if="profileTab === 'mistakes'">
                <div v-if="profileMistakesFiltered.length === 0" class="text-center py-10">
                  <p class="text-4xl mb-3">{{ heatmapFilterDate ? '🔍' : '✅' }}</p>
                  <p class="text-sm font-bold text-slate-500 mb-1">
                    {{ heatmapFilterDate ? heatmapFilterDate + ' 当天没有错题记录' : '暂无错题记录' }}
                  </p>
                  <button v-if="heatmapFilterDate" @click="heatmapFilterDate = null"
                    class="mt-2 text-xs text-violet-500 font-semibold hover:underline">
                    清除过滤，查看全部
                  </button>
                </div>
                <div v-else class="space-y-3">
                  <div v-for="m in profileMistakesFiltered" :key="m.id"
                    class="rounded-2xl border border-slate-100 overflow-hidden">
                    <div class="px-4 pt-3.5 pb-3 bg-red-50">
                      <div class="flex items-center justify-between mb-1.5">
                        <span class="text-xs font-bold text-red-500">{{ m.subject }} · {{ m.course }}</span>
                        <span class="text-xs text-slate-400">{{ m.createdAt }}</span>
                      </div>
                      <p class="text-sm font-black text-slate-800">{{ m.question }}</p>
                    </div>
                    <div class="px-4 py-3 bg-white space-y-1.5">
                      <div v-for="(opt, idx) in m.options" :key="idx"
                        class="flex items-center gap-2.5 px-3 py-2 rounded-xl text-xs font-semibold"
                        :class="idx === m.correct ? 'bg-emerald-50 text-emerald-700' : idx === m.selected ? 'bg-red-50 text-red-600' : 'text-slate-400'">
                        <span v-if="idx === m.correct" class="w-4 h-4 rounded-full bg-emerald-500 flex items-center justify-center flex-shrink-0">
                          <svg class="w-2.5 h-2.5 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M5 13l4 4L19 7"/></svg>
                        </span>
                        <span v-else-if="idx === m.selected" class="w-4 h-4 rounded-full bg-red-400 flex items-center justify-center flex-shrink-0">
                          <svg class="w-2.5 h-2.5 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M6 18L18 6M6 6l12 12"/></svg>
                        </span>
                        <span v-else class="w-4 h-4 rounded-full border border-slate-200 flex-shrink-0"></span>
                        <span>{{ ['A','B','C','D'][idx] }}. {{ opt }}</span>
                        <span v-if="idx === m.selected && idx !== m.correct" class="ml-auto text-xs text-red-400 font-bold flex-shrink-0">我的答案</span>
                        <span v-if="idx === m.correct" class="ml-auto text-xs text-emerald-500 font-bold flex-shrink-0">正确答案</span>
                      </div>
                    </div>
                    <div class="px-4 pb-3.5 bg-white">
                      <button @click="m.showAIHint = !m.showAIHint"
                        class="w-full py-2 rounded-xl text-xs font-black transition-all"
                        :class="m.showAIHint
                          ? 'bg-violet-100 text-violet-700'
                          : 'bg-gradient-to-r from-violet-500 to-purple-600 text-white shadow-md shadow-violet-200/60 hover:shadow-lg'">
                        🤖 {{ m.showAIHint ? '收起 AI 启发思路' : 'AI 启发思路' }}
                      </button>
                      <Transition name="slide-up">
                        <div v-if="m.showAIHint"
                          class="mt-2.5 px-3.5 py-3 bg-violet-50 rounded-xl border border-violet-100 text-xs text-violet-800 leading-relaxed">
                          {{ m.aiHint }}
                        </div>
                      </Transition>
                    </div>
                  </div>
                </div>
              </template>

            </div>
          </div>

        </div>

        <!-- ══ VIEW 8: 我的笔记 ══ -->
        <div v-else-if="currentView === 'notes'" class="p-6 space-y-5">
          <!-- 页头 -->
          <div class="flex items-center justify-between flex-wrap gap-3">
            <div>
              <h1 class="text-xl font-black text-slate-800">我的笔记</h1>
              <p class="text-sm text-slate-400 mt-0.5">{{ myNotes.length }} 条 AI 智能笔记</p>
            </div>
            <div class="flex items-center gap-2 flex-wrap">
              <!-- 搜索框 -->
              <div class="relative">
                <svg class="absolute left-3 top-1/2 -translate-y-1/2 w-4 h-4 text-slate-400 pointer-events-none" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"/>
                </svg>
                <input v-model="notesSearch" placeholder="搜索笔记标题或内容…"
                  class="pl-9 pr-4 py-2 text-sm bg-slate-50 border border-slate-200 rounded-2xl
                         w-52 focus:outline-none focus:ring-2 focus:ring-violet-300 focus:border-transparent transition-all"/>
              </div>
              <!-- 标签筛选 -->
              <div class="flex items-center gap-1">
                <button v-for="tag in noteTagList" :key="tag"
                  @click="notesTag = tag"
                  class="px-3 py-1.5 text-xs font-bold rounded-full transition-all"
                  :class="notesTag === tag
                    ? 'bg-violet-600 text-white shadow-sm shadow-violet-200'
                    : 'bg-slate-100 text-slate-500 hover:bg-slate-200'">
                  {{ tag }}
                </button>
              </div>
            </div>
          </div>

          <!-- 空状态 -->
          <div v-if="filteredNotes.length === 0" class="flex flex-col items-center justify-center py-20">
            <p class="text-5xl mb-4">📒</p>
            <p class="text-sm font-bold text-slate-500">暂无匹配笔记</p>
            <p class="text-xs text-slate-400 mt-1">在视频播放器中点击「📸 一键 AI 快照笔记」生成笔记</p>
          </div>

          <!-- 笔记 Grid -->
          <div v-else class="grid grid-cols-3 gap-4">
            <div v-for="note in filteredNotes" :key="note.id"
              @click="openNote(note)"
              class="rounded-3xl overflow-hidden cursor-pointer group transition-all duration-200 hover:-translate-y-1 hover:shadow-lg"
              style="box-shadow: 0 1px 3px rgba(0,0,0,.08)">
              <!-- 模拟视频截图区 -->
              <div class="h-28 relative overflow-hidden"
                :class="note.tint==='violet' ? 'bg-gradient-to-br from-violet-100 to-purple-200'
                       : note.tint==='blue'  ? 'bg-gradient-to-br from-blue-100 to-cyan-200'
                       :                       'bg-gradient-to-br from-emerald-100 to-teal-200'">
                <!-- 模拟板书线条 -->
                <div class="absolute inset-4 opacity-20">
                  <div class="h-px bg-current mb-3 w-3/4"></div>
                  <div class="h-px bg-current mb-3 w-1/2"></div>
                  <div class="h-px bg-current mb-3 w-2/3"></div>
                </div>
                <!-- 播放遮罩 -->
                <div class="absolute inset-0 flex items-center justify-center opacity-0 group-hover:opacity-100 transition-opacity bg-black/10">
                  <div class="w-10 h-10 rounded-full bg-white/80 flex items-center justify-center shadow-md">
                    <svg class="w-4 h-4 text-slate-700 ml-0.5" fill="currentColor" viewBox="0 0 24 24">
                      <path d="M8 5v14l11-7z"/>
                    </svg>
                  </div>
                </div>
                <div class="absolute bottom-2 left-3 flex items-center gap-1.5">
                  <span class="text-xs font-black px-2 py-0.5 rounded-full bg-white/70 backdrop-blur-sm"
                    :class="note.tint==='violet' ? 'text-violet-600'
                           : note.tint==='blue'  ? 'text-blue-600'
                           :                       'text-emerald-600'">
                    🤖 AI 快照
                  </span>
                  <span class="text-xs text-white/90 bg-black/30 px-1.5 py-0.5 rounded-md font-mono">{{ note.time }}</span>
                </div>
              </div>
              <!-- 卡片内容 -->
              <div class="p-4 bg-white">
                <p class="text-sm font-black text-slate-800 leading-tight mb-1 line-clamp-2">{{ note.title }}</p>
                <p class="text-xs text-slate-400 mb-2 truncate">{{ note.course }}</p>
                <p class="text-xs text-slate-500 leading-relaxed line-clamp-3">{{ note.summary }}</p>
                <p class="text-xs text-slate-300 mt-2">{{ note.createdAt }}</p>
              </div>
            </div>
          </div>
        </div>

        <!-- ══ VIEW 9: 错题本 ══ -->
        <div v-else-if="currentView === 'mistakes'" class="p-6 space-y-5">
          <!-- 页头 -->
          <div class="flex items-center justify-between flex-wrap gap-3">
            <div>
              <h1 class="text-xl font-black text-slate-800">错题本</h1>
              <p class="text-sm text-slate-400 mt-0.5">{{ myMistakes.length }} 道错题已收录</p>
            </div>
            <div class="flex items-center gap-2 flex-wrap">
              <div class="relative">
                <svg class="absolute left-3 top-1/2 -translate-y-1/2 w-4 h-4 text-slate-400 pointer-events-none" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"/>
                </svg>
                <input v-model="mistakesSearch" placeholder="搜索题目或课程…"
                  class="pl-9 pr-4 py-2 text-sm bg-slate-50 border border-slate-200 rounded-2xl
                         w-52 focus:outline-none focus:ring-2 focus:ring-rose-300 focus:border-transparent transition-all"/>
              </div>
              <div class="flex items-center gap-1">
                <button v-for="tag in mistakeTagList" :key="tag"
                  @click="mistakesTag = tag"
                  class="px-3 py-1.5 text-xs font-bold rounded-full transition-all"
                  :class="mistakesTag === tag
                    ? 'bg-rose-500 text-white shadow-sm shadow-rose-200'
                    : 'bg-slate-100 text-slate-500 hover:bg-slate-200'">
                  {{ tag }}
                </button>
              </div>
            </div>
          </div>

          <!-- 空状态 -->
          <div v-if="filteredMistakes.length === 0" class="flex flex-col items-center justify-center py-20">
            <p class="text-5xl mb-4">✅</p>
            <p class="text-sm font-bold text-slate-500">暂无匹配错题</p>
            <p class="text-xs text-slate-400 mt-1">做题时的错误会自动收录至此</p>
          </div>

          <!-- 错题列表 -->
          <div v-else class="space-y-4 max-w-2xl">
            <div v-for="m in filteredMistakes" :key="m.id"
              class="rounded-2xl border border-slate-100 overflow-hidden">
              <div class="px-4 pt-3.5 pb-3 bg-red-50">
                <div class="flex items-center justify-between mb-1.5">
                  <span class="text-xs font-bold text-red-500">{{ m.subject }} · {{ m.course }}</span>
                  <span class="text-xs text-slate-400">{{ m.createdAt }}</span>
                </div>
                <p class="text-sm font-black text-slate-800">{{ m.question }}</p>
              </div>
              <div class="px-4 py-3 bg-white space-y-1.5">
                <div v-for="(opt, idx) in m.options" :key="idx"
                  class="flex items-center gap-2.5 px-3 py-2 rounded-xl text-xs font-semibold"
                  :class="idx === m.correct ? 'bg-emerald-50 text-emerald-700' : idx === m.selected ? 'bg-red-50 text-red-600' : 'text-slate-400'">
                  <span v-if="idx === m.correct" class="w-4 h-4 rounded-full bg-emerald-500 flex items-center justify-center flex-shrink-0">
                    <svg class="w-2.5 h-2.5 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M5 13l4 4L19 7"/></svg>
                  </span>
                  <span v-else-if="idx === m.selected" class="w-4 h-4 rounded-full bg-red-400 flex items-center justify-center flex-shrink-0">
                    <svg class="w-2.5 h-2.5 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M6 18L18 6M6 6l12 12"/></svg>
                  </span>
                  <span v-else class="w-4 h-4 rounded-full border border-slate-200 flex-shrink-0"></span>
                  <span>{{ ['A','B','C','D'][idx] }}. {{ opt }}</span>
                  <span v-if="idx === m.selected && idx !== m.correct" class="ml-auto text-xs text-red-400 font-bold flex-shrink-0">我的答案</span>
                  <span v-if="idx === m.correct" class="ml-auto text-xs text-emerald-500 font-bold flex-shrink-0">正确答案</span>
                </div>
              </div>
              <div class="px-4 pb-3.5 bg-white">
                <button @click="m.showAIHint = !m.showAIHint"
                  class="w-full py-2 rounded-xl text-xs font-black transition-all"
                  :class="m.showAIHint
                    ? 'bg-violet-100 text-violet-700'
                    : 'bg-gradient-to-r from-violet-500 to-purple-600 text-white shadow-md shadow-violet-200/60 hover:shadow-lg'">
                  🤖 {{ m.showAIHint ? '收起 AI 启发思路' : 'AI 启发思路' }}
                </button>
                <Transition name="slide-up">
                  <div v-if="m.showAIHint"
                    class="mt-2.5 px-3.5 py-3 bg-violet-50 rounded-xl border border-violet-100 text-xs text-violet-800 leading-relaxed">
                    {{ m.aiHint }}
                  </div>
                </Transition>
              </div>
            </div>
          </div>
        </div>

      </main>

      <!-- ── Right Sidebar ── -->
      <aside class="w-64 flex-shrink-0 bg-white border-l border-slate-100 flex flex-col overflow-y-auto p-4 gap-5" style="scrollbar-width: none; -ms-overflow-style: none;">

        <!-- Daily Tasks -->
        <div>
          <div class="flex items-center justify-between mb-3">
            <h2 class="text-sm font-black text-slate-800">每日任务</h2>
            <span class="text-xs bg-emerald-100 text-emerald-600 px-2 py-0.5 rounded-full font-bold">
              {{ dailyTasks.filter(t => t.done).length }}/{{ dailyTasks.length }}
            </span>
          </div>
          <!-- Overall progress -->
          <div class="h-2 bg-slate-100 rounded-full mb-3 overflow-hidden">
            <div class="h-full bg-gradient-to-r from-emerald-400 to-teal-500 rounded-full transition-all duration-700"
              :style="{ width: (dailyTasks.filter(t=>t.done).length / dailyTasks.length * 100) + '%' }"></div>
          </div>
          <div class="space-y-2">
            <div v-for="task in dailyTasks" :key="task.id"
              @click="completeTask(task)"
              class="flex items-center gap-3 p-3 rounded-2xl cursor-pointer transition-all select-none"
              :class="task.done ? 'bg-emerald-50' : 'bg-slate-50 hover:bg-slate-100'">
              <div class="w-6 h-6 rounded-full flex items-center justify-center flex-shrink-0 transition-all"
                :class="task.done ? 'bg-emerald-400' : 'border-2 border-slate-300'">
                <svg v-if="task.done" class="w-3.5 h-3.5 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M5 13l4 4L19 7"/>
                </svg>
              </div>
              <p class="flex-1 text-xs font-semibold truncate"
                :class="task.done ? 'text-emerald-700 line-through decoration-emerald-400' : 'text-slate-700'">
                {{ task.title }}
              </p>
              <span class="text-xs font-black text-amber-500 flex-shrink-0">+{{ task.xp }}</span>
            </div>
          </div>
        </div>

        <div class="border-t border-slate-100"></div>

        <!-- Leaderboard -->
        <div class="flex-1">
          <h2 class="text-sm font-black text-slate-800 mb-3">排行榜</h2>
          <div class="flex gap-1 bg-slate-100 p-1 rounded-xl mb-3">
            <button @click="leaderboardTab = 'streak'"
              class="flex-1 py-1.5 text-xs font-bold rounded-lg transition-all"
              :class="leaderboardTab==='streak' ? 'bg-white text-slate-800 shadow-sm' : 'text-slate-400 hover:text-slate-600'">
              🔥 坚持之星
            </button>
            <button @click="leaderboardTab = 'focus'"
              class="flex-1 py-1.5 text-xs font-bold rounded-lg transition-all"
              :class="leaderboardTab==='focus' ? 'bg-white text-slate-800 shadow-sm' : 'text-slate-400 hover:text-slate-600'">
              ⭐ 专注之星
            </button>
          </div>
          <div class="space-y-1.5">
            <div v-for="item in activeLeaderboard" :key="item.rank"
              class="flex items-center gap-2.5 px-2.5 py-2 rounded-xl transition-colors"
              :class="item.isMe ? 'bg-violet-50 border border-violet-100' : 'hover:bg-slate-50'">
              <span class="w-6 text-center text-sm flex-shrink-0">
                {{ item.rank===1 ? '🥇' : item.rank===2 ? '🥈' : item.rank===3 ? '🥉' : item.rank }}
              </span>
              <div class="w-7 h-7 rounded-xl flex items-center justify-center text-white text-xs font-black flex-shrink-0"
                :class="item.isMe ? 'bg-gradient-to-br from-violet-400 to-purple-500' : 'bg-gradient-to-br from-slate-300 to-slate-400'">
                {{ item.avatar }}
              </div>
              <p class="flex-1 text-xs font-semibold truncate" :class="item.isMe ? 'text-violet-700' : 'text-slate-700'">
                {{ item.name }}{{ item.isMe ? ' 👈' : '' }}
              </p>
              <span class="text-xs font-black flex-shrink-0" :class="item.isMe ? 'text-violet-600' : 'text-slate-500'">
                {{ leaderboardTab==='streak' ? item.days + '天' : item.hours + 'h' }}
              </span>
            </div>
          </div>
        </div>

        <div class="border-t border-slate-100"></div>

        <!-- 今日课程（紧凑版） -->
        <div>
          <div class="flex items-center justify-between mb-2">
            <h2 class="text-sm font-black text-slate-800">📅 今日课程</h2>
            <button @click="switchView('schedule')"
              class="text-xs font-bold text-violet-500 hover:text-violet-700 transition-colors">
              课表 →
            </button>
          </div>
          <div class="space-y-1.5">
            <div v-for="ev in dayEvents(0)" :key="ev.id"
              class="flex items-center gap-2.5 px-3 py-2 rounded-xl border cursor-pointer hover:shadow-sm transition-all"
              :style="{
                borderColor: (EVENT_COLOR_MAP[ev.colorKey]?.hex || '#8b5cf6') + '30',
                backgroundColor: EVENT_COLOR_MAP[ev.colorKey]?.light || '#ede9fe',
              }"
              @click="switchView('schedule')">
              <div class="w-0.5 self-stretch rounded-full flex-shrink-0"
                :style="{ backgroundColor: EVENT_COLOR_MAP[ev.colorKey]?.hex }"></div>
              <div class="min-w-0">
                <p class="text-xs font-black leading-tight truncate"
                  :style="{ color: EVENT_COLOR_MAP[ev.colorKey]?.text }">{{ ev.subject }}</p>
                <p class="text-xs opacity-60 font-mono"
                  :style="{ color: EVENT_COLOR_MAP[ev.colorKey]?.text }">
                  {{ String(ev.startHour).padStart(2,'0') }}:{{ String(ev.startMin).padStart(2,'0') }} · {{ ev.teacher }}
                </p>
              </div>
            </div>
            <button @click="openAddEvent(0, 16, 0)"
              class="w-full flex items-center justify-center gap-1.5 px-3 py-2 rounded-xl border border-dashed border-slate-200 hover:border-violet-300 text-slate-400 hover:text-violet-500 text-xs font-semibold transition-all">
              + 添加课外课程
            </button>
          </div>
        </div>

      </aside>

    </div>
  </div>

  <!-- ══ Series Preview Modal ══ -->
  <Transition name="modal">
    <div v-if="showSeriesModal && previewSeriesData"
      class="fixed inset-0 z-50 flex items-center justify-center bg-black/40 backdrop-blur-sm p-4"
      @click.self="showSeriesModal = false">
      <div class="bg-white rounded-3xl w-full max-w-md shadow-2xl overflow-hidden">

        <!-- 头部 -->
        <div class="p-6 pb-4 border-b border-slate-100">
          <div class="flex items-start justify-between">
            <div>
              <div class="flex items-center gap-2 mb-2">
                <span class="text-2xl">{{ getCourseEmoji(previewSeriesData.subject) }}</span>
                <span class="text-xs bg-violet-100 text-violet-600 px-2.5 py-1 rounded-full font-bold">
                  {{ previewSeriesData.tag }}
                </span>
                <span class="text-xs bg-slate-100 text-slate-500 px-2 py-0.5 rounded-full font-medium">
                  {{ previewSeriesData.nodes.length }} 个节点
                </span>
              </div>
              <h3 class="text-base font-black text-slate-800">{{ previewSeriesData.title }}</h3>
              <p class="text-xs text-slate-400 mt-0.5">{{ previewSeriesData.teacher }}</p>
            </div>
            <button @click="showSeriesModal = false"
              class="text-slate-400 hover:text-slate-600 text-xl leading-none mt-1 flex-shrink-0">×</button>
          </div>
          <p class="text-xs text-slate-500 mt-3 leading-relaxed bg-slate-50 rounded-2xl p-3">
            {{ previewSeriesData.description }}
          </p>
          <!-- 进度概览（已加入时显示） -->
          <div v-if="isEnrolled(previewSeriesData.id)" class="mt-3 flex items-center gap-3">
            <div class="flex-1 h-2 bg-slate-100 rounded-full overflow-hidden">
              <div class="h-full rounded-full bg-gradient-to-r from-violet-400 to-purple-500 transition-all duration-700"
                :style="{ width: seriesProgress(previewSeriesData) + '%' }"></div>
            </div>
            <span class="text-xs font-black text-violet-600 flex-shrink-0">
              {{ seriesProgress(previewSeriesData) }}% 完成
            </span>
          </div>
        </div>

        <!-- 节点列表 -->
        <div class="px-6 py-4 space-y-0 max-h-64 overflow-y-auto">
          <div v-for="(node, i) in previewSeriesData.nodes" :key="node.id" class="relative">
            <div v-if="i < previewSeriesData.nodes.length - 1"
              class="absolute left-4 top-9 w-px h-4 z-0"
              :style="{ borderLeft: '1.5px dashed', borderColor: node.status==='completed' ? '#c4b5fd' : '#e2e8f0' }">
            </div>
            <div class="relative z-10 flex items-center gap-3 py-2">
              <!-- completed -->
              <div v-if="node.status==='completed'"
                class="w-8 h-8 rounded-xl flex items-center justify-center flex-shrink-0
                       bg-violet-500 shadow-md shadow-violet-200">
                <svg class="w-3.5 h-3.5 text-white" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="3" d="M5 13l4 4L19 7"/>
                </svg>
              </div>
              <!-- in-progress -->
              <div v-else-if="node.status==='in-progress'"
                class="w-8 h-8 rounded-xl flex items-center justify-center flex-shrink-0
                       bg-gradient-to-br from-amber-400 to-orange-500 shadow-md shadow-orange-200">
                <svg class="w-3.5 h-3.5 text-white" fill="currentColor" viewBox="0 0 24 24">
                  <path d="M8 5v14l11-7z"/>
                </svg>
              </div>
              <!-- locked -->
              <div v-else
                class="w-8 h-8 rounded-xl flex items-center justify-center flex-shrink-0 bg-slate-100">
                <svg class="w-3.5 h-3.5 text-amber-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                    d="M12 15v2m-6 4h12a2 2 0 002-2v-6a2 2 0 00-2-2H6a2 2 0 00-2 2v6a2 2 0 002 2zm10-10V7a4 4 0 00-8 0v4h8z"/>
                </svg>
              </div>
              <div class="flex-1 min-w-0">
                <p class="text-sm font-semibold"
                  :class="node.status==='locked' ? 'text-slate-400' : 'text-slate-800'">
                  {{ node.title }}
                </p>
                <p class="text-xs text-slate-400">+{{ node.xp }} XP · {{ node.duration }}</p>
              </div>
              <span v-if="node.status==='in-progress'"
                class="flex-shrink-0 text-xs bg-amber-100 text-amber-600 px-2 py-0.5 rounded-full font-semibold">
                进行中
              </span>
              <span v-else-if="node.status==='completed'"
                class="flex-shrink-0 text-xs text-violet-400 font-bold">✓</span>
            </div>
          </div>
        </div>

        <!-- 底部按钮 -->
        <div class="p-6 pt-4 border-t border-slate-100 flex gap-2">
          <button @click="showSeriesModal = false"
            class="flex-1 py-2.5 bg-slate-100 text-slate-600 font-black rounded-2xl text-sm hover:bg-slate-200 transition-colors">
            关闭
          </button>
          <button @click="enrollSeries(previewSeriesData)"
            class="flex-1 py-2.5 font-black rounded-2xl text-sm transition-all"
            :class="isEnrolled(previewSeriesData.id)
              ? 'bg-emerald-50 text-emerald-600 cursor-default'
              : 'bg-violet-600 text-white hover:bg-violet-700 active:scale-95'">
            {{ isEnrolled(previewSeriesData.id) ? '✓ 已加入学习计划' : '一键加入学习计划' }}
          </button>
        </div>

      </div>
    </div>
  </Transition>

  <!-- ── 屏幕闪光（AI 快照模拟） ── -->
  <Transition name="flash">
    <div v-if="showFlash" class="fixed inset-0 z-[200] bg-white pointer-events-none"></div>
  </Transition>

  <!-- ── AI 笔记 Toast ── -->
  <Transition name="toast-up">
    <div v-if="showNoteToast"
      class="fixed bottom-8 left-1/2 -translate-x-1/2 z-[150]
             flex items-center gap-3 bg-slate-900 text-white
             px-5 py-3.5 rounded-2xl shadow-2xl text-sm font-semibold whitespace-nowrap">
      <span class="text-lg">📸</span>
      AI 已截取当前板书并生成知识摘要，已存入<span class="text-violet-300 font-black">个人中心</span>
    </div>
  </Transition>

  <!-- ── 错题收录 Toast ── -->
  <Transition name="toast-up">
    <div v-if="showMistakeToast"
      class="fixed bottom-8 left-1/2 -translate-x-1/2 z-[150]
             flex items-center gap-3 bg-red-600 text-white
             px-5 py-3.5 rounded-2xl shadow-2xl text-sm font-semibold whitespace-nowrap">
      <span class="text-lg">⚠️</span>
      知识点未掌握，已自动收录至<span class="font-black underline cursor-pointer" @click="profileTab='mistakes'; switchView('profile')">个人中心 · 错题本</span>
    </div>
  </Transition>

  <!-- ── 热力图方格 Toast ── -->
  <Transition name="toast-up">
    <div v-if="showHeatmapToast"
      class="fixed bottom-8 left-1/2 -translate-x-1/2 z-[150]
             flex items-center gap-3 bg-slate-900 text-white
             px-5 py-3.5 rounded-2xl shadow-2xl text-sm font-semibold whitespace-nowrap">
      {{ heatmapToastMsg }}
    </div>
  </Transition>

  <!-- ── 快照命名对话框 ── -->
  <Transition name="modal">
    <div v-if="showSnapshotModal"
      class="fixed inset-0 z-[200] flex items-center justify-center bg-black/40 backdrop-blur-sm p-4"
      @click.self="showSnapshotModal = false">
      <div class="bg-white rounded-3xl w-full max-w-sm shadow-2xl p-6 space-y-5">
        <div class="flex items-center gap-3">
          <div class="w-10 h-10 rounded-2xl bg-gradient-to-br from-violet-500 to-purple-600
                      flex items-center justify-center text-lg shadow-md shadow-violet-200/60 flex-shrink-0">
            📸
          </div>
          <div>
            <p class="text-sm font-black text-slate-800">保存 AI 快照笔记</p>
            <p class="text-xs text-slate-400 mt-0.5">为本次截屏命名后保存</p>
          </div>
        </div>
        <input v-model="snapshotPendingName"
          class="w-full px-4 py-3 bg-slate-50 border-2 border-slate-200 focus:border-violet-400
                 rounded-2xl text-sm font-semibold text-slate-800 focus:outline-none transition-colors"
          placeholder="输入笔记名称…"
          @keydown.enter="confirmSnapshot"/>
        <div class="flex gap-2">
          <button @click="showSnapshotModal = false"
            class="flex-1 py-2.5 bg-slate-100 text-slate-600 font-black rounded-2xl text-sm hover:bg-slate-200 transition-colors">
            取消
          </button>
          <button @click="confirmSnapshot"
            class="flex-1 py-2.5 bg-gradient-to-r from-violet-500 to-purple-600 text-white font-black
                   rounded-2xl text-sm hover:shadow-lg hover:shadow-violet-200/60 active:scale-95 transition-all">
            确认保存
          </button>
        </div>
      </div>
    </div>
  </Transition>

  <!-- ── 笔记详情弹窗 ── -->
  <Transition name="modal">
    <div v-if="showNoteModal && activeNote"
      class="fixed inset-0 z-[200] flex items-center justify-center bg-black/40 backdrop-blur-sm p-6"
      @click.self="showNoteModal = false">
      <div class="bg-white rounded-3xl w-full max-w-lg shadow-2xl overflow-hidden">
        <!-- 截图大图区 -->
        <div class="h-52 relative"
          :class="activeNote.tint==='violet' ? 'bg-gradient-to-br from-violet-100 to-purple-200'
                 : activeNote.tint==='blue'  ? 'bg-gradient-to-br from-blue-100 to-cyan-200'
                 :                             'bg-gradient-to-br from-emerald-100 to-teal-200'">
          <!-- 模拟板书 -->
          <div class="absolute inset-8 opacity-20 space-y-4">
            <div class="h-px bg-current w-3/4"></div>
            <div class="h-px bg-current w-1/2"></div>
            <div class="h-px bg-current w-2/3"></div>
            <div class="h-px bg-current w-2/5"></div>
          </div>
          <button @click="showNoteModal = false"
            class="absolute top-4 right-4 w-8 h-8 rounded-full bg-black/30 hover:bg-black/50
                   flex items-center justify-center text-white transition-colors text-sm font-bold">
            ✕
          </button>
          <div class="absolute bottom-4 left-4 flex items-center gap-2">
            <span class="text-xs font-black bg-white/70 backdrop-blur-sm px-2.5 py-1 rounded-full"
              :class="activeNote.tint==='violet' ? 'text-violet-700'
                     : activeNote.tint==='blue'  ? 'text-blue-700'
                     :                             'text-emerald-700'">
              🤖 AI 快照 · {{ activeNote.time }}
            </span>
          </div>
        </div>
        <!-- 内容区 -->
        <div class="p-6 space-y-4">
          <div>
            <h3 class="text-base font-black text-slate-800">{{ activeNote.title }}</h3>
            <p class="text-xs text-slate-400 mt-1">{{ activeNote.course }} · {{ activeNote.teacher }} · {{ activeNote.createdAt }}</p>
          </div>
          <div class="bg-violet-50 rounded-2xl p-4">
            <p class="text-xs font-black text-violet-500 mb-2">🤖 AI 核心摘要</p>
            <p class="text-sm text-slate-700 leading-relaxed whitespace-pre-line">{{ activeNote.summary }}</p>
          </div>
          <div class="bg-slate-50 rounded-2xl p-4">
            <p class="text-xs font-black text-slate-400 mb-2">✏️ 我的补充</p>
            <p class="text-sm text-slate-500 leading-relaxed">对称轴公式记忆口诀：负 b 除以 2a，顶点纵坐标用判别式……</p>
          </div>
        </div>
      </div>
    </div>
  </Transition>

</template>

<script setup>
import { ref, computed, reactive } from 'vue'

// ── State ──
const currentView = ref('home')
const currentGrade = ref('初中')
const showAIModal = ref(false)
const aiModalType = ref('preview')
const selectedCourse = ref(null)
const leaderboardTab = ref('streak')
const reviewTab = ref('targeted')
const courseTab = ref('basic')
const quizStarted = ref(false)
const randomReviewStarted = ref(false)
const currentQuizIndex = ref(0)
const selectedAnswer = ref(null)
const showExplanation = ref(false)
const searchQuery = ref('')

// ── Focus Mode ──
const focusMode = ref(false)
const showFocusScore = ref(false)
const focusMinutes = ref(45)   // mock: 45 min session
const focusNotes = ref('')

// ── Class Auth ──
const showClassModal = ref(false)
const classCodeInput = ref('')
const className = ref('')
const showJoinSuccess = ref(false)

// ── AI Snapshots ──
const snapshots = ref([])

// ── Notes & Mistakes ──
const showFlash       = ref(false)
const showNoteToast   = ref(false)
const showMistakeToast = ref(false)

const myNotes = ref([
  {
    id: 1, title: '函数单调性的判定定理', subject: '数学', teacher: '张华老师',
    tint: 'violet', time: '10:23', course: '初中数学函数专线',
    summary: '单调递增区间内，对任意 x₁<x₂ 恒有 f(x₁)<f(x₂)。可用导数判断：f\'(x)>0 则递增，f\'(x)<0 则递减。',
    createdAt: '04/20 14:23',
  },
  {
    id: 2, title: 'y=ax²+bx+c 顶点与对称轴', subject: '数学', teacher: '张华老师',
    tint: 'blue', time: '18:45', course: '初中数学函数专线',
    summary: '对称轴 x=−b/(2a)，顶点坐标 (−b/2a, Δ/−4a)，Δ=b²−4ac 决定与 x 轴交点个数。',
    createdAt: '04/21 09:12',
  },
  {
    id: 3, title: '议论文三段式结构要点', subject: '语文', teacher: '王芳老师',
    tint: 'emerald', time: '08:11', course: '语文现代文阅读体系',
    summary: '引论（提出问题）→ 本论（分析问题，3 分论点+例证）→ 结论（解决问题）。逻辑须层层递进。',
    createdAt: '04/21 10:05',
  },
])

const myMistakes = ref([
  {
    id: 1,
    question: '下列哪个是二次函数？',
    options: ['y = 3x + 1', 'y = x² − 4x + 3', 'y = 1/x', 'y = √x'],
    selected: 2, correct: 1,
    subject: '数学', course: '初中数学函数专线', createdAt: '04/20 15:14',
    showAIHint: false,
    aiHint: '你选了 C（y=1/x），是不是忽略了"最高次项"的要求？二次函数定义是最高次为 2 次的多项式。试着把每个选项的最高次项写出来，再比较看看……',
  },
  {
    id: 2,
    question: '函数 y = x² − 2x + 3 的对称轴是？',
    options: ['x = −1', 'x = 1', 'x = 2', 'x = −2'],
    selected: 0, correct: 1,
    subject: '数学', course: '初中数学函数专线', createdAt: '04/21 09:33',
    showAIHint: false,
    aiHint: '你选了 A（x=−1）。注意对称轴公式 x=−b/(2a)，这里 a=1，b=−2，代入算算？负负得正，方向别算反了哦～',
  },
])

// ── User ──
const user = reactive({
  name: '林小萱', avatar: '林', level: 12,
  xp: 2340, xpToNext: 3000, streak: 7, totalStudyDays: 45
})

const grades = ['小学', '初中', '高中', '大学']

const xpPercent = computed(() => Math.round((user.xp / user.xpToNext) * 100))

// ── Nav ──
const navItems = [
  {
    view: 'home', label: '学习主页',
    icon: '<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6"/>'
  },
  {
    view: 'courses', label: '课程中心',
    icon: '<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6.253v13m0-13C10.832 5.477 9.246 5 7.5 5S4.168 5.477 3 6.253v13C4.168 18.477 5.754 18 7.5 18s3.332.477 4.5 1.253m0-13C13.168 5.477 14.754 5 16.5 5c1.747 0 3.332.477 4.5 1.253v13C19.832 18.477 18.247 18 16.5 18c-1.746 0-3.332.477-4.5 1.253"/>'
  },
  {
    view: 'analytics', label: 'AI 成绩看板',
    icon: '<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z"/>'
  },
  {
    view: 'review', label: '复习题库',
    icon: '<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-6 9l2 2 4-4"/>'
  },
  {
    view: 'schedule', label: '我的课表',
    icon: '<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 7V3m8 4V3m-9 8h10M5 21h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v12a2 2 0 002 2z"/>'
  },
  {
    view: 'profile', label: '个人中心',
    icon: '<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z"/>'
  },
  {
    view: 'notes', label: '我的笔记',
    icon: '<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z"/>'
  },
  {
    view: 'mistakes', label: '错题本',
    icon: '<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2"/><line x1="9" y1="12" x2="15" y2="18" stroke-linecap="round" stroke-width="2"/><line x1="15" y1="12" x2="9" y2="18" stroke-linecap="round" stroke-width="2"/>'
  },
]

// ── Daily Tasks ──
const dailyTasks = ref([
  { id: 1, title: '完成今日课程', xp: 30, done: true },
  { id: 2, title: '练习 10 道题', xp: 20, done: true },
  { id: 3, title: 'AI 预习一门课', xp: 25, done: false },
  { id: 4, title: '坚持打卡签到', xp: 15, done: false },
])

function completeTask(task) {
  if (!task.done) {
    task.done = true
    user.xp = Math.min(user.xp + task.xp, user.xpToNext)
  }
}

// ── Leaderboards ──
const streakLeaderboard = [
  { rank: 1, name: '陈子涵', avatar: '陈', days: 32 },
  { rank: 2, name: '王梓轩', avatar: '王', days: 28 },
  { rank: 3, name: '林小萱', avatar: '林', days: 7, isMe: true },
  { rank: 4, name: '张思远', avatar: '张', days: 6 },
  { rank: 5, name: '李晨曦', avatar: '李', days: 5 },
]
const focusLeaderboard = [
  { rank: 1, name: '王梓轩', avatar: '王', hours: 4.2 },
  { rank: 2, name: '赵嘉怡', avatar: '赵', hours: 3.8 },
  { rank: 3, name: '林小萱', avatar: '林', hours: 2.5, isMe: true },
  { rank: 4, name: '陈子涵', avatar: '陈', hours: 2.1 },
  { rank: 5, name: '刘宇航', avatar: '刘', hours: 1.9 },
]
const activeLeaderboard = computed(() =>
  leaderboardTab.value === 'streak' ? streakLeaderboard : focusLeaderboard
)

// ── Course Series ──
const courseSeries = [
  {
    id: 's1', title: '初中数学函数专线', subject: '数学', teacher: '张华老师',
    color: 'purple', tag: '数学专线', rating: 4.9, views: 32840,
    tags: ['体系化', '重难点'], avatar: ['A','B','C'],
    description: '从代数基础到统计概率，5 大关卡系统突破初中数学核心知识体系',
    nodes: [
      { id:'n11', title:'代数基础',     status:'completed',   xp:100, duration:'25 分钟', courseData:{ tags:['代数','基础'],    rating:4.9, views:8200, progress:100 } },
      { id:'n12', title:'方程与不等式', status:'completed',   xp:120, duration:'30 分钟', courseData:{ tags:['方程','不等式'], rating:4.8, views:7100, progress:100 } },
      { id:'n13', title:'函数初步',     status:'in-progress', xp:150, duration:'35 分钟', courseData:{ tags:['函数','图像'],   rating:4.9, views:6800, progress:38  } },
      { id:'n14', title:'几何证明',     status:'locked',      xp:180, duration:'40 分钟', courseData:{ tags:['几何','证明'],   rating:4.7, views:5300, progress:0   } },
      { id:'n15', title:'统计与概率',   status:'locked',      xp:200, duration:'30 分钟', courseData:{ tags:['统计','概率'],   rating:4.8, views:4900, progress:0   } },
    ],
  },
  {
    id: 's2', title: '英语语法攻关体系', subject: '英语', teacher: '李明老师',
    color: 'blue', tag: '英语专线', rating: 4.8, views: 25600,
    tags: ['语法', '词汇'], avatar: ['G','H','I'],
    description: '词法→句法→篇章，层层递进系统掌握初中英语语法全貌',
    nodes: [
      { id:'n21', title:'词性与词法', status:'completed',   xp:80,  duration:'20 分钟', courseData:{ tags:['词法','词汇'],  rating:4.8, views:6500, progress:100 } },
      { id:'n22', title:'时态与语态', status:'in-progress', xp:120, duration:'30 分钟', courseData:{ tags:['时态','语态'],  rating:4.7, views:5900, progress:52  } },
      { id:'n23', title:'从句精讲',   status:'locked',      xp:150, duration:'35 分钟', courseData:{ tags:['从句','语法'],  rating:4.8, views:4800, progress:0   } },
      { id:'n24', title:'写作应用',   status:'locked',      xp:180, duration:'40 分钟', courseData:{ tags:['写作','应用'],  rating:4.7, views:3900, progress:0   } },
    ],
  },
  {
    id: 's3', title: '语文现代文阅读体系', subject: '语文', teacher: '王芳老师',
    color: 'green', tag: '语文专线', rating: 4.8, views: 28100,
    tags: ['阅读', '写作'], avatar: ['D','E','F'],
    description: '精读方法论 + 题型模板，5 关递进直击阅读理解满分策略',
    nodes: [
      { id:'n31', title:'阅读方法论',   status:'completed',   xp:80,  duration:'20 分钟', courseData:{ tags:['阅读','方法'],  rating:4.9, views:9200, progress:100 } },
      { id:'n32', title:'记叙文精讲',   status:'completed',   xp:100, duration:'25 分钟', courseData:{ tags:['记叙文','解析'], rating:4.8, views:8100, progress:100 } },
      { id:'n33', title:'说明文技巧',   status:'completed',   xp:120, duration:'25 分钟', courseData:{ tags:['说明文','技巧'], rating:4.8, views:7600, progress:100 } },
      { id:'n34', title:'议论文解析',   status:'in-progress', xp:150, duration:'30 分钟', courseData:{ tags:['议论文','解析'], rating:4.9, views:7200, progress:45  } },
      { id:'n35', title:'写作综合训练', status:'locked',      xp:200, duration:'40 分钟', courseData:{ tags:['写作','综合'],  rating:4.8, views:5100, progress:0   } },
    ],
  },
]

// ── Enrollment State ──
// 单体课 ids（basic + interest 统一用 type:'basic' 标记）
const myEnrolledBasicIds = ref([201, 202])
// 套系 ids
const myEnrolledSeriesIds = ref(['s1', 's3'])

// _allBasicMap 在 allCourses 定义后填充（见下方）
const _allBasicMap = {}

// 合并后的已加入列表（每项含 type 字段）
const myEnrolledCourses = computed(() => {
  const basics = myEnrolledBasicIds.value
    .map(id => _allBasicMap[id])
    .filter(Boolean)
    .map(c => ({ ...c, type: 'basic' }))
  const series = courseSeries
    .filter(s => myEnrolledSeriesIds.value.includes(s.id))
    .map(s => ({ ...s, type: 'series' }))
  return [...basics, ...series]
})

// 当前闯关图显示的套系
const activeSeriesId = ref('s1')
const activeSeriesData = computed(() =>
  courseSeries.find(s => s.id === activeSeriesId.value) ?? courseSeries[0]
)
const seriesSelectStyle = {
  backgroundImage: "url(\"data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='none' viewBox='0 0 24 24' stroke='%237c3aed' stroke-width='2.5'%3E%3Cpath stroke-linecap='round' stroke-linejoin='round' d='M19 9l-7 7-7-7'%2F%3E%3C%2Fsvg%3E\")",
  backgroundRepeat: 'no-repeat',
  backgroundPosition: 'right 8px center',
  backgroundSize: '12px 12px',
}
function cycleActiveSeries() {
  const enrolled = myEnrolledCourses.value
  if (enrolled.length < 2) return
  const idx = enrolled.findIndex(s => s.id === activeSeriesId.value)
  activeSeriesId.value = enrolled[(idx + 1) % enrolled.length].id
}

function seriesProgress(c) {
  if (c.type === 'basic' || c.type === 'series-node' || !c.nodes) return c.progress ?? 0
  const done = c.nodes.filter(n => n.status === 'completed').length
  return Math.round(done / c.nodes.length * 100)
}
function continueSeriesNode(c) {
  if (c.type === 'series-node') {
    openVideoPlayer({ ...c._series, title: `${c.seriesTitle} · ${c.title}` }); return
  }
  if (c.type === 'basic' || !c.nodes) { openVideoPlayer(c); return }
  const node = c.nodes.find(n => n.status === 'in-progress') ?? c.nodes[0]
  openVideoPlayer({ ...c, title: `${c.title} · ${node.title}` })
}

// 套系 enrollment
function isEnrolled(seriesId) {
  return myEnrolledSeriesIds.value.includes(seriesId)
}
function enrollSeries(series) {
  if (isEnrolled(series.id)) return
  myEnrolledSeriesIds.value.push(series.id)
  activeSeriesId.value = series.id
}

// 单体课 enrollment
function isBasicEnrolled(courseId) {
  return myEnrolledBasicIds.value.includes(courseId)
}
function enrollBasicCourse(course) {
  if (isBasicEnrolled(course.id)) return
  myEnrolledBasicIds.value.push(course.id)
}

// 个人中心课程 tab (legacy, kept for profileEnrolledList computed)
const profileCourseTab = ref('basic')

// ── 个人中心新版 profileTab (3-tab: courses | notes | mistakes) ──
const profileTab = ref('courses')

// 热力图：最近 84 天 mock 数据（确定性伪随机，刷新不变）
const heatmapData = (() => {
  const cells = []
  const today = new Date()
  for (let i = 83; i >= 0; i--) {
    const d = new Date(today)
    d.setDate(d.getDate() - i)
    const MM = String(d.getMonth() + 1).padStart(2, '0')
    const DD = String(d.getDate()).padStart(2, '0')
    const dateStr = `${MM}/${DD}`
    const absR = Math.abs((Math.sin(i * 127.1 + 311.7) * 43758.5453) % 1)
    const status = absR < 0.28 ? 'mistake' : absR < 0.62 ? 'perfect' : 'empty'
    cells.push({ dateStr, fullDate: `${d.getMonth() + 1}月${d.getDate()}日`, status })
  }
  return cells
})()

// 热力图：每列（周）的月份标签
const heatmapWeekLabels = (() => {
  const labels = []
  for (let w = 0; w < 12; w++) {
    const cell = heatmapData[w * 7]
    const m = cell.fullDate.split('月')[0]
    const prev = w > 0 ? heatmapData[(w - 1) * 7].fullDate.split('月')[0] : null
    labels.push(m !== prev ? m + '月' : '')
  }
  return labels
})()

const heatmapFilterDate = ref(null)  // null = 全部，string = 仅当日

const showHeatmapToast = ref(false)
const heatmapToastMsg  = ref('')

function clickCell(cell) {
  if (cell.status === 'mistake') {
    heatmapFilterDate.value = cell.dateStr
    profileTab.value = 'mistakes'
  } else {
    heatmapToastMsg.value = cell.status === 'perfect'
      ? '🎉 当天的作业完成的很棒，再接再厉！'
      : '📭 当天没有做题哦，开始新的练习吧！'
    showHeatmapToast.value = true
    setTimeout(() => { showHeatmapToast.value = false }, 3000)
  }
}

// 错题按热力图日期过滤
const profileMistakesFiltered = computed(() =>
  heatmapFilterDate.value
    ? myMistakes.value.filter(m => m.createdAt.startsWith(heatmapFilterDate.value))
    : myMistakes.value
)

// 首页：单体基础课 + 每个已加入套系课当前 in-progress 节点（展开为独立卡片）
const homeEnrolledCourses = computed(() => {
  const basics = myEnrolledCourses.value.filter(c => c.type === 'basic')
  const seriesNodes = myEnrolledCourses.value
    .filter(c => c.type === 'series')
    .map(series => {
      const activeNode = series.nodes?.find(n => n.status === 'in-progress')
      if (!activeNode) return null
      return {
        id: `${series.id}_active`,
        title: activeNode.title,
        subject: series.subject,
        teacher: series.teacher,
        color: series.color,
        type: 'series-node',
        progress: activeNode.courseData?.progress ?? 35,
        tags: activeNode.courseData?.tags ?? [],
        seriesTitle: series.title,
        nodeXp: activeNode.xp,
        nodeDuration: activeNode.duration,
        _series: series,
        _node: activeNode,
      }
    })
    .filter(Boolean)
  return [...basics, ...seriesNodes]
})

// 个人中心按 tab 过滤
const profileEnrolledList = computed(() =>
  myEnrolledCourses.value.filter(c => c.type === profileCourseTab.value)
)

// 首页"查看全部 →" 跳转个人中心并定位到基础课 tab
function goToProfileBasic() {
  profileCourseTab.value = 'basic'
  switchView('profile')
}

// 套系预览弹窗
const showSeriesModal  = ref(false)
const previewSeriesId  = ref(null)
const previewSeriesData = computed(() =>
  courseSeries.find(s => s.id === previewSeriesId.value) ?? null
)
function openSeriesModal(series) {
  previewSeriesId.value = series.id
  showSeriesModal.value = true
}

// ── Calendar ──
const EVENT_COLOR_MAP = {
  math:    { hex: '#7c3aed', light: '#ede9fe', text: '#5b21b6' },
  chinese: { hex: '#059669', light: '#d1fae5', text: '#065f46' },
  english: { hex: '#2563eb', light: '#dbeafe', text: '#1e40af' },
  physics: { hex: '#d97706', light: '#fef3c7', text: '#92400e' },
  chem:    { hex: '#ea580c', light: '#ffedd5', text: '#9a3412' },
  bio:     { hex: '#0d9488', light: '#ccfbf1', text: '#134e4a' },
  history: { hex: '#dc2626', light: '#fee2e2', text: '#991b1b' },
  geo:     { hex: '#16a34a', light: '#dcfce7', text: '#14532d' },
  pe:      { hex: '#65a30d', light: '#ecfccb', text: '#3f6212' },
  music:   { hex: '#c026d3', light: '#fae8ff', text: '#701a75' },
  extra:   { hex: '#7c3aed', light: '#f5f3ff', text: '#4c1d95' },
}

let _nextEventId = 100
const scheduleEvents = ref([
  { id: _nextEventId++, day: 0, startHour: 8,  startMin: 0,  duration: 45, subject: '数学', teacher: '张华老师', type: 'school', colorKey: 'math' },
  { id: _nextEventId++, day: 0, startHour: 10, startMin: 0,  duration: 45, subject: '语文', teacher: '王芳老师', type: 'school', colorKey: 'chinese' },
  { id: _nextEventId++, day: 0, startHour: 14, startMin: 0,  duration: 45, subject: '英语', teacher: '李明老师', type: 'school', colorKey: 'english' },
  { id: _nextEventId++, day: 1, startHour: 8,  startMin: 0,  duration: 45, subject: '物理', teacher: '赵辉老师', type: 'school', colorKey: 'physics' },
  { id: _nextEventId++, day: 1, startHour: 10, startMin: 0,  duration: 45, subject: '数学', teacher: '张华老师', type: 'school', colorKey: 'math' },
  { id: _nextEventId++, day: 1, startHour: 15, startMin: 0,  duration: 45, subject: '体育', teacher: '刘辉老师', type: 'school', colorKey: 'pe' },
  { id: _nextEventId++, day: 2, startHour: 8,  startMin: 0,  duration: 45, subject: '化学', teacher: '陈蕾老师', type: 'school', colorKey: 'chem' },
  { id: _nextEventId++, day: 2, startHour: 10, startMin: 0,  duration: 45, subject: '英语', teacher: '李明老师', type: 'school', colorKey: 'english' },
  { id: _nextEventId++, day: 2, startHour: 14, startMin: 0,  duration: 45, subject: '历史', teacher: '吴天浩老师', type: 'school', colorKey: 'history' },
  { id: _nextEventId++, day: 3, startHour: 8,  startMin: 0,  duration: 45, subject: '语文', teacher: '王芳老师', type: 'school', colorKey: 'chinese' },
  { id: _nextEventId++, day: 3, startHour: 10, startMin: 0,  duration: 45, subject: '生物', teacher: '周杰老师', type: 'school', colorKey: 'bio' },
  { id: _nextEventId++, day: 3, startHour: 14, startMin: 0,  duration: 45, subject: '数学', teacher: '张华老师', type: 'school', colorKey: 'math' },
  { id: _nextEventId++, day: 4, startHour: 8,  startMin: 0,  duration: 45, subject: '物理', teacher: '赵辉老师', type: 'school', colorKey: 'physics' },
  { id: _nextEventId++, day: 4, startHour: 10, startMin: 0,  duration: 45, subject: '政治', teacher: '林峰老师', type: 'school', colorKey: 'geo' },
  { id: _nextEventId++, day: 4, startHour: 14, startMin: 0,  duration: 90, subject: '自习', teacher: '',         type: 'school', colorKey: 'extra' },
])

const CAL_START_HOUR = 7
const CAL_PX_PER_HOUR = 64
const calDayLabels = ['周一', '周二', '周三', '周四', '周五', '周六', '周日']
const calDates     = ['4/21', '4/22', '4/23', '4/24', '4/25', '4/26', '4/27']
const calHours     = Array.from({ length: 15 }, (_, i) => i + CAL_START_HOUR) // 7–21

const showAddEventModal = ref(false)
const selectedEvent     = ref(null)
const addEventForm = reactive({
  day: 0, startHour: 9, startMin: 0, duration: 45, subject: '', teacher: '', colorKey: 'extra',
})

function evTop(ev) {
  return ((ev.startHour - CAL_START_HOUR) * 60 + ev.startMin) * (CAL_PX_PER_HOUR / 60)
}
function evHeight(ev) {
  return Math.max(22, ev.duration * (CAL_PX_PER_HOUR / 60))
}
function dayEvents(dayIndex) {
  return scheduleEvents.value
    .filter(ev => ev.day === dayIndex)
    .sort((a, b) => a.startHour * 60 + a.startMin - (b.startHour * 60 + b.startMin))
}
function handleColClick(dayIndex, e) {
  const rect = e.currentTarget.getBoundingClientRect()
  const y = e.clientY - rect.top
  const totalMin = Math.round((y / CAL_PX_PER_HOUR) * 60) + CAL_START_HOUR * 60
  const h = Math.floor(totalMin / 60)
  const m = Math.round((totalMin % 60) / 15) * 15
  openAddEvent(dayIndex, h, m)
}
function openAddEvent(day, hour, min) {
  addEventForm.day       = day
  addEventForm.startHour = Math.min(21, Math.max(CAL_START_HOUR, hour))
  addEventForm.startMin  = min % 60
  addEventForm.duration  = 45
  addEventForm.subject   = ''
  addEventForm.teacher   = ''
  addEventForm.colorKey  = 'extra'
  selectedEvent.value    = null
  showAddEventModal.value = true
}
function addEvent() {
  if (!addEventForm.subject.trim()) return
  scheduleEvents.value.push({
    id: _nextEventId++,
    day: addEventForm.day,
    startHour: addEventForm.startHour,
    startMin: addEventForm.startMin,
    duration: addEventForm.duration,
    subject: addEventForm.subject,
    teacher: addEventForm.teacher,
    type: 'extra',
    colorKey: addEventForm.colorKey,
  })
  showAddEventModal.value = false
}
function deleteEvent(id) {
  scheduleEvents.value = scheduleEvents.value.filter(ev => ev.id !== id)
  showAddEventModal.value = false
  selectedEvent.value = null
}
function openEventDetail(ev) {
  selectedEvent.value = ev
  showAddEventModal.value = true
}

// ── Diagnostics ──
const diagMetrics = [
  { label: '知识留存率', value: '78%', color: 'text-violet-600', trend: 5, unit: '%' },
  { label: '本周错题率', value: '12%', color: 'text-amber-500', trend: -3, unit: '%' },
  { label: '专注总时长', value: '8.4h', color: 'text-blue-600', trend: 1.2, unit: 'h' },
  { label: '综合学情', value: '优秀', color: 'text-emerald-600', trend: 1, unit: '档' },
]

const diagDetails = [
  { name: '代数方程', score: 94 },
  { name: '一次函数', score: 88 },
  { name: '数列规律', score: 76 },
  { name: '平面几何', score: 68 },
  { name: '立体几何', score: 52 },
  { name: '统计概率', score: 80 },
]

const microCourses = [
  { id: 1, emoji: '📐', title: '3 分钟搞定立体几何辅助线', teacher: '张华老师', duration: '3分钟', tag: '🔥 急需', urgent: true },
  { id: 2, emoji: '🎯', title: '体积公式推导全解析', teacher: '赵辉老师', duration: '8分钟', tag: '薄弱点', urgent: true },
  { id: 3, emoji: '✨', title: '空间想象力特训营（图解版）', teacher: '李明老师', duration: '15分钟', tag: '进阶', urgent: false },
]

// ── Home Stats ──
// 返回"位于班级前 xx%"：全班最高时为 0%，否则用名次/总人数估算
function classPct(myVal, dist) {
  const above = dist.filter(v => v > myVal).length
  return above === 0 ? 0 : Math.round((above + 1) / (dist.length + 1) * 100)
}
const _daysDist   = [12,18,22,25,28,30,33,36,40,42,44,47,50,55,60,65]
const _levelDist  = [4,5,6,7,7,8,8,9,9,10,10,11,11,12,13,14]
const _streakDist = [32,28,6,5]
const _courseDist = [1,1,1,2,2,2,2,3,3,3,3,4,4,4,5,6]

const homeStats = computed(() => [
  { value: user.totalStudyDays, label: '累计打卡天数', color: 'text-violet-600', pct: classPct(user.totalStudyDays, _daysDist) },
  { value: `Lv ${user.level}`,  label: '当前等级',     color: 'text-emerald-500', pct: classPct(user.level, _levelDist) },
  { value: user.streak,         label: '当前连胜天数', color: 'text-orange-500', pct: classPct(user.streak, _streakDist) },
  { value: myEnrolledCourses.value.length, label: '在学课程数', color: 'text-blue-500', pct: classPct(myEnrolledCourses.value.length, _courseDist) },
])

// ── Course Data ──
const courseTabs = [
  { value: 'basic',    label: '📚 基础学科' },
  { value: 'interest', label: '🌟 兴趣拓展' },
  { value: 'series',   label: '🗺️ 体系化套系课' },
]
const reviewTabs = [
  { value: 'targeted', label: '📌 针对性复习' },
  { value: 'smart', label: '🧠 智能抽查（艾宾浩斯）' },
]

const allCourses = {
  '小学': {
    basic: [
      { id: 101, title: '小学数学图解思维课', subject: '数学', teacher: '刘慧老师', rating: 4.9, views: 15420, tags: ['基础', '趣味'], color: 'purple', avatar: ['赵','钱','孙'], progress: 30 },
      { id: 102, title: '语文字词大闯关', subject: '语文', teacher: '陈婷老师', rating: 4.8, views: 12300, tags: ['识字','阅读'], color: 'green', avatar: ['李','周','吴'], progress: 55 },
      { id: 103, title: '英语启蒙会话训练', subject: '英语', teacher: 'Ms. Wang', rating: 4.7, views: 9800, tags: ['口语','趣味'], color: 'blue', avatar: ['郑','王','冯'], progress: 20 },
    ],
    interest: [
      { id: 104, title: '儿童创意绘画营', subject: '美术', teacher: '赵敏老师', rating: 4.9, views: 8900, tags: ['绘画','创意'], color: 'orange', avatar: ['陈','楚','魏'], progress: 15 },
      { id: 105, title: '趣味编程入门（Scratch）', subject: '编程', teacher: '李涛老师', rating: 4.8, views: 7600, tags: ['Scratch','逻辑'], color: 'teal', avatar: ['蒋','沈','韩'], progress: 0 },
    ]
  },
  '初中': {
    basic: [
      { id: 201, title: '初中数学函数专题精讲', subject: '数学', teacher: '张华老师', rating: 4.9, views: 32840, tags: ['重难点','突破'], color: 'purple', avatar: ['A','B','C'], progress: 65 },
      { id: 202, title: '语文现代文阅读技巧', subject: '语文', teacher: '王芳老师', rating: 4.8, views: 28100, tags: ['阅读','写作'], color: 'green', avatar: ['D','E','F'], progress: 40 },
      { id: 203, title: '英语语法全解析', subject: '英语', teacher: '李明老师', rating: 4.7, views: 25600, tags: ['语法','词汇'], color: 'blue', avatar: ['G','H','I'], progress: 80 },
    ],
    interest: [
      { id: 204, title: '历史文化探秘', subject: '历史', teacher: '吴天浩老师', rating: 4.6, views: 18200, tags: ['通史','趣闻'], color: 'orange', avatar: ['J','K','L'], progress: 25 },
      { id: 205, title: '科学实验探究', subject: '科学', teacher: '赵辉老师', rating: 4.8, views: 22000, tags: ['实验','探究'], color: 'teal', avatar: ['M','N','O'], progress: 10 },
    ]
  },
}
allCourses['高中'] = allCourses['初中']
allCourses['大学'] = allCourses['初中']

// 填充全量单体课平铺索引（allCourses 定义完成后执行）
Object.values(allCourses).forEach(grade => {
  ;(grade.basic    || []).forEach(c => { _allBasicMap[c.id] = c })
  ;(grade.interest || []).forEach(c => { _allBasicMap[c.id] = c })
})

const currentCourses = computed(() => allCourses[currentGrade.value] || allCourses['初中'])

function getCourseEmoji(subject) {
  return { '数学':'📐', '语文':'📖', '英语':'🌏', '历史':'🏛️', '科学':'🔬', '美术':'🎨', '编程':'💻' }[subject] || '📚'
}
function formatNumber(n) {
  if (n >= 10000) return (n / 10000).toFixed(1) + 'w'
  if (n >= 1000) return (n / 1000).toFixed(1) + 'k'
  return String(n)
}

// ── AI Modal ──
const aiModalContent = {
  preview: {
    title: 'AI 预习导读', icon: '🔮',
    content: [
      { type: 'summary', text: '本节课深入探讨一次函数与二次函数的核心概念，重点掌握图像变换规律及实际应用场景。建议预留 25 分钟，带着问题进入课程。' },
      { type: 'keyPoints', items: ['函数的定义域与值域', '一次函数的斜率与截距', '二次函数顶点公式', '图像平移与对称变换'] },
      { type: 'tip', text: '建议先复习初一的坐标系知识，理解象限划分，有助于快速理解本节函数图像变换。' },
    ]
  },
  summary: {
    title: 'AI 智能总结', icon: '✨',
    content: [
      { type: 'summary', text: '通过本节课的学习，你将系统掌握函数知识体系，建立从代数表达式到几何图像的映射思维。' },
      { type: 'keyPoints', items: ['y=kx+b 中 k>0 图像过一、三象限', 'y=ax²+bx+c 对称轴 x=-b/(2a)', '配方法求顶点坐标', '判别式 Δ 判断交点数量'] },
      { type: 'tip', text: 'AI 已整理本课重点公式卡片。点击「加入复习计划」可自动添加到错题本，下次复习时优先推送。' },
    ]
  },
  deepread: {
    title: 'AI 精读解析', icon: '🧠',
    content: [
      { type: 'summary', text: '精读模式下，AI 逐层拆解知识难点，关联高频考点与易错陷阱，帮助你构建深度认知。' },
      { type: 'analysis', items: [
        { title: '难点①：图像平移方向与符号的反直觉关系', desc: 'y=f(x+h) 中，h>0 时图像向左移 h 个单位（而非右移），这与直觉相反，是最常见的失分点。', level: 'hard' },
        { title: '难点②：开口方向与最值判断', desc: '当 a>0 时开口向上，顶点处取得最小值；a<0 时开口向下取最大值。注意题目问"最大"还是"最小"。', level: 'medium' },
        { title: '高频联考考点', desc: '近三年统考中约 42% 的代数大题涉及二次函数，配方法和判别式是必考知识点，务必熟练掌握。', level: 'exam' },
      ]},
      { type: 'tip', text: '完成精读后，建议立即进行「课后限时自测」——趁记忆最鲜活的时刻检验理解深度。' },
    ]
  }
}

const currentAIContent = computed(() => aiModalContent[aiModalType.value] || null)

function openAIModal(type, course) {
  aiModalType.value = type
  selectedCourse.value = course
  showAIModal.value = true
}
function closeAIModal() { showAIModal.value = false }

// ── Navigation ──
function switchView(view) {
  currentView.value = view
  resetQuiz()
}

function openVideoPlayer(course) {
  selectedCourse.value = course
  currentView.value = 'player'
  resetQuiz()
}

function goBack() {
  currentView.value = 'courses'
  resetQuiz()
}

// ── Quiz ──
const quizQuestions = [
  {
    question: '已知函数 f(x) = 2x + 3，当 x = 5 时，f(x) 的值为？',
    options: ['10', '13', '11', '15'],
    correct: 1,
    explanation: '将 x=5 代入：f(5) = 2×5 + 3 = 10 + 3 = 13，故选 B。'
  },
  {
    question: '下列哪个是二次函数？',
    options: ['y = 3x + 1', 'y = x² - 4x + 3', 'y = 1/x', 'y = √x'],
    correct: 1,
    explanation: '二次函数最高次项为 x²，只有 y = x² - 4x + 3 满足，故选 B。'
  },
  {
    question: '函数 y = x² - 2x + 3 的对称轴是？',
    options: ['x = -1', 'x = 1', 'x = 2', 'x = -2'],
    correct: 1,
    explanation: '由对称轴公式 x = -b/(2a) = -(-2)/(2×1) = 1，故选 B。'
  }
]

function startQuiz() { quizStarted.value = true }
function startRandomReview() {
  randomReviewStarted.value = true
  currentQuizIndex.value = 0
  selectedAnswer.value = null
  showExplanation.value = false
}
function resetQuiz() {
  quizStarted.value = false
  randomReviewStarted.value = false
  currentQuizIndex.value = 0
  selectedAnswer.value = null
  showExplanation.value = false
}
function selectAnswer(i) {
  if (selectedAnswer.value !== null) return
  selectedAnswer.value = i
  showExplanation.value = true
  const q = quizQuestions[currentQuizIndex.value]
  if (i !== q.correct) {
    const already = myMistakes.value.find(m => m.question === q.question)
    if (!already) {
      const labels = ['A','B','C','D']
      myMistakes.value.unshift({
        id: Date.now(),
        question: q.question,
        options: q.options,
        selected: i, correct: q.correct,
        subject: selectedCourse.value?.subject || '数学',
        course: selectedCourse.value?.title || '函数专题精讲',
        createdAt: (() => { const d = new Date(); return `${String(d.getMonth()+1).padStart(2,'0')}/${String(d.getDate()).padStart(2,'0')} ${String(d.getHours()).padStart(2,'0')}:${String(d.getMinutes()).padStart(2,'0')}` })(),
        showAIHint: false,
        aiHint: `你选了 ${labels[i]}，思路上可能忽略了某个关键条件。试着重新审题，把 ${labels[q.correct]} 选项的逻辑逐步推导一遍，看看哪一步想法不同？`,
      })
    }
    showMistakeToast.value = true
    setTimeout(() => { showMistakeToast.value = false }, 3500)
  }
}
function nextQuestion() {
  if (currentQuizIndex.value < quizQuestions.length - 1) {
    currentQuizIndex.value++
    selectedAnswer.value = null
    showExplanation.value = false
  }
}
function getOptionClass(i) {
  if (selectedAnswer.value === null)
    return 'border-slate-200 bg-white hover:border-violet-200 hover:bg-violet-50 text-slate-700'
  if (i === quizQuestions[currentQuizIndex.value].correct)
    return 'border-emerald-300 bg-emerald-50 text-emerald-700'
  if (i === selectedAnswer.value)
    return 'border-red-300 bg-red-50 text-red-700'
  return 'border-slate-100 bg-slate-50 text-slate-400'
}

// ── Focus Mode ──
function toggleFocusMode() {
  if (focusMode.value) {
    focusMode.value = false
    showFocusScore.value = true
    setTimeout(() => { showFocusScore.value = false }, 4500)
  } else {
    focusMode.value = true
    focusMinutes.value = 45   // mock: already been focused 45 min
  }
}

// ── Class Auth ──
function joinClass() {
  if (!classCodeInput.value.trim()) return
  showJoinSuccess.value = true
  setTimeout(() => {
    const code = classCodeInput.value.toUpperCase()
    className.value = code.includes('802') ? '初三(2)班'
      : code.includes('701') ? '初二(1)班'
      : code.includes('901') ? '初三毕业班'
      : '高二(3)班'
    showJoinSuccess.value = false
    showClassModal.value = false
    classCodeInput.value = ''
  }, 1800)
}

// ── AI Snapshot ──
const snapshotNotes = [
  '• y=ax²+bx+c 对称轴：x = -b/(2a)\n• 顶点：(-b/2a, Δ/-4a)\n• Δ=b²-4ac 判断根的个数',
  '• 一次函数 y=kx+b：k 为斜率，b 为截距\n• k>0 图像从左下到右上\n• k<0 图像从左上到右下',
  '• 图像平移口诀：左加右减，上加下减\n• y=f(x+h)+k：先移轴再上下\n• 变换顺序影响最终结果',
]
const noteTitles = ['函数图像的平移变换规律', '一次函数斜率与图像关系', '二次函数顶点公式推导']

// ── 时间戳格式化 ──
function formatNow() {
  const d = new Date()
  const MM = String(d.getMonth() + 1).padStart(2, '0')
  const DD = String(d.getDate()).padStart(2, '0')
  const HH = String(d.getHours()).padStart(2, '0')
  const mm = String(d.getMinutes()).padStart(2, '0')
  return `${MM}${DD}_${HH}${mm}`
}

// ── 笔记视图检索状态 ──
const notesSearch   = ref('')
const notesTag      = ref('全部')
const noteTagList   = ['全部', '数学', '英语', '语文', '重点']
const filteredNotes = computed(() => {
  let list = myNotes.value
  if (notesTag.value !== '全部') list = list.filter(n => n.subject === notesTag.value)
  const q = notesSearch.value.trim().toLowerCase()
  if (q) list = list.filter(n => n.title.toLowerCase().includes(q) || n.summary.toLowerCase().includes(q))
  return list
})

// 笔记详情弹窗
const showNoteModal = ref(false)
const activeNote    = ref(null)
function openNote(note) { activeNote.value = note; showNoteModal.value = true }

// ── 错题视图检索状态 ──
const mistakesSearch   = ref('')
const mistakesTag      = ref('全部')
const mistakeTagList   = ['全部', '数学', '英语', '语文']
const filteredMistakes = computed(() => {
  let list = myMistakes.value
  if (mistakesTag.value !== '全部') list = list.filter(m => m.subject === mistakesTag.value)
  const q = mistakesSearch.value.trim().toLowerCase()
  if (q) list = list.filter(m => m.question.toLowerCase().includes(q) || m.course.toLowerCase().includes(q))
  return list
})

// ── 快照命名弹窗状态 ──
const showSnapshotModal   = ref(false)
const snapshotPendingName = ref('')
let _snapshotPending = null

function takeSnapshot() {
  const tints = ['violet', 'blue', 'emerald']
  const n = snapshots.value.length
  const timeStr = '10:' + String(23 + n * 3).padStart(2, '0')
  _snapshotPending = { n, tint: tints[n % tints.length], timeStr }
  snapshotPendingName.value = `课程截屏_${formatNow()}`
  showSnapshotModal.value = true
}

function confirmSnapshot() {
  const { n, tint, timeStr } = _snapshotPending
  const entry = {
    id: Date.now(),
    time: timeStr,
    course: selectedCourse.value?.title || '函数专题精讲',
    note: snapshotNotes[n % snapshotNotes.length],
    tint,
  }
  snapshots.value.unshift(entry)
  const now = new Date()
  myNotes.value.unshift({
    id: entry.id + 1,
    title: snapshotPendingName.value || noteTitles[n % noteTitles.length],
    subject: selectedCourse.value?.subject || '数学',
    teacher: selectedCourse.value?.teacher || '张华老师',
    tint,
    time: timeStr,
    course: entry.course,
    summary: entry.note,
    createdAt: `${String(now.getMonth()+1).padStart(2,'0')}/${String(now.getDate()).padStart(2,'0')} ${String(now.getHours()).padStart(2,'0')}:${String(now.getMinutes()).padStart(2,'0')}`,
  })
  showSnapshotModal.value = false
  showFlash.value = true
  setTimeout(() => { showFlash.value = false }, 350)
  showNoteToast.value = true
  setTimeout(() => { showNoteToast.value = false }, 3000)
}
</script>

<style>
* { box-sizing: border-box; }
main::-webkit-scrollbar,
aside::-webkit-scrollbar { display: none; }
body { margin: 0; }

/* ── Transitions ── */
.modal-enter-active, .modal-leave-active { transition: opacity 0.2s ease; }
.modal-enter-from, .modal-leave-to { opacity: 0; }

.flash-enter-active { transition: opacity 0.05s ease; }
.flash-leave-active { transition: opacity 0.3s ease; }
.flash-enter-from, .flash-leave-to { opacity: 0; }

.toast-up-enter-active { transition: opacity 0.3s ease, transform 0.35s cubic-bezier(0.34,1.56,0.64,1); }
.toast-up-leave-active { transition: opacity 0.2s ease, transform 0.2s ease; }
.toast-up-enter-from   { opacity: 0; transform: translateX(-50%) translateY(16px); }
.toast-up-leave-to     { opacity: 0; transform: translateX(-50%) translateY(8px); }

.focus-enter-active { transition: opacity 0.4s ease, transform 0.4s ease; }
.focus-leave-active { transition: opacity 0.3s ease, transform 0.3s ease; }
.focus-enter-from  { opacity: 0; transform: scale(1.03); }
.focus-leave-to    { opacity: 0; transform: scale(0.97); }

.slide-up-enter-active { transition: opacity 0.35s ease, transform 0.35s cubic-bezier(0.34,1.56,0.64,1); }
.slide-up-leave-active { transition: opacity 0.2s ease, transform 0.2s ease; }
.slide-up-enter-from   { opacity: 0; transform: translateY(20px); }
.slide-up-leave-to     { opacity: 0; transform: translateY(10px); }

.badge-enter-active { transition: opacity 0.3s ease, transform 0.3s cubic-bezier(0.34,1.56,0.64,1); }
.badge-leave-active { transition: opacity 0.2s ease, transform 0.15s ease; }
.badge-enter-from   { opacity: 0; transform: scale(0) rotate(-10deg); }
.badge-leave-to     { opacity: 0; transform: scale(0); }

/* ── Scrollbar ── */
::-webkit-scrollbar { width: 4px; height: 4px; }
::-webkit-scrollbar-track { background: transparent; }
::-webkit-scrollbar-thumb { background: #e2e8f0; border-radius: 4px; }
::-webkit-scrollbar-thumb:hover { background: #cbd5e1; }

/* ── Focus Mode Stars ── */
.star {
  position: absolute;
  background: white;
  border-radius: 50%;
  animation: twinkle 3s ease-in-out infinite;
  opacity: 0;
}
@keyframes twinkle {
  0%, 100% { opacity: 0; transform: scale(0.5); }
  50%       { opacity: 0.6; transform: scale(1); }
}

/* ── Snapshot polaroid hover straighten ── */
.snapshot-card:hover { transform: rotate(0deg) !important; }

.line-clamp-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.line-clamp-3 {
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
</style>
