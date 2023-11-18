<template>
        <tr class="grouping-row">
            <td v-for="i in level" style="width: 50px;"/>
          <td class="grouping-row-item" @click="group.opened = !group.opened" :colspan="columns.length + 2">
            <div>
                <span>{{ group.value }}</span>
            <el-icon><ArrowUp v-if="group.opened"/>
            <ArrowDown v-else/></el-icon>
            </div>
          </td>
        </tr>
        <template v-if="isGroup(group) && group.opened">
            <GroupRow v-for="g in group.items" :groupsCount="groupsCount" :level="level + 1" :group="g" :columns="columns"/>
        </template>
        <template v-else-if="group.opened && !isGroup(group)">
          <tr v-for="(item, idx) in group.items">
            <td v-for="i in groupsCount" style="width: 50px;"/>
            <td
              class="table__cell"
              v-for="column in columns"
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
const props = defineProps(['group', 'columns', 'level', 'groupsCount'])

const isSelectCheckboxField = (column) => column?.field === "selected";

const isGroup = (group) => group.isGroup
</script>

<style lang="scss">
.grouping-row {
    padding: 12px;
}

.grouping-row-item {
    padding: 12px;
    border-bottom: 1px solid #e2e5e8;
    border-left: 1px solid #e2e5e8;

    div {
        display: flex;
        align-items: center;
        gap: 12px;
    }

}
</style>