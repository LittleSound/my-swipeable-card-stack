<script lang="ts" setup>
import type { CardType } from '~/components/GameCard.vue'
const props = defineProps<{
  cards: CardType[]
}>()

const hideCards = ref<CardType[]>([])

function onHideCard(card: CardType) {
  hideCards.value.push(card)
}

const visibleCards = computed(() => props.cards.filter(item => !hideCards.value.includes(item)).slice(0, 3))
</script>

<template>
  <div z-1 flex relative m="y-50px x-auto" w="300px" h="350px" preserve-3d class="expand-boundary">
    <GameCard
      v-for="(card, index) of visibleCards"
      :key="card.keyword"
      :card="card"
      :is-current="index === 0"
      @hide="onHideCard"
    />
  </div>
</template>

<style lang="scss" scoped>
.expand-boundary::after {
  content: '';
  position: absolute;
  top: -40px;
  left: -40px;
  right: -40px;
  bottom: -40px;
}
</style>
