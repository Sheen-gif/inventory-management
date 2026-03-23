<template>
  <div class="app-shell">

    <!-- ── Left Sidebar ── -->
    <aside :class="['sidebar', { collapsed: sidebarCollapsed }]">

      <!-- Brand + toggle -->
      <div class="sidebar-brand">
        <div class="brand-icon">C</div>
        <div v-show="!sidebarCollapsed" class="brand-text">
          <span class="brand-name">{{ t('nav.companyName') }}</span>
          <span class="brand-subtitle">{{ t('nav.subtitle') }}</span>
        </div>
        <button
          class="sidebar-toggle"
          @click="toggleSidebar"
          :title="sidebarCollapsed ? 'Expand sidebar' : 'Collapse sidebar'"
        >
          <svg width="14" height="14" viewBox="0 0 14 14" fill="none">
            <path d="M9 2L4 7L9 12" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/>
          </svg>
        </button>
      </div>

      <!-- Navigation -->
      <nav class="sidebar-nav">
        <router-link class="nav-item" to="/" :class="{ active: $route.path === '/' }" :title="sidebarCollapsed ? t('nav.overview') : undefined">
          <svg class="nav-icon" width="16" height="16" viewBox="0 0 16 16" fill="none"><rect x="1" y="1" width="6" height="6" rx="1.5" stroke="currentColor" stroke-width="1.5"/><rect x="9" y="1" width="6" height="6" rx="1.5" stroke="currentColor" stroke-width="1.5"/><rect x="1" y="9" width="6" height="6" rx="1.5" stroke="currentColor" stroke-width="1.5"/><rect x="9" y="9" width="6" height="6" rx="1.5" stroke="currentColor" stroke-width="1.5"/></svg>
          <span v-show="!sidebarCollapsed" class="nav-label">{{ t('nav.overview') }}</span>
        </router-link>

        <router-link class="nav-item" to="/inventory" :class="{ active: $route.path === '/inventory' }" :title="sidebarCollapsed ? t('nav.inventory') : undefined">
          <svg class="nav-icon" width="16" height="16" viewBox="0 0 16 16" fill="none"><path d="M2 5.5L8 2.5L14 5.5V10.5L8 13.5L2 10.5V5.5Z" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/><path d="M8 2.5V13.5M2 5.5L8 8.5L14 5.5" stroke="currentColor" stroke-width="1.5" stroke-linejoin="round"/></svg>
          <span v-show="!sidebarCollapsed" class="nav-label">{{ t('nav.inventory') }}</span>
        </router-link>

        <router-link class="nav-item" to="/orders" :class="{ active: $route.path === '/orders' }" :title="sidebarCollapsed ? t('nav.orders') : undefined">
          <svg class="nav-icon" width="16" height="16" viewBox="0 0 16 16" fill="none"><path d="M2 4H14M2 8H10M2 12H7" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/></svg>
          <span v-show="!sidebarCollapsed" class="nav-label">{{ t('nav.orders') }}</span>
        </router-link>

        <router-link class="nav-item" to="/spending" :class="{ active: $route.path === '/spending' }" :title="sidebarCollapsed ? t('nav.finance') : undefined">
          <svg class="nav-icon" width="16" height="16" viewBox="0 0 16 16" fill="none"><path d="M2 12L6 7L9 10L12 5L14 7" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/><path d="M2 14H14" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/></svg>
          <span v-show="!sidebarCollapsed" class="nav-label">{{ t('nav.finance') }}</span>
        </router-link>

        <router-link class="nav-item" to="/demand" :class="{ active: $route.path === '/demand' }" :title="sidebarCollapsed ? t('nav.demandForecast') : undefined">
          <svg class="nav-icon" width="16" height="16" viewBox="0 0 16 16" fill="none"><path d="M2 12L6 8L9 10.5L14 4" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/><path d="M11 4H14V7" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></svg>
          <span v-show="!sidebarCollapsed" class="nav-label">{{ t('nav.demandForecast') }}</span>
        </router-link>

        <router-link class="nav-item" to="/restocking" :class="{ active: $route.path === '/restocking' }" :title="sidebarCollapsed ? t('nav.restocking') : undefined">
          <svg class="nav-icon" width="16" height="16" viewBox="0 0 16 16" fill="none"><path d="M13.5 8A5.5 5.5 0 1 1 8 2.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/><path d="M8 1L11 4L8 7" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round"/></svg>
          <span v-show="!sidebarCollapsed" class="nav-label">{{ t('nav.restocking') }}</span>
        </router-link>

        <router-link class="nav-item" to="/reports" :class="{ active: $route.path === '/reports' }" :title="sidebarCollapsed ? 'Reports' : undefined">
          <svg class="nav-icon" width="16" height="16" viewBox="0 0 16 16" fill="none"><rect x="2" y="1.5" width="12" height="13" rx="1.5" stroke="currentColor" stroke-width="1.5"/><path d="M5 5.5H11M5 8.5H11M5 11.5H8.5" stroke="currentColor" stroke-width="1.5" stroke-linecap="round"/></svg>
          <span v-show="!sidebarCollapsed" class="nav-label">Reports</span>
        </router-link>
      </nav>

      <!-- Sidebar Footer -->
      <div class="sidebar-footer">
        <LanguageSwitcher />
        <ProfileMenu
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
      </div>

    </aside>

    <!-- ── Main Area ── -->
    <div class="main-area">
      <div class="content-topbar">
        <FilterBar />
      </div>
      <main class="main-content">
        <router-view />
      </main>
    </div>

    <!-- Modals — unchanged -->
    <ProfileDetailsModal
      :is-open="showProfileDetails"
      @close="showProfileDetails = false"
    />

    <TasksModal
      :is-open="showTasks"
      :tasks="tasks"
      @close="showTasks = false"
      @add-task="addTask"
      @delete-task="deleteTask"
      @toggle-task="toggleTask"
    />

  </div>
</template>

<script>
import { ref, onMounted, onUnmounted, computed } from 'vue'
import { api } from './api'
import { useAuth } from './composables/useAuth'
import { useI18n } from './composables/useI18n'
import FilterBar from './components/FilterBar.vue'
import ProfileMenu from './components/ProfileMenu.vue'
import ProfileDetailsModal from './components/ProfileDetailsModal.vue'
import TasksModal from './components/TasksModal.vue'
import LanguageSwitcher from './components/LanguageSwitcher.vue'

export default {
  name: 'App',
  components: {
    FilterBar,
    ProfileMenu,
    ProfileDetailsModal,
    TasksModal,
    LanguageSwitcher
  },
  setup() {
    const { currentUser } = useAuth()
    const { t } = useI18n()
    const showProfileDetails = ref(false)
    const showTasks = ref(false)

    // Sidebar collapse — auto-collapses on narrow viewports
    const sidebarCollapsed = ref(window.innerWidth < 1024)

    const toggleSidebar = () => {
      sidebarCollapsed.value = !sidebarCollapsed.value
    }

    const handleResize = () => {
      sidebarCollapsed.value = window.innerWidth < 1024
    }

    onUnmounted(() => {
      window.removeEventListener('resize', handleResize)
    })

    const apiTasks = ref([])

    // Merge mock tasks from currentUser with API tasks
    const tasks = computed(() => {
      return [...currentUser.value.tasks, ...apiTasks.value]
    })

    const loadTasks = async () => {
      try {
        apiTasks.value = await api.getTasks()
      } catch (err) {
        console.error('Failed to load tasks:', err)
      }
    }

    const addTask = async (taskData) => {
      try {
        const newTask = await api.createTask(taskData)
        // Add new task to the beginning of the array
        apiTasks.value.unshift(newTask)
      } catch (err) {
        console.error('Failed to add task:', err)
      }
    }

    const deleteTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const isMockTask = currentUser.value.tasks.some(t => t.id === taskId)

        if (isMockTask) {
          // Remove from mock tasks
          const index = currentUser.value.tasks.findIndex(t => t.id === taskId)
          if (index !== -1) {
            currentUser.value.tasks.splice(index, 1)
          }
        } else {
          // Remove from API tasks
          await api.deleteTask(taskId)
          apiTasks.value = apiTasks.value.filter(t => t.id !== taskId)
        }
      } catch (err) {
        console.error('Failed to delete task:', err)
      }
    }

    const toggleTask = async (taskId) => {
      try {
        // Check if it's a mock task (from currentUser)
        const mockTask = currentUser.value.tasks.find(t => t.id === taskId)

        if (mockTask) {
          // Toggle mock task status
          mockTask.status = mockTask.status === 'pending' ? 'completed' : 'pending'
        } else {
          // Toggle API task
          const updatedTask = await api.toggleTask(taskId)
          const index = apiTasks.value.findIndex(t => t.id === taskId)
          if (index !== -1) {
            apiTasks.value[index] = updatedTask
          }
        }
      } catch (err) {
        console.error('Failed to toggle task:', err)
      }
    }

    onMounted(() => {
      loadTasks()
      window.addEventListener('resize', handleResize)
    })

    return {
      t,
      showProfileDetails,
      showTasks,
      tasks,
      addTask,
      deleteTask,
      toggleTask,
      sidebarCollapsed,
      toggleSidebar
    }
  }
}
</script>

<style>
/* ── Design Tokens ── */
:root {
  --sidebar-width: 232px;
  --sidebar-bg: #0f172a;
  --sidebar-border: rgba(255, 255, 255, 0.07);
  --sidebar-text: #94a3b8;
  --sidebar-text-active: #f1f5f9;
  --sidebar-hover-bg: rgba(255, 255, 255, 0.06);
  --sidebar-active-bg: rgba(255, 255, 255, 0.1);
  --sidebar-active-indicator: #3b82f6;

  --content-bg: #f8fafc;
  --topbar-bg: #ffffff;
  --topbar-border: #e2e8f0;

  --card-bg: #ffffff;
  --card-border: #e2e8f0;
  --card-radius: 12px;
  --card-shadow: 0 1px 3px rgba(0, 0, 0, 0.05), 0 1px 2px rgba(0, 0, 0, 0.04);

  --text-primary: #0f172a;
  --text-secondary: #64748b;
  --text-muted: #94a3b8;

  --accent: #2563eb;
  --content-padding: 28px;
}

/* ── Reset ── */
*, *::before, *::after {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html, body {
  height: 100%;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: var(--content-bg);
  color: var(--text-primary);
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

/* ── App Shell ── */
.app-shell {
  display: flex;
  height: 100vh;
  overflow: hidden;
}

/* ── Sidebar ── */
.sidebar {
  width: var(--sidebar-width);
  min-width: var(--sidebar-width);
  background: var(--sidebar-bg);
  display: flex;
  flex-direction: column;
  height: 100vh;
  overflow: hidden;
  border-right: 1px solid var(--sidebar-border);
  flex-shrink: 0;
  /* Smooth width transition for collapse/expand */
  transition: width 0.22s cubic-bezier(0.4, 0, 0.2, 1),
              min-width 0.22s cubic-bezier(0.4, 0, 0.2, 1);
}

/* Icons-only collapsed state */
.sidebar.collapsed {
  width: 52px;
  min-width: 52px;
}

.sidebar-brand {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 18px 14px 16px;
  border-bottom: 1px solid var(--sidebar-border);
  flex-shrink: 0;
}

/* Toggle button — sits at right of brand row */
.sidebar-toggle {
  margin-left: auto;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 22px;
  height: 22px;
  background: transparent;
  border: none;
  border-radius: 4px;
  color: var(--sidebar-text);
  cursor: pointer;
  transition: background 0.12s, color 0.12s, transform 0.22s cubic-bezier(0.4, 0, 0.2, 1);
  padding: 0;
}

.sidebar-toggle:hover {
  background: var(--sidebar-hover-bg);
  color: var(--sidebar-text-active);
}

/* Flip chevron when collapsed */
.sidebar.collapsed .sidebar-toggle {
  transform: rotate(180deg);
}

.brand-icon {
  width: 30px;
  height: 30px;
  border-radius: 8px;
  background: var(--accent);
  color: #fff;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 800;
  font-size: 0.85rem;
  flex-shrink: 0;
  letter-spacing: -0.02em;
}

.brand-text {
  display: flex;
  flex-direction: column;
  gap: 1px;
  min-width: 0;
}

.brand-name {
  font-size: 0.82rem;
  font-weight: 700;
  color: var(--sidebar-text-active);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  letter-spacing: -0.01em;
}

.brand-subtitle {
  font-size: 0.63rem;
  color: var(--sidebar-text);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

/* Nav */
.sidebar-nav {
  flex: 1;
  overflow-y: auto;
  padding: 10px 8px;
  display: flex;
  flex-direction: column;
  gap: 1px;
}

.sidebar-nav::-webkit-scrollbar { width: 0; }

.nav-item {
  display: flex;
  align-items: center;
  gap: 9px;
  padding: 7px 10px;
  border-radius: 7px;
  color: var(--sidebar-text);
  text-decoration: none;
  font-size: 0.83rem;
  font-weight: 500;
  transition: background 0.12s, color 0.12s;
  white-space: nowrap;
  position: relative;
}

.nav-item:hover {
  background: var(--sidebar-hover-bg);
  color: var(--sidebar-text-active);
}

.nav-item.active {
  background: var(--sidebar-active-bg);
  color: var(--sidebar-text-active);
}

.nav-item.active::before {
  content: '';
  position: absolute;
  left: 0;
  top: 25%;
  bottom: 25%;
  width: 3px;
  border-radius: 0 3px 3px 0;
  background: var(--sidebar-active-indicator);
}

.nav-icon {
  flex-shrink: 0;
  opacity: 0.75;
}

.nav-item.active .nav-icon,
.nav-item:hover .nav-icon {
  opacity: 1;
}

.nav-label {
  overflow: hidden;
  text-overflow: ellipsis;
}

/* In collapsed mode, center the icon and remove the active left-bar indicator */
.sidebar.collapsed .nav-item {
  justify-content: center;
  padding: 8px;
}

.sidebar.collapsed .nav-item::before {
  display: none;
}

/* Sidebar footer */
.sidebar-footer {
  flex-shrink: 0;
  padding: 8px;
  border-top: 1px solid var(--sidebar-border);
  display: flex;
  flex-direction: column;
  gap: 2px;
}

/* Hide text labels in footer components when collapsed */
.sidebar.collapsed .language-label,
.sidebar.collapsed .profile-name,
.sidebar.collapsed .chevron {
  display: none;
}

.sidebar.collapsed .language-button,
.sidebar.collapsed .profile-button {
  justify-content: center;
  padding: 8px;
}

/* ── Main Area ── */
.main-area {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: hidden;
  min-width: 0;
}

.content-topbar {
  flex-shrink: 0;
  background: var(--topbar-bg);
  border-bottom: 1px solid var(--topbar-border);
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.04);
}

.main-content {
  flex: 1;
  overflow-y: auto;
  padding: var(--content-padding);
  background: var(--content-bg);
}

/* ── Page Header ── */
.page-header {
  margin-bottom: 24px;
}

.page-header h2 {
  font-size: 1.5rem;
  font-weight: 700;
  color: var(--text-primary);
  margin-bottom: 4px;
  letter-spacing: -0.02em;
}

.page-header p {
  color: var(--text-secondary);
  font-size: 0.875rem;
}

/* ── Stats Grid ── */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 16px;
  margin-bottom: 20px;
}

.stat-card {
  background: var(--card-bg);
  padding: 20px;
  border-radius: var(--card-radius);
  border: 1px solid var(--card-border);
  box-shadow: var(--card-shadow);
  transition: box-shadow 0.15s, border-color 0.15s;
}

.stat-card:hover {
  border-color: #cbd5e1;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.07);
}

.stat-label {
  color: var(--text-secondary);
  font-size: 0.72rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.07em;
  margin-bottom: 10px;
}

.stat-value {
  font-size: 2rem;
  font-weight: 700;
  color: var(--text-primary);
  letter-spacing: -0.03em;
  line-height: 1;
}

.stat-card.warning .stat-value { color: #ea580c; }
.stat-card.success .stat-value { color: #059669; }
.stat-card.danger  .stat-value { color: #dc2626; }
.stat-card.info    .stat-value { color: var(--accent); }

/* ── Cards ── */
.card {
  background: var(--card-bg);
  border-radius: var(--card-radius);
  padding: 20px;
  border: 1px solid var(--card-border);
  box-shadow: var(--card-shadow);
  margin-bottom: 20px;
}

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
  padding-bottom: 14px;
  border-bottom: 1px solid var(--card-border);
}

.card-title {
  font-size: 0.95rem;
  font-weight: 700;
  color: var(--text-primary);
  letter-spacing: -0.01em;
}

/* ── Tables ── */
.table-container {
  overflow-x: auto;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background: #f8fafc;
  border-top: 1px solid var(--card-border);
  border-bottom: 1px solid var(--card-border);
}

th {
  text-align: left;
  padding: 9px 12px;
  font-weight: 600;
  color: var(--text-secondary);
  font-size: 0.72rem;
  text-transform: uppercase;
  letter-spacing: 0.06em;
}

td {
  padding: 11px 12px;
  border-top: 1px solid #f1f5f9;
  color: #334155;
  font-size: 0.875rem;
}

tbody tr { transition: background-color 0.1s; }
tbody tr:hover { background: #f8fafc; }

/* ── Badges ── */
.badge {
  display: inline-block;
  padding: 3px 9px;
  border-radius: 6px;
  font-size: 0.7rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.04em;
}

.badge.success    { background: #dcfce7; color: #166534; }
.badge.warning    { background: #fef3c7; color: #92400e; }
.badge.danger     { background: #fee2e2; color: #991b1b; }
.badge.info       { background: #dbeafe; color: #1e40af; }
.badge.increasing { background: #dcfce7; color: #166534; }
.badge.decreasing { background: #fee2e2; color: #991b1b; }
.badge.stable     { background: #e0e7ff; color: #3730a3; }
.badge.high       { background: #fee2e2; color: #991b1b; }
.badge.medium     { background: #fef3c7; color: #92400e; }
.badge.low        { background: #dbeafe; color: #1e40af; }

/* ── States ── */
.loading {
  text-align: center;
  padding: 48px;
  color: var(--text-secondary);
  font-size: 0.875rem;
}

.error {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #991b1b;
  padding: 14px 16px;
  border-radius: 8px;
  margin: 12px 0;
  font-size: 0.875rem;
}
</style>
