<script setup lang="ts">
import { ref, onBeforeMount } from 'vue';
import { getGamesInStock } from '../services/games';
import Game from '../components/Home/GameCard.vue';
import LoadingGame from '../components/Common/LoadingGameCard.vue';

const isLoading = ref<boolean>(false);
const isError = ref<boolean>(false);
const gamesInStock = ref<any>([]);
const sort = ref<string>('date');

async function fetchGamesInStock() {
  try {

    isLoading.value = true;
    gamesInStock.value = await getGamesInStock()
    isLoading.value = false;
    isError.value = false;

  } catch (error: any) {

    isLoading.value = false;
    isError.value = true;
    throw error.message;

  }
}

const searchInput = ref<string>('');

function searchGames(search?: string) {

  if (!search) {
    fetchGamesInStock();
    return;
  }

  gamesInStock.value = gamesInStock.value.filter((game: any) => {
    return game.name.toLowerCase().includes(search.toLowerCase()) ||
      game.console.toLowerCase().includes(search.toLowerCase())
  });
}

function clearSearch() {
  searchInput.value = '';
  fetchGamesInStock();
}

function sortGames(sort: string) {
  if (sort === 'name') {
    gamesInStock.value.sort((a: any, b: any) => {
      return a.name.localeCompare(b.name);
    });
  } else if (sort === 'date') {
    gamesInStock.value.sort((a: any, b: any) => {
      return new Date(b.created_at).getTime() - new Date(a.created_at).getTime();
    });
  } else if (sort === 'value') {
    gamesInStock.value.sort((a: any, b: any) => {
      return (b.estimated_price - b.purchased_price) - (a.estimated_price - a.purchased_price);
    });
  }
}

onBeforeMount(async () => {
  await fetchGamesInStock();
  sortGames(sort.value);
});

</script>

<template>
  <main id="Home" class="container p-2 mx-auto">
    <h1>Home</h1>

    <section class="sales">
      <h2>Games in stock</h2>

      <div id="controls" class="flex items-center gap-4">
        <div id="search" class="flex items-center gap-2">
          <input @input="searchGames(searchInput)" v-model="searchInput" type="text" placeholder="Rechercher un jeu" />
          <button v-if="searchInput" @click="clearSearch">❌</button>
        </div>
        <div id="sort" class="flex items-center gap-2">
          <label for="sort">Sort by</label>
          <select name="sort" id="sort" v-model="sort" @change="sortGames(sort)">
            <option value="date">Date</option>
            <option value="name">Name</option>
            <option value="value">Value</option>
          </select>
        </div>
      </div>

      <div id="games-list">
        <ul class="grid grid-cols-1 gap-2 mt-4 md:grid-cols-2 xl:grid-cols-3 md:gap-4 xl:gap-8">
          <li v-for="game in gamesInStock" :key="game.id">
            <Game :game="game" />
          </li>
          <li v-if="isLoading" v-for="n in 10" :key="n">
            <LoadingGame />
          </li>
        </ul>
        <div v-if="gamesInStock.length === 0">
          <p v-if="!isLoading && !isError" class="p-4 text-center text-red-500 bg-red-100 rounded-md">No
            games in stock</p>
          <p v-if="isError" class="p-4 text-center text-red-500 bg-red-100 rounded-md">Can't fetch games</p>
        </div>
      </div>


    </section>
  </main>
</template>