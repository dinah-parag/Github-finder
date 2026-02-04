<script setup>
import { ref } from 'vue'
import axios from 'axios'
import LanguageChart from './components/LanguageChart.vue'

const username = ref('')
const userProfile = ref(null)
const languages = ref({})
const loading = ref(false)
const error = ref(null)

const searchUser = async () => {
  if (!username.value) return
  
  loading.value = true
  error.value = null
  userProfile.value = null
  languages.value = {}

  try {
    // 1. Busca os dados do Perfil
    const profileRes = await axios.get(`https://api.github.com/users/${username.value}`)
    userProfile.value = profileRes.data

    // 2. Busca os repositórios
    const reposRes = await axios.get(`https://api.github.com/users/${username.value}/repos?per_page=100&sort=updated`)
    
    // 3. PROCESSAMENTO DE DADOS (Data Science no Front)
    // Contagem de quantas vezes cada linguagem aparece
    const stats = {}
    
    reposRes.data.forEach(repo => {
      const lang = repo.language
      if (lang) {
        // Se já existe, soma 1. Se não, inicia com 1.
        stats[lang] = (stats[lang] || 0) + 1
      }
    })
    
    languages.value = stats

  } catch (err) {
    error.value = "Usuário não encontrado ou erro na API."
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
        @keyup.enter="searchUser"
        placeholder="Digite o usuário (ex: dinah-parag)..." 
      />
      <button @click="searchUser" :disabled="loading">
        {{ loading ? 'Buscando...' : 'Buscar' }}
      </button>
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
/* CSS Focado em Layout Grid e Cartões */
.container { max-width: 900px; margin: 0 auto; padding: 40px 20px; font-family: 'Segoe UI', sans-serif; }
h1 { text-align: center; color: #7693ae; margin-bottom: 30px; }

.search-box { display: flex; gap: 10px; justify-content: center; margin-bottom: 40px; }
input { padding: 15px; width: 60%; border: 2px solid #ddd; border-radius: 8px; font-size: 1rem; }
button { padding: 15px 30px; background: #64829f; color: white; border: none; border-radius: 8px; cursor: pointer; font-weight: bold; }
button:disabled { opacity: 0.7; }

.result-area { display: grid; grid-template-columns: 1fr 1fr; gap: 30px; animation: fadeIn 0.5s ease; }

.profile-card { background: rgb(162, 179, 199); padding: 30px; border-radius: 12px; box-shadow: 0 4px 12px rgba(0,0,0,0.1); text-align: center; }
.profile-card img { width: 120px; height: 120px; border-radius: 50%; margin-bottom: 15px; border: 4px solid #f0f0f0; }
.stats-row { display: flex; justify-content: space-around; margin: 20px 0; color: #094045; }
.btn-visit { display: inline-block; padding: 10px 20px; background: #2d805b; color: white; text-decoration: none; border-radius: 6px; }

.error { text-align: center; color: red; background: #ffebeb; padding: 10px; border-radius: 8px; }

/* Responsividade para mobile */
@media (max-width: 768px) {
  .result-area { grid-template-columns: 1fr; }
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}
</style>