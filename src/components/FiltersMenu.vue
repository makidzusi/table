<template>
  <div>
    <svg
    @click="isOpened = !isOpened" ref="reference"
      width="4"
      height="14"
      viewBox="0 0 4 14"
      fill="none"
      xmlns="http://www.w3.org/2000/svg"
    >
      <path
        d="M2.66634 7.00065C2.66634 6.63246 2.36786 6.33398 1.99967 6.33398C1.63148 6.33398 1.33301 6.63246 1.33301 7.00065C1.33301 7.36884 1.63148 7.66732 1.99967 7.66732C2.36786 7.66732 2.66634 7.36884 2.66634 7.00065Z"
        stroke="#C4CAD4"
        stroke-width="1.4"
        stroke-linecap="round"
        stroke-linejoin="round"
      />
      <path
        d="M2.66634 2.33398C2.66634 1.96579 2.36786 1.66732 1.99967 1.66732C1.63148 1.66732 1.33301 1.96579 1.33301 2.33398C1.33301 2.70217 1.63148 3.00065 1.99967 3.00065C2.36786 3.00065 2.66634 2.70217 2.66634 2.33398Z"
        stroke="#C4CAD4"
        stroke-width="1.4"
        stroke-linecap="round"
        stroke-linejoin="round"
      />
      <path
        d="M2.66634 11.6673C2.66634 11.2991 2.36786 11.0007 1.99967 11.0007C1.63148 11.0007 1.33301 11.2991 1.33301 11.6673C1.33301 12.0355 1.63148 12.334 1.99967 12.334C2.36786 12.334 2.66634 12.0355 2.66634 11.6673Z"
        stroke="#C4CAD4"
        stroke-width="1.4"
        stroke-linecap="round"
        stroke-linejoin="round"
      />
    </svg>
    <div v-if="isOpened" class="drop" ref="floating" :style="floatingStyles">
      <div class="option">Выбор колонок</div>
      <div class="option" @click="$emit('group'), (isOpened = false)">
        Группировать по колонке
      </div>
      <div
        class="option"
        @click="$emit('showFilters', !isFiltesOpened), (isOpened = false)"
      >
        {{ isFiltesOpened ? "Скрыть автофильтры" : "Показать автофильтиры" }}
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import {
  useFloating,
  offset,
  flip,
  shift,
  autoPlacement,
} from "@floating-ui/vue";

const emit = defineEmits(["showFilters", "group"]);
const props = defineProps(["isFiltesOpened"]);

const reference = ref(null);
const floating = ref(null);
const { floatingStyles } = useFloating(reference, floating, {
  middleware: [offset(10), flip(), shift()],
  strategy: "fixed",
});

const isOpened = ref(false);
</script>

<style scoped lang="scss">
.drop {
  background-color: rgba(12, 18, 30, 0.8);
  border-radius: 8px;
  z-index: 10;
  position: fixed;
}

.option {
  padding: 12px 12px 12px 8px;
  color: white;
  border-radius: 8px;
  &:hover {
    cursor: pointer;
    transition: all 0.3s;
    background-color: rgba(255, 255, 255, 0.08);
  }
}
</style>
