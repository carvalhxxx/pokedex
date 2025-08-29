<template>

  <!-- <input type="text" v-model="busca" placeholder="Digite o nome de um Pokemon"/> -->
  <!-- <hr> -->
  <div class="pokedex-wrapper">
      <img class="logo" src="/public/pokedex.png">
        <div class="filtro-tipos">
          <span 
            v-for="tipo in Object.keys(coresPorTipo)" 
            :key="tipo"
            class="tipo-icone"
            :style="{ backgroundColor: coresPorTipo[tipo] }"
            @click="tipoSelecionado = (tipoSelecionado === tipo ? null : tipo)"
            :class="{ativo: tipoSelecionado === true}"
          >
            <img :src="iconesPorTipo[tipo]" :alt="tipo" class="icone-tipo" />
          </span>
        </div>

    <div class="lista">
      <PokemonCard
        v-for="pokemon in pokemonsFiltrados"
        :key="pokemon.name"
        :pokemon="pokemon"
        @abrir-modal="abrirModal"
      />
    </div>
    <button @click="carregarMais" class="btn-carregar-mais">Carregar Mais</button>
  </div>

<div v-if="pokemonSelecionado" class="modal-overlay" @click.self="fecharModal">
  <div class="modal-content" :class="{ aberto: modalAtivo }" :style="{ backgroundColor: pokemonSelecionado.cor }">
    <span class="fechar" @click="fecharModal"><i class="fas fa-arrow-left"></i></span>

    <!-- Nome do Pokémon -->
    <h2 class="nome-pokemon">{{ pokemonSelecionado.name }}</h2>

    <!-- Tipos abaixo do nome -->
    <div class="tipos-modal">
      <span v-for="tipo in pokemonSelecionado.tipos" 
            :key="tipo"
            class="tipo"
            :style="{
              backgroundColor: pokemonSelecionado.corBorda,
              borderColor: pokemonSelecionado.corBorda,
              boxShadow: '1px 8px 20px rgba(0, 0, 0, 0.1)'
            }">
        {{ tipo }}
      </span>
    </div>

    <!-- Imagem centralizada um pouco abaixo -->
    <div class="imagem-modal">
      <img :src="pokemonSelecionado.imagem" :alt="pokemonSelecionado.name"/>
    </div>

    <div class="infos-modal">
      <div class="tabs">
        <button :class="{ ativa: abaAtiva === 'sobre' }" 
                @click="abaAtiva = 'sobre'" 
                :style="{ borderBottomColor: abaAtiva === 'sobre' ? pokemonSelecionado?.cor : '' }">Sobre</button>
        <button :class="{ ativa: abaAtiva === 'stats' }" 
                @click="abaAtiva = 'stats'" 
                :style="{ borderBottomColor: abaAtiva === 'stats' ? pokemonSelecionado?.cor : '' }">Base Stats</button>
        <button :class="{ ativa: abaAtiva === 'evolution' }" 
                @click="abaAtiva = 'evolution'" 
                :style="{ borderBottomColor: abaAtiva === 'evolution' ? pokemonSelecionado?.cor : '' }">Evoluções</button>
      </div>

      <div class="about" v-if="abaAtiva === 'sobre'" >
        <div class="linha" >
          <span class="titulo" :style="{ color: pokemonSelecionado.cor}">Altura:</span>
          <span class="valor">{{ pokemonSelecionado.height / 10 }} m</span>
        </div>
        <div class="linha" >
          <span class="titulo" :style="{ color: pokemonSelecionado.cor}">Peso:</span>
          <span class="valor">{{ pokemonSelecionado.weight / 10 }} kg</span>
        </div>
        <div class="linha">
          <span class="titulo" :style="{ color: pokemonSelecionado.cor}">Tipos:</span>
          <span class="valor">{{ pokemonSelecionado.tipos.join(', ') }}</span>
        </div>
        <div class="linha">
          <span class="titulo" :style="{ color: pokemonSelecionado.cor}">Habilidades:</span>
          <span class="valor">{{ pokemonSelecionado.abilities.map(a => a.ability.name).join(', ') }}</span>
        </div>
      </div>



      <div v-if="abaAtiva === 'stats'">
        <ul>
          <li v-for="s in pokemonSelecionado.stats" :key="s.stat.name" class="stat-item">
            <span class="stat-name" :style=" { color: pokemonSelecionado.cor}"> {{ statAbreviaturas[s.stat.name] || s.stat.name }}: </span>
            <span class="stat-value">{{ s.base_stat }} </span>
            <div class="progress-bar">
              <div class="progress-fill" :style="{ width: (s.base_stat / 255 * 100) + '%', backgroundColor: pokemonSelecionado.cor}"></div>
            </div>
          </li>
        </ul>
      </div>

      <div v-if="abaAtiva === 'evolution'">
        <ul>
          <li v-for="e in pokemonSelecionado.evolucoes" :key="e">
            {{ e }}
          </li>
        </ul>
      </div>
    </div>
  </div>
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
  max-height: 100vh;
  height: 75vh;
  width: 100%;
  text-align: center;
  transform: translateY(100%);
  transition: transform 0.5s ease-out;

}
.modal-overlay .modal-content.aberto {
  transform: translateY(0);
}
.modal-content img {
  width: 250px;
  height: auto;
  position: relative;
  top: 7vh;
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
  position: relative;
  top: 10px;
}
.modal-content button {
  align-items: flex-end;
}
.modal-content .fechar {
  position: absolute;
  top: 35px;
  left: 20px;
  cursor: pointer;
  z-index: 10;
}
.modal-content .fechar:hover {
  color: #82caff;
}
.infos-modal {
  background-color: white;
  height: 100%;        /* já está */
  border-radius: 25px;
  display: flex;
  flex-direction: column;
  padding: 20px;
}

.tabs {
  display: flex;
  justify-content: space-around;
  margin-top: 15px;
  border-bottom: none;
  padding-top: 30px;
}
.tabs button {
  background: none;
  border: none;
  padding: 10px 15px;
  cursor: pointer;
  font-weight: 300;
  color: #555;
  border-bottom: 1px solid transparent;
  transition: border-color 0.2s;
  border-radius: 0;
}
.tabs button.ativa {
  font-weight: 400;
  border-bottom: 2px solid #333 ;
  color: #000;
}
.stat-item {
  margin: 20px 0;
  list-style: none;
  display: flex;
  align-items: center;
  justify-content: flex-start;
  gap: 8px;
  width: 100%;
}
.stat-name {
  display: inline-block;
  width: 45px;
  text-transform: capitalize;
  font-weight: bold;
  text-align: left;
}
.nome-pokemon {
  color: white;
}
.stat-value {
  width: 35px;
  text-align: left;
}
.progress-bar {
  flex: 1;
  vertical-align: middle;
  height: 12px;
  background: #eee;
  border-radius: 5px;
  overflow: hidden;
  max-width: 55%;       /* opcional: limite em telas grandes */
  min-width: 150px; 
}
.progress-fill {
  height: 100%;
  transition: width 0.3 ease;
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
.tipos-modal {
  display: flex;
  justify-content: center;
  gap: 8px;
  margin-top: 10px;
  border-radius: 25px;
}
.tipos-modal .tipo {
  border: 1px solid white;
  padding: 4px 10px;
  border-radius: 15px;
  font-size: 14px;
  text-transform: capitalize;
  text-align: center;
  color: white;
}

.about {
  flex: 1;             /* ocupa todo o espaço disponível */
  display: flex;
  flex-direction: column;
  gap: 25px;
  padding-left: 40px;
  overflow-y: auto;    /* se houver muito conteúdo, aparece scroll */
      margin-top: 16px; 
}

.linha {
  display: grid;
  grid-template-columns: 100px auto; /* aumentei a largura da coluna do título */
  align-items: start;
  gap: 10px; /* 👈 espaço extra entre colunas */
}

.titulo {

  text-align: left;
  text-transform: uppercase;
  font-weight: bold;
}

.valor {
  text-align: left;
    font-weight: 500;
    grid-template-columns: 200px auto;
    text-transform: capitalize;
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
    max-width: 75%;
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
@media (max-width: 600px) {
  .modal-content {
    width: 95%;
    max-height: 90vh;
    padding: 15px;
  }
}

</style>