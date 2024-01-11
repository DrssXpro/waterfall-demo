<template>
  <div class="app">
    <div class="container" ref="fContainerRef">
      <fs-waterfall :bottom="20" :column="column" :gap="10" :page-size="20" :request="getData">
        <template #item="{ index }">
          <div
            class="card-box"
            :style="{
              background: colorArr[index % (colorArr.length - 1)],
            }"
          >
            <!-- <img :src="item.url" /> -->
          </div>
        </template>
      </fs-waterfall>
    </div>
  </div>
</template>

<script setup lang="ts">
import data1 from "./config/data1.json";
import data2 from "./config/data2.json";
import FsWaterfall from "./components/FsWaterfall.vue";
import { ICardItem } from "./components/type";
import { onMounted, onUnmounted, ref } from "vue";

const colorArr = ["#409eff", "#67c23a", "#e6a23c", "#f56c6c", "#909399"];

const fContainerRef = ref<HTMLDivElement | null>(null);
const column = ref(4);
const fContainerObserver = new ResizeObserver((entries) => {
  changeColumn(entries[0].target.clientWidth);
});

const changeColumn = (width: number) => {
  if (width > 960) {
    column.value = 5;
  } else if (width >= 690 && width < 960) {
    column.value = 4;
  } else if (width >= 500 && width < 690) {
    column.value = 3;
  } else {
    column.value = 2;
  }
};

onMounted(() => {
  fContainerRef.value && fContainerObserver.observe(fContainerRef.value);
});

onUnmounted(() => {
  fContainerRef.value && fContainerObserver.unobserve(fContainerRef.value);
});

const list1: ICardItem[] = data1.data.items.map((i) => ({
  id: i.id,
  url: i.note_card.cover.url_pre,
  width: i.note_card.cover.width,
  height: i.note_card.cover.height,
}));
const list2: ICardItem[] = data2.data.items.map((i) => ({
  id: i.id,
  url: i.note_card.cover.url_pre,
  width: i.note_card.cover.width,
  height: i.note_card.cover.height,
}));

const list = [...list1, ...list2];

console.log(list.length);

const getData = (page: number, pageSize: number) => {
  return new Promise<ICardItem[]>((resolve) => {
    setTimeout(() => {
      resolve(list.slice((page - 1) * pageSize, (page - 1) * pageSize + pageSize));
    }, 1000);
  });
};
</script>

<style scoped lang="scss">
.app {
  width: 100vw;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  .container {
    width: 1400px;
    height: 600px;
    border: 1px solid red;
  }
  .card-box {
    position: relative;
    width: 100%;
    height: 100%;
    border-radius: 10px;
  }
}
</style>
