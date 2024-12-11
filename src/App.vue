<template>
  <div id="app">
    <h1>Adivina el Pokémon</h1>
    <div class="row justify-content-center">
      <div 
        class="col-md-4" 
        v-for="(pokemon, index) in pokemons" 
        :key="pokemon.id"
      >
        <PokemonCard 
          :pokemon="pokemon" 
          @pokemon-revealed="incrementCounter" 
          @incorrect-guess="incrementErrors" 
        />
      </div>
    </div>
    <h2>Pokémon descubiertos: {{ discoveredCount }}</h2>
    <h2>Intentos Fallidos: {{ errorCount }}</h2>
  </div>
</template>

<script>
import axios from 'axios';
import PokemonCard from './components/PokemonCard.vue';

export default {
  components: {
    PokemonCard
  },
  data() {
    return {
      pokemons: [],
      discoveredCount: 0,
      errorCount: 0
    };
  },
  created() {
    this.loadPokemons();
  },
  methods: {
    async loadPokemons() {
      const response = await axios.get('https://pokeapi.co/api/v2/pokemon?limit=20');
      this.pokemons = response.data.results.map((pokemon, index) => ({
        ...pokemon,
        id: index + 1,
        silhouette: `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/back/${index + 1}.png`
      }));
    },
    incrementCounter() {
      this.discoveredCount++;
    },
    incrementErrors() {
      this.errorCount++;
    }
  }
};
</script>

<style>
.row {
  margin-top: 20px;
}
</style>