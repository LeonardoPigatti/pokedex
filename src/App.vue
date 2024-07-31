<template>
  <v-app>
    <v-container>
      <v-container>
        <v-row>
          <v-container>
            <v-img
              :src="require('../src/assets/pokedex.png')"
              class="my-3"
              contain
              height="200"
            />
            <h1 class="text-center white--text mb-2" style="font-size: 5rem">
              Pokedex
            </h1>
            <h1 class="text-center white--text mb-8">
              Created by
              <a class="pink--text" style="text-decoration: none;" href="https://github.com/LeonardoPigatti">
                Leonardo Vinicius Milani Pigatti
              </a>
            </h1>
          </v-container>
        </v-row>

        <v-row align="center">
          <button @click="toggleSearchField" class="button-style">
            <img
              :src="require('../src/assets/pokeball.png')"
              height="100"
              :class="{ 'flip-x': showSearchField }"
            />
          </button>

          <transition name="slide-fade">
            <v-col v-if="showSearchField">
              <v-text-field
                v-model="search"
                label="Choose your Pokémon to start your journey"
                placeholder="Pikachu or Charmander? Whatever everyone knows that Squirtle is THE BEST"
                solo
                class="search-field"
              ></v-text-field>
            </v-col>
          </transition>
        </v-row>

        <!-- Botões de Tipo -->
        <v-row class="type-buttons-row">
          <v-col
            v-for="(type, index) in allTypes"
            :key="index"
            cols="auto"
            class="type-button-col"
          >
            <v-btn
              @click="toggleType(type)"
              :style="{ 'background-color': getButtonColor(type), 'color': '#fff', 'border': '2px solid #000' }"
              class="type-button"
            >
              {{ type }}
            </v-btn>
          </v-col>
        </v-row>

        <v-row>
          <v-col
            cols="6"
            md="2"
            v-for="pokemon in filtered_pokemons"
            :key="pokemon.name"
          >
            <PokemonCard :pokemon="pokemon" @clicked="show_pokemon" />
          </v-col>
        </v-row>
      </v-container>
    </v-container>

    <PokemonInfoDialog
      :show.sync="show_dialog"
      :selected_pokemon="selected_pokemon"
    />
  </v-app>
</template>

<script>
import axios from "axios";
import PokemonCard from "./components/PokemonCard.vue";
import PokemonInfoDialog from "./components/PokemonInfoDialog.vue";

export default {
  name: "App",
  components: {
    PokemonCard,
    PokemonInfoDialog,
  },
  data() {
    return {
      pokemons: [],
      search: "",
      allTypes: [], // Lista de todos os tipos de Pokémon
      selectedTypes: [], // Tipos selecionados
      typeColors: {
        fire: '#F08030',
        water: '#6890F0',
        grass: '#78C850',
        electric: '#F8D030',
        ice: '#98D8D8',
        fighting: '#C03028',
        poison: '#A040A0',
        ground: '#E0C068',
        flying: '#A890F0',
        psychic: '#F85888',
        bug: '#A8B820',
        rock: '#B8A038',
        ghost: '#705898',
        dragon: '#7038F8',
        dark: '#705848',
        steel: '#B8B8D0',
        fairy: '#F0B6BC'
        // Adicione mais tipos e cores conforme necessário
      },
      show_dialog: false,
      selected_pokemon: null,
      showSearchField: false,
    };
  },
  mounted() {
    axios
      .get("https://pokeapi.co/api/v2/pokemon?limit=20") // Fetch more Pokémons
      .then((response) => {
        this.pokemons = response.data.results;
        this.fetchPokemonDetails();
      });

    // Fetch all Pokémon types
    axios.get("https://pokeapi.co/api/v2/type")
      .then((response) => {
        this.allTypes = response.data.results.map(type => type.name);
      });
  },
  methods: {
    toggleSearchField() {
      this.showSearchField = !this.showSearchField;
    },
    show_pokemon(id) {
      axios.get(`https://pokeapi.co/api/v2/pokemon/${id}`).then((response) => {
        this.selected_pokemon = response.data;
        this.show_dialog = !this.show_dialog;
      });
    },
    fetchPokemonDetails() {
      this.pokemons = this.pokemons.map(pokemon => {
        return axios.get(`https://pokeapi.co/api/v2/pokemon/${pokemon.name}`)
          .then(response => {
            return {
              ...pokemon,
              types: response.data.types.map(t => t.type.name),
            };
          });
      });
      // Wait until all fetches are complete
      Promise.all(this.pokemons).then(results => {
        this.pokemons = results;
      });
    },
    toggleType(type) {
      const index = this.selectedTypes.indexOf(type);
      if (index === -1) {
        this.selectedTypes.push(type);
      } else {
        this.selectedTypes.splice(index, 1);
      }
    },
    getButtonColor(type) {
      return this.selectedTypes.includes(type)
        ? this.typeColors[type] || '#000' // Default color if type not found
        : '#ccc'; // Color for unselected types
    },
  },
  computed: {
    filtered_pokemons() {
      return this.pokemons.filter((pokemon) => {
        const matchesName = pokemon.name.includes(this.search);
        const matchesType = this.selectedTypes.length === 0
          ? true
          : pokemon.types.some(type => this.selectedTypes.includes(type));
        return matchesName && matchesType;
      });
    },
  },
};
</script>

<style>
#app {
  background: linear-gradient(
      to bottom right,
      rgba(10, 10, 10, 1),
      rgb(7, 20, 31)
      )
    no-repeat center center fixed !important;
  -webkit-background-size: cover;
  -moz-background-size: cover;
  -o-background-size: cover;
  background-size: cover !important;
  background-position: center;
  min-height: 100vh;
}

.button-style {
  background: none;
  border: none;
  padding: 0;
  cursor: pointer;
}

.flip-x {
  transform: scaleX(-1);
}

.selected-type {
  background-color: #ffcc00; /* Cor de destaque para tipos selecionados */
  color: #000;
}

.slide-fade-enter-active {
  transition: all 0.5s ease;
}
.slide-fade-leave-active {
  transition: all 0.5s ease;
}
.slide-fade-enter, .slide-fade-leave-to /* .slide-fade-leave-active in <2.1.8 */ {
  transform: translateX(-100%);
  opacity: 0;
}

.search-field {
  position: relative;
  top: 20px;
}

/* Estilo adicional para o layout dos botões de tipo */
.type-buttons-row {
  flex-wrap: wrap;
  justify-content: flex-start;
  margin: 0;
}

.type-button-col {
  flex: 1 0 10%; /* Cada coluna ocupa até 10% da largura disponível */
  max-width: 10%; /* Máximo de 10% da largura */
  padding: 0; /* Remove o padding */
}

.type-button {
  width: 100%; /* Ocupa toda a largura da coluna */
  margin: 0; /* Remove a margem */
  border: 2px solid #000; /* Adiciona a borda aos botões */
  border-radius: 4px; /* Adiciona borda arredondada para um visual mais limpo */
  text-transform: capitalize; /* Capitaliza o texto dos botões */
  background-color: #ada8a8;
}
</style>
