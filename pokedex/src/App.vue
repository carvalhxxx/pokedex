<template>

  <!-- <input type="text" v-model="busca" placeholder="Digite o nome de um Pokemon"/> -->
  <!-- <hr> -->
  <div class="pokedex-wrapper">
      <img class="logo" src="/public/pokedex.png">
    <div class="lista">
      <PokemonCard
        v-for="pokemon in pokemonsFiltrados"
        :key="pokemon.name"
        :pokemon="pokemon"
        @abrir-modal="abrirModal"
      />
    </div>
  </div>

  <div v-if="pokemonSelecionado" class="modal-overlay" @click.self="fecharModal">
    <div class="modal-content" :class="{ aberto: modalAtivo }">
      <span class="fechar" @click="fecharModal"><i class="fas fa-arrow-left"></i></span>
      <h2>{{ pokemonSelecionado.name }}</h2>
      <img :src="pokemonSelecionado.imagem" :alt="pokemonSelecionado.name"/>
      <p>{{ pokemonSelecionado.tipos.join (',') }}</p>
    </div>
  </div>

</template>

<script setup>
import { onMounted, ref, computed, nextTick, watch } from 'vue'
import '@fortawesome/fontawesome-free/css/all.css'
import PokemonCard from './components/PokemonCard.vue'

const pokemons = ref([])
const busca = ref([])

async function carregarPokemons() {
  const res = await fetch('https://pokeapi.co/api/v2/pokemon?limit=35')
  const data = await res.json()
  pokemons.value = data.results
}

onMounted(() => {
  carregarPokemons()
})

const pokemonsFiltrados = computed(() => {
  const termo = String(busca.value || '').toLowerCase() // garante string
  return pokemons.value.filter(p =>
    p.name.toLowerCase().includes(termo)
  )
})

const pokemonSelecionado = ref(null)
const modalAtivo = ref(false)

function abrirModal(pokemon) {
  pokemonSelecionado.value = pokemon
  nextTick(() => {
    setTimeout(() => {
    modalAtivo.value = true
    document.body.style.overflow = 'hidden'
    }, 10)
  })
}

function fecharModal() {
  modalAtivo.value = false
  setTimeout(() => {
    pokemonSelecionado.value = null
        document.body.style.overflow = ''
  }, 500)
}


</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Roboto:ital,wght@0,100..900;1,100..900&display=swap');
body {
  display: block;
  font-family: 'Roboto', sans-serif;
  margin: 0;
  box-sizing: border-box;
      background-color: #ca4a58;

}

#app {
  /* padding: 1rem; */

    margin: 0;
}

.pokedex-wrapper {
  width: 100%;
  display: block;
  text-align: left;
  background-color: white;
  border-radius: 15px;
  padding: 1rem;
}
.pokedex-wrapper input {
    width: 100%;              /* ocupa toda a largura do container */
    max-width: 400px;         /* opcional: não fica gigante em telas enormes */
    padding: 8px 12px;
    border-radius: 15px;
    border: 2px solid #ccc;
    background-color: #f4f4f4;
    font-size: 14px;
    color: #333;
    outline: none;
    transition: all 0.2s ease;
    box-sizing: border-box;
    margin-bottom: 8px;
}

.pokedex-wrapper input:focus {
    border-color: #82CAFF;
    box-shadow: 0 0 6px rgba(130, 202, 255, 0.5);
    background-color: #ffffff;
}

.lista {
  display: grid;
  flex-wrap: wrap;
  padding-top: 10px;
  /* width: 100%; */
}
h1 {
  text-align: left;
}
.modal-overlay {
  position: fixed;
  top: 0; left: 0;
  width: 100%; height: 100%;
  background: rgba(0,0,0,0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  align-items: flex-end;
  z-index: 1000;
}
.modal-content {
  position: relative;
  background: white;
  border-radius: 20px 20px 0 0;
  padding: 20px;
  max-height: 90vh;
  height: 80vh;
  width: 100%;
  text-align: center;

  transform: translateY(100%);
  transition: transform 0.5s ease-out;

}
.modal-overlay .modal-content.aberto {
  transform: translateY(0);
}
.modal-content img {
  width: 150px;
  height: auto;
}
.logo {
    width: 40%;
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 0px 0px 10px 29%;
}
.modal-content p, h2 {
  text-transform: capitalize;
}
.modal-content button {
  align-items: flex-end;
}
.modal-content .fechar {
  position: absolute;
  top: 30px;
  left: 20px;
  cursor: pointer;
  z-index: 10;
}
.modal-content .fechar:hover {
  color: #82caff;
}

@media screen and (min-width: 300px) {
  .pokedex-wrapper {
  max-width: 300px;
  margin: 1rem auto;
  border-radius: 1rem;
  }
}
</style>