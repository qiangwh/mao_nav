<template>
  <!-- 锁定界面 -->
  <div v-if="isLocked && !isUnlocked" class="lock-container">
    <div class="lock-box">
      <h1>🔐 访问验证</h1>
      <p class="lock-description">此导航站已启用访问保护</p>
      <form @submit.prevent="handleUnlock">
        <div class="form-group">
          <label for="unlock-password">请输入访问密钥:</label>
          <input
            id="unlock-password"
            type="password"
            v-model="unlockPassword"
            placeholder="请输入访问密钥"
            required
            class="form-input"
          />
        </div>
        <button type="submit" class="unlock-btn" :disabled="unlocking">
          {{ unlocking ? '验证中...' : '进入导航' }}
        </button>
      </form>
      <div v-if="unlockError" class="error-message">
        {{ unlockError }}
      </div>
    </div>
  </div>

  <!-- 正常导航界面 -->
  <div v-else class="nav-home">
    <!-- 左侧边栏 -->
    <aside class="sidebar">
      <!-- Logo区域 -->
      <div class="logo-section">
        <div class="logo-badge" aria-hidden="true"><img src="/favicon.svg" alt="" class="logo-image" /></div>
        <h1 class="site-title">{{ title || '导航栏' }}</h1>
      </div>

      <!-- 分类导航 -->
      <nav class="category-nav">
        <h2 class="nav-title">分类导航</h2>
                <ul class="category-list">
          <li
            v-for="category in categories"
            :key="category.id"
            class="category-item"
            @click="scrollToCategory(category.id)"
          >
            <span class="category-icon">{{ category.icon }}</span>
            <span class="category-name">{{ category.name }}</span>
          </li>
        </ul>
      </nav>

    </aside>

    <!-- 右侧主内容区 -->
    <main class="main-content">
                  <!-- 顶部搜索栏 -->
      <header class="search-header">
        <div class="search-container">
          <div class="search-engine-selector">
            <img :src="searchEngines[selectedEngine].icon" :alt="selectedEngine" class="engine-logo" />
            <select v-model="selectedEngine" class="engine-select">
              <option value="google">Google</option>
              <option value="baidu">Baidu</option>
              <option value="bing">Bing</option>
              <option value="duckduckgo">DuckDuckGo</option>
            </select>
          </div>
          <input
            type="text"
            v-model="searchQuery"
            :placeholder="searchEngines[selectedEngine].placeholder"
            class="search-input"
            @keyup.enter="handleSearch"
          />
        </div>

        <!-- 主题切换按钮 -->
        <button class="theme-toggle-btn" @click="themeStore.toggleTheme" :title="themeStore.isDarkMode ? '切换到日间模式' : '切换到夜间模式'">
          <svg v-if="!themeStore.isDarkMode" width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
            <path d="M12 18C8.68629 18 6 15.3137 6 12C6 8.68629 8.68629 6 12 6C15.3137 6 18 8.68629 18 12C18 15.3137 15.3137 18 12 18ZM12 16C14.2091 16 16 14.2091 16 12C16 9.79086 14.2091 8 12 8C9.79086 8 8 9.79086 8 12C8 14.2091 9.79086 16 12 16ZM11 1H13V4H11V1ZM11 20H13V23H11V20ZM3.51472 4.92893L4.92893 3.51472L7.05025 5.63604L5.63604 7.05025L3.51472 4.92893ZM16.9497 18.364L18.364 16.9497L20.4853 19.0711L19.0711 20.4853L16.9497 18.364ZM19.0711 3.51472L20.4853 4.92893L18.364 7.05025L16.9497 5.63604L19.0711 3.51472ZM5.63604 16.9497L7.05025 18.364L4.92893 20.4853L3.51472 19.0711L5.63604 16.9497ZM23 11V13H20V11H23ZM4 11V13H1V11H4Z"/>
          </svg>
          <svg v-else width="20" height="20" viewBox="0 0 24 24" fill="currentColor">
            <path d="M10 7C10 10.866 13.134 14 17 14C18.9584 14 20.729 13.1957 21.9995 11.8995C22 11.933 22 11.9665 22 12C22 17.5228 17.5228 22 12 22C6.47715 22 2 17.5228 2 12C2 6.47715 6.47715 2 12 2C12.0335 2 12.067 2 12.1005 2.00049C10.8043 3.27098 10 5.04157 10 7ZM4 12C4 16.4183 7.58172 20 12 20C15.0583 20 17.7158 18.2839 19.062 15.7621C18.3945 15.9187 17.7035 16 17 16C12.0294 16 8 11.9706 8 7C8 6.29648 8.08133 5.60547 8.2379 4.938C5.71611 6.28423 4 8.9417 4 12Z"/>
          </svg>
        </button>

        <!-- 移动端菜单按钮 -->
        <button class="mobile-menu-btn" @click="toggleMobileMenu">
          <svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M3 12H21M3 6H21M3 18H21" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
          </svg>
        </button>

        <!-- 移动端分类菜单 -->
        <div class="mobile-menu" :class="{ active: showMobileMenu }">
          <div class="mobile-menu-header">
            <div class="header-left">
              <h3>分类导航</h3>
              <img :src="githubLogo" alt="GitHub" class="header-github-icon" @click="openGitHub" />
            </div>
            <button class="close-btn" @click="closeMobileMenu">×</button>
          </div>
                    <ul class="mobile-category-list">
            <li
              v-for="category in categories"
              :key="category.id"
              class="mobile-category-item"
              @click="scrollToCategoryMobile(category.id)"
            >
              <span class="category-icon">{{ category.icon }}</span>
              <span class="category-name">{{ category.name }}</span>
            </li>
          </ul>
        </div>

        <!-- 移动端菜单遮罩 -->
        <div class="mobile-menu-overlay" :class="{ active: showMobileMenu }" @click="closeMobileMenu"></div>
      </header>

      <!-- 导航内容区 -->
      <div class="content-area">
        <!-- 加载状态 -->
        <div v-if="loading" class="loading">
          <div class="loading-spinner"></div>
          <p>加载中...</p>
        </div>

        <!-- 错误状态 -->
        <div v-else-if="error" class="error">
          <p>{{ error }}</p>
          <button @click="fetchCategories" class="retry-btn">重试</button>
        </div>

                <!-- 分类内容 -->
        <div v-else class="categories-container">
          <section
            v-for="category in categories"
            :key="category.id"
            class="category-section"
            :id="`category-${category.id}`"
          >
            <h2 class="category-title">
              <span class="category-icon">{{ category.icon }}</span>
              <span class="category-name">{{ category.name }}</span>
            </h2>

            <div class="sites-grid">
              <a
                v-for="site in category.sites"
                :key="site.id"
                :href="site.url"
                target="_blank"
                rel="noopener noreferrer"
                class="site-card"
              >
                <div class="site-icon">
                  <img :src="site.icon" :alt="site.name" @error="handleImageError" />
                </div>
                <div class="site-info">
                  <h3 class="site-name">{{ site.name }}</h3>
                  <p class="site-description">{{ site.description }}</p>
                </div>
              </a>
            </div>
          </section>

          <!-- 页面底部信息 -->
          <footer class="page-footer" hidden="true">
            <div class="footer-content">
              <div class="footer-info">
                <h3>{{ title || '导航栏' }}</h3>
                <p>一个简洁、美观的导航网站，收录优质网站资源</p>
              </div>

              <div class="footer-links">
                <a
                  href="https://github.com/maodeyu180/mao_nav"
                  target="_blank"
                  rel="noopener noreferrer"
                  class="footer-link"
                >
                  <svg width="18" height="18" viewBox="0 0 24 24" fill="currentColor">
                    <path d="M12 0c-6.626 0-12 5.373-12 12 0 5.302 3.438 9.8 8.207 11.387.599.111.793-.261.793-.577v-2.234c-3.338.726-4.033-1.416-4.033-1.416-.546-1.387-1.333-1.756-1.333-1.756-1.089-.745.083-.729.083-.729 1.205.084 1.839 1.237 1.839 1.237 1.07 1.834 2.807 1.304 3.492.997.107-.775.418-1.305.762-1.604-2.665-.305-5.467-1.334-5.467-5.931 0-1.311.469-2.381 1.236-3.221-.124-.303-.535-1.524.117-3.176 0 0 1.008-.322 3.301 1.23.957-.266 1.983-.399 3.003-.404 1.02.005 2.047.138 3.006.404 2.291-1.552 3.297-1.23 3.297-1.23.653 1.653.242 2.874.118 3.176.77.84 1.235 1.911 1.235 3.221 0 4.609-2.807 5.624-5.479 5.921.43.372.823 1.102.823 2.222v3.293c0 .319.192.694.801.576 4.765-1.589 8.199-6.086 8.199-11.386 0-6.627-5.373-12-12-12z"/>
                  </svg>
                  开源项目
                </a>
              </div>
            </div>

            <div class="footer-bottom">
              <p>&copy; {{ new Date().getFullYear() }} 导航栏 - 由 <a href="https://github.com/maodeyu180" target="_blank" rel="noopener noreferrer">maodeyu180</a> 用 ❤️ 制作</p>
              <p class="footer-tech">基于 Vue.js 构建 | <a href="https://github.com/maodeyu180/mao_nav" target="_blank" rel="noopener noreferrer">查看源代码</a></p>
            </div>
          </footer>
        </div>
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { useNavigation } from '@/apis/useNavigation.js'
import { useThemeStore } from '@/stores/counter.js'
// 导入搜索引擎logo图片
import googleLogo from '@/assets/goolge.png'
import baiduLogo from '@/assets/baidu.png'
import bingLogo from '@/assets/bing.png'
import duckLogo from '@/assets/duck.png'
// 导入GitHub logo
import githubLogo from '@/assets/github.png'

// 使用导航API
const { categories, title, defaultSearchEngine, loading, error, fetchCategories } = useNavigation()

// 使用主题store
const themeStore = useThemeStore()

// 响应式数据
const searchQuery = ref('') // 搜索查询
const selectedEngine = ref('bing') // 选中的搜索引擎，初始值会在组件挂载后更新
const showMobileMenu = ref(false) // 移动端菜单显示状态

// 锁定功能相关
const isLocked = ref(false) // 是否启用锁定功能
const isUnlocked = ref(false) // 是否已解锁
const unlockPassword = ref('') // 解锁密码输入
const unlocking = ref(false) // 解锁中状态
const unlockError = ref('') // 解锁错误信息

// 搜索引擎配置
const searchEngines = {
  google: {
    url: 'https://www.google.com/search?q=',
    icon: googleLogo,
    placeholder: 'Google (点logo切换搜索引擎'
  },
  baidu: {
    url: 'https://www.baidu.com/s?wd=',
    icon: baiduLogo,
    placeholder: '百度一下(点logo切换搜索引擎'
  },
  bing: {
    url: 'https://www.bing.com/search?q=',
    icon: bingLogo,
    placeholder: 'Bing (点logo切换搜索引擎)'
  },
  duckduckgo: {
    url: 'https://duckduckgo.com/?q=',
    icon: duckLogo,
    placeholder: 'DuckDuckGo (点logo切换搜索引擎)'
  }
}

// 自定义固定时间滚动函数
const smoothScrollTo = (container, targetTop, duration = 600) => {
  const startTop = container.scrollTop
  const distance = targetTop - startTop
  let startTime = null

  const animateScroll = (currentTime) => {
    if (startTime === null) startTime = currentTime
    const timeElapsed = currentTime - startTime
    const progress = Math.min(timeElapsed / duration, 1)

    // 使用缓动函数 (easeInOutCubic)
    const ease = progress < 0.5
      ? 4 * progress * progress * progress
      : 1 - Math.pow(-2 * progress + 2, 3) / 2

    container.scrollTop = startTop + distance * ease

    if (progress < 1) {
      requestAnimationFrame(animateScroll)
    }
  }

  requestAnimationFrame(animateScroll)
}

// 滚动到指定分类
const scrollToCategory = (categoryId) => {
  const element = document.getElementById(`category-${categoryId}`)
  const container = document.querySelector('.content-area')

  if (element && container) {
    // 检查是否为移动端
    const isMobile = window.innerWidth <= 768

    let targetTop = 0

    if (isMobile) {
      // 移动端：在 content-area 容器内滚动
      const elementOffsetTop = element.offsetTop
      const searchHeaderHeight = 80 // 固定高度，因为搜索框是fixed定位
      targetTop = elementOffsetTop - searchHeaderHeight
    } else {
      // 桌面端：在容器内滚动
      const searchHeader = document.querySelector('.search-header')
      const elementOffsetTop = element.offsetTop
      const searchHeaderHeight = searchHeader ? searchHeader.offsetHeight + 20 : 100
      targetTop = elementOffsetTop - searchHeaderHeight
    }

    // 使用固定时间滚动（600毫秒）
    smoothScrollTo(container, Math.max(0, targetTop), 600)
  }
}

// 检查是否启用锁定功能
const checkLockStatus = () => {
  const openLock = import.meta.env.VITE_OPEN_LOCK
  if (openLock && openLock.trim() !== '') {
    isLocked.value = true
    // 检查是否已经解锁过
    const savedUnlock = localStorage.getItem('nav_unlocked')
    if (savedUnlock === 'true') {
      isUnlocked.value = true
    }
  } else {
    isLocked.value = false
    isUnlocked.value = true // 如果没有启用锁定，默认为解锁状态
  }
}

// 处理解锁
const handleUnlock = async () => {
  unlocking.value = true
  unlockError.value = ''

    try {
    const adminPassword = import.meta.env.VITE_ADMIN_PASSWORD

    if (!adminPassword) {
      throw new Error('访问密钥未配置')
    }

    if (unlockPassword.value === adminPassword) {
      isUnlocked.value = true
      localStorage.setItem('nav_unlocked', 'true')
      unlockPassword.value = ''
      console.log('导航站解锁成功')
    } else {
      throw new Error('访问密钥错误，请重新输入')
    }
  } catch (error) {
    unlockError.value = error.message
  } finally {
    unlocking.value = false
  }
}

// 处理搜索
const handleSearch = () => {
  if (!searchQuery.value.trim()) return

  const engine = searchEngines[selectedEngine.value]
  const url = engine.url + encodeURIComponent(searchQuery.value)
  window.open(url, '_blank')
}

// 处理图片加载错误
const handleImageError = (event) => {
  // 设置默认的 favicon.ico 作为 fallback 图片
  event.target.src = '/favicon.ico'
  event.target.onerror = null // 防止无限循环
}

// 移动端菜单控制
const toggleMobileMenu = () => {
  showMobileMenu.value = !showMobileMenu.value
  // 控制body滚动
  if (showMobileMenu.value) {
    document.body.style.overflow = 'hidden'
  } else {
    document.body.style.overflow = ''
  }
}

const closeMobileMenu = () => {
  showMobileMenu.value = false
  // 恢复body滚动
  document.body.style.overflow = ''
}

// 移动端分类滚动
const scrollToCategoryMobile = (categoryId) => {
  closeMobileMenu() // 先关闭菜单

  // 稍微延迟一下再滚动，确保菜单关闭动画完成
  setTimeout(() => {
    scrollToCategory(categoryId)
  }, 200)
}

// 打开GitHub项目页面
const openGitHub = () => {
  window.open('https://github.com/maodeyu180/mao_nav', '_blank')
}

// 组件挂载时获取数据
onMounted(async () => {
  checkLockStatus() // 检查锁定状态
  await fetchCategories()
  // 设置默认搜索引擎
  selectedEngine.value = defaultSearchEngine.value
})

// 组件卸载时清理样式
onUnmounted(() => {
  // 确保卸载时恢复body滚动
  document.body.style.overflow = ''
})
</script>

<style scoped>
/* =========================================================================
   猫猫导航 · 优美慵懒暖调
   一套连贯样式：暖纸背景 · 陶土主色 · 衬体标题 · 柔和卡片 · 缓慢过渡
   颜色全部走 base.css 的令牌，日/夜自动切换。
   ========================================================================= */

/* ---------- 布局骨架 ---------- */
.nav-home {
  display: flex;
  min-height: 100vh;
  position: relative;
  isolation: isolate;
}

/* 午后光影 —— 两团暖光极缓漂移，全页唯一点睛 */
.nav-home::before,
.nav-home::after {
  content: '';
  position: fixed;
  z-index: -1;
  pointer-events: none;
  will-change: transform;
}

.nav-home::before {
  inset: -12% 0 auto 0;
  height: 62vh;
  background: radial-gradient(56% 100% at 82% -8%, rgba(193, 122, 82, 0.18), transparent 72%);
  animation: drift-a 34s ease-in-out infinite alternate;
}

.nav-home::after {
  inset: auto 0 -18% 0;
  height: 52vh;
  background: radial-gradient(52% 100% at 12% 112%, rgba(134, 148, 113, 0.14), transparent 70%);
  animation: drift-b 46s ease-in-out infinite alternate;
}

.dark .nav-home::before {
  background: radial-gradient(56% 100% at 82% -8%, rgba(215, 155, 114, 0.08), transparent 72%);
}

.dark .nav-home::after {
  background: radial-gradient(52% 100% at 12% 112%, rgba(159, 174, 138, 0.05), transparent 70%);
}

@keyframes drift-a {
  from { transform: translate3d(-3%, -1%, 0) scale(1); }
  to { transform: translate3d(4%, 2%, 0) scale(1.08); }
}

@keyframes drift-b {
  from { transform: translate3d(2%, 1%, 0) scale(1.05); }
  to { transform: translate3d(-4%, -2%, 0) scale(1); }
}

/* ---------- 侧栏 ---------- */
.sidebar {
  width: 276px;
  flex-shrink: 0;
  height: 100vh;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  background: var(--color-surface);
  backdrop-filter: blur(18px);
  -webkit-backdrop-filter: blur(18px);
  border-right: 1px solid var(--color-border);
}

.logo-section {
  display: flex;
  align-items: center;
  gap: 14px;
  padding: 26px 22px 20px;
}

.logo-badge {
  width: 50px;
  height: 50px;
  border-radius: 16px;
  display: grid;
  place-items: center;
  flex-shrink: 0;
  background: linear-gradient(150deg, var(--color-accent-soft), rgba(134, 148, 113, 0.18));
  border: 1px solid var(--color-border);
  box-shadow: var(--shadow-soft);
}

.logo-image {
  width: 30px;
  height: 30px;
  object-fit: contain;
  border-radius: 8px;
}

.site-title {
  margin: 0;
  font-family: var(--font-serif);
  font-size: 24px;
  font-weight: 500;
  letter-spacing: 0.01em;
  color: var(--color-heading);
}

.category-nav {
  flex: 1;
  overflow-y: auto;
  padding: 6px 14px 24px;
  scrollbar-width: thin;
  scrollbar-color: color-mix(in srgb, var(--color-accent) 40%, transparent) transparent;
}

.nav-title {
  margin: 14px 12px 10px;
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 0.22em;
  text-transform: uppercase;
  color: var(--color-text-muted);
}

.category-list {
  list-style: none;
  display: flex;
  flex-direction: column;
  gap: 2px;
  padding: 0;
  margin: 0;
}

.category-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 9px 12px;
  border-radius: 12px;
  cursor: pointer;
  color: var(--color-text);
  transition:
    background-color 0.35s var(--ease-lazy),
    color 0.35s var(--ease-lazy),
    transform 0.35s var(--ease-lazy);
}

.category-item:hover {
  background: var(--color-accent-soft);
  color: var(--color-accent-strong);
  transform: translateX(3px);
}

.category-item .category-icon {
  width: 30px;
  height: 30px;
  display: grid;
  place-items: center;
  font-size: 16px;
  border-radius: 9px;
  background: var(--color-surface-strong);
  border: 1px solid var(--color-border);
  flex-shrink: 0;
}

.category-name {
  font-size: 14.5px;
  font-weight: 500;
  color: inherit;
}

/* ---------- 主内容 ---------- */
.main-content {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
  height: 100vh;
  overflow: hidden;
}

/* ---------- 搜索栏 ---------- */
.search-header {
  position: sticky;
  top: 0;
  z-index: 100;
  display: flex;
  align-items: center;
  gap: 14px;
  padding: 18px 30px;
  background: color-mix(in srgb, var(--color-background) 72%, transparent);
  backdrop-filter: blur(18px);
  -webkit-backdrop-filter: blur(18px);
  border-bottom: 1px solid var(--color-border);
}

.search-container {
  flex: 1;
  max-width: 680px;
  margin: 0 auto;
  display: flex;
  align-items: center;
  border-radius: 999px;
  background: var(--color-surface-strong);
  border: 1px solid var(--color-border);
  box-shadow: var(--shadow-soft);
  overflow: hidden;
  transition:
    border-color 0.35s var(--ease-lazy),
    box-shadow 0.35s var(--ease-lazy);
}

.search-container:focus-within {
  border-color: color-mix(in srgb, var(--color-accent) 55%, transparent);
  box-shadow: 0 0 0 4px var(--color-accent-soft), var(--shadow-soft);
}

.search-engine-selector {
  position: relative;
  display: flex;
  align-items: center;
  align-self: stretch;
  padding: 0 4px 0 8px;
  cursor: pointer;
}

.engine-logo {
  width: 22px;
  height: 22px;
  margin: 8px;
  object-fit: contain;
  border-radius: 6px;
  pointer-events: none;
}

.engine-select {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  opacity: 0;
  cursor: pointer;
  border: none;
  outline: none;
  background: transparent;
}

.search-input {
  flex: 1;
  min-width: 0;
  border: none;
  outline: none;
  background: transparent;
  padding: 13px 18px 13px 4px;
  font-size: 15px;
  color: var(--color-text);
}

.search-input::placeholder {
  color: var(--color-text-muted);
}

/* 主题切换 / 移动端菜单按钮 */
.theme-toggle-btn,
.mobile-menu-btn {
  display: grid;
  place-items: center;
  width: 42px;
  height: 42px;
  flex-shrink: 0;
  border: 1px solid var(--color-border);
  border-radius: 50%;
  background: var(--color-surface-strong);
  color: var(--color-text);
  cursor: pointer;
  transition:
    transform 0.35s var(--ease-lazy),
    background-color 0.35s var(--ease-lazy),
    color 0.35s var(--ease-lazy);
}

.theme-toggle-btn:hover,
.mobile-menu-btn:hover {
  color: var(--color-accent-strong);
  background: var(--color-accent-soft);
  transform: translateY(-2px);
}

.mobile-menu-btn {
  display: none;
}

/* ---------- 内容区 ---------- */
.content-area {
  flex: 1;
  overflow-y: auto;
  padding: 46px 30px 140px;
  scrollbar-width: thin;
  scrollbar-color: color-mix(in srgb, var(--color-accent) 42%, transparent) transparent;
}

.content-area::-webkit-scrollbar {
  width: 10px;
}

.content-area::-webkit-scrollbar-thumb {
  background: color-mix(in srgb, var(--color-accent) 40%, transparent);
  border-radius: 999px;
  border: 3px solid transparent;
  background-clip: padding-box;
}

.categories-container {
  max-width: 1180px;
  margin: 0 auto;
}

/* ---------- 分类区块 ---------- */
.category-section {
  margin-bottom: 58px;
  animation: lazy-rise 0.95s var(--ease-lazy) both;
}

/* 轻微错峰入场，像午后慢慢铺开 */
.category-section:nth-of-type(1) { animation-delay: 0.03s; }
.category-section:nth-of-type(2) { animation-delay: 0.12s; }
.category-section:nth-of-type(3) { animation-delay: 0.21s; }
.category-section:nth-of-type(4) { animation-delay: 0.30s; }
.category-section:nth-of-type(5) { animation-delay: 0.39s; }
.category-section:nth-of-type(n + 6) { animation-delay: 0.46s; }

@keyframes lazy-rise {
  from { opacity: 0; transform: translateY(18px); }
  to { opacity: 1; transform: translateY(0); }
}

.category-title {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-bottom: 28px;
  font-family: var(--font-serif);
  font-size: 28px;
  font-weight: 500;
  letter-spacing: 0.01em;
  color: var(--color-heading);
}

.category-title .category-icon {
  width: 48px;
  height: 48px;
  display: grid;
  place-items: center;
  font-size: 24px;
  border-radius: 16px;
  flex-shrink: 0;
  background: var(--color-surface);
  border: 1px solid var(--color-border);
  box-shadow: var(--shadow-soft);
}

.category-title .category-name {
  line-height: 1.25;
  font-size: 28px;
  font-weight: 500;
  letter-spacing: 0.01em;
}

/* 标题后延展的细线 —— 编辑式点缀 */
.category-title::after {
  content: '';
  flex: 1;
  height: 1px;
  margin-left: 4px;
  background: linear-gradient(90deg, var(--color-border), transparent 78%);
}

/* ---------- 站点卡片 ---------- */
.sites-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(258px, 1fr));
  gap: 18px;
}

.site-card {
  position: relative;
  isolation: isolate;
  overflow: hidden;
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 18px 20px;
  border-radius: var(--radius-md);
  text-decoration: none;
  color: inherit;
  background: var(--color-surface-strong);
  border: 1px solid var(--color-border);
  box-shadow: 0 10px 30px rgba(74, 55, 36, 0.05);
  transition:
    transform 0.55s var(--ease-lazy),
    box-shadow 0.55s var(--ease-lazy),
    border-color 0.55s var(--ease-lazy);
}

/* 悬停时极淡暖晕从右上角缓缓浮现 */
.site-card::before {
  content: '';
  position: absolute;
  inset: 0;
  z-index: -1;
  pointer-events: none;
  background: radial-gradient(130% 150% at 88% -25%, var(--color-accent-soft), transparent 62%);
  opacity: 0;
  transition: opacity 0.55s var(--ease-lazy);
}

.site-card:hover {
  transform: translateY(-3px);
  border-color: color-mix(in srgb, var(--color-accent) 38%, transparent);
  box-shadow: 0 26px 50px rgba(74, 55, 36, 0.13);
}

.site-card:hover::before {
  opacity: 1;
}

.site-icon {
  position: relative;
  width: 46px;
  height: 46px;
  flex-shrink: 0;
  display: grid;
  place-items: center;
  border-radius: 14px;
  overflow: hidden;
  background: var(--color-surface);
  border: 1px solid var(--color-border);
  transition: transform 0.55s var(--ease-lazy);
}

.site-card:hover .site-icon {
  transform: scale(1.05) rotate(-1.5deg);
}

.site-icon img {
  width: 28px;
  height: 28px;
  object-fit: contain;
}

.site-info {
  position: relative;
  flex: 1;
  min-width: 0;
}

.site-name {
  font-size: 16px;
  font-weight: 600;
  color: var(--color-heading);
  margin: 0 0 3px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.site-description {
  font-size: 13px;
  color: var(--color-text-muted);
  margin: 0;
  line-height: 1.45;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* ---------- 加载 / 错误 ---------- */
.loading,
.error {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 14px;
  min-height: 240px;
  color: var(--color-text-muted);
}

.loading-spinner {
  width: 38px;
  height: 38px;
  border: 3px solid var(--color-border);
  border-top-color: var(--color-accent);
  border-radius: 50%;
  animation: spin 0.9s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.retry-btn {
  padding: 9px 20px;
  border: 1px solid var(--color-border);
  border-radius: 999px;
  background: var(--color-accent-soft);
  color: var(--color-accent-strong);
  cursor: pointer;
  transition: background-color 0.3s var(--ease-lazy);
}

.retry-btn:hover {
  background: color-mix(in srgb, var(--color-accent) 22%, transparent);
}

/* ---------- 移动端菜单 ---------- */
.mobile-menu {
  position: fixed;
  top: 0;
  right: -100%;
  width: 262px;
  height: 100vh;
  z-index: 1001;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  background: var(--color-background-soft);
  border-left: 1px solid var(--color-border);
  box-shadow: -22px 0 60px rgba(74, 55, 36, 0.18);
  transition: right 0.4s var(--ease-lazy);
}

.mobile-menu.active {
  right: 0;
}

.mobile-menu-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 20px;
  flex-shrink: 0;
  border-bottom: 1px solid var(--color-border);
}

.header-left {
  display: flex;
  align-items: center;
  gap: 12px;
}

.mobile-menu-header h3 {
  margin: 0;
  font-family: var(--font-serif);
  font-size: 19px;
  font-weight: 500;
  color: var(--color-heading);
}

.header-github-icon {
  width: 26px;
  height: 26px;
  border-radius: 7px;
  cursor: pointer;
  opacity: 0.85;
  transition: transform 0.3s var(--ease-lazy), opacity 0.3s ease;
}

.header-github-icon:hover {
  opacity: 1;
  transform: scale(1.08);
}

.close-btn {
  width: 32px;
  height: 32px;
  display: grid;
  place-items: center;
  border: none;
  background: transparent;
  border-radius: 9px;
  font-size: 22px;
  line-height: 1;
  color: var(--color-text);
  cursor: pointer;
  transition: background-color 0.3s ease, color 0.3s ease;
}

.close-btn:hover {
  background: var(--color-accent-soft);
  color: var(--color-accent-strong);
}

.mobile-category-list {
  list-style: none;
  flex: 1;
  overflow-y: auto;
  margin: 0;
  padding: 10px 12px 160px;
}

.mobile-category-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 13px 12px;
  border-radius: 12px;
  cursor: pointer;
  color: var(--color-text);
  transition: background-color 0.3s var(--ease-lazy), color 0.3s ease;
}

.mobile-category-item:hover {
  background: var(--color-accent-soft);
  color: var(--color-accent-strong);
}

.mobile-category-item .category-icon {
  width: 32px;
  height: 32px;
  display: grid;
  place-items: center;
  font-size: 17px;
  border-radius: 9px;
  flex-shrink: 0;
  background: var(--color-surface-strong);
  border: 1px solid var(--color-border);
}

.mobile-category-item .category-name {
  font-size: 15px;
  font-weight: 500;
  color: inherit;
}

.mobile-menu-overlay {
  position: fixed;
  inset: 0;
  z-index: 999;
  background: rgba(33, 28, 22, 0.42);
  backdrop-filter: blur(2px);
  -webkit-backdrop-filter: blur(2px);
  opacity: 0;
  visibility: hidden;
  transition: opacity 0.4s var(--ease-lazy), visibility 0.4s ease;
}

.mobile-menu-overlay.active {
  opacity: 1;
  visibility: visible;
}

.dark .mobile-menu-overlay {
  background: rgba(0, 0, 0, 0.56);
}

/* ---------- 锁屏 ---------- */
.lock-container {
  position: fixed;
  inset: 0;
  z-index: 9999;
  display: grid;
  place-items: center;
  padding: 20px;
  background:
    radial-gradient(1000px 620px at 22% -4%, rgba(193, 122, 82, 0.18), transparent 60%),
    linear-gradient(180deg, var(--color-background-soft), var(--color-background));
}

.dark .lock-container {
  background:
    radial-gradient(1000px 620px at 22% -4%, rgba(215, 155, 114, 0.16), transparent 60%),
    linear-gradient(180deg, var(--color-background-soft), var(--color-background));
}

.lock-box {
  width: 100%;
  max-width: 400px;
  text-align: center;
  padding: 40px 36px;
  background: var(--color-surface-strong);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-xl);
  backdrop-filter: blur(18px);
  -webkit-backdrop-filter: blur(18px);
}

.lock-box h1 {
  margin: 0 0 10px;
  font-family: var(--font-serif);
  font-size: 27px;
  font-weight: 500;
  color: var(--color-heading);
}

.lock-description {
  margin: 0 0 28px;
  color: var(--color-text-muted);
  font-size: 15px;
}

.lock-box .form-group {
  margin-bottom: 18px;
  text-align: left;
}

.lock-box .form-group label {
  display: block;
  margin-bottom: 8px;
  font-size: 13px;
  font-weight: 500;
  color: var(--color-text);
}

.lock-box .form-input {
  width: 100%;
  padding: 12px 16px;
  font-size: 15px;
  border: 1px solid var(--color-border);
  border-radius: 12px;
  background: var(--color-surface);
  color: var(--color-text);
  transition: border-color 0.3s ease, box-shadow 0.3s ease;
}

.lock-box .form-input:focus {
  outline: none;
  border-color: color-mix(in srgb, var(--color-accent) 55%, transparent);
  box-shadow: 0 0 0 4px var(--color-accent-soft);
}

.unlock-btn {
  width: 100%;
  margin-top: 8px;
  padding: 13px 24px;
  border: none;
  border-radius: 12px;
  background: var(--color-accent-strong);
  color: #fff;
  font-size: 15px;
  font-weight: 600;
  cursor: pointer;
  transition: transform 0.3s var(--ease-lazy), box-shadow 0.3s ease, background-color 0.3s ease;
}

.unlock-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  background: color-mix(in srgb, var(--color-accent-strong) 88%, #000);
  box-shadow: 0 14px 30px rgba(193, 122, 82, 0.3);
}

.unlock-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.dark .unlock-btn {
  background: var(--color-accent);
  color: #241f18;
}

.dark .unlock-btn:hover:not(:disabled) {
  background: var(--color-accent-strong);
}

.lock-box .error-message {
  margin-top: 14px;
  padding: 12px;
  border-radius: 10px;
  font-size: 13px;
  background: rgba(180, 70, 50, 0.1);
  color: #b4482f;
  border: 1px solid rgba(180, 70, 50, 0.24);
}

.dark .lock-box .error-message {
  background: rgba(220, 120, 90, 0.14);
  color: #e8a488;
  border-color: rgba(220, 120, 90, 0.3);
}

/* ---------- 页脚（默认 hidden，仅备用样式） ---------- */
.page-footer {
  margin-top: 50px;
  padding: 34px 28px;
  background: var(--color-surface);
  border: 1px solid var(--color-border);
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-soft);
}

.footer-content {
  display: flex;
  justify-content: space-between;
  gap: 28px;
  margin-bottom: 24px;
  flex-wrap: wrap;
}

.footer-info h3 {
  margin: 0 0 6px;
  font-family: var(--font-serif);
  font-size: 19px;
  color: var(--color-heading);
}

.footer-info p {
  margin: 0;
  color: var(--color-text-muted);
  font-size: 14px;
}

.footer-links {
  display: flex;
  gap: 12px;
}

.footer-link {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 9px 16px;
  border-radius: 999px;
  background: var(--color-surface-strong);
  border: 1px solid var(--color-border);
  color: var(--color-accent-strong);
  text-decoration: none;
  font-size: 14px;
  font-weight: 500;
  transition: transform 0.3s var(--ease-lazy), background-color 0.3s ease, color 0.3s ease;
}

.footer-link:hover {
  transform: translateY(-2px);
  background: var(--color-accent);
  color: #fff;
}

.dark .footer-link:hover {
  color: #241f18;
}

.footer-bottom {
  padding-top: 18px;
  border-top: 1px solid var(--color-border);
  text-align: center;
}

.footer-bottom p {
  margin: 4px 0;
  color: var(--color-text-muted);
  font-size: 13px;
}

.footer-bottom a {
  color: var(--color-accent-strong);
  text-decoration: none;
}

.footer-bottom a:hover {
  text-decoration: underline;
}

.footer-tech {
  font-size: 12px;
  opacity: 0.85;
}

/* ---------- 暗色微调（大多数已由令牌自动切换） ---------- */
.dark .site-card {
  box-shadow: 0 12px 32px rgba(0, 0, 0, 0.34);
}

.dark .site-card:hover {
  box-shadow: 0 24px 48px rgba(0, 0, 0, 0.44);
}

/* ---------- 响应式 ---------- */
@media (max-width: 768px) {
  .nav-home {
    flex-direction: column;
    height: 100vh;
    height: 100svh;
    overflow: hidden;
  }

  .sidebar {
    display: none;
  }

  .main-content {
    height: 100vh;
    height: 100svh;
  }

  .search-header {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    padding: 14px 16px;
  }

  .mobile-menu-btn {
    display: grid;
  }

  .search-container {
    max-width: none;
    margin: 0;
  }

  .content-area {
    padding: 92px 16px 200px;
    -webkit-overflow-scrolling: touch;
  }

  .sites-grid {
    grid-template-columns: 1fr 1fr;
    gap: 12px;
  }

  .site-card {
    flex-direction: column;
    text-align: center;
    gap: 10px;
    padding: 16px 12px;
  }

  .site-name {
    font-size: 14.5px;
    white-space: normal;
  }

  .site-description {
    font-size: 12px;
    white-space: normal;
  }

  .category-title {
    font-size: 21px;
    gap: 12px;
  }

  .category-title .category-name {
    font-size: 21px;
  }

  .category-title .category-icon {
    width: 40px;
    height: 40px;
    font-size: 21px;
  }
}

/* ---------- 降低动效偏好 ---------- */
@media (prefers-reduced-motion: reduce) {
  .nav-home::before,
  .nav-home::after {
    animation: none;
  }

  .category-section {
    animation: none;
  }

  .site-card,
  .category-item,
  .site-icon {
    transition: none;
  }
}
</style>
