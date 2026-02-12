<script setup>
defineProps({
  active: {
    type: String,
    required: true
  }
})

defineEmits(['navigate'])
</script>

<template>
  <nav class="navbar">
    <div class="nav-container">
      <span class="logo">Pokébattle</span>
      <div class="nav-buttons">
        <button
          @click="$emit('navigate', 'search')"
          class="nav-button"
          :class="{ active: active === 'search' }"
        >
          🔍 Recherche
        </button>
        <button
          @click="$emit('navigate', 'game')"
          class="nav-button"
          :class="{ active: active === 'game' }"
        >
          🎮 Stat Battle
        </button>
      </div>
    </div>
  </nav>
</template>

<style scoped>
.navbar {
  background: linear-gradient(135deg, white 0%, var(--pokemon-light) 100%);
  border-bottom: 4px solid var(--pokemon-red);
  padding: 0;
  position: sticky;
  top: 0;
  z-index: 100;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

.nav-container {
  max-width: 1400px;
  margin: 0 auto;
  padding: 1rem 1.5rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 2rem;
}

.logo {
  font-size: clamp(1.3rem, 3vw, 1.8rem);
  font-weight: 800;
  color: var(--pokemon-blue);
  letter-spacing: 1px;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.05);
}

.nav-buttons {
  display: flex;
  gap: 0.75rem;
}

.nav-button {
  background-color: transparent;
  border: 2px solid var(--pokemon-blue);
  padding: 0.7rem 1.5rem;
  border-radius: 12px;
  cursor: pointer;
  font-size: clamp(0.9rem, 2vw, 1rem);
  font-weight: 700;
  color: var(--pokemon-blue);
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
  white-space: nowrap;
  letter-spacing: 0.5px;
}

.nav-button:hover {
  border-color: var(--pokemon-red);
  color: var(--pokemon-red);
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(238, 21, 21, 0.2);
}

.nav-button.active {
  background: linear-gradient(135deg, var(--pokemon-red), #d41010);
  color: white;
  border-color: var(--pokemon-red);
  box-shadow: 0 6px 20px rgba(238, 21, 21, 0.3);
}

.nav-button.active:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 24px rgba(238, 21, 21, 0.4);
}

@media (max-width: 768px) {
  .nav-container {
    padding: 0.75rem 1rem;
    gap: 1rem;
  }

  .logo {
    font-size: 1.2rem;
  }

  .nav-button {
    padding: 0.6rem 1rem;
    font-size: 0.85rem;
  }
}

@media (max-width: 480px) {
  .nav-container {
    padding: 0.5rem 0.75rem;
  }

  .logo {
    font-size: 1rem;
  }

  .nav-button {
    padding: 0.5rem 0.75rem;
    font-size: 0.75rem;
  }

  .nav-buttons {
    gap: 0.5rem;
  }
}
</style>
