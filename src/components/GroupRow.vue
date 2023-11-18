<template>
  <tr class="grouping-row">
    <td v-for="i in level" style="width: 50px; background-color: #f6f7f9; border-left: 1px solid #e2e5e8; border-right: 1px solid #e2e5e8" />
    <td
      class="grouping-row-item"
      @click="group.opened = !group.opened"
      :colspan="columns.length + 2"
    >
      <div class="grouping-row-item-cell">
        <div>
          <div class="arrow">
            <svg
              v-if="!group.opened"
              width="7"
              height="12"
              viewBox="0 0 7 12"
              fill="none"
              xmlns="http://www.w3.org/2000/svg"
            >
              <path
                d="M1 11L6 6L1 1"
                stroke="#C4CAD4"
                stroke-width="1.4"
                stroke-linecap="round"
                stroke-linejoin="round"
              />
            </svg>
            <svg
              v-else
              width="12"
              height="7"
              viewBox="0 0 12 7"
              fill="none"
              xmlns="http://www.w3.org/2000/svg"
            >
              <path
                d="M1 1L6 6L11 1"
                stroke="#1755E7"
                stroke-width="1.4"
                stroke-linecap="round"
                stroke-linejoin="round"
              />
            </svg>
          </div>

          <span>{{ group.value }}</span>
        </div>
        <div class="ung">
          <span>{{ group.items.length }}</span>
          <Menu @click.stop>
            <div class="option" @click="handleUngroup">Разгрупировать</div>
          </Menu>
        </div>
      </div>
    </td>
  </tr>
  <template v-if="isGroup(group) && group.opened">
    <GroupRow
      @ungroup="emit('ungroup', $event)"
      v-for="g in group.items"
      :groupsCount="groupsCount"
      :level="level + 1"
      :group="g"
      :columns="columns"
    />
  </template>
  <template v-else-if="group.opened && !isGroup(group)">
    <tr v-for="(item, idx) in group.items">
      <td
        v-for="i in groupsCount"
        style="width: 50px; background-color: #f6f7f9; border-left: 1px solid #e2e5e8; border-right: 1px solid #e2e5e8;"
      />
      <td
        class="table__cell cell"
        v-for="column in columns"
        :style="{
                textAlign: isSelectCheckboxField(column) ? 'center' : null
            }"
        :key="`row-${idx}-col${column.field}`"
      >
        <template v-if="isSelectCheckboxField(column)">
          <el-checkbox v-model="item[column.field]" />
        </template>
        <template v-else>
          {{ item[column.field] }}
        </template>
      </td>
    </tr>
  </template>
</template>

<script setup>
import Menu from "./Menu.vue";

const props = defineProps(["group", "columns", "level", "groupsCount"]);
const emit = defineEmits(["ungroup"]);

const isSelectCheckboxField = (column) => column?.field === "selected";

const handleUngroup = () => {
  emit("ungroup", props.group.field);
};

const isGroup = (group) => group.isGroup;
</script>

<style lang="scss">
$light-gray: #f6f7f9;
.grouping-row {
  padding: 12px;
  background-color: $light-gray;
}

.grouping-row-item {
  border-bottom: 1px solid #e2e5e8;
  border-left: 1px solid #e2e5e8;
  border-top: 1px solid #e2e5e8;

  &-cell {
    display: flex;
    justify-content: space-between;
    align-items: center;
    // border-top: 1px solid #e2e5e8;
  }

  div {
    display: flex;
    align-items: center;
    gap: 12px;
  }
}

.arrow {
  width: 50px;
  height: 50px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.ung {
  margin-right: 12px;
  display: flex;
  align-items: center;
  gap: 20px;
}
</style>
