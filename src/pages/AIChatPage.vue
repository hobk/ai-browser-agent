<template>
  <div class="flex h-screen flex-col bg-gray-50">
    <!-- Header -->
    <div class="flex items-center justify-between border-b bg-white px-6 py-3 shadow-sm">
      <div class="flex items-center gap-4">
        <h1 class="text-xl font-bold text-gray-900">AI
           Brower Agent</h1>
        <span v-if="isSessionActive" class="flex items-center gap-1.5 text-sm text-green-600">
          <span class="h-2 w-2 rounded-full bg-green-500 animate-pulse"></span>
          Connected
        </span>
        <span v-else class="flex items-center gap-1.5 text-sm text-gray-400">
          <span class="h-2 w-2 rounded-full bg-gray-300"></span>
          Disconnected
        </span>
      </div>
      <div class="flex items-center gap-4">
        <button
          @click="startSession"
          :disabled="isSessionActive || isStarting"
          class="rounded-md bg-blue-600 px-4 py-2 text-sm font-medium text-white hover:bg-blue-700 disabled:opacity-50"
        >
          {{ isStarting ? '启动中...' : isSessionActive ? '已连接' : '启动会话' }}
        </button>
        <button
          v-if="isSessionActive"
          @click="stopSession"
          class="rounded-md bg-red-600 px-4 py-2 text-sm font-medium text-white hover:bg-red-700"
        >
          停止会话
        </button>
      </div>
    </div>

    <!-- Main Content -->
    <div class="flex flex-1 overflow-hidden">
      <!-- Left: Chat Panel (1/6 width) -->
      <div class="flex w-1/6 min-w-[280px] flex-col border-r bg-white">
        <!-- Chat Header -->
        <div class="flex items-center justify-between border-b bg-gray-50 px-4 py-2">
          <h2 class="font-medium text-gray-700">AI 指令执行</h2>
          <button @click="clearMessages" class="text-xs text-gray-500 hover:text-gray-700">
            清空对话
          </button>
        </div>

        <!-- Chat Messages -->
        <div class="flex-1 overflow-y-auto p-4 space-y-4" ref="chatContainerRef">
          <div v-if="messages.length === 0" class="flex flex-col items-center justify-center h-full text-gray-400">
            <svg class="w-16 h-16 mb-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.5"
                d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z" />
            </svg>
            <p class="text-center">
              开始对话吧！<br />
              <span class="text-sm">输入指令让 AI 帮你操作浏览器</span>
            </p>
          </div>

          <div v-for="(msg, index) in messages" :key="index" class="flex" :class="msg.role === 'user' ? 'justify-end' : 'justify-start'">
            <div
              class="max-w-[80%] rounded-lg px-4 py-2 shadow-sm"
              :class="msg.role === 'user' ? 'bg-blue-600 text-white' : msg.type === 'error' ? 'bg-red-100 text-red-800' : 'bg-gray-100 text-gray-800'"
            >
              <div class="text-sm whitespace-pre-wrap">{{ msg.content }}</div>
              <div class="mt-1 text-xs opacity-70">{{ msg.time }}</div>
            </div>
          </div>

          <!-- Loading indicator -->
          <div v-if="isExecuting" class="flex justify-start">
            <div class="bg-gray-100 rounded-lg px-4 py-3">
              <div class="flex items-center gap-2">
                <div class="flex gap-1">
                  <span class="w-2 h-2 bg-gray-400 rounded-full animate-bounce" style="animation-delay: 0ms"></span>
                  <span class="w-2 h-2 bg-gray-400 rounded-full animate-bounce" style="animation-delay: 150ms"></span>
                  <span class="w-2 h-2 bg-gray-400 rounded-full animate-bounce" style="animation-delay: 300ms"></span>
                </div>
                <span class="text-sm text-gray-500">AI 正在执行...</span>
              </div>
            </div>
          </div>
        </div>

        <!-- Chat Input -->
        <div class="border-t bg-white p-4">
          <div class="relative">
            <textarea
              ref="inputRef"
              v-model="inputMessage"
              @keydown.enter.exact="handleEnter"
              @keydown.shift.enter="insertNewLine"
              @compositionstart="isComposing = true"
              @compositionend="isComposing = false"
              :disabled="!isSessionActive || isExecuting"
              placeholder="请在此描述要执行的操作，如点击或输入xx"
              class="w-full resize-none rounded-lg border border-gray-300 px-4 py-3 pr-14 text-sm focus:border-blue-500 focus:outline-none focus:ring-2 focus:ring-blue-500/20 disabled:bg-gray-100"
              rows="2"
            ></textarea>
            <button
              @click="sendMessage"
              :disabled="!isSessionActive || isExecuting || !inputMessage.trim()"
              class="absolute bottom-2 right-2 h-8 w-8 rounded-full bg-blue-600 hover:bg-blue-700 disabled:opacity-50 disabled:bg-gray-300 flex items-center justify-center transition-colors"
              title="发送"
            >
              <svg class="h-5 w-5 text-white" fill="currentColor" viewBox="0 0 20 20">
                <path d="M10.894 2.553a1 1 0 00-1.788 0l-7 14a1 1 0 001.169 1.409l5.951-1.429 5.951 1.429a1 1 0 001.169-1.409l-7-14z" />
              </svg>
            </button>
          </div>
       <!--    <div class="mt-2 text-xs text-gray-400">
            按 Enter 发送，Shift + Enter 换行
          </div> -->
        </div>
      </div>

      <!-- Right: Browser View -->
      <div class="flex flex-1 flex-col bg-gray-100 overflow-hidden">
        <!-- Browser Frame Container -->
        <div class="flex-1 flex flex-col p-4 min-h-0">
          <!-- Simulated Browser Window -->
          <div class="flex flex-col h-full w-full overflow-hidden rounded-lg border border-gray-300 bg-white shadow-lg">
            <!-- Browser Toolbar -->
            <div class="flex items-center gap-2 border-b bg-gray-100 px-3 py-2">
              <div class="flex gap-1.5">
                <div class="h-3 w-3 rounded-full bg-red-400"></div>
                <div class="h-3 w-3 rounded-full bg-yellow-400"></div>
                <div class="h-3 w-3 rounded-full bg-green-400"></div>
              </div>
              <!-- Address Bar Input -->
              <div class="flex-1 flex items-center gap-2">
                <span class="text-xs text-gray-500 whitespace-nowrap">请在此输入网址:</span>
                <div class="relative flex-1">
                  <input
                    v-model="addressBarInput"
                    @keydown.enter="visitInputUrl"
                    :disabled="!isSessionActive"
                    type="text"
                    placeholder="https://example.com"
                    class="w-full rounded-md bg-white px-3 py-1 text-xs text-gray-700 border border-gray-300 focus:border-blue-500 focus:outline-none focus:ring-1 focus:ring-blue-500 disabled:bg-gray-100"
                  />
                </div>
                <button
                  @click="visitInputUrl"
                  :disabled="!isSessionActive || !addressBarInput.trim()"
                  class="px-3 py-1 rounded-md bg-blue-600 text-white text-xs font-medium hover:bg-blue-700 disabled:opacity-50 disabled:bg-gray-300 whitespace-nowrap transition-colors"
                  title="访问输入的URL"
                >
                  访问
                </button>
              </div>
              <!-- Refresh Button -->
              <button
                @click="refreshPage"
                :disabled="!isSessionActive"
                class="p-1 rounded hover:bg-gray-200 disabled:opacity-50"
                title="刷新页面"
              >
                <svg class="w-4 h-4 text-gray-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                    d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" />
                </svg>
              </button>
            </div>

            <!-- Browser Content (Image) -->
            <div class="relative flex-1 overflow-hidden bg-white">
              <img
                v-if="isSessionActive && screenshotBlobUrl"
                :src="screenshotBlobUrl"
                class="h-full w-full object-contain"
                alt="Browser Preview"
              />
              <div v-else class="flex h-full flex-col items-center justify-center text-gray-400">
                <svg class="mb-4 h-16 w-16" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                    d="M9.75 17L9 20l-1 1h8l-1-1-.75-3M3 13h18M5 17h14a2 2 0 002-2V5a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z"
                  ></path>
                </svg>
                <p>{{ isStarting ? '正在启动浏览器...' : '点击"启动会话"开始' }}</p>
              </div>
            </div>
          </div>
        </div>

        <!-- Quick Actions -->
        <div class="border-t bg-white p-3">
          <div class="flex items-center gap-2 flex-wrap">
            <span class="text-xs text-gray-500 mr-2">快捷指令：</span>
            <button
              v-for="action in quickActions"
              :key="action.label"
              @click="executeQuickAction(action)"
              :disabled="!isSessionActive || isExecuting"
              class="rounded-full bg-gray-100 px-3 py-1 text-xs text-gray-700 hover:bg-gray-200 disabled:opacity-50"
            >
              {{ action.label }}
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { STEP_TYPES } from '@/constants/stepTypes'
import { authAPI, authToken, debugAPI } from '@/utils/api'
import { toast } from '@/utils/toast'
import { computed, nextTick, onMounted, onUnmounted, ref } from 'vue'
import { onBeforeRouteLeave, useRouter } from 'vue-router'

const router = useRouter()

// Session state
const sessionId = ref(null)
const isStarting = ref(false)
const screenshotBlobUrl = ref('')
const currentUrl = ref('')
const addressBarInput = ref('') // 地址栏输入值
const isExecuting = ref(false)

// Chat state
const messages = ref([])
const inputMessage = ref('')
const chatContainerRef = ref(null)
const inputRef = ref(null) // 输入框DOM元素
const isComposing = ref(false) // 追踪输入法状态

// WebSocket
let ws = null
let reconnectTimer = null
let reconnectAttempts = 0

// 系统自动登录配置（前端平台登录）
const SYSTEM_LOGIN_EMAIL = 'hu.bo2@trs.com.cn'
const SYSTEM_LOGIN_PASSWORD = '@pa123sw0rd1'

// 默认打开的URL
const DEFAULT_URL = 'https://www.aliyun.com'

// 快捷指令
const quickActions = [
  { label: '阿里云官网', command: 'https://www.aliyun.com' },
  { label: '截图', type: 'screenshot' },
  { label: '返回顶部', command: '滚动到页面顶部' },
  { label: '返回底部', command: '滚动到页面底部' },
]

const isSessionActive = computed(() => !!sessionId.value)

// Auto scroll chat to bottom
const scrollToBottom = () => {
  nextTick(() => {
    if (chatContainerRef.value) {
      chatContainerRef.value.scrollTop = chatContainerRef.value.scrollHeight
    }
  })
}

// Add message to chat
const addMessage = (content, role = 'assistant', type = 'normal') => {
  const time = new Date().toLocaleTimeString()
  messages.value.push({ content, role, type, time })
  scrollToBottom()
}

// Clear messages
const clearMessages = () => {
  messages.value = []
}

// WebSocket connection
const connectWebSocket = (sid) => {
  if (ws) {
    ws.close()
    ws = null
  }

  const protocol = window.location.protocol === 'https:' ? 'wss:' : 'ws:'
  const host = window.location.host
  const wsUrl = `https://llmys.trscd.com.cn/autotest/ws?token=${authToken.get()}&sessionId=${sid}`

  ws = new WebSocket(wsUrl)
  ws.binaryType = 'blob'

  ws.onopen = () => {
    console.log('WS Connected')
    reconnectAttempts = 0
  }

  ws.onmessage = (event) => {
    if (event.data instanceof Blob) {
      if (screenshotBlobUrl.value) {
        URL.revokeObjectURL(screenshotBlobUrl.value)
      }
      screenshotBlobUrl.value = URL.createObjectURL(event.data)
    }
  }

  ws.onclose = () => {
    console.log('WS Disconnected')

    if (!sessionId.value || sessionId.value !== sid) return
    if (reconnectTimer) return

    const delay = Math.min(5000, 500 * Math.pow(2, reconnectAttempts))
    reconnectAttempts += 1
    reconnectTimer = setTimeout(async () => {
      reconnectTimer = null
      try {
        const st = await debugAPI.status(sid)
        if (st?.active) {
          connectWebSocket(sid)
        }
      } catch {
        // ignore
      }
    }, delay)
  }
}

const disconnectClient = () => {
  if (reconnectTimer) {
    clearTimeout(reconnectTimer)
    reconnectTimer = null
  }
  reconnectAttempts = 0
  if (ws) {
    ws.close()
    ws = null
  }
  if (screenshotBlobUrl.value) {
    URL.revokeObjectURL(screenshotBlobUrl.value)
    screenshotBlobUrl.value = ''
  }
}

// Start debug session
const startSession = async () => {
  if (isStarting.value || isSessionActive.value) return

  isStarting.value = true
  try {
    const res = await debugAPI.start()
    sessionId.value = res.sessionId
    addMessage('浏览器会话已启动', 'assistant')
    toast.success('会话已启动')

    connectWebSocket(res.sessionId)

    // 等待一下让浏览器准备好
    await new Promise(resolve => setTimeout(resolve, 1000))

    // 默认打开百度
    await navigateToUrl(DEFAULT_URL)
    addMessage('✅ 已打开默认页面，现在为您演示当前页面的操作，请等待演示完毕后可自行操作', 'assistant')
    
    // 发送演示消息
    const demoMessage = '【演示】搜索esa产品，然后找到定价'
    await new Promise(resolve => setTimeout(resolve, 500))
    addMessage(demoMessage, 'user')
    
    // 自动执行演示指令
    isExecuting.value = true
    try {
      const step = {
        type: STEP_TYPES.AI_ACTION,
        prompt: demoMessage,
      }
      const res = await executeStep(step)
      if (res?.result) {
        addMessage(`执行完成：${res.result}`, 'assistant')
      } else {
        addMessage('✅ 演示指令执行完成，您可以在下方输入操作指令，快来试试吧', 'assistant')
      }
    } catch (e) {
      addMessage(`❌ 演示指令执行失败: ${e.message}`, 'assistant', 'error')
    } finally {
      isExecuting.value = false
      nextTick(() => {
        if (inputRef.value) {
          inputRef.value.focus()
        }
      })
    }
  } catch (e) {
    if(e.meesage.indexOf('401')!==-1){
      addMessage(`授权过期，正在为您重新初始化系统...`,'assistant')
      // 重新登录
      authToken.clear()
      // 刷新页面
      setTimeout(() => {
        window.location.reload()
      }, 500)
      return
    }
    addMessage(`启动失败: ${e.message}`, 'assistant', 'error')
    toast.error('启动失败: ' + e.message)
  } finally {
    isStarting.value = false
  }
}

// Stop debug session
const stopSession = async () => {
  disconnectClient()

  if (sessionId.value) {
    try {
      await debugAPI.stop(sessionId.value)
      addMessage('会话已停止', 'assistant')
    } catch (e) {
      // ignore
    }
    sessionId.value = null
  }

  currentUrl.value = ''
}

// 系统自动登录（前端平台）
const autoSystemLogin = async () => {
  try {
    const {data:res} = await authAPI.login({
      email: SYSTEM_LOGIN_EMAIL,
      password: SYSTEM_LOGIN_PASSWORD,
    })
    debugger
    if (res?.token) {
      authToken.set(res.token)
      localStorage.setItem('auth_user', JSON.stringify(res.user))
      if (res.tenants) {
        localStorage.setItem('login_data', JSON.stringify({ tenants: res.tenants }))
      }
      return true
    }
    return false
  } catch (e) {
    console.error('自动登录失败:', e)
    return false
  }
}

// 校验是否是合法的URL
const isValidUrl = (urlString) => {
  try {
    const url = new URL(urlString)
    // 只允许 http 和 https 协议
    return url.protocol === 'http:' || url.protocol === 'https:'
  } catch {
    return false
  }
}

// 标准化URL（补全协议）
const normalizeUrl = (urlString) => {
  const trimmed = urlString.trim()
  // 如果没有协议，添加 https://
  if (!trimmed.startsWith('http://') && !trimmed.startsWith('https://')) {
    return 'https://' + trimmed
  }
  return trimmed
}

// 访问地址栏输入的URL
const visitInputUrl = async () => {
  const input = addressBarInput.value.trim()
  if (!input || !isSessionActive.value) return

  // 标准化URL
  const url = normalizeUrl(input)

  // 校验URL合法性
  if (!isValidUrl(url)) {
    toast.error('请输入有效的网址，例如：https://www.aliyun.com')
    return
  }

  addMessage(`访问: ${url}`, 'user')
  await navigateToUrl(url)
  addressBarInput.value = ''
}

// 导航到指定URL
const navigateToUrl = async (url) => {
  if (!sessionId.value) return
  
  addMessage(`正在打开: ${url}`, 'assistant')
  await executeStep({
    type: STEP_TYPES.NAVIGATE,
    url: url,
  })
  currentUrl.value = url
  await new Promise(resolve => setTimeout(resolve, 1500))
}

// 从文本中提取URL（只匹配有效的URL，排除汉字和其他非ASCII字符）
const extractUrls = (text) => {
  // 匹配 http:// 或 https:// 开头，后面跟有效的URL字符（排除汉字、空格）
  // 有效字符包括：a-z A-Z 0-9 以及 - . _ ~ : / ? # [ ] @ ! $ & ' ( ) * + , ; = % 等
  const urlRegex = /(https?:\/\/[a-zA-Z0-9\-._~:/?#\[\]@!$&'()*+,;=%.]+)/gi
  const matches = text.match(urlRegex) || []
  
  // 进一步过滤，移除以中文字符结尾的URL（保险措施）
  return matches.filter(url => {
    // 检查URL是否以标点符号结尾（可能是汉字导致的错误匹配）
    const endsWithPunctuation = /[、。，；：？！（）]/g.test(url)
    return !endsWithPunctuation
  })
}

// Execute a single step
const executeStep = async (step) => {
  if (!sessionId.value) throw new Error('Session not active')

  const res = await debugAPI.executeStep(sessionId.value, step)
  if (res?.status !== 'success') {
    throw new Error(res?.error || 'Unknown error')
  }
  return res
}

// Send message (AI command)
const sendMessage = async () => {
  const text = inputMessage.value.trim()
  if (!text || !isSessionActive.value || isExecuting.value) return

  inputMessage.value = ''
  addMessage(text, 'user')

  isExecuting.value = true
  try {
    // 检测消息中是否包含URL
    const urls = extractUrls(text)
    
    if (urls.length > 0) {
      // 如果包含URL，先导航到该URL
      const targetUrl = urls[0]
      await navigateToUrl(targetUrl)
      
      // 从原始文本中移除URL，获取剩余的指令
      let remainingPrompt = text
      urls.forEach(url => {
        remainingPrompt = remainingPrompt.replace(url, '').trim()
      })
      
      // 如果有剩余指令，继续执行AI操作
      if (remainingPrompt) {
        addMessage(`正在执行: ${remainingPrompt}`, 'assistant')
        const step = {
          type: STEP_TYPES.AI_ACTION,
          prompt: remainingPrompt,
        }
        const res = await executeStep(step)
        if (res?.result) {
          addMessage(`执行完成：${res.result}`, 'assistant')
        } else {
          addMessage('✅ 执行完成', 'assistant')
        }
      } else {
        addMessage('✅ 页面已打开', 'assistant')
      }
    } else {
      // 没有URL，直接执行AI智能操作
      const step = {
        type: STEP_TYPES.AI_ACTION,
        prompt: text,
      }

      const res = await executeStep(step)

      if (res?.result) {
        addMessage(`执行完成：${res.result}`, 'assistant')
      } else {
        addMessage('✅ 执行完成', 'assistant')
      }
    }
  } catch (e) {
    addMessage(`❌ 执行失败: ${e.message}`, 'assistant', 'error')
  } finally {
    isExecuting.value = false
    // 执行完成后自动focus到输入框
    nextTick(() => {
      if (inputRef.value) {
        inputRef.value.focus()
      }
    })
  }
}

// Handle Enter key with composition support
const handleEnter = (e) => {
  // 如果正在使用输入法（如中文输入法），不触发发送
  if (isComposing.value) {
    return
  }
  e.preventDefault()
  sendMessage()
}

// Insert new line in textarea
const insertNewLine = (e) => {
  const textarea = e.target
  const start = textarea.selectionStart
  const end = textarea.selectionEnd
  inputMessage.value = inputMessage.value.substring(0, start) + '\n' + inputMessage.value.substring(end)
  nextTick(() => {
    textarea.selectionStart = textarea.selectionEnd = start + 1
  })
}

// Refresh page
const refreshPage = async () => {
  if (!isSessionActive.value || isExecuting.value) return

  isExecuting.value = true
  addMessage('刷新页面', 'user')
  try {
    await executeStep({
      type: STEP_TYPES.RELOAD,
    })
    addMessage('✅ 页面已刷新', 'assistant')
  } catch (e) {
    addMessage(`❌ 刷新失败: ${e.message}`, 'assistant', 'error')
  } finally {
    isExecuting.value = false
  }
}

// Execute quick action
const executeQuickAction = async (action) => {
  if (!isSessionActive.value || isExecuting.value) return

  if (action.type === 'screenshot') {
    addMessage('截图功能通过实时预览已实现', 'assistant')
    return
  }

  if (action.command) {
    inputMessage.value = action.command
    await sendMessage()
  }
}

// Lifecycle hooks
onMounted(async () => {
  // 检查是否已登录系统，如果未登录则自动登录
  const token = authToken.get()
  if (!token) {
    addMessage('检测到初次访问，正在为您初始化系统...', 'assistant')
    const loginSuccess = await autoSystemLogin()
    if (!loginSuccess) {
      addMessage('❌ 自动登录失败，请手动登录', 'assistant', 'error')
      router.push({ name: 'Login' })
      return
    }
    addMessage('环境初始化成功，正在启动浏览器会话...', 'assistant')
  }else{
    addMessage('正在启动浏览器，请稍后...', 'assistant')
  }
  
  // 页面加载时自动启动会话
  await startSession()
})

onBeforeRouteLeave(async () => {
  await stopSession()
})

onUnmounted(() => {
  stopSession()
})
</script>

<style scoped>
@keyframes bounce {
  0%, 80%, 100% {
    transform: translateY(0);
  }
  40% {
    transform: translateY(-6px);
  }
}
</style>
