<template>
  <div class="lhe-table">
    <div class="lhe-table-container">
      <div class="lhe-table-content">
        <table style="table-layout: auto">
          <thead class="lhe-table-thead">
            <tr
              v-for="(rows, index) in finalColumns"
              :key="'rows:' + index"
              class="lhe-table-cell"
            >
              <th
                v-for="(cell, celidx) in rows"
                :key="'cell:' + celidx"
                :rowspan="cell?.rowspan"
                :colspan="cell?.colspan"
                :class="`${cell?.align}`"
              >
                <span>{{ cell?.title }}</span>
              </th>
            </tr>
          </thead>
          <tbody class="lhe-table-tbody">
            <template v-if="records && records.length > 0">
              <tr
                v-for="(record, index) in records"
                :key="'record:' + index"
                class="lhe-table-row"
              >
                <td
                  v-for="(column, colidx) in flattenColumns"
                  :key="colidx + ':' + index"
                  class="lhe-table-cell"
                >
                  <slot
                    v-if="hasSlots(column)"
                    :name="column?.dataIndex"
                    v-bind="slotProps(column, record, index)"
                  />
                  <span v-else>{{ renderRecord(record, column) }}</span>
                </td>
              </tr>
            </template>
            <slot v-else name="empty"> No data available </slot>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script lang="js">
export default {
  name: 'LHETable',
  props: {
    columns: {
      type: Array,
      default: () => []
    },
    dataSource: {
      type: Array,
      default: () => []
    },
  },
  data() {
    return {
      records: this.dataSource
    }
  },
  computed: {
    finalColumns() {
      return this.generateColumns(this.columns)
    },
    flattenColumns() {
      return this.flattenChildrenColumn(this.columns)
    },
  },
  watch: {
    dataSource(value) {
      this.records = value
    },
  },
  methods: {
    slotProps(column, record, index) {
      return { column, record: this.renderRecord(record, column), index }
    },
    hasSlots(column) {
      return this.$scopedSlots[column?.dataIndex] || false
    },
    generateColumns(columns) {
      const newColumns = [];
      this.columnsGenerate(columns, newColumns, 0);
      return newColumns;
    },
    columnsGenerate(columns, newColumns, depth) {
      if (columns.length === 0) return;
      if (newColumns.length <= depth) {
        newColumns.push([]);
      }
      columns.forEach((item) => {
        const cellObj = Object.keys(item).reduce((obj, key) => {
          if (key !== 'children') {
            obj[key] = item[key];
          }
          return obj;
        }, {});
        if (item?.children) {
          const rowspan = this.getMaxChildrenDepth(item?.children)
          const colspan = this.getTotalColumns(item?.children);
          if (rowspan > 1 && rowspan > colspan) {
            cellObj.rowspan = rowspan
          }
          if (colspan > 1 && colspan > rowspan) {
            cellObj.colspan = colspan
          }
          this.columnsGenerate(item?.children, newColumns, depth + 1);
        } else {
          const rowspan = this.getMaxDepth(columns);
          if (rowspan > 1) {
            cellObj.rowspan = rowspan
          }
        }
        newColumns[depth].push(cellObj);
      });
    },
    getMaxDepth(columns) {
      if (columns.length === 0) return 1;
      let maxDepth = 0;
      columns.forEach((item) => {
        if (item?.children) {
          const childDepth = this.getMaxDepth(item?.children);
          maxDepth = Math.max(maxDepth, childDepth);
        }
      });
      return maxDepth + 1;
    },
    getMaxChildrenDepth(columns) {
      if (!columns || columns.length === 0) {
        return 0;
      }
      let maxDepth = 0;
      columns.forEach((item) => {
        if (item?.children) {
          const childDepth = this.getMaxChildrenDepth(item?.children);
          maxDepth = Math.max(maxDepth, childDepth);
        }
      });
      return maxDepth + 1;
    },
    getTotalColumns(columns) {
      if (!columns || columns.length === 0) {
        return 1;
      }
      let totalColumns = 0;
      columns.forEach((item) => {
        totalColumns += item?.children ? this.getTotalColumns(item?.children) : 1;
      });
      return totalColumns;
    },
    flattenChildrenColumn(columns, parentKeys = []) {
      const newColumns = [];
      for (const column of columns) {
        const { title, dataIndex, children } = column;

        if (children) {
          const newParentKeys = [...parentKeys, dataIndex];
          newColumns.push(...this.flattenChildrenColumn(children, newParentKeys));
        } else {
          newColumns.push({
            title,
            dataIndex,
            parentKeys,
          });
        }
      }
      return newColumns;
    },
    getValue(data, keys) {
      let result = data;
      for (const key of keys) {
        result = result[key];
      }
      return result;
    },
    renderRecord(record, column) {
      const parentKeys = column?.parentKeys.slice();
      parentKeys.push(column?.dataIndex);
      return this.getValue(record, parentKeys)
    }
  }
}
</script>

<style lang="sass" scoped>
@import "table"
</style>
