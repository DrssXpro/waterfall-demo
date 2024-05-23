<template>
  <div class="fs-waterfall-container" ref="containerRef" @scroll="handleScroll">
    <div class="fs-waterfall-list">
      <div
        class="fs-waterfall-item"
        v-for="(item, index) in state.cardList"
        :key="item.id"
        :style="{
          width: `${state.cardPos[index].width}px`,
          height: `${state.cardPos[index].height}px`,
          transform: `translate3d(${state.cardPos[index].x}px, ${state.cardPos[index].y}px, 0)`,
        }"
      >
        <slot name="item" :item="item" :index="index"></slot>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, onUnmounted, reactive, ref, watch } from "vue";
import { debounce, rafThrottle } from "./tool";
import type { IWaterFallProps, ICardItem, ICardPos } from "./type";

const props = defineProps<IWaterFallProps>();

defineSlots<{
  item(props: { item: ICardItem; index: number }): any;
}>();

const containerRef = ref<HTMLDivElement | null>(null);

const resizeObserver = new ResizeObserver(() => {
  handleResize();
});

const state = reactive({
  isFinish: false,
  loading: false,
  page: 1,
  cardWidth: 0,
  cardList: [] as ICardItem[],
  cardPos: [] as ICardPos[],
  columnHeight: new Array(props.column).fill(0) as number[],
});

const minColumn = computed(() => {
  let minIndex = -1,
    minHeight = Infinity;

  state.columnHeight.forEach((item, index) => {
    if (item < minHeight) {
      minHeight = item;
      minIndex = index;
    }
  });

  return {
    minIndex,
    minHeight,
  };
});

watch(
  () => props.column,
  () => {
    handleResize();
  }
);

const getCardList = async (page: number, pageSize: number) => {
  if (state.isFinish) return;
  state.loading = true;
  const list = await props.request(page, pageSize);
  state.page++;
  if (!list.length) {
    state.isFinish = true;
    return;
  }
  state.cardList = [...state.cardList, ...list];
  computedCardPos(list);
  state.loading = false;
};

const computedCardPos = (list: ICardItem[]) => {
  list.forEach((item, index) => {
    const cardHeight = Math.floor((item.height * state.cardWidth) / item.width);
    if (index < props.column && state.cardList.length <= props.pageSize) {
      state.cardPos.push({
        width: state.cardWidth,
        height: cardHeight,
        x: index ? index * (state.cardWidth + props.gap) : 0,
        y: 0,
      });
      state.columnHeight[index] = cardHeight + props.gap;
    } else {
      const { minIndex, minHeight } = minColumn.value;
      state.cardPos.push({
        width: state.cardWidth,
        height: cardHeight,
        x: minIndex ? minIndex * (state.cardWidth + props.gap) : 0,
        y: minHeight,
      });
      state.columnHeight[minIndex] += cardHeight + props.gap;
    }
  });
};

const init = () => {
  if (containerRef.value) {
    const containerWidth = containerRef.value.clientWidth;
    state.cardWidth = (containerWidth - props.gap * (props.column - 1)) / props.column;
    getCardList(state.page, props.pageSize);
    resizeObserver.observe(containerRef.value);
  }
};

const handleScroll = rafThrottle(() => {
  const { scrollTop, clientHeight, scrollHeight } = containerRef.value!;
  const bottom = scrollHeight - clientHeight - scrollTop;
  if (bottom <= props.bottom) {
    !state.loading && getCardList(state.page, props.pageSize);
  }
});

const handleResize = debounce(() => {
  const containerWidth = containerRef.value!.clientWidth;
  state.cardWidth = (containerWidth - props.gap * (props.column - 1)) / props.column;
  state.columnHeight = new Array(props.column).fill(0);
  state.cardPos = [];
  computedCardPos(state.cardList);
});

onMounted(() => {
  init();
});

onUnmounted(() => {
  containerRef.value && resizeObserver.unobserve(containerRef.value);
});
</script>

<style scoped lang="scss">
.fs-waterfall {
  &-container {
    width: 100%;
    height: 100%;
    overflow-y: scroll;
    overflow-x: hidden;
  }

  &-list {
    width: 100%;
    position: relative;
  }
  &-item {
    position: absolute;
    left: 0;
    top: 0;
    box-sizing: border-box;
    transition: all 0.3s;
  }
}
</style>
