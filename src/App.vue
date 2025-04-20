<script setup>
import PocketBase from 'pocketbase'
import { ref, onMounted, computed, watch } from 'vue'

const pb = new PocketBase('http://127.0.0.1:8090/');
const records = ref([]);
const todoInput = ref("");
const animeList = ref([]);
const allAnime = ref([]);
const genresList = ref([]);
const selectedGenre = ref("");
const selectedScore = ref(0);
const searchQuery = ref("");
const isDarkMode = ref(false);

// Fetch anime
function fetchAnime(apiType) {
  const apiUrl = apiType === "top"
    ? "https://api.jikan.moe/v4/top/anime"
    : "https://api.jikan.moe/v4/seasons/now";

  fetch(apiUrl)
    .then(res => res.json())
    .then(data => {
      allAnime.value = data.data;
      animeList.value = data.data;
      genresList.value = Array.from(new Set(data.data.flatMap(anime => anime.genres)));
    });
}

function loadTodos() {
  pb.collection('todos').getFullList().then((data) => {
    records.value = data;
  });
}

function createTodo() {
  if (!todoInput.value) return;
  pb.collection('todos').create({ Item: todoInput.value, isDone: false }).then((data) => {
    records.value.push(data);
  });
}


function markAsDone(record) {
  pb.collection('todos').update(record.id, { isDone: true }).then(() => {
    const target = records.value.find(r => r.id === record.id);
    if (target) target.isDone = true;
  });
}

function deleteTodo(record) {
  pb.collection('todos').delete(record.id).then(() => {
    records.value = records.value.filter(r => r.id !== record.id);
  });
}

function filterAnime() {
  let filtered = allAnime.value;

  if (selectedGenre.value) {
    filtered = filtered.filter(anime => anime.genres.some(g => g.name === selectedGenre.value));
  }
  if (selectedScore.value > 0) {
    filtered = filtered.filter(anime => anime.score >= selectedScore.value);
  }
  if (searchQuery.value) {
    filtered = filtered.filter(anime => anime.title.toLowerCase().includes(searchQuery.value.toLowerCase()));
  }

  animeList.value = filtered;
}

watch([selectedGenre, selectedScore, searchQuery], filterAnime);

onMounted(() => {
  fetchAnime("top");
  loadTodos();
});
</script>

<template>
  <div :class="`min-h-screen transition ${isDarkMode ? 'bg-slate-900 text-slate-100' : 'bg-slate-100 text-slate-900'}`">
    <div class="max-w-7xl mx-auto p-6 flex flex-col md:flex-row gap-6">

      <!-- SIDEBAR TODO WIDGET -->
      <aside :class="`md:w-1/3 w-full p-4 rounded-xl shadow-lg ${isDarkMode ? 'bg-slate-800' : 'bg-white'}`">
        <div class="flex justify-between items-center mb-4">
          <h2 class="text-xl font-bold">Anime Tracker</h2>
          <button @click="isDarkMode = !isDarkMode" class="px-3 py-1 text-sm rounded bg-indigo-600 text-white hover:bg-indigo-500">
            {{ isDarkMode ? 'Light' : 'Dark' }}
          </button>
        </div>

        <select v-model="todoInput" class="w-full p-2 mb-4 rounded bg-white text-black">
          <option value="" disabled>Select Anime</option>
          <option v-for="anime in animeList" :key="anime.mal_id" :value="anime.title">
            {{ anime.title }}
          </option>
        </select>

        <button @click="createTodo" class="w-full mb-4 py-2 bg-indigo-600 text-white rounded hover:bg-indigo-500">Add to Watchlist</button>

        <ul class="space-y-3">
          <li v-for="record in records" :key="record.id" class="p-3 bg-white text-black dark:bg-slate-700 dark:text-white rounded-lg flex justify-between items-center">
            <span>{{ record.Item }} - <strong>{{ record.isDone ? '✅' : '❌' }}</strong></span>
            <div class="flex gap-2">
              <button v-if="!record.isDone" @click="markAsDone(record)" class="bg-blue-600 text-white px-2 py-1 rounded">Done</button>
              <button @click="deleteTodo(record)" class="bg-red-500 text-white px-2 py-1 rounded">Delete</button>
            </div>
          </li>
        </ul>
      </aside>

      <!-- MAIN CONTENT -->
      <main class="md:w-2/3 w-full space-y-6">

        <!-- Fetch Controls -->
        <div :class="`rounded-lg p-4 shadow-lg ${isDarkMode ? 'bg-slate-800' : 'bg-white'}`">
          <h2 class="text-2xl font-bold mb-4">Fetch Anime</h2>
          <div class="flex gap-4 flex-wrap">
            <button @click="fetchAnime('top')" class="bg-indigo-600 text-white px-4 py-2 rounded hover:bg-indigo-500">Top Anime</button>
            <button @click="fetchAnime('current')" class="bg-indigo-600 text-white px-4 py-2 rounded hover:bg-indigo-500">Current Season</button>
          </div>
        </div>

        <!-- Filters -->
<div :class="`rounded-lg p-4 shadow-lg ${isDarkMode ? 'bg-slate-800' : 'bg-white'}`">
  <h2 class="text-2xl font-bold mb-4">Filter Anime</h2>
  <div class="flex flex-wrap gap-4 items-end">
    <select v-model="selectedGenre" class="p-2 bg-white text-black rounded w-full md:w-auto">
      <option value="">All Genres</option>
      <option v-for="genre in genresList" :key="genre.name" :value="genre.name">{{ genre.name }}</option>
    </select>

    <div>
      <label class="block text-sm mb-1">Minimum Rating</label>
      <input 
        type="number" 
        v-model="selectedScore" 
        min="0" max="10" 
        class="p-2 bg-white text-black rounded w-full md:w-36"
      />
    </div>

    <div class="flex-1">
      <label class="block text-sm mb-1">Search by Name</label>
      <input 
        type="text" 
        v-model="searchQuery" 
        placeholder="Search anime..."
        class="p-2 bg-white text-black rounded w-full"
      />
    </div>

    <button @click="filterAnime" class="bg-indigo-600 text-white px-4 py-2 rounded hover:bg-indigo-500">
      Search
    </button>
  </div>
</div>


        <!-- Anime List -->
        <div :class="`rounded-lg p-4 shadow-lg ${isDarkMode ? 'bg-slate-800' : 'bg-white'}`">
          <p class="text-xl font-bold mb-4">Anime List:</p>
          <div class="flex flex-wrap justify-start gap-4">
            <div 
              v-for="anime in animeList" 
              :key="anime.mal_id" 
              class="bg-white dark:bg-slate-700 text-black dark:text-white p-4 rounded-lg w-full sm:w-[48%] md:w-[30%]"
            >
              <h3 class="font-semibold text-lg mb-1">{{ anime.title }}</h3>
              <p class="text-sm">Score: {{ anime.score }}</p>
              <p class="text-sm">Episodes: {{ anime.episodes || 'N/A' }}</p>
              <p class="text-sm">Genres:
                <span v-for="(genre, index) in anime.genres" :key="index">
                  {{ genre.name }}<span v-if="index < anime.genres.length - 1">, </span>
                </span>
              </p>
              <img :src="anime.images.jpg.image_url" alt="Anime" class="mt-2 rounded-lg w-full h-auto" />
            </div>
          </div>
        </div>

      </main>
    </div>
  </div>
</template>



<style scoped>
.bg-maroon-900 {
  background-color: #3e1f1f;
}
.bg-maroon-800 {
  background-color: #5a2d2d;
}
.bg-maroon-700 {
  background-color: #7b3f3f;
}
</style>
