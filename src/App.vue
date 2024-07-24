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
              Developed by
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
      show_dialog: false,
      selected_pokemon: null,
      showSearchField: false,
    };
  },

  mounted() {
    axios
      .get("https://pokeapi.co/api/v2/pokemon?limit=493")
      .then((response) => {
        this.pokemons = response.data.results;
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
    get_move_level(move) {
      for (let version of move.version_group_details) {
        if (
          version.version_group.name == "sword-shield" &&
          version.move_learn_method.name == "level-up"
        ) {
          return version.level_learned_at;
        }
      }
      return 0;
    },
  },
  computed: {
    filtered_pokemons() {
      return this.pokemons.filter((item) => {
        return item.name.includes(this.search);
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
</style>
