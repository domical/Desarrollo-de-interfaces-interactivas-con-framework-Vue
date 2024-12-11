<template>
    <div class="row">
      <div class="col-3 p-2" v-for="pokemon in randomPokemons" :key="pokemon.name">
        <div class="card pokemon-card">
          <div class="pokemon-Background">
            <img :src="pokemon.sprite" class="pokemon" :class="pokemon.class" :alt="pokemon.name">
          </div>
          <div class="p-3">
            <div v-if="pokemon.correct || pokemon.skip">
              <h6 class="pokemon-name">¡Es {{ pokemon.name }}!</h6>
              <p v-if="pokemon.correct" class="correct-text">Número de errores: {{ pokemon.errors }}</p>
              <p v-if="pokemon.skip" class="skipped-text">¡Omitido!</p>
            </div>
            <form v-else @submit.prevent="submitForm">
              <p v-if="pokemon.errors > 3" class="hint-text">Pista: <span :class="pokemon.errors < 6 ? 'hint1' : 'hint2'">{{ pokemon.name }}</span></p>
              <div class="mb-3">
                <label for="PokemonName" class="form-label">Nombre Pokémon</label>
                <input type="text" class="form-control" id="PokemonName" v-model="pokemon.answer">
              </div>
              <div class="d-flex justify-content-evenly">
                <button type="button" class="btn btn-primary pokemon-btn" @click="checkAnswer(pokemon)">Responder</button>
                <button type="button" class="btn btn-secondary pokemon-btn" @click="skipAnswer(pokemon)">Omitir</button>
              </div>
            </form>
          </div>
        </div>
      </div>
      <button v-if="randomPokemons.length === 0" type="button" class="btn btn-primary start-game-btn" @click="getRandomPokemons">¡Iniciar Juego!</button>
    </div>
  </template>
  
  <script>
  import axios from 'axios';
  
  export default {
    name: 'PokemonCard',
    emits: ['results'],
    data() {
      return {
        pokemons: [],
        randomPokemons: [],
        contador: {
          corrects: 0,
          skips: 0,
          errors: 0,
        }
      };
    },
    created() {
      this.fetchData();
    },
    computed: {
      totalCorrectsAndSkips() {
        return this.contador.corrects + this.contador.skips;
      },
    },
    watch: {
      totalCorrectsAndSkips(newTotal) {
        if (newTotal === 20) {
          this.$emit('results', this.contador);
        }
      },
    },
    methods: {
      async fetchData() {
        try {
          const response = await axios.get('https://pokeapi.co/api/v2/pokemon?limit=100000');
          this.pokemons = response.data.results;
        } catch (error) {
          console.error('Vaya... algo ha pasado: ', error);
        }
      },
      async getRandomPokemons() {
        Object.keys(this.contador).forEach(key => this.contador[key] = 0);
        const totalPokemons = 649;
        const randomIndices = new Set();
  
        // Generar 20 índices únicos aleatorios
        while (randomIndices.size < 20) {
          const randomIndex = Math.floor(Math.random() * totalPokemons);
          randomIndices.add(randomIndex);
        }
  
        // Creando cada Pokémon nuevo
        const pokemonRequests = Array.from(randomIndices).map(index => {
          const pokemon = this.pokemons[index];
          return axios.get(pokemon.url).then(response => ({
            name: pokemon.name,
            sprite: response.data.sprites.front_default, // Imagen del Pokémon
            answer: '',
            correct: false,
            skip: false,
            errors: 0,
            class: 'hide-pokemon'
          }));
        });
  
        // Esperar a que todas las solicitudes se completen
        this.randomPokemons = await Promise.all(pokemonRequests);
      },
      checkAnswer(pokemon) {
        if (pokemon.name === pokemon.answer.toLowerCase()) {
          pokemon.correct = true;
          pokemon.class = 'show-pokemon';
          this.contador.corrects++;
        } else {
          pokemon.errors++;
          this.contador.errors++;
        }
      },
      skipAnswer(pokemon) {
        pokemon.skip = true;
        pokemon.class = 'show-pokemon';
        this.contador.skips++;
      }
    }
  };
  </script>
  
  <style scoped>
  .pokemon-card {
    border: 2px solid #ddd;
    border-radius: 12px;
    background-color: #fff;
    box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
    transition: transform 0.3s ease;
  }
  .pokemon-card:hover {
    transform: scale(1.05);
  }
  .pokemon-Background {
    background-image: url(../assets/img/background.png);
    background-size: cover;
    background-position: center;
    background-repeat: no-repeat;
    border-top-left-radius: 10px;
    border-top-right-radius: 10px;
    overflow: hidden;
    position: relative;
  }
  
  .pokemon {
    width: 150px;
    height: auto;
    margin: 0 auto;
  }
  .hide-pokemon {
    filter: brightness(0%);
  }
  .show-pokemon {
    transition: 0.5s;
    filter: brightness(100%);
  }
  .hint1 {
    filter: blur(3px) grayscale(100%);
  }
  .hint2 {
    filter: blur(1.5px) grayscale(100%);
  }
  .pokemon-name {
    color: #3d7dca;
    font-size: 20px;
    font-weight: bold;
    text-align: center;
    margin-bottom: 10px;
  }
  
  .correct-text {
    color: #28a745;
    font-size: 16px;
    text-align: center;
  }
  
  .skipped-text {
    color: #e60000;
    font-size: 16px;
    text-align: center;
  }
  
  /* Estilos de pistas */
  .hint-text {
    font-size: 16px;
    text-align: center;
    color: #ffcb05;
  }
  
  /* Botones */
  .pokemon-btn {
    background-color: #ffcb05;
    color: #3d7dca;
    padding: 10px 20px;
    border: none;
    font-size: 16px;
    font-weight: bold;
    border-radius: 8px;
    cursor: pointer;
    transition: background-color 0.3s ease;
  }
  
  .pokemon-btn:hover {
    background-color: #e6b800;
  }
  
  .start-game-btn {
    background-color: #3d7dca;
    color: white;
    font-size: 18px;
    font-weight: bold;
    padding: 12px 25px;
    border-radius: 8px;
    cursor: pointer;
    display: block;
    width: 100%;
    margin-top: 20px;
    transition: background-color 0.3s ease;
  }
  
  .start-game-btn:hover {
    background-color: #2a64b3;
  }
  
  /* Ajustes para móvil */
  @media (max-width: 768px) {
    .pokemon-card {
      margin-bottom: 15px;
    }
    .pokemon-name {
      font-size: 18px;
    }
    .pokemon-btn {
      font-size: 14px;
      padding: 8px 16px;
    }
    .start-game-btn {
      font-size: 16px;
      padding: 10px 20px;
    }
  }
  </style>