<template>
  <div>
    <div class="from-blue-50 to-indigo-50">
      <!-- 背景 IP 文案 -->
      <div class="absolute inset-0 overflow-hidden select-none pointer-events-none">
        <div class="absolute inset-0">
          <div class="relative w-full h-full" style="transform: rotate(-45deg)">
            <!-- 使用网格布局创建固定间距的背景 -->
            <div class="absolute inset-[-100%] grid gap-16" style="grid-template-columns: repeat(auto-fill, 160px);">
              <!-- 大号 IP -->
              <div v-for="i in 100" :key="i" class="text-[12vh] font-black text-black/[0.02] whitespace-nowrap">
                IP
              </div>
              <!-- 中号 IP -->
              <div v-for="i in 100" :key="`m-${i}`" class="text-[8vh] font-black text-black/[0.015] whitespace-nowrap translate-x-8 translate-y-8">
                IP
              </div>
              <!-- 小号 IP -->
              <div v-for="i in 100" :key="`s-${i}`" class="text-[5vh] font-black text-black/[0.01] whitespace-nowrap translate-x-16 translate-y-16">
                IP
              </div>
            </div>
          </div>
        </div>
      </div>
      
      <main class="relative ">
        <IpInfoTable />
    <footer class="text-center text-sm text-gray-500 py-4 border-t mt-8">
      <div>踏上取经路比抵达灵山更重要™旗下网站，本站由 HTML 强力驱动！本站源代码：<a href="https://github.com" class="text-blue-500 hover:underline">github</a> | Copyright@牛逼IP查询 保留所有权利 | Powered&Developed by 牛逼IP查询</div>
      <div class="mt-2">法务支持与联系：<a href="mailto:comments@whitehouse.gov" class="text-blue-500 hover:underline">comments@whitehouse.gov</a> | Hosting by Cloudflare，受相关法律保护！<a href="#" class="text-blue-500 hover:underline">免责声明</a> | <a href="#" class="text-blue-500 hover:underline">DMCA</a> | <a href="#" class="text-blue-500 hover:underline">隐私政策</a></div>
    </footer>
      </main>
  </div>
  </div>
</template>

<script setup>
import IpInfoTable from './components/IpInfoTable.vue'
</script>