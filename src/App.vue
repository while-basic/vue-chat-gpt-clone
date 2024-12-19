<template>
  <div class="app dark">
    <header>
      <div class="header-content">
        <div class="header-left">
          <i class="fas fa-robot logo-icon"></i>
          <h1>ChatGPT Clone</h1>
        </div>
        <div class="header-right">
          <div class="theme-badge">AI Assistant</div>
          <button class="mobile-menu-btn">
            <i class="fas fa-bars"></i>
          </button>
        </div>
      </div>
    </header>
    <main>
      <ChatInterface />
    </main>
  </div>
</template>

<script setup lang="ts">
import ChatInterface from './components/ChatInterface.vue'
</script>

<style>
@import url('https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.1/css/all.min.css');
@import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600&family=Fira+Code:wght@400;500&display=swap');

:root {
  /* Light theme */
  --primary-color: #2563eb;
  --primary-dark: #1d4ed8;
  --success-color: #059669;
  --success-dark: #047857;
  --background-color: #0f172a;
  --surface-color: #1e293b;
  --text-primary: #e2e8f0;
  --text-secondary: #94a3b8;
  --border-color: #334155;
}

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

html, body {
  height: 100%;
  overflow: hidden;
}

body {
  margin: 0;
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen,
    Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  background: var(--background-color);
  color: var(--text-primary);
  line-height: 1.5;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

header {
  background: var(--surface-color);
  color: var(--text-primary);
  padding: 1rem;
  border-bottom: 1px solid var(--border-color);
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 1000;
}

.header-content {
  max-width: 1400px;
  margin: 0 auto;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 2rem;
}

.header-left {
  display: flex;
  align-items: center;
  gap: 0.75rem;
}

.logo-icon {
  font-size: 1.5rem;
  color: var(--primary-color);
}

.header-right {
  display: flex;
  align-items: center;
  gap: 1rem;
}

h1 {
  margin: 0;
  font-size: 1.25rem;
  font-weight: 600;
}

.theme-badge {
  background: var(--primary-color);
  color: var(--surface-color);
  padding: 0.5rem 1rem;
  border-radius: 9999px;
  font-size: 0.875rem;
  font-weight: 500;
}

.mobile-menu-btn {
  display: none;
  background: none;
  border: none;
  color: var(--text-primary);
  font-size: 1.25rem;
  cursor: pointer;
  padding: 0.5rem;
}

main {
  flex: 1;
  padding-top: 4rem;
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100vh;
  overflow: hidden;
}

/* Tablet Styles */
@media (max-width: 1024px) {
  .header-content {
    padding: 0 1.5rem;
  }
}

/* Mobile Styles */
@media (max-width: 768px) {
  .theme-badge {
    display: none;
  }

  .mobile-menu-btn {
    display: block;
  }

  .header-content {
    padding: 0 1rem;
  }

  h1 {
    font-size: 1.125rem;
  }
}

/* Small Mobile Styles */
@media (max-width: 480px) {
  .header-left {
    gap: 0.5rem;
  }

  .logo-icon {
    font-size: 1.25rem;
  }

  .header-content {
    padding: 0 0.75rem;
  }
}

/* Handle iOS Safari viewport height */
@supports (-webkit-touch-callout: none) {
  .app {
    min-height: -webkit-fill-available;
  }
}
</style>
