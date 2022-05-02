<!-- 游戏卡片 -->
<script lang="ts" setup>
import interact from 'interactjs'

export interface CardType {
  keyword: string
}

const props = defineProps<{
  card: CardType
  /** 是第一张 */
  isCurrent: boolean
}>()
const emits = defineEmits<{
  (event: Interaction): void
  (event: 'acceptCard'): void
  (event: 'rejectCard'): void
  (event: 'skipCard'): void
  (event: 'hide', card: CardType): void
}>()

/** 卡片交互触发阈值 X */
const interactXThreshold = 100
/** 卡片交互触发阈值 Y */
const interactYThreshold = 150
/** 卡片退出位置 X */
const interactOutOfSightXCoordinate = 1000
/** 卡片退出位置 Y */
const interactOutOfSightYCoordinate = 1000

/** 最大旋转角度 */
const interactMaxRotation = 15
/** 最大 3D 旋转角度 */
const interactMax3DRotation = 15

/** 卡片位置 */
const interactPosition = reactive({
  x: 0,
  y: 0,
  rotationX: 0,
  rotationY: 0,
  rotationZ: 0,
})

function setPosition(coordinates: Partial<typeof interactPosition> = {}) {
  for (const _item in interactPosition) {
    const item = _item as keyof typeof interactPosition
    interactPosition[item] = coordinates[item] || 0
  }
}

/** 卡片缓动样式开关 */
const isAnimating = ref(true)

/** 卡片元素 */
const cardElement = ref<HTMLDivElement>()

/** 卡片位置的 CSS 变换字符串 */
const transformString = computed(() => {
  if (props.isCurrent) {
    const { x, y, rotationX, rotationY, rotationZ } = interactPosition
    return `translate3D(${x}px, ${y}px, 500px) rotateX(${rotationX}deg) rotateY(${rotationY}deg) rotateZ(${rotationZ}deg)`
  }
  return ''
})

/** 重置卡片坐标 */
function resetCardPosition() {
  setPosition()
}

const isShowing = ref(true)

function hideCard() {
  setTimeout(() => {
    isShowing.value = false
    emits('hide', props.card)
  }, 300)
}

function playCard(interaction: Interaction) {
  interactUnsetElement()
  unsetHoverMove()

  switch (interaction) {
    case Interaction.AcceptCard:
      setPosition({
        x: interactOutOfSightXCoordinate,
      })
      break
    case Interaction.RejectCard:
      setPosition({
        x: -interactOutOfSightXCoordinate,
      })
      break
    case Interaction.SkipCard:
      setPosition({
        y: interactOutOfSightYCoordinate,
      })
      break
  }
  emits(interaction)
  hideCard()
}

function initInteract() {
  if (cardElement.value && props.isCurrent) {
    /** 使用 interactjs 来实现拖拽效果 */
    interact(cardElement.value).draggable({
      onmove: (event) => {
        interactPosition.x += event.dx
        interactPosition.y += event.dy

        let rotation = interactMaxRotation * (interactPosition.x / interactXThreshold)

        if (rotation > interactMaxRotation)
          rotation = interactMaxRotation
        else if (rotation < -interactMaxRotation)
          rotation = -interactMaxRotation

        interactPosition.rotationZ = rotation
      },
      onstart: () => {
        isAnimating.value = false
      },
      onend: () => {
        isAnimating.value = true

        if (interactPosition.y > interactYThreshold)
          playCard(Interaction.SkipCard)
        else if (interactPosition.x > interactXThreshold)
          playCard(Interaction.AcceptCard)
        else if (interactPosition.x < -interactXThreshold)
          playCard(Interaction.RejectCard)
        else
          resetCardPosition()
      },
    })
  }
}

function interactUnsetElement() {
  if (cardElement.value)
    interact(cardElement.value).unset()
}

function onMouseMove(event: MouseEvent) {
  if (!isAnimating.value)
    return
  const rotationY = (event.offsetX / (cardElement.value!.offsetWidth / 2) - 1) * interactMax3DRotation
  const rotationX = -(event.offsetY / (cardElement.value!.offsetHeight / 2) - 1) * interactMax3DRotation
  setPosition({
    rotationX,
    rotationY,
  })
}

function onMouseOut(event: MouseEvent) {
  if (!isAnimating.value)
    return
  resetCardPosition()
}

function initHoverMove() {
  if (cardElement.value) {
    cardElement.value.parentElement!.addEventListener('mousemove', onMouseMove)
    cardElement.value.parentElement!.addEventListener('mouseout', onMouseOut)
  }
}

function unsetHoverMove() {
  if (cardElement.value) {
    cardElement.value.parentElement!.removeEventListener('mousemove', onMouseMove)
    cardElement.value.parentElement!.removeEventListener('mouseout', onMouseOut)
  }
}

watch(() => props.isCurrent, () => {
  initInteract()
  initHoverMove()
})
onMounted(() => {
  initInteract()
  initHoverMove()
})
</script>

<script lang="ts">
export enum Interaction {
  AcceptCard = 'acceptCard',
  RejectCard = 'rejectCard',
  SkipCard = 'skipCard',
}
</script>

<template>
  <div
    ref="cardElement"
    class="card"
    :class="{
      isCurrent,
      isAnimating,
    }"
    :style="{ transform: transformString }"
  >
    <div class="card-content">
      <h3 class="cardTitle">
        {{ card.keyword }}
      </h3>
    </div>
  </div>
</template>

<style lang="scss" scoped>
@import "../styles/index.scss";

$cardsTotal: 3;
$cardsWidth: 300px;
$cardsPositionOffset: 55vh * 0.06;
$cardsScaleOffset: 0.08;
$defaultTranslation: $cardsPositionOffset * $cardsTotal;
$defaultScale: 1 - ($cardsScaleOffset * $cardsTotal);
$fs-card-title: 1.125em;

.card {
  @include card();
  @include absolute(0);
  @include sizing(100% 80vw);
  @include flex-center();

  @include after() {
    @include sizing(21px 3px);
    @include absolute(right 0 bottom 11px left 0);

    margin: auto;
    border-radius: 100px;
    background: rgba($c-black, 0.3);
  }

  touch-action: none;
  user-select: none;

  display: flex;
  max-height: 350px;
  margin: auto;
  font-size: $fs-h2;
  font-weight: $fw-bold;
  color: $c-white;
  background-image: linear-gradient(
    -180deg,
    $primary-gradient-start 2%,
    $primary-gradient-end 100%
  );
  opacity: 0;
  transform: translateY($defaultTranslation) scale($defaultScale);
  /* transform-origin: 50%, 100%; */
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  user-select: none;
  pointer-events: none;
  will-change: transform, opacity;

  height: 100vw;

  &.isCurrent {
    pointer-events: auto;
    .cardTitle {
      transform-style: preserve-3d;
      transform: translateZ(100px);
    }
  }
  transform-style: preserve-3d;

  &.isAnimating {
    transition: transform 0.7s $ease-out-back;
  }
}

.card-content {
  transform-style: preserve-3d;
  pointer-events: none;
}
.cardTitle {
  margin: 0 0 15px;
  font-size: $fs-card-title;

  pointer-events: none;
  touch-action: none;
  user-select: none;
}

@for $i from 1 through $cardsTotal {
  $index: $i - 1;
  $translation: $cardsPositionOffset * $index;
  $scale: 1 - ($cardsScaleOffset * $index);

  .card:nth-child(#{$i}) {
    z-index: $cardsTotal - $index;
    opacity: 1;
    transform: translateY($translation) scale($scale);

    @if $i == 3 {
      color: $c-red-25;
      background-color: $c-red-25;
    } @else if $i == 2 {
      color: $c-red-50;
      background-color: $c-red-50;
    }

    @if $i != 1 {
      background-image: none;

      @include after() {
        @include sizing(0 0);
      }
    }
  }
}
</style>
