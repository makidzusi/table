<template>
  <div>
    <DataTable v-model:columns="columns" v-model:data="data" />
    <br>
    <br>
    <DataTable v-model:columns="normalWidthCols" v-model:data="data" />
  </div>
</template>

<script setup>
import { ref } from "vue";
import DataTable from "./components/DataTable.vue";

const columns = ref([
  {
    field: "selected",
  },
  {
    field: "user",
    title: "Пользователь",
    minWidth: '1000px'
  },
  {
    field: "edited",
    title: "Изменено",
    minWidth: '1000px'
  },
  {
    field: "date",
    title: "Дата", width: '1000px'
  },
  {
    field: 'fio',
    title: 'Фамилия', width: '1000px'
  }
]);

const normalWidthCols = ref([
  {
    field: "selected",
  },
  {
    field: "user",
    title: "Пользователь",

  },
  {
    field: "edited",
    title: "Изменено",

  },
  {
    field: "date",
    title: "Дата",
  },
  {
    field: 'fio',
    title: 'Фамилия', 
  }
])

const data = ref([
  { user: "User-0", edited: "18.11.2023", date: "Сегодня", fio: 'Пупкин' },
  { user: "User-0", edited: "18.11.2023", date: "Завтра",fio: 'Пупкин'  },
  { user: "User-0", edited: "16.11.2023", date: "Сегодня",fio: 'Иванов'  },
  { user: "User-0", edited: "16.11.2023", date: "Сегодня",fio: 'Пупкин'  },
  { user: "User-0", edited: "16.11.2023", date: "Завтра",fio: 'Иванов'  },
  { user: "User-1", edited: "18.11.2023", date: "Сегодня",fio: 'Пупкин'  },
  { user: "User-1", edited: "18.11.2023", date: "Сегодня",fio: 'Пупкин'  },
  { user: "User-1", edited: "18.11.2023", date: "Завтра",fio: 'Иванов'  },
  { user: "User-1", edited: "16.11.2023", date: "Сегодня",fio: 'Иванов'  },
]);
</script>

<style>
* {
  box-sizing: border-box;
}
</style>
