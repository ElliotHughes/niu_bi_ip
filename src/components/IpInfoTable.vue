<template>
  <div class="h-full flex items-center justify-center p-4">
    <div id="map" style="height: 300px;"></div>
    <div class="w-full max-w-6xl flex flex-col md:flex-row gap-4">
      <!-- 左侧搜索卡片 - 移动端调整为全宽 -->
      <div class="w-full md:w-[400px] flex-shrink-0 flex flex-col gap-4">
        <div class="rounded-lg border bg-white shadow-sm p-6">
          <form @submit.prevent="fetchIpInfo" class="space-y-4">
            <div class="space-y-2">
              <label class="text-sm font-medium text-gray-700">
                IP 地址
              </label>
              <div class="relative">
                <input 
                  v-model="ipInput" 
                  type="text" 
                  placeholder="支持 IPv4 / IPv6 地址查询（留空查询当前 IP）" 
                  class="w-full h-14 px-5 rounded-md border border-gray-200 focus:border-gray-400 focus:ring-1 focus:ring-gray-400 transition-all duration-200 text-base font-mono"
                  @paste="handlePaste"
                  @focus="onInputFocus"
                  @blur="onInputBlur" 
                />
                <p class="text-sm text-gray-500 mt-2 flex items-center gap-1">
                  <svg class="w-4 h-4" viewBox="0 0 24 24" fill="none" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 16h-1v-4h-1m1-4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
                  </svg>
                  支持 IPv4（如：8.8.8.8）或 IPv6（如：2001:4860:4860::8888）
                </p>
              </div>
            </div>
            
            <button 
              type="submit"
              class="w-full h-12 bg-gray-900 text-white rounded-md transition-all duration-200 flex items-center justify-center gap-2 hover:bg-gray-800 text-base"
              :disabled="loading"
            >
              <span v-if="loading" class="h-5 w-5 border-2 border-white/30 border-t-white rounded-full animate-spin"></span>
              <span class="font-medium">{{ loading ? '查询中' : '查询 IP' }}</span>
            </button>
          </form>
        </div>

        <!-- 错误提示 -->
        <div v-if="error" class="rounded-lg bg-red-50 border border-red-100 p-4">
          <div class="flex items-center gap-2 text-red-600">
            <svg class="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
              <path fill-rule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z" clip-rule="evenodd"/>
            </svg>
            <span class="text-sm">{{ error }}</span>
          </div>
        </div>
      </div>

      <!-- 右侧结果展示 - 移动端调整为全宽 -->
      <div class="w-full md:w-[600px] flex-shrink-0">
        <!-- 加载状态 -->
        <div v-if="loading" class="rounded-lg border bg-white shadow-sm overflow-hidden">
          <div class="px-6 py-4 border-b border-gray-100 bg-gray-50">
            <h2 class="text-lg font-medium text-gray-900">IP 信息</h2>
          </div>
          <div class="divide-y divide-gray-100">
            <div v-for="i in 8" :key="i" class="px-6 py-3 hover:bg-gray-50">
              <div class="flex justify-between items-center">
                <div class="flex items-center gap-2">
                  <div class="w-4 h-4 bg-gray-200 rounded animate-pulse"></div>
                  <div class="h-5 w-24 bg-gray-200 rounded animate-pulse"></div>
                </div>
                <div class="h-5 w-48 bg-gray-200 rounded animate-pulse"></div>
              </div>
            </div>
          </div>
        </div>

        <!-- IP 信息展示 -->
        <div v-else-if="ipInfo" class="rounded-lg border bg-white shadow-sm overflow-hidden">
          <div class="px-6 py-4 border-b border-gray-100 bg-gray-50">
            <h2 class="text-lg font-medium text-gray-900">IP 信息</h2>
          </div>
          <div class="divide-y divide-gray-100">
            <div v-for="(item, key) in displayData" :key="key" 
                 class="px-6 py-3 hover:bg-gray-50 transition-colors duration-200">
              <div class="flex justify-between items-center">
                <div class="flex items-center gap-2 flex-shrink-0">
                  <svg v-if="item.label === 'IP'" class="w-4 h-4 text-gray-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 12a9 9 0 01-9 9m9-9a9 9 0 00-9-9m9 9H3m9 9a9 9 0 01-9-9m9 9c1.657 0 3-4.03 3-9s-1.343-9-3-9m0 18c-1.657 0-3-4.03-3-9s1.343-9 3-9m-9 9a9 9 0 019-9" />
                  </svg>
                  <svg v-else-if="item.label === '国家'" class="w-4 h-4 text-gray-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 21v-4m0 0V5a2 2 0 012-2h6.5l1 1H21l-3 6 3 6h-8.5l-1-1H5a2 2 0 00-2 2zm9-13.5V9" />
                  </svg>
                  <svg v-else-if="item.label === '城市'" class="w-4 h-4 text-gray-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 21V5a2 2 0 00-2-2H7a2 2 0 00-2 2v16m14 0h2m-2 0h-5m-9 0H3m2 0h5M9 7h1m-1 4h1m4-4h1m-1 4h1m-5 10v-5a1 1 0 011-1h2a1 1 0 011 1v5m-4 0h4" />
                  </svg>
                  <svg v-else-if="item.label === 'ISP'" class="w-4 h-4 text-gray-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M8 14v3m4-3v3m4-3v3M3 21h18M3 10h18M3 7l9-4 9 4M4 10h16v11H4V10z" />
                  </svg>
                  <svg v-else-if="item.label === '经度' || item.label === '纬度'" class="w-4 h-4 text-gray-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z" />
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 11a3 3 0 11-6 0 3 3 0 016 0z" />
                  </svg>
                  <svg v-else-if="item.label === '时区'" class="w-4 h-4 text-gray-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" />
                  </svg>
                  <svg v-else-if="item.label === '邮编'" class="w-4 h-4 text-gray-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 8l7.89 5.26a2 2 0 002.22 0L21 8M5 19h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z" />
                  </svg>
                  <span class="text-base font-semibold text-gray-700">{{ item.label }}</span>
                </div>
                <span class="text-base font-medium text-blue-700 font-mono break-all">{{ item.value }}</span>
              </div>
            </div>
          </div>
        </div>

        <!-- 空状态 -->
        <div v-else-if="!loading" class="rounded-lg border bg-white shadow-sm p-8 text-center">
          <div class="text-sm text-gray-500">
            请输入 IP 地址进行查询，或留空查询当前 IP
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, computed } from 'vue'
import axios from 'axios'

import 'leaflet/dist/leaflet.css'
import L from 'leaflet'

const ipInput = ref('')
const ipInfo = ref(null)
const loading = ref(false)
const error = ref(null)

const displayData = computed(() => {
  if (!ipInfo.value) return []
  return [
    { label: 'IP', value: ipInfo.value.ip },
    { label: '国家', value: ipInfo.value.ip_data.country },
    { label: '城市', value: ipInfo.value.ip_data.city },
    { label: 'ISP', value: ipInfo.value.ip_data.isp },
    { label: '经度', value: ipInfo.value.ip_data.longitude },
    { label: '纬度', value: ipInfo.value.ip_data.latitude },
    { label: '时区', value: ipInfo.value.ip_data.timezone },
    { label: '邮编', value: ipInfo.value.ip_data.zipcode }
  ]
})

const fetchIpInfo = async () => {
  loading.value = true
  error.value = null
  
  try {
    const url = `https://ip-scan.adspower.net/sys/config/ip/get-visitor-ip?type=ip-api&ip=${ipInput.value}`
    const response = await axios.get(url)
    
    if (response.data && response.data.data) {
      ipInfo.value = response.data.data
      const { latitude, longitude } = response.data.data.ip_data
      displayMap(latitude, longitude)
    } else {
      error.value = '获取 IP 信息失败，请稍后再试'
    }
  } catch (err) {
    console.error('Error fetching IP info:', err)
    error.value = '获取 IP 信息失败：' + (err.message || '未知错误')
  } finally {
    loading.value = false
  }
}

// 输入框聚焦时移除全局粘贴事件
const onInputFocus = () => {
  document.removeEventListener('paste', handleGlobalPaste)
}

// 输入框失焦时重新绑定全局粘贴事件
const onInputBlur = () => {
  document.addEventListener('paste', handleGlobalPaste)
}

// 添加全局粘贴事件监听
onMounted(() => {
  // 页面加载时自动查询当前 IP
  fetchIpInfo()
  // 添加全局粘贴事件监听
  document.addEventListener('paste', handleGlobalPaste)
})

// 在组件卸载时移除事件监听
onUnmounted(() => {
  document.removeEventListener('paste', handleGlobalPaste)
})

// 全局粘贴事件处理
const handleGlobalPaste = async (event) => {
  // 获取粘贴的文本
  const pastedText = (event.clipboardData || window.clipboardData).getData('text').trim()
  
  // 检查是否是IP地址
  if (ipv4Regex.test(pastedText) || ipv6Regex.test(pastedText)) {
    // 如果是IP地址，设置输入值并立即查询
    ipInput.value = pastedText
    await fetchIpInfo()
  }
}

// IP地址正则表达式
const ipv4Regex = /^(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$/
const ipv6Regex = /^(?:(?:[0-9a-fA-F]{1,4}:){7}[0-9a-fA-F]{1,4}|(?:[0-9a-fA-F]{1,4}:){1,7}:|(?:[0-9a-fA-F]{1,4}:){1,6}:[0-9a-fA-F]{1,4}|(?:[0-9a-fA-F]{1,4}:){1,5}(?::[0-9a-fA-F]{1,4}){1,2}|(?:[0-9a-fA-F]{1,4}:){1,4}(?::[0-9a-fA-F]{1,4}){1,3}|(?:[0-9a-fA-F]{1,4}:){1,3}(?::[0-9a-fA-F]{1,4}){1,4}|(?:[0-9a-fA-F]{1,4}:){1,2}(?::[0-9a-fA-F]{1,4}){1,5}|[0-9a-fA-F]{1,4}:(?:(?::[0-9a-fA-F]{1,4}){1,6})|:(?:(?::[0-9a-fA-F]{1,4}){1,7}|:)|fe80:(?::[0-9a-fA-F]{0,4}){0,4}%[0-9a-zA-Z]{1,}|::(?:ffff(?::0{1,4}){0,1}:){0,1}(?:(?:25[0-5]|(?:2[0-4]|1{0,1}[0-9]){0,1}[0-9])\.){3,3}(?:25[0-5]|(?:2[0-4]|1{0,1}[0-9]){0,1}[0-9])|(?:[0-9a-fA-F]{1,4}:){1,4}:(?:(?:25[0-5]|(?:2[0-4]|1{0,1}[0-9]){0,1}[0-9])\.){3,3}(?:25[0-5]|(?:2[0-4]|1{0,1}[0-9]){0,1}[0-9]))$/

// 处理粘贴事件
const handlePaste = async (event) => {
  // 获取粘贴的文本
  const pastedText = (event.clipboardData || window.clipboardData).getData('text')
  
  // 检查是否是IP地址
  if (ipv4Regex.test(pastedText) || ipv6Regex.test(pastedText)) {
    // 如果是IP地址，设置输入值并立即查询
    ipInput.value = pastedText
    await fetchIpInfo()
  }
}


const displayMap = (latitude, longitude) => {
  const map = L.map('map').setView([latitude, longitude], 13)
  L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
    attribution: '© OpenStreetMap contributors'
  }).addTo(map)
  L.marker([latitude, longitude]).addTo(map)
}

</script>