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
    <button @click="carregarMais" class="btn-carregar-mais">Carregar Mais</button>
  </div>

<div v-if="pokemonSelecionado" class="modal-overlay" @click.self="fecharModal">
  <div class="modal-content" :class="{ aberto: modalAtivo }" :style="{ backgroundColor: pokemonSelecionado.cor }">
    <span class="fechar" @click="fecharModal"><i class="fas fa-arrow-left"></i></span>
    <h2>{{ pokemonSelecionado.name }}</h2>
    <img :src="pokemonSelecionado.imagem" :alt="pokemonSelecionado.name"/>
    <p class="tipo-modal">{{ pokemonSelecionado.tipos.join(', ') }}</p>

    <div class="infos-modal">
      <div class="tabs">
        <button :class="{ ativa: abaAtiva === 'sobre' }" @click="abaAtiva = 'sobre'" :style="{ borderBottomColor: abaAtiva === 'sobre' ? pokemonSelecionado?.cor : '' }" >Sobre</button>
        <button :class="{ ativa: abaAtiva === 'stats' }" @click="abaAtiva = 'stats'" :style="{ borderBottomColor: abaAtiva === 'stats' ? pokemonSelecionado?.cor : '' }">Base Stats</button>
        <button :class="{ ativa: abaAtiva === 'evolution' }" @click="abaAtiva = 'evolution'" :style="{ borderBottomColor: abaAtiva === 'evolution' ? pokemonSelecionado?.cor : '' }">Evoluções</button>
      </div>

      <!-- Conteúdo das abas -->
      <div v-if="abaAtiva === 'sobre'" >
        <p>Altura: {{ pokemonSelecionado.weight / 10 }} m</p>
          <p><strong>Peso:</strong> {{ pokemonSelecionado.weight / 10 }} kg</p>
            <p><strong>Tipos:</strong> {{ pokemonSelecionado.tipos.join(', ') }}</p>
        <p><strong>Habilidades:</strong> {{ pokemonSelecionado.abilities.map(a => a.ability.name).join(', ') }}</p>
        <p><strong>Experiência Base:</strong> {{ pokemonSelecionado.base_experience }}</p>
        <p><strong>Descrição:</strong> {{ pokemonSelecionado.descricao }}</p> 
      </div>

      <div v-if="abaAtiva === 'stats'">
        <ul>
          <li v-for="s in pokemonSelecionado.stats" :key="s.stat.name" class="stat-item">
            <span class="stat-name"> {{ statAbreviaturas[s.stat.name] || s.stat.name }}: </span>
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
import { onMounted, ref, computed, nextTick, watch } from 'vue'
import '@fortawesome/fontawesome-free/css/all.css'
import PokemonCard from './components/PokemonCard.vue'

const pokemons = ref([])
let offset = 0
const limit = 35
const busca = ref([])

async function carregarPokemons() {
  const res = await fetch(`https://pokeapi.co/api/v2/pokemon?limit=${limit}&offset=${offset}`)
  const data = await res.json()
  pokemons.value.push(...data.results)
  offset += limit
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

// async function abrirModal(pokemon) {
//   try {
//     const res = await fetch(pokemon.url)
//     const data = await  res.json()

//     const speciesRes = await fetch(data.species.url)
//     const speciesData = await speciesRes.json()

//     const descricao = speciesData.flavor_text_entries.find(
//       entry => entry.language.name === 'en'
//     )?.flavor_text.replace(/\n|\f/g, '') || ''

//     const evolucoesRes = await fetch(speciesData.evolution_chain.url)
//     const evolucoesData = await evolucoesRes.json()

//     function extrairEvolucoes(chain) {
//       const nomes = [chain.species.name]
//       if (chain.evolves_to.length > 0) {
//         chain.evolves_to.forEach(e => nomes.push(...extrairEvolucoes(e)))
//       }
//       return nomes
//     }
//     const evolucoes = extrairEvolucoes(evolucoesData.chain)

//     pokemonSelecionado.value = {
//       ...data,
//       tipos: data.types.map(t => t.type.name),
//       abilities: data.abilities,
//       evolucoes,
//       descricao,
//        imagem: data.sprites.other['official-artwork'].front_default || data.sprites.front_default,
//        cor : corFundo.value
//     }

//     nextTick(() => {
//       setTimeout(() => {
//         modalAtivo.value = true
//         document.body.style.overflow = 'hidden'
//       }, 10)
//     })
//   } catch (err) {
//     console.error('Erro ao abrir modal do Pokémon:', err)
//   }
// }

// function abrirModal(pokemonCompleto) {
//   // pokemonCompleto já tem cor, imagem, stats, evoluções
//   pokemonSelecionado.value = pokemonCompleto

//   nextTick(() => {
//     setTimeout(() => {
//       modalAtivo.value = true
//       document.body.style.overflow = 'hidden'
//     }, 10)
//   })
// }

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

    // Mantendo a cor que veio do card
    pokemonSelecionado.value = {
      ...data,
      tipos: data.types.map(t => t.type.name),
      abilities: data.abilities,
      evolucoes,
      descricao,
      imagem: data.sprites.other['official-artwork'].front_default || data.sprites.front_default,
      cor: pokemon.cor  // <- usa a cor que veio do card
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

const abaAtiva = ref('sobre');

const statAbreviaturas = {
  hp: 'HP',
  attack: 'Attack',
  defense: 'Defense',
  'special-attack': 'SP. Attack',
  'special-defense': 'SP. Defense',
  speed: 'Speed'
}

function carregarMais() {
  carregarPokemons()
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
  height: 100vh;
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
  height: 100%;
  border-radius: 25px;
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
  font-weight: bold;
  color: #555;
  border-bottom: 1px solid transparent;
  transition: border-color 0.2s;
  border-radius: 0;
}
.tabs button.ativa {
  border-bottom: 2px solid #333 ;
  color: #000;
}
.stat-item {
  margin: 40px 0;
  list-style: none;
  display: flex;
  align-items: center;
  justify-content: flex-start;
  gap: 8px;
}
.stat-name {
  display: inline-block;
  width: 90px;
  text-transform: capitalize;
  font-weight: bold;
  text-align: left;
}
.stat-value {
  width: 35px;
  text-align: left;
}
.progress-bar {
  display: inline-block;
  vertical-align: middle;
  width: 220px;
  height: 10px;
  background: #eee;
  border-radius: 5px;
  overflow: hidden;
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
@media screen and (min-width: 300px) {
  .pokedex-wrapper {
  max-width: 300px;
  margin: -1rem auto;
  border-radius: 1rem;
  }
}
@media screen and (min-width: 550px) {
    .pokedex-wrapper {
  max-width: 650px;
  margin: -1rem auto;
  border-radius: 1rem;
  }
  .lista {
    grid-template-columns: 1fr 1fr;
  }
}
@media screen and (min-width: 750px) {
    .pokedex-wrapper {
  max-width: 950px;
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
</style>