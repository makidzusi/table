<template>
  <div class="table__wrapper">
  <table class="table">
    <colgroup>
      <col v-for="group in groups" width="50px" />
      <col
        v-for="column in columns"
        :style="{
          minWidth: column?.minWidth ? column.minWidth : '50px',
          width: isSelectCheckboxField(column) ? '50px' : column?.width ? column.width : null,
        }"
      />
    </colgroup>
    <thead class="table__head">
      <tr ref="thead">
        <td v-for="group in groups" style="width: 50px; background-color: #f6f7f9; border: 1px solid #e2e5e8;"></td>
        <td
          :data-field="column?.field"
          class="table__cell"
          :style="{
            textAlign: isSelectCheckboxField(column) ? 'center' : null,
          }"
          v-for="column in columns"
          :key="column?.field"
        >
          <div v-if="isSelectCheckboxField(column)">
            <el-checkbox v-model="isAllRowsSelected" />
          </div>
          <div v-else class="table__head-cell">
            <div class="table__head-cell-top">
              <span>{{ column?.title }}</span>
              <FiltersMenu
                :isFiltesOpened="isShowFilters"
                @showFilters="handleToggleFilters"
                @group="handleGroup(column)"
              />
            </div>
            <div class="table__head-cell-bottom" v-if="isShowFilters">
              <el-input
                v-model="filtersModel[column.field]"
                placeholder="Please input"
              />
              <el-icon><Operation /></el-icon>
            </div>
          </div>
          <span v-if="column.field !== 'selected'"  @mousedown.stop="startResize(column)" class="table__resizer"></span>
        </td>
      </tr>
    </thead>
    <tbody>
      <template v-if="groups.length">
        <GroupRow
          v-for="group in groupedData"
          :groups-count="groups.length"
          :group="group"
          :level="1"
          @ungroup="handleUngroup"
          :columns="columns"
        />
      </template>
      <template v-else>
        <tr v-for="(row, idx) in data" :key="`row-${idx}`">
          <td
            class="table__cell"
            v-for="column in columns"
            :style="{
                textAlign: isSelectCheckboxField(column) ? 'center' : null,
            }"
            :key="`row-${idx}-col${column.field}`"
          >
            <template v-if="isSelectCheckboxField(column)">
              <el-checkbox v-model="row[column.field]" />
            </template>
            <template v-else>
              {{ row[column.field] }}
            </template>
          </td>
        </tr>
      </template>
    </tbody>
  </table>
</div>
</template>

<script setup>
import { computed, ref } from "@vue/reactivity";
import FiltersMenu from "./FiltersMenu.vue";
import Sortable from "sortablejs";
import { nextTick, onMounted } from "vue";
import GroupRow from "./GroupRow.vue";

const props = defineProps(["columns", "data"]);
const emit = defineEmits(["update:data", "update:columns"]);
const thead = ref(null);

const groupedData = ref([]);

const isSelectCheckboxField = (column) => column?.field === "selected";
const isShowFilters = ref(false);
const filtersModel = ref({});
const groups = ref([]);
const isResize = ref(false)
const lastX = ref(0)

const startResize = (column) => {
  isResize.value = true;
  const colElement = getColumnElement(column.field)
  let w = colElement.clientWidth

  const onMove = (e) => {
    if(e.clientX - lastX.value > 0) {
      w += 7;
    }  else {
      w -= 7
    }
    console.log(w)
    lastX.value = e.clientX
    colElement.style.width = `${w}px`
  }
  document.addEventListener('mousemove', onMove)
  document.addEventListener('mouseup', () => {
    document.removeEventListener('mousemove', onMove)
  })
}

const getColumnElement = (field) => {
  const childs = thead.value.children;
  for (let i = 0; i < childs.length; i++) {
    const child = childs[i];
    if (child.className === "") continue;
    const fName = child.dataset["field"];
    if(fName === field) {
      return child
    }
  }
}

const handleToggleFilters = (e) => {
  isShowFilters.value = e;
};
onMounted(async () => {
  await nextTick();
  Sortable.create(thead.value, {
    animation: 150,
    onEnd: reorderEvent,
    onChange: reorderEvent,
  });
});

const reorderEvent = (e) => {
  const childs = thead.value.children;
  const reorderedColumns = [];
  for (let i = 0; i < childs.length; i++) {
    const child = childs[i];
    if (child.className === "") continue;
    const fName = child.dataset["field"];
    reorderedColumns.push(props.columns.find((el) => el.field === fName));
  }
  emit("update:columns", reorderedColumns);
};

const handleGroup = (e) => {
  const el = groups.value.find(_el => _el === e.field)
  if(el) {
    groups.value = groups.value.filter(el => el !== e.field)
  } else {
    groups.value.push(e.field);
  }
  groupData()
};

const groupData = () => {
  const firstGroup = groups.value[0];
  groupedData.value = mapData(props.data, firstGroup);
  const slicedGroups = groups.value.slice(1);

  slicedGroups.forEach((group, idx) => {
    groupedData.value.forEach((el) => {
  
      const nestedValues = getNestValues(el);

      nestedValues.forEach((el) => {
        el.items = mapData(el.items, group);
        el.isGroup = true;
      })
    });
  });
}

const handleUngroup = (e) => {
  groups.value = groups.value.filter(el => el !== e)
  groupData()
}

const getNestValues = (group) => {
  if (group.isGroup === false) {

    return [group];
  }
  const nestedValues = []
  group.items.forEach((el) => {
    nestedValues.push(...getNestValues(el))
  })
  return nestedValues;
};

const mapData = (items, field) => {
  const mappedData = [];
  if(!items) return []
  items.map((el) => {
    const value = el[field];
    const container = mappedData.find((el) => el.value === value);
    if (container) {
      container.items.push(el);
    } else {
      mappedData.push({
        field: field,
        value: value,
        opened: false,
        isGroup: false,
        items: [el],
      });
    }
  });
  console.log(mappedData);
  return mappedData;
};

const isAllRowsSelected = computed({
  get() {
    for (let i = 0; i < props.data.length; i++) {
      const row = props.data[i];
      if (row?.selected !== true) {
        return false;
      }
    }
    return true;
  },
  set() {
    if (!isAllRowsSelected.value) {
      emit(
        "update:data",
        props.data.map((el) => ({
          ...el,
          selected: true,
        }))
      );
      return;
    }
    emit(
      "update:data",
      props.data.map((el) => ({
        ...el,
        selected: false,
      }))
    );
  },
});
</script>

<style lang="scss">
$light-gray: #f6f7f9;
$font-size-regular: 16;
// Colors
$black: #0C121E;
$black-light: #161B27;
$gray-dark: #566376;
$black-80: #0C121ECC;
$gray: #959CB2;
$gray-medium: #C4CAD4;
$gray-light: #F6F7F9;
$white: #FFFFFF;
$gray-blue: #B4BBC6;
$blue-primary-button: #DFE8FD;
$gray-very-light: #BEC3CC;

$blue: #1755E7;
$blue-medium: #2C61F2;
$blue-dark: #0F42CF;
$red: #F54B31;
$green: #54CC29;
$orange: #FFA033;
$purple: #7070FF;

// Animation
$transition-time: 300ms;

.table {
  width: 100%;
  border: 1px solid #e2e5e8;
  border-radius: 8px;
  border-spacing: 0px;
  // overflow: hidden;
  color: #0C121E;
  font-weight: 400;
  font-size: 14px;
  font-family: 'Inter', sans-serif;
  border-collapse: collapse;

  &__wrapper {
    border: 1px solid #e2e5e8;
    border-radius: 8px;
    overflow: auto;
    &::-webkit-scrollbar {
    width: 4px;
    height: 4px;
  }

  /* Track */
  &::-webkit-scrollbar-track {
    background: $white;
    margin: 4px 0;
  }

  /* Handle */
  &::-webkit-scrollbar-thumb {
    background: $gray-medium;
    border-radius: 8px;
    margin-right: 4px;
  }
  }

  &__head {
    background-color: $light-gray;
    position: sticky;
    top: 0;
    z-index: 10;
  }

  &__head-cell {
    display: flex;
    flex-direction: column;
    justify-content: space-between;

    &-top {
      display: flex;
      align-items: center;
      justify-content: space-between;
    }

    &-bottom {
      display: flex;
      margin-top: 10px;
      align-items: center;
      gap: 10px;
    }
  }

  &__cell {
    padding: 9px;
    position: relative;
    border-left: 1px solid #e2e5e8;
    // &:not(:last-child) {
    //   border-right: 1px solid #e2e5e8;
    // }
    border-bottom: 1px solid #e2e5e8;
  }

  &__resizer {
    cursor: col-resize;
    position: absolute;
    right: -5px;
    height: 100%;
    top: 0;
    width: 15px;
    z-index: 5;
  }
}
</style>
