<script setup>
import { onMounted, ref } from 'vue'
import axios from 'axios'
import LanguageChart from './components/LanguageChart.vue'

const username = ref('')
const userProfile = ref(null)
const languages = ref({})
const loading = ref(false)
const error = ref(null)

// --- CORREÇÃO 1: Definimos a variável ANTES de usar no onMounted ---
const searchHistory = ref([])

onMounted(() => {
  console.log("Iniciando App... Buscando histórico salvo.")
  const saved = localStorage.getItem('github_history')
  if (saved) {
    searchHistory.value = JSON.parse(saved)
  }
})

// Função para adicionar ao histórico (sem repetir)
const addToHistory = (name) => {
  const filtered = searchHistory.value.filter(item => item !== name)
  filtered.unshift(name)
  searchHistory.value = filtered.slice(0, 5)
  localStorage.setItem('github_history', JSON.stringify(searchHistory.value))
}

const clearHistory = () => {
  searchHistory.value = []
  localStorage.removeItem('github_history')
}

const searchUser = async (searchName = null) => {
  const query = searchName || username.value
  if (!query) return
  
  // Atualiza o input visualmente se veio do histórico
  if(searchName) username.value = searchName

  loading.value = true
  error.value = null
  userProfile.value = null
  languages.value = {}

  try {
    // 1. Busca os dados do Perfil
    const profileRes = await axios.get(`https://api.github.com/users/${query}`)
    userProfile.value = profileRes.data

    // 2. Busca os repositórios
    const reposRes = await axios.get(`https://api.github.com/users/${query}/repos?per_page=100&sort=updated`)
    
    // 3. Processamento de dados
    const stats = {}
    reposRes.data.forEach(repo => {
      const lang = repo.language
      if (lang) {
        stats[lang] = (stats[lang] || 0) + 1
      }
    })
    
    languages.value = stats

    // Adiciona ao histórico se deu tudo certo
    addToHistory(query)

  } catch (err) {
    error.value = "Usuário não encontrado."
    console.error(err)
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <div class="container">
    <h1>GitHub DevFinder 🔍</h1>
    
    <div class="search-box">
      <input 
        v-model="username" 
        @keyup.enter="() => searchUser()"
        placeholder="Digite o usuário (ex: dinah-parag)..." 
      />
      <button @click="() => searchUser()" :disabled="loading">
        {{ loading ? 'Buscando...' : 'Buscar' }}
      </button>
    </div>

    <div class="history-area" v-if="searchHistory.length > 0">
      <span class="label">Recentes:</span>
      <span 
        v-for="item in searchHistory" 
        :key="item" 
        class="history-tag"
        @click="searchUser(item)"
      >
        {{ item }}
      </span>
      <button class="btn-clear" @click="clearHistory" title="Limpar histórico">×</button>
    </div>

    <div v-if="error" class="error">{{ error }}</div>

    <div v-if="userProfile" class="result-area">
      
      <div class="profile-card">
        <img :src="userProfile.avatar_url" alt="Avatar" />
        <h2>{{ userProfile.name || userProfile.login }}</h2>
        <p>{{ userProfile.bio }}</p>
        <div class="stats-row">
          <span><strong>{{ userProfile.followers }}</strong> Seguidores</span>
          <span><strong>{{ userProfile.public_repos }}</strong> Repositórios</span>
        </div>
        <a :href="userProfile.html_url" target="_blank" class="btn-visit">Ver no GitHub</a>
      </div>

      <div class="chart-area">
        <LanguageChart :languages="languages" />
      </div>

    </div>
  </div>
</template>

<style scoped>
.container { max-width: 900px; margin: 0 auto; padding: 40px 20px; font-family: 'Segoe UI', sans-serif; }
h1 { text-align: center; color: #7693ae; margin-bottom: 30px; }

.search-box { display: flex; gap: 10px; justify-content: center; margin-bottom: 30px; }
input { padding: 15px; width: 60%; border: 2px solid #ddd; border-radius: 8px; font-size: 1rem; }
button { padding: 15px 30px; background: #64829f; color: white; border: none; border-radius: 8px; cursor: pointer; font-weight: bold; }
button:disabled { opacity: 0.7; }

/* CSS do Histórico */
.history-area {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 10px;
  margin-bottom: 40px; /* Espaço para não grudar no resultado */
  flex-wrap: wrap;
}
.label { font-size: 0.9rem; color: #888; margin-right: 5px; }

.history-tag {
  background: #e0e6ed;
  color: #2c3e50;
  padding: 5px 12px;
  border-radius: 20px;
  font-size: 0.85rem;
  cursor: pointer;
  transition: all 0.2s;
}
.history-tag:hover {
  background: #2e5f49;
  color: white;
  transform: translateY(-2px);
}
.btn-clear {
  background: none; border: none; color: #999; cursor: pointer; font-size: 1.2rem; margin-left: 5px; padding: 0 5px;
}
.btn-clear:hover { color: red; }

/* CSS dos Resultados */
.result-area { display: grid; grid-template-columns: 1fr 1fr; gap: 30px; animation: fadeIn 0.5s ease; }

.profile-card { background: rgb(162, 179, 199); padding: 30px; border-radius: 12px; box-shadow: 0 4px 12px rgba(0,0,0,0.1); text-align: center; }
.profile-card img { width: 120px; height: 120px; border-radius: 50%; margin-bottom: 15px; border: 4px solid #f0f0f0; }
.stats-row { display: flex; justify-content: space-around; margin: 20px 0; color: #094045; }
.btn-visit { display: inline-block; padding: 10px 20px; background: #2d805b; color: white; text-decoration: none; border-radius: 6px; }

.error { text-align: center; color: red; background: #ffebeb; padding: 10px; border-radius: 8px; }

@media (max-width: 768px) {
  .result-area { grid-template-columns: 1fr; }
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}
</style>