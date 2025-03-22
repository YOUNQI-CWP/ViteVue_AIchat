<script setup>
import { ref, onMounted } from 'vue'
import { ElMessage } from 'element-plus'
import { marked } from 'marked'
import hljs from 'highlight.js'
import 'highlight.js/styles/github.css'
import { CopyDocument, RefreshRight } from '@element-plus/icons-vue'

const messages = ref([])
const inputMessage = ref('')
const apiKey = ref(import.meta.env.VITE_OPENAI_API_KEY || '')
const apiEndpoint = ref(import.meta.env.VITE_OPENAI_API_ENDPOINT || '')
const showSettings = ref(false)
const selectedModel = ref('')
const availableModels = ref([
  { value: 'Qwen/Qwen2.5-Coder-7B-Instruct', label: 'Qwen/Qwen2.5-Coder-7B-Instruct' },
  { value: 'gpt-4', label: 'GPT-4' },
  { value: 'gpt-4-32k', label: 'GPT-4 32K' }
])

onMounted(() => {
  const savedApiKey = localStorage.getItem('apiKey')
  const savedApiEndpoint = localStorage.getItem('apiEndpoint')
  const savedModel = localStorage.getItem('selectedModel')
  if (savedApiKey) apiKey.value = savedApiKey
  if (savedApiEndpoint) apiEndpoint.value = savedApiEndpoint
  if (savedModel) selectedModel.value = savedModel

  marked.setOptions({
    highlight: function(code, lang) {
      if (lang && hljs.getLanguage(lang)) {
        return hljs.highlight(code, { language: lang }).value
      }
      return hljs.highlightAuto(code).value
    },
    breaks: true
  })
})

const saveSettings = () => {
  localStorage.setItem('apiKey', apiKey.value)
  localStorage.setItem('apiEndpoint', apiEndpoint.value)
  localStorage.setItem('selectedModel', selectedModel.value)
  showSettings.value = false
  ElMessage.success('设置已保存')
}

const copyMessage = async (content) => {
  try {
    await navigator.clipboard.writeText(content)
    ElMessage.success('复制成功')
  } catch (error) {
    ElMessage.error('复制失败：' + error.message)
  }
}

const regenerateMessage = async (index) => {
  // 确保index是有效的，并且对应的消息是assistant的回复
  if (index < 0 || index >= messages.value.length || messages.value[index].role !== 'assistant') return
  if (!apiKey.value || !apiEndpoint.value) {
    ElMessage.warning('请先配置API设置')
    showSettings.value = true
    return
  }
  
  // 保留用户的问题和之前的历史消息
  const previousMessages = messages.value.slice(0, index)
  const userMessage = messages.value[index - 1]
  
  // 删除从当前assistant消息开始的所有后续消息
  messages.value = previousMessages
  
  // 创建一个空的助手消息
  const assistantMessage = { role: 'assistant', content: '' }
  messages.value.push(assistantMessage)

  try {
    const response = await fetch(apiEndpoint.value, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${apiKey.value}`
      },
      body: JSON.stringify({
        messages: messages.value.slice(0, -1), // 不包含空的助手消息
        model: selectedModel.value,
        temperature: 0.7,
        stream: true // 启用流式输出
      })
    })

    if (!response.ok) {
      throw new Error(`API请求失败: ${response.status} ${response.statusText}`)
    }

    const reader = response.body.getReader()
    const decoder = new TextDecoder()
    let buffer = ''

    while (true) {
      const { done, value } = await reader.read()
      if (done) break

      buffer += decoder.decode(value, { stream: true })
      const lines = buffer.split('\n')
      buffer = lines.pop() || ''

      for (const line of lines) {
        if (line.trim() === '') continue
        if (line.trim() === 'data: [DONE]') continue

        try {
          const data = JSON.parse(line.replace(/^data: /, ''))
          if (data.choices && data.choices[0] && data.choices[0].delta && data.choices[0].delta.content) {
            assistantMessage.content += data.choices[0].delta.content
            // 强制Vue更新视图
            messages.value = [...messages.value]
          }
        } catch (error) {
          console.error('解析流数据出错:', error)
        }
      }
    }
  } catch (error) {
    messages.value.pop() // 移除助手消息
    ElMessage.error('重新生成失败：' + error.message)
  }
}

const sendMessage = async () => {
  if (!inputMessage.value.trim()) return
  if (!apiKey.value || !apiEndpoint.value) {
    ElMessage.warning('请先配置API设置')
    showSettings.value = true
    return
  }

  const userMessage = { role: 'user', content: inputMessage.value }
  messages.value.push(userMessage)
  const tempMessage = inputMessage.value
  inputMessage.value = ''

  // 创建一个空的助手消息
  const assistantMessage = { role: 'assistant', content: '' }
  messages.value.push(assistantMessage)

  try {
    const response = await fetch(apiEndpoint.value, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${apiKey.value}`
      },
      body: JSON.stringify({
        messages: messages.value.slice(0, -1), // 不包含空的助手消息
        model: selectedModel.value,
        temperature: 0.7,
        stream: true // 启用流式输出
      })
    })

    if (!response.ok) {
      throw new Error(`API请求失败: ${response.status} ${response.statusText}`)
    }

    const reader = response.body.getReader()
    const decoder = new TextDecoder()
    let buffer = ''

    while (true) {
      const { done, value } = await reader.read()
      if (done) break

      buffer += decoder.decode(value, { stream: true })
      const lines = buffer.split('\n')
      buffer = lines.pop() || ''

      for (const line of lines) {
        if (line.trim() === '') continue
        if (line.trim() === 'data: [DONE]') continue

        try {
          const data = JSON.parse(line.replace(/^data: /, ''))
          if (data.choices && data.choices[0] && data.choices[0].delta && data.choices[0].delta.content) {
            assistantMessage.content += data.choices[0].delta.content
            // 强制Vue更新视图
            messages.value = [...messages.value]
          }
        } catch (error) {
          console.error('解析流数据出错:', error)
        }
      }
    }
  } catch (error) {
    messages.value.pop() // 移除助手消息
    inputMessage.value = tempMessage
    ElMessage.error('发送消息失败：' + error.message)
  }
}
</script>

<template>
  <div class="chat-container">
    <el-button class="settings-button" @click="showSettings = true" type="primary" circle>
      <el-icon><Setting /></el-icon>
    </el-button>

    <div class="messages-container">
      <div v-for="(msg, index) in messages" :key="index" :class="['message', msg.role]">
        <div class="message-content">
          <div class="message-inner">
            <div v-html="marked(msg.content)"></div>
          </div>
          <div class="message-actions">
            <el-button
              class="action-button"
              :icon="CopyDocument"
              circle
              @click="copyMessage(msg.content)"
              title="复制消息"
            />
            <el-button
              v-if="msg.role === 'assistant'"
              class="action-button"
              :icon="RefreshRight"
              circle
              @click="regenerateMessage(index + 1)"
              title="重新生成"
            />
          </div>
        </div>
      </div>
    </div>

    <div class="input-container">
      <el-input
        v-model="inputMessage"
        placeholder="输入消息..."
        @keyup.enter="sendMessage"
        :rows="3"
        type="textarea"
      />
      <el-button type="primary" @click="sendMessage">发送</el-button>
    </div>

    <el-dialog v-model="showSettings" title="API设置" width="400px">
      <el-form label-position="top">
        <el-form-item label="API密钥">
          <el-input v-model="apiKey" placeholder="输入API密钥" show-password />
        </el-form-item>
        <el-form-item label="API地址">
          <el-input v-model="apiEndpoint" placeholder="输入API地址" />
        </el-form-item>
        <el-form-item label="模型名称">
          <el-input v-model="selectedModel" placeholder="输入模型名称" />
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="showSettings = false">取消</el-button>
        <el-button type="primary" @click="saveSettings">保存</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<style scoped>
.chat-container {
  width: 100%;
  height: 100vh;
  display: flex;
  flex-direction: column;
  background-color: var(--assistant-bg);
  position: relative;
}

.settings-button {
  position: fixed;
  top: 1rem;
  right: 1rem;
  z-index: 100;
  opacity: 0.8;
  transition: opacity 0.2s;
}

.settings-button:hover {
  opacity: 1;
}

.messages-container {
  flex: 1;
  overflow-y: auto;
  padding: 0.8rem;
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
}

.message {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  max-width: 85%;
  padding: 0.15rem;
  position: relative;
  margin: 0.1rem 0;
}

.message.user {
  align-self: flex-end;
  align-items: flex-end;
  .message-content {
    background-color: #007AFF;
    color: white;
    margin-left: auto;
  }
}

.message.assistant {
  align-self: flex-start;
  .message-content {
    background-color: #f0f0f0;
    margin-right: auto;
  }
}

.message::before {
  content: '';
  width: 2rem;
  height: 2rem;
  border-radius: 50%;
  background-size: cover;
  margin-right: 0.75rem;
}

.message.user {
  flex-direction: row-reverse;
}

.message.user::before {
  content: '👤';
  margin-right: 0;
  margin-left: 0.75rem;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: var(--primary-color);
  color: white;
}

.message.assistant::before {
  content: '🤖';
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #f0f0f0;
  color: var(--primary-color);
}

.message-content {
  line-height: 1.4;
  white-space: normal;
  width: 100%;
  max-width: 100%;
  font-size: 1rem;
  color: inherit;
  padding: 0;
  border-radius: 0.8rem;
  position: relative;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  margin: 0.15rem 0;
  word-wrap: break-word;
  text-align: left;
}

.message-inner {
  padding: 0.8rem 1rem;
  margin: 0;
  width: 100%;
  box-sizing: border-box;
}

.message-inner :deep(p) {
  margin: 0;
  padding: 0;
  line-height: 1.4;
}

.message-inner :deep(pre) {
  margin: 0.3rem 0;
  border-radius: 0.4rem;
}

.message-inner :deep(code) {
  padding: 0.15rem 0.3rem;
  border-radius: 0.25rem;
  background-color: rgba(0, 0, 0, 0.04);
}

.message-inner :deep(pre code) {
  padding: 0;
  background-color: transparent;
}

.message-inner :deep(p:first-child) {
  margin-top: 0;
}

.message-inner :deep(p:last-child) {
  margin-bottom: 0;
}

.message-actions {
  position: absolute;
  bottom: -1.5em;
  left: 0;
  display: none;
  gap: 0.25rem;
  opacity: 0;
  transition: opacity 0.2s ease;
  z-index: 1;
}

.message.user .message-actions {
  left: auto;
  right: 0;
}

.message {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  max-width: 85%;
  padding: 0.15rem;
  position: relative;
  margin: 0.3rem 0;
}

.message:hover .message-actions {
  display: flex;
  opacity: 1;
  transform: translateY(-0.2em);
}

.action-button {
  padding: 4px;
  height: 24px;
  width: 24px;
  font-size: 12px;
}

.action-button:hover {
  opacity: 1;
}

.message-content :deep(h1),
.message-content :deep(h2),
.message-content :deep(h3),
.message-content :deep(h4),
.message-content :deep(h5),
.message-content :deep(h6) {
  margin: 1.5rem 0 1rem;
  font-weight: 600;
  line-height: 1.25;
}

.message-content :deep(h1) { font-size: 2em; }
.message-content :deep(h2) { font-size: 1.5em; }
.message-content :deep(h3) { font-size: 1.25em; }
.message-content :deep(h4) { font-size: 1em; }
.message-content :deep(h5) { font-size: 0.875em; }
.message-content :deep(h6) { font-size: 0.85em; }

.message-content :deep(pre) {
  background-color: #282c34;
  padding: 1rem;
  border-radius: 0.5rem;
  overflow-x: auto;
  margin: 1rem 0;
}

.message-content :deep(code) {
  font-family: Monaco, Consolas, 'Courier New', monospace;
  font-size: 0.9em;
  padding: 0.2em 0.4em;
  border-radius: 0.3em;
  background-color: rgba(0, 0, 0, 0.1);
}

.message-content :deep(pre code) {
  background-color: transparent;
  padding: 0;
  color: #abb2bf;
}

.message-content :deep(ul),
.message-content :deep(ol) {
  padding-left: 1.5em;
  margin: 1rem 0;
}

.message-content :deep(li) {
  margin: 0.5rem 0;
}

.message-content :deep(blockquote) {
  margin: 1rem 0;
  padding: 0.5rem 1rem;
  border-left: 4px solid #ddd;
  background-color: rgba(0, 0, 0, 0.05);
  border-radius: 0.3rem;
}

.message-content :deep(p) {
  margin: 0;
}

.message-content :deep(table) {
  border-collapse: collapse;
  width: 100%;
  margin: 1rem 0;
}

.message-content :deep(th),
.message-content :deep(td) {
  border: 1px solid #ddd;
  padding: 0.5rem;
  text-align: left;
}

.message-content :deep(th) {
  background-color: rgba(0, 0, 0, 0.05);
}

.message-content :deep(p) {
  margin: 0;
  padding: 0;
}

.message-content :deep(pre) {
  background: #f8f9fa;
  border-radius: 0.5rem;
  padding: 1rem;
  margin: 0.5em 0;
  overflow-x: auto;
}

.message-content :deep(code) {
  font-family: 'Fira Code', monospace;
  font-size: 0.9em;
}

.message-content :deep(p code) {
  background: rgba(0,0,0,0.1);
  padding: 0.2em 0.4em;
  border-radius: 0.3em;
}

.message.user .message-content :deep(pre),
.message.user .message-content :deep(code) {
  background: rgba(255,255,255,0.1);
}

.message.user .message-content :deep(a) {
  color: #fff;
  text-decoration: underline;
}

.message.assistant .message-content :deep(a) {
  color: var(--primary-color);
}

.message.user .message-content {
  background-color: var(--primary-color);
  color: white;
  border-top-right-radius: 0;
  margin-right: 0.5rem;
}

.message.assistant .message-content {
  background-color: #f0f0f0;
  border-top-left-radius: 0;
  margin-left: 0.5rem;
}

.input-container {
  position: sticky;
  bottom: 0;
  left: 0;
  right: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.5rem;
  padding: 1rem;
  background-color: var(--assistant-bg);
  border-top: 1px solid var(--border-color);
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

.input-container .el-input {
  width: 100%;
  max-width: 48rem;
}

.input-container .el-button {
  align-self: flex-end;
  margin-right: calc((100% - 48rem) / 2);
}

.el-dialog {
  border-radius: 0.5rem;
  overflow: hidden;
  max-width: 90vw;
}

.el-dialog__header {
  margin: 0;
  padding: 1rem 1.5rem;
  background-color: var(--message-bg);
  border-bottom: 1px solid var(--border-color);
}

.el-dialog__body {
  padding: 1.5rem;
}

.el-dialog__footer {
  padding: 1rem 1.5rem;
  border-top: 1px solid var(--border-color);
}

@media (max-width: 768px) {
  .message {
    padding: 1rem 0.75rem;
  }

  .message-content {
    font-size: 0.9rem;
  }

  .input-container {
    padding: 0.75rem;
  }

  .input-container .el-button {
    margin-right: 0;
    width: 100%;
  }
}

</style>
