<template>
  <div>
    <!-- source -->
    <el-row class="padding-lr">
      <span style="width: 80px;text-align: right;">
        <span>{{ $t('plugin_view_sankey.source') }}</span>/<span>{{ $t('chart.dimension') }}</span>
      </span>
      <draggable
        v-model="source"
        :move="onMove"
        animation="300"
        class="drag-block-style"
        group="drag"
        @add="addSource"
        @update="calcData(true)"
      >
        <transition-group class="draggable-group">
          <sankey-dimension-item
            v-for="(item,index) in source"
            :key="index"
            :bus="bus"
            :chart="chart"
            :dimension-data="dimension"
            :index="0"
            :item="item"
            :param="param"
            :quota-data="quotaData"
            dimension-name="source"
            @onDimensionItemChange="dimensionItemChange"
            @onDimensionItemRemove="locationItemRemove"
            @onNameEdit="showRename"
          />
        </transition-group>
      </draggable>
      <div v-if="!source || source.length === 0" class="drag-placeholder-style">
        <span class="drag-placeholder-style-span">{{ $t('chart.placeholder_field') }}</span>
      </div>
    </el-row>

    <!-- target -->
    <el-row class="padding-lr" style="margin-top: 6px;">
      <span style="width: 80px;text-align: right;">
        <span>{{ $t('plugin_view_sankey.target') }}</span>/<span>{{ $t('chart.dimension') }}</span>
      </span>
      <draggable
        v-model="target"
        :move="onMove"
        animation="300"
        class="drag-block-style"
        group="drag"
        @add="addTarget"
        @update="calcData(true)"
      >
        <transition-group class="draggable-group">
          <sankey-dimension-item
            v-for="(item,index) in target"
            :key="index"
            :bus="bus"
            :chart="chart"
            :dimension-data="dimension"
            :index="1"
            :item="item"
            :param="param"
            :quota-data="quotaData"
            dimension-name="target"
            @onDimensionItemRemove="locationItemRemove"
            @onNameEdit="showRename"
          />
        </transition-group>
      </draggable>
      <div v-if="!target || target.length === 0" class="drag-placeholder-style">
        <span class="drag-placeholder-style-span">{{ $t('chart.placeholder_field') }}</span>
      </div>
    </el-row>

    <!-- 符号大小 -->
    <el-row class="padding-lr" style="margin-top: 6px;">
      <span style="width: 80px;text-align: right;">
        <span>{{ $t('plugin_view_sankey.mark_size') }}</span>/<span>{{ $t('chart.quota') }}</span>
      </span>
      <draggable
        v-model="view.yaxis"
        :move="onMove"
        animation="300"
        class="drag-block-style"
        group="drag"
        @add="addYaxis"
        @update="calcData(true)"
      >
        <transition-group class="draggable-group">
          <sankey-quota-item
            v-for="(item,index) in view.yaxis"
            :key="item.id"
            :chart="chart"
            :dimension-data="dimension"
            :index="index"
            :item="item"
            :param="param"
            :quota-data="quota"
            @editItemCompare="showQuotaEditCompare"
            @editItemFilter="showQuotaEditFilter"
            @onNameEdit="showRename"
            @onQuotaItemChange="quotaItemChange"
            @onQuotaItemRemove="quotaItemRemove"
          />
        </transition-group>
      </draggable>
      <div v-if="!view.yaxis || view.yaxis.length === 0" class="drag-placeholder-style">
        <span class="drag-placeholder-style-span">{{ $t('chart.placeholder_field') }}</span>
      </div>
    </el-row>

    <!-- 结果过滤器 -->
    <el-row class="padding-lr" style="margin-top: 6px;">
      <span class="data-area-label">
          <span>{{ $t('chart.result_filter') }}</span>
          <span
            v-if="!!view.customFilter.logic"
            class="setting"
          >{{ $t('chart.is_set') }}</span>
          <i
            class="el-icon-arrow-down el-icon-delete data-area-clear"
            @click="deleteTreeFilter"
          />
        </span>
        <div
          class="tree-btn"
          :class="!!view.customFilter.logic && 'active'"
          @click="openTreeFilter"
        >
          <svg-icon
            class="svg-background"
            icon-class="icon-filter_outlined"
          />
          <span>{{ $t('chart.filter') }}</span>
        </div>
    </el-row>
    <FilterTree
      ref="filterTree"
     @filter-data="changeFilterData"    @execute-axios="executeAxios"                               
    />

  </div>
</template>

<script>
import SankeyDimensionItem from '../../../components/views/SankeyDimensionItem'
import SankeyQuotaItem from '../../../components/views/SankeyQuotaItem'
import FilterItem from '../../../components/views/FilterItem'
import messages from '@/de-base/lang/messages'
import FilterTree from '@/components/views/filter/FilterTree.vue'

export default {
  provide() {
    return {
      filedList: () => this.filedList
    }
  },
  props: {
    obj: {
      type: Object,
      default: () => {
      }
    },
    bus: {
      type: Object,
      required: false,
      default: null
    }
  },
  components: {
    SankeyDimensionItem,
    SankeyQuotaItem,
    FilterItem,
    FilterTree
  },
  data() {
    return {
      widgets: [],
      places: [],
      moveId: -1,
      showDrill: false,
      options: [{
        value: 'point',
        label: '点'
      },
        {
          value: 'line',
          label: '线'
        }
      ],
      source: [],
      target: []
    }
  },
  computed: {
    filedList() {
      return [...this.dimension, ...this.quota].filter(ele => ele.id !== 'count')
    },
    param() {
      return this.obj.param
    },
    view() {
      return this.obj.view
    },
    chart() {
      return this.obj.chart
    },
    dimensionData() {
      return this.obj.dimensionData
    },
    dimension() {
      return this.obj.dimension
    },
    quota() {
      return this.obj.quota
    },
    quotaData() {
      return this.obj.quotaData
    },
    selectedDimension() {
      return this.obj.selectedDimension
    },
    selectedQuota() {
      return this.obj.selectedQuota
    }
  },
  created() {
    this.source = this.view.xaxis && this.view.xaxis.length && [this.view.xaxis[0]] || []
    this.target = this.view.xaxis && this.view.xaxis.length > 1 && [this.view.xaxis[1]] || []
    this.$emit('on-add-languages', messages)
  },
  watch: {
    source(val) {
      this.view.xaxis = [...this.source, ...this.target]
    },
    target(val) {
      this.view.xaxis = [...this.source, ...this.target]
    },
    'view.xaxis': function(val, oldVal) {
      if (val) {
        if (val[0] && this.source[0]) {
          this.source[0].name = val[0].name
        }
        if (val[1] && this.target[0]) {
          this.target[0].name = val[1].name
        }
      }
    }
  },
  methods: {
    changeFilterData(customFilter) {
      this.view.customFilter =JSON.parse(JSON.stringify(customFilter))
      this.calcData(true)
    },
    openTreeFilter() {
      this.$refs.filterTree.init(JSON.parse(JSON.stringify(this.view.customFilter)))
    },
    deleteTreeFilter() {
      this.changeFilterData({})
    },
    executeAxios(url, type, data, callBack) {
      const param = {
        url: url,
        type: type,
        data: data,
        callBack: callBack
      }
      this.$emit('execute-axios', param)
      if (process.env.NODE_ENV === 'development') {
        execute(param).then(res => {
          if (param.callBack) {
            param.callBack(res)
          }
        }).catch(e => {
          if (param.error) {
            param.error(e)
          }
        })
      }
    },

    dimensionItemChange(item) {
      this.calcData(true)
    },

    onMove(e, originalEvent) {
      this.moveId = e.draggedContext.element.id
      return true
    },

    addSource(e) {
      this.multiAdd(e, this.source)
      this.dragMoveDuplicate(this.source, e)
      this.dragCheckType(this.source, 'd')
      if (this.source.length > 1) {
        this.source = [this.source[0]]
      }
      this.calcData(true)
    },
    addTarget(e) {
      this.multiAdd(e, this.target)
      this.dragMoveDuplicate(this.target, e)
      this.dragCheckType(this.target, 'd')
      if (this.target.length > 1) {
        this.target = [this.target[0]]
      }
      this.calcData(true)
    },
    addYaxis(e) {
      this.multiAdd(e, this.view.yaxis)
      this.dragMoveDuplicate(this.view.yaxis, e)
      this.dragCheckType(this.view.yaxis, 'q')
      if (this.view.yaxis.length > 1) {
        this.view.yaxis = [this.view.yaxis[0]]
      }
      this.calcData(true)
    },
    calcData(cache) {
      this.view.xaxis = [...this.source, ...this.target]

      this.$emit('plugin-call-back', {
        eventName: 'calc-data',
        eventParam: {
          cache
        }
      })
    },

    locationItemRemove(item) {
      if (item.removeType === 'source') {
        this.source.splice(0, 1)
        //this.view.xaxis[0] = undefined;
      } else if (item.removeType === 'target') {
        this.target.splice(0, 1)
        //this.view.xaxis[1] = undefined;
      }
      this.calcData(true)
    },
    showDimensionEditFilter(item) {

      this.$emit('plugin-call-back', {
        eventName: 'show-dimension-edit-filter',
        eventParam: item
      })
    },
    showRename(item) {

      this.$emit('plugin-call-back', {
        eventName: 'show-rename',
        eventParam: item
      })
    },

    quotaItemChange(item) {
      this.calcData(true)
    },
    quotaItemRemove(item) {
      if (item.removeType === 'quota') {
        this.view.yaxis.splice(item.index, 1)
      } else if (item.removeType === 'quotaExt') {
        this.view.yaxisExt.splice(item.index, 1)
      }
      this.calcData(true)
    },
    showQuotaEditFilter(item) {

      this.$emit('plugin-call-back', {
        eventName: 'show-quota-edit-filter',
        eventParam: item
      })
    },
    showQuotaEditCompare(item) {

      this.$emit('plugin-call-back', {
        eventName: 'show-quota-edit-compare',
        eventParam: item
      })
    },
    dragCheckType(list, type) {
      if (list && list.length > 0) {
        for (let i = list.length - 1; i >= 0; i--) {
          if (list[i].groupType !== type) {
            list.splice(i, 1)
          }
        }
      }
    },
    multiAdd(e, itemList) {
      // 只处理原始字段拖拽
      if (!e.item.classList.contains('selected-item')) {
        return
      }
      const groupDie = e.item.classList.contains('group-dimension')
      const sourceList = groupDie ? this.selectedDimension : this.selectedQuota
      if (sourceList.length > 1) {
        const qdList = groupDie ? this.dimensionData : this.quotaData
        const sourceIds = sourceList.map(i => i.id)
        const sortedList = qdList.filter(i => sourceIds.includes(i.id))
        itemList.splice(e.newIndex, 1, ...sortedList)
      }
    },
    dragMoveDuplicate(list, e) {
      let newItems = [list[e.newDraggableIndex]]
      if (e.item.classList.contains('selected-item')) {
        const groupDie = e.item.classList.contains('group-dimension')
        newItems = groupDie ? this.selectedDimension : this.selectedQuota
      }
      const preIds = list
          .filter((_, i) => i < e.newDraggableIndex || i >= e.newDraggableIndex + newItems.length)
          .map(i => i.id)
      // 倒序删除
      for (let i = e.newDraggableIndex + newItems.length - 1; i >= e.newDraggableIndex; i--) {
        if (preIds.includes(list[i].id)) {
          list.splice(i, 1)
        }
      }
    },
    addCustomFilter(e) {
      this.multiAdd(e, this.view.customFilter)
      this.dragMoveDuplicate(this.view.customFilter, e)
      // 记录数等自动生成字段不做为过滤条件
      if (this.view.customFilter && this.view.customFilter.length > 0) {
        for (let i = 0; i < this.view.customFilter.length; i++) {
          if (this.view.customFilter[i].id === 'count') {
            this.view.customFilter.splice(i, 1)
          }
        }
      }
      this.calcData(true)
    },
    filterItemRemove(item) {
      this.view.customFilter.splice(item.index, 1)
      this.calcData(true)
    },
    showEditFilter(item) {
      this.$emit('plugin-call-back', {
        eventName: 'show-edit-filter',
        eventParam: item
      })
    }
  }
}

</script>

<style lang="scss" scoped>
.padding-lr {
  padding: 0 6px;
  .data-area-label {
    text-align: left;
    position: relative;
    width: 100%;
    display: inline-block;
    .setting {
      padding: 0px 4px 0px 4px;
      border-radius: 2px;
      background-color: #1F23291A;
      color: #646A73;
      position: absolute;
      top: 1px;
      right: 23px;
      font-size: 10px;
      font-weight: 500;
      line-height: 14px;
      height: 16px;
      z-index: 1;
    }
  }

  .tree-btn {
      width: 100%;
      background: #fff;
      height: 32px;
      border-radius: 4px;
      border: 1px solid #DCDFE6;
      display: flex;
      color: #CCCCCC;
      align-items: center;
      cursor: pointer;
      justify-content: center;
      font-size: 12px;

      &.active {
        color: #3370FF;
        border-color: #3370FF;
      }
    }
    .data-area-clear {
      position: absolute;
      top: 4px;
      right: 6px;
      color: rgb(135, 141, 159);
      cursor: pointer;
      z-index: 1;
    }
}

.itxst {
  margin: 10px;
  text-align: left;
}

.col {
  width: 40%;
  flex: 1;
  padding: 10px;
  border: solid 1px #eee;
  border-radius: 5px;
  float: left;
}

.col + .col {
  margin-left: 10px;
}

.view-panel {
  display: flex;
  height: calc(100% - 40px);
  background-color: #f7f8fa;
}

.blackTheme .view-panel {
  background-color: var(--MainBG);
}

.drag-list {
  height: calc(100% - 26px);
  overflow: auto;
}

.item-dimension {
  padding: 2px 10px;
  margin: 2px 2px 0 2px;
  border: solid 1px #eee;
  text-align: left;
  color: #606266;
  /*background-color: rgba(35,46,64,.05);*/
  background-color: white;
  display: block;
  word-break: break-all;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

.blackTheme .item-dimension {
  border: solid 1px;
  border-color: var(--TableBorderColor);
  color: var(--TextPrimary);
  background-color: var(--MainBG);
}

.item-dimension + .item-dimension {
  margin-top: 2px;
}

.item-dimension:hover {
  color: #1890ff;
  background: #e8f4ff;
  border-color: #a3d3ff;
  cursor: pointer;
}

.blackTheme .item-dimension:hover {
  color: var(--Main);
  background: var(--ContentBG);
  cursor: pointer;
}

.item-quota {
  padding: 2px 10px;
  margin: 2px 2px 0 2px;
  border: solid 1px #eee;
  text-align: left;
  color: #606266;
  background-color: white;
  display: block;
  word-break: break-all;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}

.blackTheme .item-quota {
  border: solid 1px;
  border-color: var(--TableBorderColor);
  color: var(--TextPrimary);
  background-color: var(--MainBG);
}

.item-quota + .item-quota {
  margin-top: 2px;
}

.item-quota:hover {
  color: #67c23a;
  background: #f0f9eb;
  border-color: #b2d3a3;
  cursor: pointer;
}

.blackTheme .item-quota:hover {
  background: var(--ContentBG);
}

.el-form-item {
  margin-bottom: 0;
}

span {
  font-size: 12px;
}

.tab-header > > > .el-tabs__header {
  border-top: solid 1px #eee;
  border-right: solid 1px #eee;
}

.tab-header > > > .el-tabs__item {
  font-size: 12px;
  padding: 0 60px !important;
}

.blackTheme .tab-header > > > .el-tabs__item {
  background-color: var(--MainBG);
}

.tab-header > > > .el-tabs__nav-scroll {
  padding-left: 0 !important;
}

.tab-header > > > .el-tabs__header {
  margin: 0 !important;
}

.tab-header > > > .el-tabs__content {
  height: 100%;
}

.draggable-group {
  display: block;
  width: 100%;
  height: calc(100% - 6px);
}

.chart-icon {
  width: 20px;
  height: 20px;
}

.el-radio {
  margin: 5px;
}

.el-radio > > > .el-radio__label {
  padding-left: 0;
}

.attr-style {
  height: calc(100vh - 56px - 60px - 40px - 40px);
}

.blackTheme .attr-style {
  color: var(--TextPrimary);
}

.attr-selector {
  width: 100%;
  height: 100%;
  margin: 6px 0;
  padding: 0 4px;
  display: flex;
  align-items: center;
  background-color: white
}

.blackTheme .attr-selector {

  background-color: var(--MainBG)
}

.disabled-none-cursor {
  cursor: not-allowed;
  pointer-events: none;
}

.chart-class {
  height: 100%;
  padding: 10px;
}

.table-class {
  height: calc(100% - 20px);
}

.dialog-css > > > .el-dialog__title {
  font-size: 14px;
}

.dialog-css > > > .el-dialog__header {
  padding: 20px 20px 0;
}

.dialog-css > > > .el-dialog__body {
  padding: 10px 20px 20px;
}

.filter-btn-class {
  padding: 6px;
  border: none;
  width: 100%;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.chart-error-class {
  text-align: center;
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #ece7e7;
}

.blackTheme .chart-error-class {

  background-color: var(--MainBG)
}

.field-height {
  height: calc(50% - 20px);
  border-top: 1px solid #E6E6E6;
}

.blackTheme .field-height {

  border-top: 1px solid;
  border-color: var(--TableBorderColor) !important;
}

.padding-tab {
  padding: 0;
  height: 100%;
}

.tree-select-span {
  > > > div.vue-treeselect__control {
    height: 32px !important;
    font-weight: normal !important;
  }
}

.drag-block-style {
  padding: 2px 0 0 0;
  width: 100%;
  min-height: 32px;
  border-radius: 4px;
  border: 1px solid #DCDFE6;
  overflow-x: hidden;
  display: flex;
  align-items: center;
  background-color: white;
}

.blackTheme .drag-block-style {
  border: 1px solid;
  border-color: var(--TableBorderColor);
  background-color: var(--ContentBG);
}

.drag-placeholder-style {
  position: absolute;
  top: calc(50% - 2px);
  left: 0;
  width: 100%;
  color: #CCCCCC;
}

.blackTheme .drag-placeholder-style {
  color: var(--TextPrimary);
}

.drag-placeholder-style-span {
  padding-left: 16px;
}

.blackTheme .theme-border-class {
  color: var(--TextPrimary) !important;
  background-color: var(--ContentBG);
}

.blackTheme .padding-lr {
  border-color: var(--TableBorderColor) !important;
}

.blackTheme .theme-item-class {
  background-color: var(--MainBG) !important;
  border-color: var(--TableBorderColor) !important;
}

.icon-class {
  color: #6c6c6c;
}

.blackTheme .icon-class {
  color: #cccccc;
}

.result-count {
  width: 80px;
}

.radio-span > > > .el-radio__label {
  margin-left: 4px;
}

</style>
