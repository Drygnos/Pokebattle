<script setup>
import { ref } from 'vue'

const searchQuery = ref('')
const pokemon = ref(null)
const loading = ref(false)
const error = ref('')

const searchPokemon = async () => {
  if (!searchQuery.value.trim()) {
    error.value = 'Veuillez entrer un nom de Pokémon'
    pokemon.value = null
    return
  }

  loading.value = true
  error.value = ''
  pokemon.value = null

  try {
    const response = await fetch(
      `https://pokeapi.co/api/v2/pokemon/${searchQuery.value.toLowerCase()}`
    )

    if (!response.ok) {
      throw new Error('Pokémon non trouvé')
    }

    const data = await response.json()
    pokemon.value = {
      name: data.name.charAt(0).toUpperCase() + data.name.slice(1),
      id: data.id,
      image: data.sprites.other['official-artwork'].front_default,
      types: data.types.map(t => t.type.name),
      height: data.height / 10,
      weight: data.weight / 10,
      stats: data.stats.map(s => ({
        name: s.stat.name,
        value: s.base_stat
      }))
    }
  } catch (err) {
    error.value = err.message
    pokemon.value = null
  } finally {
    loading.value = false
  }
}

const handleKeyPress = (event) => {
  if (event.key === 'Enter') {
    searchPokemon()
  }
}
</script>

<template>
  <div class="pokemon-container">
    <h1>Pokédex</h1>

    <div class="search-box">
      <input
        v-model="searchQuery"
        type="text"
        placeholder="Entrez le nom d'un Pokémon..."
        @keypress="handleKeyPress"
        class="search-input"
      />
      <button @click="searchPokemon" class="search-button" :disabled="loading">
        {{ loading ? 'Recherche...' : 'Rechercher' }}
      </button>
    </div>

    <div v-if="error" class="error-message">
      ❌ {{ error }}
    </div>

    <div v-if="pokemon" class="pokemon-card">
      <div class="pokemon-header">
        <h2>{{ pokemon.name }}</h2>
        <span class="pokemon-id">#{{ pokemon.id }}</span>
      </div>

      <img v-if="pokemon.image" :src="pokemon.image" :alt="pokemon.name" class="pokemon-image" />
      <div v-else class="no-image">Image non disponible</div>

      <div class="pokemon-types">
        <span v-for="type in pokemon.types" :key="type" class="type-badge">
          {{ type }}
        </span>
      </div>

      <div class="pokemon-info">
        <div class="info-item">
          <strong>Hauteur:</strong> {{ pokemon.height }} m
        </div>
        <div class="info-item">
          <strong>Poids:</strong> {{ pokemon.weight }} kg
        </div>
      </div>

      <div class="pokemon-stats">
        <h3>Statistiques</h3>
        <div v-for="stat in pokemon.stats" :key="stat.name" class="stat">
          <div class="stat-name">{{ stat.name }}</div>
          <div class="stat-bar">
            <div class="stat-fill" :style="{ width: (stat.value / 255) * 100 + '%' }"></div>
          </div>
          <div class="stat-value">{{ stat.value }}</div>
        </div>
      </div>
    </div>

    <div v-if="!pokemon && !error && !loading" class="welcome-message">
      <p>🔍 Recherchez un Pokémon pour commencer!</p>
      <p class="examples">Exemples: ditto, pikachu, charizard</p>
    </div>
  </div>
</template>

<style scoped>
.pokemon-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 2rem 1.5rem;
  min-height: 100vh;
  background: linear-gradient(135deg, var(--pokemon-light) 0%, #f0f0f0 100%);
}

h1 {
  text-align: center;
  color: var(--pokemon-blue);
  margin-bottom: 2.5rem;
  font-size: clamp(2rem, 5vw, 3rem);
  font-weight: 800;
  letter-spacing: 1px;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.05);
}

.search-box {
  display: flex;
  gap: 1rem;
  margin-bottom: 2.5rem;
  flex-wrap: wrap;
}

.search-input {
  flex: 1;
  min-width: 200px;
  padding: 1rem 1.25rem;
  border: 2px solid #ddd;
  border-radius: 12px;
  font-size: 1rem;
  background-color: white;
  color: var(--color-text);
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.05);
}

.search-input:focus {
  outline: none;
  border-color: var(--pokemon-blue);
  box-shadow: 0 4px 16px rgba(59, 76, 202, 0.2);
  transform: translateY(-2px);
}

.search-input::placeholder {
  color: #999;
}

.search-button {
  padding: 1rem 2rem;
  background: linear-gradient(135deg, var(--pokemon-red), #d41010);
  color: white;
  border: none;
  border-radius: 12px;
  font-size: 1rem;
  font-weight: 700;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
  box-shadow: 0 4px 15px rgba(238, 21, 21, 0.3);
  min-width: 140px;
  letter-spacing: 0.5px;
}

.search-button:hover:not(:disabled) {
  transform: translateY(-3px);
  box-shadow: 0 8px 24px rgba(238, 21, 21, 0.4);
}

.search-button:active:not(:disabled) {
  transform: translateY(-1px);
}

.search-button:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

.error-message {
  background-color: rgba(244, 67, 54, 0.1);
  border: 2px solid #f44336;
  color: #c62828;
  padding: 1.5rem;
  border-radius: 12px;
  margin-bottom: 2rem;
  text-align: center;
  font-weight: 600;
  animation: shake 0.4s ease-in-out;
}

@keyframes shake {
  0%, 100% { transform: translateX(0); }
  25% { transform: translateX(-10px); }
  75% { transform: translateX(10px); }
}

.pokemon-card {
  background: white;
  border: 3px solid var(--pokemon-blue);
  border-radius: 20px;
  padding: 2rem;
  animation: slideIn 0.4s cubic-bezier(0.34, 1.56, 0.64, 1);
  box-shadow: 0 12px 32px rgba(59, 76, 202, 0.15);
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(20px) scale(0.95);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

.pokemon-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
  border-bottom: 3px solid var(--pokemon-yellow);
  padding-bottom: 1rem;
}

.pokemon-header h2 {
  margin: 0;
  color: var(--pokemon-blue);
  font-size: clamp(1.5rem, 4vw, 2rem);
  font-weight: 800;
}

.pokemon-id {
  color: var(--pokemon-red);
  font-weight: 700;
  font-size: clamp(1rem, 2vw, 1.3rem);
  background: rgba(238, 21, 21, 0.1);
  padding: 0.5rem 1rem;
  border-radius: 8px;
}

.pokemon-image {
  width: 100%;
  max-width: 300px;
  height: auto;
  display: block;
  margin: 2rem auto;
  background: linear-gradient(135deg, rgba(59, 76, 202, 0.05) 0%, rgba(255, 204, 0, 0.05) 100%);
  padding: 1.5rem;
  border-radius: 16px;
  filter: drop-shadow(0 6px 16px rgba(0, 0, 0, 0.1));
}

.no-image {
  text-align: center;
  padding: 3rem 1rem;
  color: #999;
  font-style: italic;
  font-size: 1.1rem;
}

.pokemon-types {
  display: flex;
  gap: 0.75rem;
  margin-bottom: 2rem;
  flex-wrap: wrap;
  justify-content: center;
}

.type-badge {
  background: linear-gradient(135deg, var(--pokemon-blue), #1e3c72);
  color: white;
  padding: 0.6rem 1.25rem;
  border-radius: 20px;
  font-weight: 700;
  font-size: 0.9rem;
  text-transform: capitalize;
  box-shadow: 0 4px 12px rgba(59, 76, 202, 0.2);
}

.pokemon-info {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.5rem;
  margin-bottom: 2rem;
  padding: 1.5rem;
  background: linear-gradient(135deg, rgba(59, 76, 202, 0.05) 0%, rgba(255, 204, 0, 0.05) 100%);
  border-radius: 12px;
  border: 2px solid rgba(59, 76, 202, 0.1);
}

.info-item {
  text-align: center;
}

.info-item strong {
  display: block;
  color: var(--pokemon-blue);
  margin-bottom: 0.5rem;
  font-size: 0.95rem;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.pokemon-stats {
  margin-top: 2.5rem;
  background: linear-gradient(135deg, rgba(238, 21, 21, 0.05) 0%, rgba(255, 204, 0, 0.05) 100%);
  padding: 1.5rem;
  border-radius: 12px;
  border: 2px solid rgba(238, 21, 21, 0.1);
}

.pokemon-stats h3 {
  margin: 0 0 1.5rem 0;
  color: var(--pokemon-red);
  font-size: 1.3rem;
  font-weight: 800;
  text-transform: uppercase;
  letter-spacing: 1px;
}

.stat {
  display: grid;
  grid-template-columns: 100px 1fr 60px;
  gap: 1rem;
  align-items: center;
  margin-bottom: 1.2rem;
}

.stat-name {
  font-weight: 700;
  text-transform: capitalize;
  font-size: 0.95rem;
  color: var(--pokemon-blue);
}

.stat-bar {
  background-color: #e0e0e0;
  border: 2px solid var(--pokemon-blue);
  border-radius: 8px;
  height: 24px;
  overflow: hidden;
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.1);
}

.stat-fill {
  background: linear-gradient(90deg, var(--pokemon-red), var(--pokemon-yellow));
  height: 100%;
  transition: width 0.6s cubic-bezier(0.34, 1.56, 0.64, 1);
  box-shadow: inset 0 1px 3px rgba(0, 0, 0, 0.2);
}

.stat-value {
  text-align: right;
  font-weight: 700;
  font-size: 1rem;
  color: var(--pokemon-blue);
  min-width: 60px;
}

.welcome-message {
  text-align: center;
  padding: 3rem 1.5rem;
  color: #666;
}

.welcome-message p {
  margin: 1rem 0;
  font-size: clamp(1.1rem, 3vw, 1.4rem);
  font-weight: 600;
}

.examples {
  font-size: 1rem;
  color: #999;
  opacity: 0.9;
  margin-top: 1rem;
}

@media (max-width: 768px) {
  .pokemon-container {
    padding: 1.5rem 1rem;
  }

  .search-box {
    flex-direction: column;
    gap: 0.75rem;
  }

  .search-input,
  .search-button {
    width: 100%;
  }

  .pokemon-card {
    padding: 1.5rem;
    border-radius: 16px;
  }

  .pokemon-info {
    grid-template-columns: 1fr;
    gap: 1rem;
  }

  .stat {
    grid-template-columns: 80px 1fr 50px;
    gap: 0.75rem;
    margin-bottom: 1rem;
  }
}

@media (max-width: 480px) {
  .pokemon-container {
    padding: 1rem;
  }

  .pokemon-header {
    flex-direction: column;
    gap: 0.75rem;
    text-align: center;
  }

  .stat-name {
    font-size: 0.85rem;
  }

  .stat-value {
    min-width: 45px;
    font-size: 0.9rem;
  }
}

@media (min-width: 1024px) {
  .pokemon-container {
    padding: 3rem 2rem;
  }

  .pokemon-card {
    padding: 2.5rem;
  }
}
</style>
