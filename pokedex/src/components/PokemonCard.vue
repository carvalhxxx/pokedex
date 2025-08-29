<template>
  <div class="pokemon-card" :style="{backgroundColor: corFundo}" @click="abrirModal">
    <h3>{{ pokemon.name }}</h3>
    <div class="tipos-e-imagem">
      <div class="tipos">
        <span v-for="tipo in tipos" :key="tipo" class="tipo" :style="{ backgroundColor: corBorda, borderColor: corBorda,  boxShadow: '0 8px 20px rgba(0, 0, 0, 0.1)'}">
          {{ tipo }}
        </span>
      </div>
      <img v-if="imagem" :src="imagem" :alt="pokemon.name"/>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { computed } from 'vue'

const props = defineProps({
    pokemon: Object,
    tipos: {
        type: Array,
        required: true
    }
});

const tipos = ref([])
const imagem = ref('')
const stats = ref([])
const moves = ref([])
const evolucoes = ref([])

async function carregarDetalhes() {
    const res = await fetch(props.pokemon.url)
    const data = await res.json()

    imagem.value = data.sprites.other['official-artwork']?.front_default || data.sprites.front_default || ''
    tipos.value = data.types.map(t => t.type.name)
    stats.value = data.stats
    moves.value = data.moves

    // pegar evoluções
    const resSpecies = await fetch(data.species.url)
    const speciesData = await resSpecies.json()
    const resEvol = await fetch(speciesData.evolution_chain.url)
    const evolData = await resEvol.json()
    evolucoes.value = processarEvolucoes(evolData.chain)
}

function processarEvolucoes(chain) {
    const evols = []
    function percorrer(c) {
        evols.push(c.species.name)
        c.evolves_to.forEach(e => percorrer(e))
    }
    percorrer(chain)
    return evols
}

const coresPorTipo = {
  normal:   '#A8A878', 
  fire:     '#F08030', 
  water:    '#6890F0', 
  grass:    '#78C850', 
  electric: '#F8D030', 
  ice:      '#98D8D8', 
  fighting: '#C03028', 
  poison:   '#A040A0', 
  ground:   '#E0C068', 
  flying:   '#A890F0', 
  psychic:  '#F85888', 
  bug:      '#A8B820', 
  rock:     '#B8A038', 
  ghost:    '#705898',
  dragon:   '#7038F8', 
  dark:     '#705848', 
  steel:    '#B8B8D0', 
  fairy:    '#EE99AC' 
}

const corFundo = computed(() => { 
    if (!tipos.value.length) return "#ffffff";

    const tipoPrincipal = tipos.value[0].toLowerCase(); 
    
    return coresPorTipo[tipoPrincipal] || "white";
});

function escurecerCor(hex, percent) {
  const num = parseInt(hex.replace("#",""),16)
  let r = (num >> 16) & 0xFF
  let g = (num >> 8) & 0xFF
  let b = num & 0xFF

  r = Math.floor(r * (1 - percent/100))
  g = Math.floor(g * (1 - percent/100))
  b = Math.floor(b * (1 - percent/100))

  return "#" + ((1 << 24) + (r <<16) + (g << 8) + b).toString(16).slice(1)
}

const corBorda = computed(() => escurecerCor(corFundo.value, 20))



const emit = defineEmits(['abrir-modal'])

function abrirModal() {
    emit('abrir-modal', { 
        ...props.pokemon,
        tipos: tipos.value,
        imagem: imagem.value,
        cor: corFundo.value,
        corBorda: corBorda.value,
        stats: stats.value,
        moves: moves.value,
        evolucoes: evolucoes.value
    })
}
onMounted(() => {
    carregarDetalhes()
})
</script>

<style scoped>
.pokemon-card {
    border: 1px solid #ccc;
    padding: 10px;
    display: flex;
    flex-direction: column;      /* h3 em cima, tipos+imagem abaixo */
    justify-content: flex-start; /* tudo no topo */
    margin: 2px;
    height: 110px;
    max-width: 100%;                /* largura maior para caber imagem */
    border-radius: 15px;
    transition: all 0.1s ease;
    position: relative;
    overflow: hidden;
}
.pokemon-card::before {
    content: "";
    position: absolute;
    top: -3px;
    right: -42px;
    width: 230px;
    height: 207px;
    background-image: url(/pokebola.png);
    background-size: contain;
    background-repeat: no-repeat;
    opacity: 0.3;
    z-index: 0;        /* fica atrás */
}
.tipos-e-imagem {
    display: flex;
    flex-direction: row;         /* tipos à esquerda, imagem à direita */
    justify-content: space-between;
    align-items: flex-start;
}
.info {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
}
.tipos{
    display: flex;
    flex-direction: column;
    gap: 4px;
    margin-top: 4px;
}
.tipo {
    border: 1px solid white;
    padding: 2px 6px;
    border-radius: 15px;
    font-size: 12px;
    text-transform: capitalize;
    text-align: center;
    color: white;
}
.pokemon-card:active {
    box-shadow: 0 8px 20px rgba(0, 0, 0, 0.3);
    transform: translateY(-5px);
    cursor: pointer;
}
.pokemon-card img {
    width: 100px;
    height: 108px;
    object-fit: contain;
    position: relative;
    display: flex;
    max-width: 100%;
    max-height: 90%;
    align-self: flex-end;
    top: -50px;
}
.pokemon-card h3 {
    text-transform: capitalize;
    color: white;
    font-size: 22px;
    margin: 0 0 25px 0;
}
p {
    border: 1px solid gray;
    border-radius: 5px;
}
.pokebola {
    z-index: 0;
}

</style>