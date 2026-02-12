<script setup>
import { ref, computed, onMounted } from 'vue'

const pokemon1 = ref(null)
const pokemon2 = ref(null)
const currentStat = ref(null)
const score = ref(0)
const gameOver = ref(false)
const loading = ref(false)
const message = ref('')
const showStats = ref(false)

const statOptions = [
  { name: 'HP', key: 'hp' },
  { name: 'Attaque', key: 'attack' },
  { name: 'Défense', key: 'defense' },
  { name: 'Attaque Spé', key: 'special-attack' },
  { name: 'Défense Spé', key: 'special-defense' },
  { name: 'Vitesse', key: 'speed' }
]

const getRandomStat = () => {
  return statOptions[Math.floor(Math.random() * statOptions.length)]
}

const fetchPokemonList = async (limit = 1025) => {
  try {
    const response = await fetch(`https://pokeapi.co/api/v2/pokemon?limit=${limit}&offset=0`)
    const data = await response.json()
    return data.results.map((p, index) => ({
      name: p.name,
      id: index + 1
    }))
  } catch (error) {
    console.error('Erreur lors de la récupération de la liste:', error)
    return []
  }
}

const fetchPokemonData = async (nameOrId) => {
  try {
    const response = await fetch(`https://pokeapi.co/api/v2/pokemon/${nameOrId}`)
    const data = await response.json()

    const stats = {
      hp: data.stats.find(s => s.stat.name === 'hp')?.base_stat || 0,
      attack: data.stats.find(s => s.stat.name === 'attack')?.base_stat || 0,
      defense: data.stats.find(s => s.stat.name === 'defense')?.base_stat || 0,
      sp_atk: data.stats.find(s => s.stat.name === 'special-attack')?.base_stat || 0,
      sp_def: data.stats.find(s => s.stat.name === 'special-defense')?.base_stat || 0,
      speed: data.stats.find(s => s.stat.name === 'speed')?.base_stat || 0
    }

    return {
      name: data.name.charAt(0).toUpperCase() + data.name.slice(1),
      id: data.id,
      image: data.sprites.other['official-artwork'].front_default,
      stats: stats
    }
  } catch (error) {
    console.error('Erreur lors de la récupération du Pokémon:', error)
    return null
  }
}

const getRandomPokemon = async (allPokemon) => {
  const randomPokemon = allPokemon[Math.floor(Math.random() * allPokemon.length)]
  return await fetchPokemonData(randomPokemon.id)
}

const startGame = async () => {
  loading.value = true
  message.value = ''
  showStats.value = false
  score.value = 0

  try {
    const allPokemon = await fetchPokemonList()

    pokemon1.value = await getRandomPokemon(allPokemon)
    pokemon2.value = await getRandomPokemon(allPokemon)
    currentStat.value = getRandomStat()

    gameOver.value = false
  } catch (error) {
    message.value = 'Erreur lors du chargement du jeu'
  } finally {
    loading.value = false
  }
}

const getStatValue = (pokemon, stat) => {
  return pokemon.stats[stat.key] || 0
}

const nextRound = async () => {
  loading.value = true
  showStats.value = false

  try {
    const allPokemon = await fetchPokemonList()
    pokemon1.value = pokemon2.value
    pokemon2.value = await getRandomPokemon(allPokemon)
    currentStat.value = getRandomStat()
    message.value = ''
  } catch (error) {
    message.value = 'Erreur lors du chargement du prochain Pokémon'
  } finally {
    loading.value = false
  }
}

const checkAnswer = async (chosen) => {
  if (!pokemon1.value || !pokemon2.value || !currentStat.value) return

  const value1 = getStatValue(pokemon1.value, currentStat.value)
  const value2 = getStatValue(pokemon2.value, currentStat.value)

  showStats.value = true

  if (
    (chosen === 'left' && value1 > value2) ||
    (chosen === 'right' && value2 > value1)
  ) {
    score.value++
    message.value = '✅ Correct!'
    setTimeout(() => {
      nextRound()
    }, 2000)
  } else if (value1 === value2) {
    message.value = '🤝 Égalité! Essayez encore.'
    setTimeout(() => {
      nextRound()
    }, 2000)
  } else {
    message.value = '❌ Mauvaise réponse!'
    gameOver.value = true
  }
}

onMounted(() => {
  startGame()
})
</script>

<template>
  <div class="game-container">
    <div class="game-header">
      <h1>Pokémon Stat Battle</h1>
      <div class="score-board">
        <span class="score">Score: {{ score }}</span>
      </div>
    </div>

    <div v-if="gameOver" class="game-over">
      <h2>Game Over!</h2>
      <p class="final-score">Score final: {{ score }}</p>
      <button @click="startGame" class="restart-button">Rejouer</button>
    </div>

    <div v-else-if="loading" class="loading">
      <p>Chargement des Pokémon...</p>
    </div>

    <div v-else-if="pokemon1 && pokemon2 && currentStat" class="game-content">
      <div class="stat-challenge">
        <h2>Quel Pokémon a la plus grande valeur de <span class="stat-name">{{ currentStat.name }}</span>?</h2>
      </div>

      <div class="pokemon-comparison">
        <button
          @click="checkAnswer('left')"
          class="pokemon-option left-pokemon"
          :disabled="loading"
        >
          <div class="pokemon-display">
            <img v-if="pokemon1.image" :src="pokemon1.image" :alt="pokemon1.name" />
            <div class="pokemon-name">{{ pokemon1.name }}</div>
            <div v-if="showStats" class="pokemon-stat-value">{{ getStatValue(pokemon1, currentStat) }}</div>
          </div>
        </button>

        <div class="vs-divider">
          <span class="vs-text">VS</span>
        </div>

        <button
          @click="checkAnswer('right')"
          class="pokemon-option right-pokemon"
          :disabled="loading"
        >
          <div class="pokemon-display">
            <img v-if="pokemon2.image" :src="pokemon2.image" :alt="pokemon2.name" />
            <div class="pokemon-name">{{ pokemon2.name }}</div>
            <div v-if="showStats" class="pokemon-stat-value">{{ getStatValue(pokemon2, currentStat) }}</div>
          </div>
        </button>
      </div>

      <div v-if="message" class="message" :class="{ correct: message.includes('✅'), wrong: message.includes('❌'), tie: message.includes('🤝') }">
        {{ message }}
      </div>
    </div>
  </div>
</template>

<style scoped>
.game-container {
  min-height: 100vh;
  background: linear-gradient(135deg, var(--pokemon-blue) 0%, #1e3c72 100%);
  padding: 2rem 1rem;
  display: flex;
  flex-direction: column;
}

.game-header {
  text-align: center;
  margin-bottom: 2rem;
}

.game-header h1 {
  color: white;
  font-size: clamp(2rem, 5vw, 3rem);
  margin: 0 0 1.5rem 0;
  text-shadow: 3px 3px 6px rgba(0, 0, 0, 0.4);
  font-weight: 800;
  letter-spacing: 2px;
}

.score-board {
  background: linear-gradient(135deg, rgba(238, 21, 21, 0.2) 0%, rgba(255, 204, 0, 0.1) 100%);
  backdrop-filter: blur(10px);
  padding: 1rem 2rem;
  border-radius: 16px;
  display: inline-block;
  border: 3px solid var(--pokemon-red);
  margin: 0 auto;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.2);
}

.score {
  color: white;
  font-size: clamp(1.2rem, 3vw, 1.8rem);
  font-weight: bold;
}

.game-content {
  max-width: 1200px;
  margin: 0 auto;
  width: 100%;
}

.stat-challenge {
  text-align: center;
  margin-bottom: 2.5rem;
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(10px);
  padding: 2rem;
  border-radius: 16px;
  border: 2px solid rgba(255, 255, 255, 0.3);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
}

.stat-challenge h2 {
  color: white;
  margin: 0;
  font-size: clamp(1.3rem, 4vw, 1.8rem);
  font-weight: 700;
}

.stat-name {
  background: linear-gradient(135deg, var(--pokemon-red), var(--pokemon-yellow));
  padding: 0.5rem 1.5rem;
  border-radius: 20px;
  font-weight: bold;
  display: inline-block;
  margin-top: 0.75rem;
  color: white;
  box-shadow: 0 4px 15px rgba(238, 21, 21, 0.3);
}

.pokemon-comparison {
  display: grid;
  grid-template-columns: 1fr auto 1fr;
  gap: 1.5rem;
  align-items: center;
  margin-bottom: 2.5rem;
  width: 100%;
}

.pokemon-option {
  background: white;
  border: none;
  border-radius: 16px;
  padding: 1.5rem;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.15);
  position: relative;
  overflow: hidden;
}

.pokemon-option::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 4px;
  background: linear-gradient(90deg, var(--pokemon-red), var(--pokemon-blue));
}

.pokemon-option:hover:not(:disabled) {
  transform: translateY(-8px) scale(1.02);
  box-shadow: 0 16px 48px rgba(0, 0, 0, 0.25);
}

.pokemon-option:active:not(:disabled) {
  transform: translateY(-2px);
}

.pokemon-option:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.pokemon-display {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
  position: relative;
  z-index: 1;
}

.pokemon-display img {
  width: clamp(120px, 25vw, 220px);
  height: clamp(120px, 25vw, 220px);
  object-fit: contain;
  filter: drop-shadow(0 4px 8px rgba(0, 0, 0, 0.1));
}

.pokemon-name {
  font-size: clamp(1.1rem, 3vw, 1.5rem);
  font-weight: 700;
  color: var(--pokemon-blue);
  text-align: center;
}

.pokemon-stat-value {
  font-size: clamp(1.5rem, 4vw, 2.5rem);
  font-weight: 900;
  color: white;
  background: linear-gradient(135deg, var(--pokemon-red), #d41010);
  padding: 0.75rem 1.5rem;
  border-radius: 12px;
  min-width: 100px;
  text-align: center;
  box-shadow: 0 6px 20px rgba(238, 21, 21, 0.3);
}

.vs-divider {
  display: flex;
  justify-content: center;
  align-items: center;
}

.vs-text {
  background: linear-gradient(135deg, var(--pokemon-red), var(--pokemon-yellow));
  color: white;
  font-weight: bold;
  font-size: clamp(1.3rem, 3vw, 1.8rem);
  padding: 1.2rem;
  border-radius: 50%;
  width: 80px;
  height: 80px;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 8px 24px rgba(238, 21, 21, 0.3);
  border: 3px solid white;
}

.message {
  text-align: center;
  padding: 1.5rem;
  border-radius: 12px;
  font-size: clamp(1rem, 2vw, 1.3rem);
  font-weight: bold;
  animation: slideIn 0.4s cubic-bezier(0.34, 1.56, 0.64, 1);
  backdrop-filter: blur(10px);
  border: 2px solid;
}

@keyframes slideIn {
  from {
    opacity: 0;
    transform: translateY(-20px) scale(0.95);
  }
  to {
    opacity: 1;
    transform: translateY(0) scale(1);
  }
}

.message.correct {
  background-color: rgba(76, 175, 80, 0.15);
  color: #2e7d32;
  border-color: #4caf50;
}

.message.wrong {
  background-color: rgba(244, 67, 54, 0.15);
  color: #c62828;
  border-color: #f44336;
}

.message.tie {
  background-color: rgba(255, 193, 7, 0.15);
  color: #f57f17;
  border-color: #ffc107;
}

.game-over {
  max-width: 600px;
  margin: 2rem auto 0;
  text-align: center;
  background: linear-gradient(135deg, rgba(238, 21, 21, 0.15) 0%, rgba(255, 204, 0, 0.1) 100%);
  backdrop-filter: blur(10px);
  padding: 3.5rem 2rem;
  border-radius: 20px;
  border: 3px solid var(--pokemon-red);
  box-shadow: 0 16px 48px rgba(0, 0, 0, 0.2);
}

.game-over h2 {
  color: white;
  font-size: clamp(2rem, 5vw, 3rem);
  margin: 0 0 1rem 0;
  font-weight: 800;
  letter-spacing: 1px;
}

.final-score {
  color: var(--pokemon-yellow);
  font-size: clamp(1.5rem, 4vw, 2rem);
  margin: 1.5rem 0 2.5rem 0;
  font-weight: 700;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
}

.restart-button {
  background: linear-gradient(135deg, var(--pokemon-red), #d41010);
  color: white;
  border: none;
  padding: 1.2rem 2.5rem;
  font-size: clamp(1rem, 2vw, 1.2rem);
  border-radius: 12px;
  cursor: pointer;
  font-weight: 700;
  transition: all 0.3s cubic-bezier(0.34, 1.56, 0.64, 1);
  box-shadow: 0 8px 24px rgba(238, 21, 21, 0.3);
  letter-spacing: 1px;
}

.restart-button:hover {
  transform: translateY(-3px);
  box-shadow: 0 12px 32px rgba(238, 21, 21, 0.4);
}

.restart-button:active {
  transform: translateY(-1px);
}

.loading {
  text-align: center;
  color: white;
  font-size: clamp(1.1rem, 2vw, 1.5rem);
  padding: 3rem 1rem;
  font-weight: 600;
  animation: pulse 1.5s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% {
    opacity: 1;
  }
  50% {
    opacity: 0.6;
  }
}

/* Responsive Design */
@media (max-width: 768px) {
  .game-container {
    padding: 1rem;
  }

  .pokemon-comparison {
    grid-template-columns: 1fr;
    gap: 1.2rem;
    margin-bottom: 1.5rem;
  }

  .vs-divider {
    margin: 0.5rem 0;
  }

  .vs-text {
    width: 70px;
    height: 70px;
    font-size: 1.3rem;
    padding: 1rem;
  }

  .stat-challenge {
    padding: 1.5rem 1rem;
    margin-bottom: 1.5rem;
  }

  .game-over {
    margin: 1rem auto 0;
    padding: 2.5rem 1.5rem;
  }
}

@media (max-width: 480px) {
  .pokemon-option {
    padding: 1rem;
  }

  .game-header h1 {
    margin-bottom: 1rem;
  }

  .score-board {
    padding: 0.8rem 1.5rem;
  }

  .stat-challenge h2 {
    font-size: 1.2rem;
  }
}

@media (min-width: 1200px) {
  .game-container {
    padding: 3rem 2rem;
  }

  .pokemon-comparison {
    gap: 2rem;
  }
}
</style>
