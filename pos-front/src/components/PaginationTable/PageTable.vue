<template>
  <div class="pagination-table-container">
    <!-- 表格部分 -->
    <div class="table-container">
      <!-- {{ $attrs }} -->
      <el-table ref="tableRef" v-bind="$attrs" :height="tableHeight" class="pagination-table">
        <!-- 表格列 -->
        <slot></slot>

        <!-- 自定义空数据显示 -->
        <template v-if="$slots.empty" #empty>
          <slot name="empty"></slot>
        </template>
        <template v-else #empty>
          <el-empty description="暂无数据"></el-empty>
        </template>

        <!-- 自定义表尾合计行 -->
        <template v-if="$slots.append" #append>
          <slot name="append"></slot>
        </template>
      </el-table>
    </div>

    <!-- 分页部分 -->
    <div class="pagination-container" v-if="props.showPagination">
      <div>
        <slot name="total"></slot>
      </div>
      <!-- <el-pagination
        v-model:current-page="currentPage"
        v-model:page-size="pageSize"
        :page-sizes="props.pageSizes"
        :small="props.small"
        :background="props.background"
        :layout="props.layout"
        :total="props.total"
        :disabled="props.disabled"
        :hide-on-single-page="props.hideOnSinglePage"
        @size-change="handlePaginationSizeChange"
        @current-change="handlePaginationCurrentChange"
        @prev-click="handlePaginationPrevClick"
        @next-click="handlePaginationNextClick"
        size="small"
      /> -->
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted, nextTick } from 'vue';
import type { ElTable, TableProps } from 'element-plus';

interface Props extends Partial<TableProps<any>> {
  showPagination?: boolean;
  currentPage?: number;
  pageSize?: number;
  pageSizes?: number[];
  layout?: string;
  total?: number;
  small?: boolean;
  background?: boolean;
  disabled?: boolean;
  hideOnSinglePage?: boolean;
}

// 定义事件
const $emit = defineEmits([
  // 表格事件
  'selection-change',
  'cell-mouse-enter',
  'cell-mouse-leave',
  'cell-click',
  'cell-dblclick',
  'cell-contextmenu',
  'row-click',
  'row-contextmenu',
  'row-dblclick',
  'header-click',
  'header-contextmenu',
  'sort-change',
  'filter-change',
  'current-change',
  'header-dragend',
  'expand-change',
  // 分页事件
  'update:currentPage',
  'update:pageSize',
  'size-change',
  'pagination-current-change',
  'prev-click',
  'next-click',
]);

// 定义属性和默认值
const props = withDefaults(defineProps<Props>(), {
  // 表格默认值
  data: () => [],
  height: '100%',
  containerHeight: '100%',
  maxHeight: 'auto',
  stripe: true,
  border: true,
  size: 'default',
  fit: true,
  showHeader: true,
  highlightCurrentRow: true,
  emptyText: '暂无数据',
  defaultExpandAll: false,
  tooltipEffect: 'dark',
  showSummary: false,
  selectOnIndeterminate: true,
  tableLayout: 'fixed',

  // 分页默认值
  showPagination: true,
  currentPage: 1,
  pageSize: 10,
  pageSizes: () => [10, 20, 30, 50, 100],
  layout: 'total, sizes, prev, pager, next, jumper',
  total: 0,
  small: false,
  background: true,
  disabled: false,
  hideOnSinglePage: false,
});

// 内部状态
const tableRef = ref<InstanceType<typeof ElTable>>();
const currentPage = ref(props.currentPage);
const pageSize = ref(props.pageSize);

// 计算表格高度，确保表头和分页固定
const tableHeight = computed(() => {
  if (typeof props.height === 'number') {
    return props.height - (props.showPagination ? 44 : 0) + 'px';
  } else if (props.height === 'auto') {
    return 'auto';
  } else {
    return props.height;
  }
});

// 监听 props 变化
// watch(
//   () => props.currentPage,
//   (newValue) => {
//     currentPage.value = newValue;
//   },
// );

// watch(
//   () => props.pageSize,
//   (newValue) => {
//     pageSize.value = newValue;
//   },
// );

// 监听内部状态变化，通知父组件更新
// watch(
//   () => currentPage.value,
//   (newValue) => {
//     $emit('update:currentPage', newValue);
//   },
// );

// watch(
//   () => pageSize.value,
//   (newValue) => {
//     $emit('update:pageSize', newValue);
//   },
// );

// #region 表格事件处理函数
/**
 * 当选择项发生变化时触发的事件处理函数
 * @param selection 已选择的行数据
 */
const handleSelectionChange = (selection: any[]) => {
  $emit('selection-change', selection);
};

/**
 * 当单元格 hover 进入时触发的事件处理函数
 * @param row 行数据
 * @param column 列数据
 * @param cell 单元格DOM元素
 * @param event 事件对象
 */
const handleCellMouseEnter = (row: any, column: any, cell: HTMLElement, event: Event) => {
  $emit('cell-mouse-enter', row, column, cell, event);
};

/**
 * 当单元格 hover 退出时触发的事件处理函数
 * @param row 行数据
 * @param column 列数据
 * @param cell 单元格DOM元素
 * @param event 事件对象
 */
const handleCellMouseLeave = (row: any, column: any, cell: HTMLElement, event: Event) => {
  $emit('cell-mouse-leave', row, column, cell, event);
};

/**
 * 当单元格被点击时触发的事件处理函数
 * @param row 行数据
 * @param column 列数据
 * @param cell 单元格DOM元素
 * @param event 事件对象
 */
const handleCellClick = (row: any, column: any, cell: HTMLElement, event: Event) => {
  $emit('cell-click', row, column, cell, event);
};

/**
 * 当单元格被双击时触发的事件处理函数
 * @param row 行数据
 * @param column 列数据
 * @param cell 单元格DOM元素
 * @param event 事件对象
 */
const handleCellDblclick = (row: any, column: any, cell: HTMLElement, event: Event) => {
  $emit('cell-dblclick', row, column, cell, event);
};

/**
 * 当单元格右键点击时触发的事件处理函数
 * @param row 行数据
 * @param column 列数据
 * @param cell 单元格DOM元素
 * @param event 事件对象
 */
const handleCellContextmenu = (row: any, column: any, cell: HTMLElement, event: Event) => {
  $emit('cell-contextmenu', row, column, cell, event);
};

/**
 * 当某一行被点击时触发的事件处理函数
 * @param row 行数据
 * @param column 列数据
 * @param event 事件对象
 */
const handleRowClick = (row: any, column: any, event: Event) => {
  $emit('row-click', row, column, event);
};

/**
 * 当某一行被右键点击时触发的事件处理函数
 * @param row 行数据
 * @param column 列数据
 * @param event 事件对象
 */
const handleRowContextmenu = (row: any, column: any, event: Event) => {
  $emit('row-contextmenu', row, column, event);
};

/**
 * 当某一行被双击时触发的事件处理函数
 * @param row 行数据
 * @param column 列数据
 * @param event 事件对象
 */
const handleRowDblclick = (row: any, column: any, event: Event) => {
  $emit('row-dblclick', row, column, event);
};

/**
 * 当某一列的表头被点击时触发的事件处理函数
 * @param column 列数据
 * @param event 事件对象
 */
const handleHeaderClick = (column: any, event: Event) => {
  $emit('header-click', column, event);
};

/**
 * 当某一列的表头被右键点击时触发的事件处理函数
 * @param column 列数据
 * @param event 事件对象
 */
const handleHeaderContextmenu = (column: any, event: Event) => {
  $emit('header-contextmenu', column, event);
};

/**
 * 当表格的排序条件发生变化时触发的事件处理函数
 * @param params 排序参数对象，包含column、prop和order属性
 */
const handleSortChange = (params: { column: any; prop: string; order: string }) => {
  $emit('sort-change', params);
};

/**
 * 当表格的筛选条件发生变化时触发的事件处理函数
 * @param filters 筛选参数对象
 */
const handleFilterChange = (filters: any) => {
  $emit('filter-change', filters);
};

/**
 * 当表格的当前行发生变化时触发的事件处理函数
 * @param currentRow 当前行数据
 * @param oldCurrentRow 旧的当前行数据
 */
const handleCurrentChange = (currentRow: any, oldCurrentRow: any) => {
  $emit('current-change', currentRow, oldCurrentRow);
};

/**
 * 当拖动表头改变列的宽度时触发的事件处理函数
 * @param newWidth 新的宽度
 * @param oldWidth 旧的宽度
 * @param column 列数据
 * @param event 事件对象
 */
const handleHeaderDragend = (newWidth: number, oldWidth: number, column: any, event: Event) => {
  $emit('header-dragend', newWidth, oldWidth, column, event);
};

/**
 * 当展开行时触发的事件处理函数
 * @param row 行数据
 * @param expanded 是否展开
 */
const handleExpandChange = (row: any, expanded: boolean) => {
  $emit('expand-change', row, expanded);
};
// #endregion

// #region 分页事件处理函数
/**
 * 当分页大小改变时触发的事件处理函数
 * @param val 新的分页大小
 */
const handlePaginationSizeChange = (val: number) => {
  // pageSize.value = val;
  $emit('size-change', val);
};

/**
 * 当分页当前页改变时触发的事件处理函数
 * @param val 新的当前页
 */
const handlePaginationCurrentChange = (val: number) => {
  // currentPage.value = val;
  $emit('pagination-current-change', val);
};

/**
 * 当点击上一页按钮时触发的事件处理函数
 * @param val 新的当前页
 */
const handlePaginationPrevClick = (val: number) => {
  $emit('prev-click', val);
};

/**
 * 当点击下一页按钮时触发的事件处理函数
 * @param val 新的当前页
 */
const handlePaginationNextClick = (val: number) => {
  $emit('next-click', val);
};
// #endregion

// 对外暴露方法和属性
defineExpose({
  // 表格实例
  tableRef,
  // 分页相关
  currentPage,
  pageSize,
});
</script>

<style lang="scss" scoped>
.pagination-table-container {
  display: flex;
  flex-direction: column;
  height: 100%;
  width: 100%;
  position: relative;

  .table-container {
    flex: 1;
    overflow: hidden;
  }

  .pagination-container {
    padding: 10px;
    display: flex;
    flex-direction: row;
    justify-content: flex-end;
    z-index: 10;

    > div:first-child {
      flex: 1;
    }
  }
}

/* 使用深度选择器修改表格表头样式 */
:deep(.pagination-table .el-table__header-wrapper th) {
  background-color: $base-child-nav-bg;
}
</style>
