<template>
  <div class="pokedex-wrapper">
    <img class="logo" src="/public/pokedex.png">

    <!-- Filtro por tipos -->
    <div class="filtro-tipos">
      <span 
        v-for="tipo in Object.keys(coresPorTipo)" 
        :key="tipo"
        class="tipo-icone"
        :style="{ backgroundColor: coresPorTipo[tipo] }"
        @click="tipoSelecionado = (tipoSelecionado === tipo ? null : tipo)"
        :class="{ativo: tipoSelecionado === tipo}"
      >
        <img :src="iconesPorTipo[tipo]" :alt="tipo" class="icone-tipo" />
      </span>
    </div>

    <!-- Lista de Pokémon -->
    <div class="lista">
      <PokemonCard
        v-for="pokemon in pokemonsFiltrados"
        :key="pokemon.name"
        :pokemon="pokemon"
      />
    </div>

    <!-- Botão carregar mais -->
    <button @click="carregarMais" class="btn-carregar-mais">Carregar Mais</button>
  </div>
</template>
<script setup>
import { onMounted, ref, computed, nextTick } from 'vue'
import '@fortawesome/fontawesome-free/css/all.css'
import PokemonCard from './components/PokemonCard.vue'

const pokemons = ref([])
let offset = 0
const limit = 35
const busca = ref('')  // string para filtro por nome
const tipoSelecionado = ref(null) // tipo selecionado na barra

const coresPorTipo = {
  normal: '#A8A878',
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
  fairy: '#EE99AC'
}

async function carregarPokemons() {
  const res = await fetch(`https://pokeapi.co/api/v2/pokemon?limit=${limit}&offset=${offset}`)
  const data = await res.json()
  // Cada Pokémon agora carrega também os tipos
  const detalhesPromises = data.results.map(async p => {
    const resDetalhes = await fetch(p.url)
    const detalhes = await resDetalhes.json()
    return {
      ...p,
      types: detalhes.types.map(t => t.type.name),
      cor: coresPorTipo[detalhes.types[0].type.name], // primeira cor do tipo
      corBorda: coresPorTipo[detalhes.types[0].type.name]
    }
  })
  const pokemonsComTipos = await Promise.all(detalhesPromises)
  pokemons.value.push(...pokemonsComTipos)
  offset += limit
}

onMounted(() => {
  carregarPokemons()
})

// Filtro de Pokémon por nome + tipo
const pokemonsFiltrados = computed(() => {
  const termo = String(busca.value || '').toLowerCase()
  return pokemons.value.filter(p => {
    const matchesNome = p.name.toLowerCase().includes(termo)
    const matchesTipo = tipoSelecionado.value
      ? p.types?.some(t => t === tipoSelecionado.value)
      : true
    return matchesNome && matchesTipo
  })
})

// Modal e abas (mantido igual)
const pokemonSelecionado = ref(null)
const modalAtivo = ref(false)
const abaAtiva = ref('sobre')

const statAbreviaturas = {
  hp: 'HP',
  attack: 'ATK',
  defense: 'DEF',
  'special-attack': 'SATK',
  'special-defense': 'SDEF',
  speed: 'SPD'
}

async function abrirModal(pokemon) {
  try {
    const res = await fetch(pokemon.url)
    const data = await res.json()
    const speciesRes = await fetch(data.species.url)
    const speciesData = await speciesRes.json()
    const descricao = speciesData.flavor_text_entries.find(
      entry => entry.language.name === 'en'
    )?.flavor_text.replace(/\n|\f/g, '') || ''

    const evolucoesRes = await fetch(speciesData.evolution_chain.url)
    const evolucoesData = await evolucoesRes.json()
    function extrairEvolucoes(chain) {
      const nomes = [chain.species.name]
      if (chain.evolves_to.length > 0) {
        chain.evolves_to.forEach(e => nomes.push(...extrairEvolucoes(e)))
      }
      return nomes
    }
    const evolucoes = extrairEvolucoes(evolucoesData.chain)

    pokemonSelecionado.value = {
      ...data,
      tipos: data.types.map(t => t.type.name),
      abilities: data.abilities,
      evolucoes,
      descricao,
      imagem: data.sprites.other['official-artwork'].front_default || data.sprites.front_default,
      cor: pokemon.cor,
      corBorda: pokemon.corBorda
    }

    nextTick(() => {
      setTimeout(() => {
        modalAtivo.value = true
        document.body.style.overflow = 'hidden'
      }, 10)
    })
  } catch (err) {
    console.error('Erro ao abrir modal do Pokémon:', err)
  }
}

function fecharModal() {
  modalAtivo.value = false
  setTimeout(() => {
    pokemonSelecionado.value = null
    document.body.style.overflow = ''
  }, 500)
}

function carregarMais() {
  carregarPokemons()
}

const iconesPorTipo = {
  normal: '/tipos/normal.svg',
  fire: '/tipos/fire.svg',
  water: '/tipos/water.svg',
  grass: '/tipos/grass.svg',
  electric: '/tipos/electric.svg',
  ice: '/tipos/ice.svg',
  fighting: '/tipos/fighting.svg',
  poison: '/tipos/poison.svg',
  ground: '/tipos/ground.svg',
  flying: '/tipos/flying.svg',
  psychic: '/tipos/psychic.svg',
  bug: '/tipos/bug.svg',
  rock: '/tipos/rock.svg',
  ghost: '/tipos/ghost.svg',
  dragon: '/tipos/dragon.svg',
  dark: '/tipos/dark.svg',
  steel: '/tipos/steel.svg',
  fairy: '/tipos/fairy.svg'
}

</script>


<style>
@import url('https://fonts.googleapis.com/css2?family=Roboto:ital,wght@0,100..900;1,100..900&display=swap');
body {
  display: block;
  font-family: 'Roboto', sans-serif;
  margin: 0;
  box-sizing: border-box; 
  background: radial-gradient(circle at top left, #fbcb33, transparent 60%),
              radial-gradient(circle at bottom right, #3b4cca, transparent 60%),
              #111;

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
    max-width: 1250px; /* limite máximo opcional */
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
  justify-content: center;
    align-content: center;
    
}
h1 {
  text-align: left;
}

.logo {
    width: 40%;
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 0px 0px 10px 29%;
}


.btn-carregar-mais {
  margin: 10px auto;
  display: block;
  padding: 10px 20px;
  border-radius: 12px;
  background-color: #82CAFF;
  border: none;
  font-weight: bold;
  cursor: pointer;
}
.btn-carregar-mais:hover {
  background-color: #4fa3e0;
}



.filtro-tipos {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin: 20px 0;
  justify-content: center;
}

.tipo-icone {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: #f0f0f0; /* fundo neutro */
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: transform 0.2s, background 0.2s;
  box-shadow: 0 2px 5px rgba(0,0,0,0.1);
}

.tipo-icone img {
  width: 24px;
  height: 24px;
}

.tipo-icone:hover {
  transform: scale(1.1);
  background: #e0e0e0;
}

.tipo-icone.ativo {
  background: #ffcc00; /* destaque quando selecionado */
  transform: scale(1.2);
}

@media screen and (min-width: 320px) {
  .pokedex-wrapper {
  max-width: 220px;
  margin: -1rem auto;
  border-radius: 1rem;
  }
}
@media screen and (min-width: 420px) {
    .pokedex-wrapper {
  max-width: 320px;
  margin: -1rem auto;
  border-radius: 1rem;
  }
  .lista {
    grid-template-columns: 1fr 1fr;
  }
}
@media screen and (min-width: 768px) {
    .pokedex-wrapper {
  max-width: 668px;
  margin: -1rem auto;
  border-radius: 1rem;
  }
  .lista {
    grid-template-columns: 1fr 1fr 1fr;
  }
  
}
@media screen and (min-width: 1250px) {
    .pokedex-wrapper {
  max-width: 1250px;
  margin: -1rem auto;
  border-radius: 1rem;
  }
  .lista {
    grid-template-columns: 1fr 1fr 1fr 1fr;
  } 
}
@media screen and (min-width: 635px) {
  .progress-bar {
    max-width: 85%;
  }
}
@media screen and (min-width: 935px) {
  .progress-bar {
    max-width: 75%;
  }
}
@media screen and (min-width: 1200px) {
  .progress-bar {
    max-width: 85%;
  }
}
/* @media (max-width: 600px) {
  .modal-content {
    width: 95%;
    max-height: 90vh;
    padding: 15px;
  }
} */

</style>