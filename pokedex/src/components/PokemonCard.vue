<template>
  <div class="pokemon-card" :style="{backgroundColor: corFundo}" @click="abrirModal">
    <h3>{{ pokemon.name }}</h3>
    <div class="tipos-e-imagem">
      <div class="tipos">
        <span v-for="tipo in tipos" :key="tipo" class="tipo" :style="{ backgroundColor: corBorda, borderColor: corBorda}">
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

async function carregarDetalhes() {
    const res = await fetch(props.pokemon.url)
    const data = await res.json()
    imagem.value = data.sprites.other['official-artwork']?.front_default || data.sprites.front_default || ''
    tipos.value = data.types.map(t => t.type.name)

    console.log('Detalhes carregados:', props.pokemon.name, imagem.value)
    console.log(tipos.value)
    console.log(coresPorTipo[tipos.value])
}

const coresPorTipo = {
    fire: '#FA6161',
    water: '#82CAFF',
    grass: '#5EE0C1',
    electric: '#FAD85F',
    poison: '#B818D6',
    bug: '#90EE90',
    normal: '#808080'
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
    emit('abrir-modal', {... props.pokemon, tipos: tipos.value, imagem: imagem.value})
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
    width: 96px;                /* largura maior para caber imagem */
    border-radius: 15px;
    transition: all 0.1s ease;
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
    width: 45px;
    height: 45px;
    object-fit: contain;
}
.pokemon-card h3 {
    text-transform: capitalize;
    color: white;
    font-size: 12px;
    margin: 0 0 25px 0;
}
p {
    border: 1px solid gray;
    border-radius: 5px;
}
</style>