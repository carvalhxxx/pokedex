<template>
    <div>
        <h1>Pokédex</h1>
        <ul>
            <li v-for="pokemon in pokemons" :key="pokemon.name">
                {{ pokemon.name }}
            </li>
        </ul>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const pokemons = ref ([])

async function carregarPokemons() {
    const res = await fetch('https://pokeapi.co/api/v2/pokemon?limit=35')
    const data = await res.json()
    pokemons.value =  data.results
}

onMounted(() => {
    carregarPokemons()
})
</script>