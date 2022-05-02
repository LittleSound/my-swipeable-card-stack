<script setup lang="ts">
import type { CardType } from '~/components/GameCard.vue'
const cards: CardType[] = [
  {
    keyword: 'Test',
  },
  {
    keyword: 'Vue.js',
  },
  {
    keyword: 'Vite.js',
  },
  {
    keyword: 'Test 2',
  },
  {
    keyword: 'Vue.js 2',
  },
  {
    keyword: 'Vite.js 2',
  },
  {
    keyword: 'Test 3',
  },
  {
    keyword: 'Vue.js 3',
  },
  {
    keyword: 'Vite.js 3',
  },
]
</script>

<template>
  <div class="z-2">
    <GameCardStack :cards="cards" />
  </div>
</template>
