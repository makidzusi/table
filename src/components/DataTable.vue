<template>
  <table class="table">
    <colgroup>
      <col v-for="group in groups" width="50px" />
      <col
        v-for="column in columns"
        :style="{
          width: isSelectCheckboxField(column) ? '50px' : column?.width ? column.width : null,
        }"
      />
    </colgroup>
    <thead class="table__head">
      <tr ref="thead">
        <td v-for="group in groups" style="width: 50px"></td>
        <td
          :data-field="column?.field"
          class="table__cell"
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
          <span  @mousedown.stop="startResize(column)" class="table__resizer"></span>
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
      w += 3;
    }  else {
      w -= 3
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
.table {
  width: 100%;
  border: 1px solid #e2e5e8;
  border-radius: 8px;
  border-spacing: 0px;
  overflow: hidden;

  &__head {
    background-color: $light-gray;
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
    padding: 16px;
    position: relative;
    &:not(:last-child) {
      border-right: 1px solid #e2e5e8;
    }
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
