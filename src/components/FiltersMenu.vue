<template>
  <div>
    <el-icon @click="isOpened = !isOpened" ref="reference"><More /></el-icon>
    <div v-if="isOpened" class="drop" ref="floating" :style="floatingStyles">
      <div class="option">Выбор колонок</div>
      <div class="option" @click="$emit('group'), (isOpened = false)">Группировать по колонке</div>
      <div class="option" @click="$emit('showFilters', !isFiltesOpened), (isOpened = false)">
        {{ isFiltesOpened ? 'Скрыть автофильтры' : 'Показать автофильтиры' }}
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import { useFloating, offset, flip, shift } from "@floating-ui/vue";

const emit = defineEmits(["showFilters", "group"]);
const props = defineProps(['isFiltesOpened'])

const reference = ref(null);
const floating = ref(null);
const { floatingStyles } = useFloating(reference, floating, {
  middleware: [offset(10), flip(), shift()],
});

const isOpened = ref(false);
</script>

<style scoped lang="scss">
.drop {
  background-color: rgba(12, 18, 30, 0.8);
  border-radius: 8px;
  z-index: 10;
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
