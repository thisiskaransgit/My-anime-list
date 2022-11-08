<script setup>
import { computed, onMounted, ref } from "vue";

const query = ref("");
const MyAnime = ref([]);
const SearchResults = ref([]);

const MyAnimeAsc = computed(() => {
  return MyAnime.value.sort((a, b) => {
    return a.title.localeCompare(b.title);
  });
});

const SearchAnime = () => {
  const url = `https://api.jikan.moe/v4/anime?q=${query.value}`;
  fetch(url)
    .then((res) => res.json())
    .then((data) => {
      SearchResults.value = data.data;
    });
};

const handleInput = (e) => {
  if (!e.target.value) {
    SearchResults.value = [];
  }
};

const addAnime = (anime) => {
  SearchResults.value = [];
  query.value = "";

  MyAnime.value.push({
    id: anime.mal_id,
    title: anime.title,
    image: anime.images.jpg.image_url,
    totalEpisodes: anime.episodes,
    watched_episodes: 0,
  });

  localStorage.setItem("MyAnime", JSON.stringify(MyAnime.value));
};

const IncreasedWatch = (anime) => {
  anime.watched_episodes++;
  localStorage.setItem("MyAnime", JSON.stringify(MyAnime.value));
};

const DecreaseWatch = (anime) => {
  anime.watched_episodes--;
  localStorage.setItem("MyAnime", JSON.stringify(MyAnime.value));
};

onMounted(() => {
  MyAnime.value = JSON.parse(localStorage.getItem("MyAnime")) || [];
});
</script>

<template>
  <main>
    <h1>MY Anime tracker</h1>

    <form @submit.prevent="SearchAnime">
      <input
        type="text"
        placeholder="Search for an anime ..."
        v-model="query"
        @input="handleInput"
      />
      <button type="submit" class="button">Search</button>
    </form>

    <div class="results" v-if="SearchResults.length > 0">
      <div class="result" v-for="anime in SearchResults">
        <img :src="anime.images.jpg.image_url" />
        <div class="details">
          <h3>{{ anime.title }}</h3>
          <p :title="anime.synopsis" v-if="anime.synopsis">
            {{ anime.synopsis.slice(0, 120) }}...
          </p>
          <span class="flex-1"></span>
          <button @click="addAnime(anime)" class="button">
            Add to my Anime
          </button>
        </div>
      </div>
    </div>
    <div class="myanime" v-if="MyAnime.length > 0">
      <h2>My Anime</h2>

      <div v-for="anime in MyAnimeAsc" class="anime">
        <img :src="anime.image" />
        <h3>{{ anime.title }}</h3>
        <div class="flex-1"></div>
        <span class="episodes">
          {{ anime.watched_episodes }} / {{ anime.totalEpisodes }}
        </span>
        <button
          v-if="anime.totalEpisodes !== anime.watched_episodes"
          @click="IncreasedWatch(anime)"
          class="button"
        >
          +
        </button>
        <button
          v-if="anime.watched_episodes > 0"
          @click="DecreaseWatch(anime)"
          class="button"
        >
          -
        </button>
      </div>
    </div>
  </main>
</template>
