<template>
  <div class="gb-ant-select-multiple-cascader"
       :class="{ 'multiple-cascader-outside': !isClickOutSide }"
       v-clickoutside="handleDropDownOutSide">
    <a-input-group :compact="compact"
                   class="input-group-class">
      <a-select v-if="compact"
                :value="modeValue"
                :default-value="0"
                class="first-select"
                @change="changeModeValue">
        <a-select-option v-for="item in modeArray"
                         :value="item.value"
                         :key="item.value">
          {{item.label}}
        </a-select-option>
      </a-select>
      <a-select v-bind="$attrs"
                mode="multiple"
                class="second-select"
                :maxTagTextLength="maxTagTextLength"
                :maxTagCount="maxTagCount"
                :placeholder="placeholder"
                :value="selectedValueArr"
                :notFoundContent="null"
                :open="isOpen"
                :allowClear="allowClear"
                optionLabelProp="text"
                :dropdownMatchSelectWidth="false"
                :getPopupContainer="tirggerNode => tirggerNode.parentNode"
                @focus="handleFocus"
                @deselect="handleDeselect">
        <a-icon type="close-circle"
                slot="clearIcon"
                theme="filled"
                @click="handleClearAll" />
        <template slot="dropdownRender"
                  slot-scope="menu">
          <v-nodes :vnodes="menu" />
          <div class="tab-warpper"
               v-if="hasTab">
            <a-tabs :activeKey="tabsValue"
                    size="small"
                    :animated="false"
                    :tabBarStyle="{
                    }"
                    @change="changeTab">
              <a-tab-pane key="0"
                          tab="包含">
              </a-tab-pane>
              <a-tab-pane key="1"
                          tab="不包含"
                          force-render>
              </a-tab-pane>
            </a-tabs>
          </div>
          <div :class="dropdownClassName">
            <div class="cascader-not-content"
                 v-if="treeDataList.length <= 0">
              <a-empty :image="simpleImage"
                       :description="noDataText"></a-empty>
            </div>
            <div class="cascader-content-wrap"
                 v-else>
              <div class="cascader-content-container">
                <ul class="cascader-content-list"
                    v-for="(itemList, levelIndex) in treeDataList"
                    :key="levelIndex">
                  <li class="cascader-content-item"
                      v-for="(subItem, subIndex) in itemList"
                      :key="subItem[selectOptionsConfig.key]"
                      :title="subItem[selectOptionsConfig.text]"
                      :class="{ 'cascader-content-item-active': subItem.$active }"
                      @click.stop="
                      handleCascaderItemClick(subItem, subIndex, levelIndex)
                    ">
                    <a-checkbox :disabled="isDisabled(subItem)"
                                :indeterminate="!!subItem.$indeterminate"
                                :checked="!!subItem.$checked"
                                @click.self="
                        e => handleCheckClick(e, subItem, subIndex, levelIndex)
                      " />
                    <span class="checkbox-text"
                          :class="{item_disabled: isDisabled(subItem)}">
                      {{ subItem[selectOptionsConfig.text] }}</span>
                    <a-icon type="right"
                            style="font-size: 10px"
                            v-if="
                        subItem[selectOptionsConfig.children] &&
                          subItem[selectOptionsConfig.children].length > 0
                      " />
                  </li>
                </ul>
              </div>
            </div>
          </div>
        </template>
        <a-select-option v-for="item in treeToList"
                         :key="item[selectOptionsConfig.key]"
                         :value="item[selectOptionsConfig.value]"
                         :text="item[selectOptionsConfig.text]">
          {{ item[selectOptionsConfig.text] }}
        </a-select-option>
      </a-select>
    </a-input-group>

  </div>
</template>
<script>
import { Empty } from 'ant-design-vue'
import { cloneDeep } from 'lodash'
const clickoutside = {
  // 初始化指令
  bind(el, binding) {
    function documentHandler(e) {
      if (el.contains(e.target)) {
        return false
      }
      if (binding.expression) {
        binding.value(e)
      }
    }
    el.__vueClickOutside__ = documentHandler
    document.addEventListener('click', documentHandler)
  },
  update() {},
  unbind(el) {
    document.removeEventListener('click', el.__vueClickOutside__)
    delete el.__vueClickOutside__
  },
}
export default {
  name: 'MultipleCascder',
  components: {
    VNodes: {
      functional: true,
      render: (h, ctx) => {
        return ctx.props.vnodes
      },
    },
  },
  directives: { clickoutside },
  props: {
    allowClear: {
      type: Boolean,
      default: true,
    },
    allText: {
      type: String,
      default: '全部',
    },
    noDataText: {
      type: String,
      default: '暂无数据',
    },
    treeData: {
      type: Array,
      default: () => {
        return []
      },
    },
    dropdownClassName: {
      type: String,
    },
    selectOptionsConfig: {
      type: Object,
      default: () => {
        return {
          key: 'id',
          value: 'value',
          text: 'title',
          children: 'children',
          disabled: 'disabled',
        }
      },
    },
    maxTagCount: {
      type: Number,
    },
    maxTagTextLength: {
      type: Number,
    },
    placeholder: {
      type: String,
      default: '请搜索',
    },
    checkAll: {
      type: Boolean,
      default: false,
    },
    value: {
      type: Array,
    },
    modeValue: {
      type: Number,
      default: 0,
    },
    tabsValue: {
      type: String,
      default: '0',
    },
    compact: {
      type: Boolean,
      default: false,
    },
    modeArray: {
      type: Array,
      default: () => {
        return [
          {
            label: '包含',
            value: 0,
          },
          {
            label: '不包含',
            value: 1,
          },
        ]
      },
    },
    hasTab: {
      type: Boolean,
      default: false,
    },
  },
  beforeCreate() {
    this.simpleImage = Empty.PRESENTED_IMAGE_SIMPLE
  },
  destroyed() {
    this.timer && clearInterval(this.timer)
  },
  data() {
    return {
      isOpen: false,
      isCheckAll: this.checkAll,
      isIndeterminateAll: false,
      isClickOutSide: true,
      selectedValueArr: this.value || undefined,
      treeDataList: cloneDeep(this.treeData),
      treeToList: [],
    }
  },
  watch: {
    checkAll: {
      handler(newVal) {
        if (newVal) {
          this.timer = setInterval(() => {
            if (this.treeDataList.length > 0) {
              clearInterval(this.timer)
              this.handleCascaderAllItemClick(true)
            }
          }, 30)
        }
      },
      immediate: true,
    },
    value(newVal) {
      this.selectedValueArr = newVal // this.treeDataList = this.treeData;
      this.initDefaultKeys(newVal, this.treeDataList)
      return newVal
    },
    treeData(newVal) {
      this.treeListToArrList(newVal[0])
      this.initDefaultKeys(this.selectedValueArr, newVal)
      this.treeDataList = newVal
      return newVal
    }, //打开组件
    isOpen(newVal) {
      //打开为true 并且 没有选择
      if (newVal && !this.value.length) {
        //执行清空操作
        this.handleClearAll()
        this.isOpen = true
      }
    },
  },
  methods: {
    handleClearAll() {
      this.treeDataList = this.treeData
      this.isOpen = false
      this.isCheckAll = false
      this.isClickOutSide = true
      this.isIndeterminateAll = false
      this.updatePostData()
    },
    initDefaultKeys(selectedArr, treeDataList) {
      if (
        selectedArr &&
        selectedArr.length > 0 &&
        treeDataList &&
        treeDataList.length > 0
      ) {
        const treeDataListClone = cloneDeep(treeDataList)
        const treeToListClone = cloneDeep(this.treeToList)
        selectedArr.forEach((item) => {
          const curretnItemList = []
          this.findKeysItem(treeDataListClone[0], item, curretnItemList)
          let currentItem = {}
          if (curretnItemList.length > 0) {
            currentItem = curretnItemList[0]
          }
          // 有2种情况 一种是删除 一种是新增
          //删除 数据减少
          //新增 数据增加
          // 一种判断 selectedArr 选择 其他的全部清空
          if (
            !treeToListClone.find(
              (listItem) =>
                listItem[this.selectOptionsConfig.value] ===
                currentItem[this.selectOptionsConfig.value]
            )
          ) {
            treeToListClone.push(currentItem)
          }
          currentItem.$checked = true
          if (
            currentItem[this.selectOptionsConfig.children] &&
            currentItem[this.selectOptionsConfig.children].length > 0
          ) {
            this.setAllDataTreeChecked(
              currentItem[this.selectOptionsConfig.children],
              true
            )
          }
        })
        this.setCheckBoxIndeteriminate(treeDataListClone[0])
        this.$nextTick(() => {
          this.treeToList = treeToListClone
          this.treeDataList = treeDataListClone
        })
      }
    },
    handleCascaderAllItemClick(checkAll) {
      const treeDataListClone = cloneDeep(this.treeDataList)
      if (checkAll) {
        this.setAllDataTreeChecked(treeDataListClone[0], checkAll)
      } else {
        this.setAllDataTreeChecked(treeDataListClone[0], !this.isCheckAll)
      }
      this.clearAllDataActive(treeDataListClone[0])
      this.setCheckBoxIndeteriminate(treeDataListClone[0])
      this.treeDataList = treeDataListClone
      this.updatePostData()
    },
    handleDeselect(value) {
      const treeDataListClone = cloneDeep(this.treeDataList)
      const currentItemList = []
      this.findKeysItem(treeDataListClone[0], value, currentItemList)
      let currentItem = {}
      if (currentItemList.length > 0) {
        currentItem = currentItemList[0]
      }
      currentItem.$checked = false
      currentItem.$active = false
      if (
        currentItem[this.selectOptionsConfig.children] &&
        currentItem[this.selectOptionsConfig.children].length > 0
      ) {
        this.setAllDataTreeChecked(
          currentItem[this.selectOptionsConfig.children],
          false
        )
      }
      this.setCheckBoxIndeteriminate(treeDataListClone[0])
      this.treeDataList = treeDataListClone
      this.updatePostData()
    },
    handleCheckClick(e, item, index, levelIndex) {
      console.log('handleCheckClick: ');
      const treeDataListClone = cloneDeep(this.treeDataList)
      const childrenTag = this.selectOptionsConfig.children
      const valueTag = this.selectOptionsConfig.value
      const currentItemList = []
      this.findKeysItem(treeDataListClone[0], item[valueTag], currentItemList)
      let currentItem = {}
      if (currentItemList.length > 0) {
        currentItem = currentItemList[0]
      }
      e.stopPropagation()
      e.cancelBubble = true
      const checked = e.target.checked
      if (!currentItem.$active && levelIndex !== treeDataListClone.length - 1) {
        // 清空所有选中状态
        this.clearAllDataActive(treeDataListClone[0])
      }
      if (currentItem[childrenTag] && currentItem[childrenTag].length > 0) {
        // 可展开
        if (!currentItem.$active) {
          const nextIndex = levelIndex + 1
          if (treeDataListClone[nextIndex]) {
            treeDataListClone.splice(nextIndex)
          }
        }
      } else {
        // 不可展开
        const nextIndex = levelIndex + 1
        if (treeDataListClone[nextIndex]) {
          treeDataListClone.splice(nextIndex)
        }
      }
      if (currentItem[childrenTag] && currentItem[childrenTag].length > 0) {
        this.setAllDataTreeChecked(currentItem[childrenTag], checked)
      } else {
        currentItem.$checked = checked
      }
      this.setCheckBoxIndeteriminate(treeDataListClone[0])
      this.treeDataList = treeDataListClone
      this.updatePostData()
    }, // 单个item点击事件
    handleCascaderItemClick(item, index, levelIndex) {
      console.log('handleCascaderItemClick: ');
      //TODO 判断 标签是否禁用
      if (item[this.selectOptionsConfig.disabled]) {
        return
      }
      const treeDataListClone = cloneDeep(this.treeDataList)
      const childrenTag = this.selectOptionsConfig.children
      const valueTag = this.selectOptionsConfig.value
      const currentItemList = []
      this.findKeysItem(treeDataListClone[0], item[valueTag], currentItemList)
      let currentItem = {}
      if (currentItemList.length > 0) {
        currentItem = currentItemList[0]
      }
      const parentItemsList = []
      this.findAllParentItems(
        currentItem[valueTag],
        treeDataListClone[0],
        treeDataListClone[0],
        parentItemsList
      )
      parentItemsList.push(currentItem)
      this.clearAllDataActive(treeDataListClone[0])
      parentItemsList.forEach((activeItem) => {
        if (activeItem[childrenTag] && activeItem[childrenTag].length > 0) {
          activeItem.$active = true
        }
      })
      if (currentItem[childrenTag] && currentItem[childrenTag].length > 0) {
        const nextIndex = levelIndex + 1
        if (treeDataListClone[nextIndex]) {
          treeDataListClone.splice(nextIndex)
          treeDataListClone[nextIndex] = currentItem[childrenTag]
        } else {
          treeDataListClone.push(currentItem[childrenTag])
        }
        this.addHideSelectedList(currentItem[childrenTag])
      } else {
        const nextIndex = levelIndex + 1
        if (treeDataListClone[nextIndex]) {
          treeDataListClone.splice(nextIndex)
        }
        currentItem.$checked = !currentItem.$checked
      }
      this.setCheckBoxIndeteriminate(treeDataListClone[0])
      this.treeDataList = treeDataListClone
      if (currentItem[childrenTag] && currentItem[childrenTag].length > 0) {
        this.updatePostData(false)
      } else {
        this.updatePostData()
      }
    },
    handleFocus() {
      this.isOpen = true
      this.isClickOutSide = false
    },
    updatePostData(type = true) {
      const that = this
      const childrenTag = that.selectOptionsConfig.children
      const valueTag = that.selectOptionsConfig.value
      const selectedValueArr = []
      const selectObjList = []
      _getAllItemChecked(that.treeDataList[0])
      that.$emit('change', selectedValueArr, selectObjList, type)
      function _getAllItemChecked(treeDataList) {
        let parentItem
        treeDataList.forEach((item) => {
          if (item[childrenTag] && item[childrenTag].length > 0) {
            parentItem = item
            const currentCheckedAll = []
            const currentCheckedAllItems = []
            _getSubChecked(parentItem[childrenTag], currentCheckedAll)
            that.getSubCheckedItems(
              parentItem[childrenTag],
              currentCheckedAllItems,
              childrenTag
            ) // _getSubCheckedItems( //   parentItem[childrenTag], //   currentCheckedAllItems // );
            if (currentCheckedAll.every((checkItem) => checkItem === true)) {
              if (parentItem[valueTag]) {
                selectedValueArr.push(parentItem[valueTag])
                selectObjList.push(parentItem)
              }
            } else if (
              currentCheckedAll.some((checkItem) => checkItem === true)
            ) {
              currentCheckedAllItems.forEach((selectedItem) => {
                if (selectedItem[valueTag]) {
                  selectedValueArr.push(selectedItem[valueTag])
                  selectObjList.push(selectedItem)
                }
              })
            }
          } else {
            if (item.$checked) {
              // selectedValueArr.push(item.code)
              selectedValueArr.push(item[valueTag])
              selectObjList.push(item)
            }
          }
        })
      }
      function _getSubChecked(itemList, arr) {
        itemList.forEach((item) => {
          arr.push(!!item.$checked)
          if (item[childrenTag] && item[childrenTag].length > 0) {
            _getSubChecked(item[childrenTag], arr)
          }
        })
      } // function _getSubCheckedItems(itemList, arr) { //   itemList.forEach((item) => { //      if (item.$checked) { //         arr.push(item); //      } else { //         if (item[childrenTag] && item[childrenTag].length > 0) { //            _getSubCheckedItems(item[childrenTag], arr); //         } //      } //   }); // }
    },
    getSubCheckedItems(itemList, arr, childrenTag) {
      itemList.forEach((item) => {
        if (item.$checked) {
          arr.push(item)
        } else {
          if (item[childrenTag] && item[childrenTag].length > 0) {
            this.getSubCheckedItems(item[childrenTag], arr, childrenTag)
          }
        }
      })
    }, // 点击组件外部区域
    handleDropDownOutSide() {
      this.isOpen = false
      this.isClickOutSide = true
    }, // 设置当前的选中框的全选等状态
    setCheckBoxIndeteriminate(treeDataList) {
      const that = this
      const childrenTag = that.selectOptionsConfig.children
      _getAllItemChecked(treeDataList)
      const checkedAllArr = []
      that.getAllItemCheckedAll(treeDataList, checkedAllArr, childrenTag) // _getAllItemCheckedAll(treeDataList, checkedAllArr);
      if (checkedAllArr.every((item) => item === true)) {
        that.isCheckAll = true
        that.isIndeterminateAll = false
      } else if (checkedAllArr.some((item) => item === true)) {
        that.isCheckAll = false
        that.isIndeterminateAll = true
      } else if (checkedAllArr.every((item) => item === false)) {
        that.isCheckAll = false
        that.isIndeterminateAll = false
      } // function _getAllItemCheckedAll(itemList, arr) { //   itemList.forEach((item) => { //      arr.push(!!item.$checked); //      if (item[childrenTag] && item[childrenTag].length > 0) { //         _getAllItemCheckedAll(item[childrenTag], arr); //      } //   }); // }
      function _getAllItemChecked(treeDataList) {
        let parentItem
        treeDataList.forEach((item) => {
          if (item[childrenTag] && item[childrenTag].length > 0) {
            parentItem = item
            _getAllItemChecked(item[childrenTag])
            const currentCheckedAll = []
            that.getSubChecked(
              parentItem[childrenTag],
              currentCheckedAll,
              childrenTag
            ) // _getSubChecked(parentItem[childrenTag], currentCheckedAll);
            if (currentCheckedAll.every((checkItem) => checkItem === true)) {
              parentItem.$checked = true
              parentItem.$indeterminate = false
            } else if (
              currentCheckedAll.some((checkItem) => checkItem === true)
            ) {
              parentItem.$checked = false
              parentItem.$indeterminate = true
            } else if (
              currentCheckedAll.every((checkItem) => checkItem === false)
            ) {
              parentItem.$checked = false
              parentItem.$indeterminate = false
            }
          }
        })
      } // function _getSubChecked(itemList, arr) { //   itemList.forEach((item) => { //      arr.push(!!item.$checked); //      if (item[childrenTag] && item[childrenTag].length > 0) { //         _getSubChecked(item[childrenTag], arr); //      } //   }); // }
    },
    getAllItemCheckedAll(itemList, arr, childrenTag) {
      itemList.forEach((item) => {
        arr.push(!!item.$checked)
        if (item[childrenTag] && item[childrenTag].length > 0) {
          this.getAllItemCheckedAll(item[childrenTag], arr, childrenTag)
        }
      })
    },
    getSubChecked(itemList, arr, childrenTag) {
      itemList.forEach((item) => {
        arr.push(!!item.$checked)
        if (item[childrenTag] && item[childrenTag].length > 0) {
          this.getSubChecked(item[childrenTag], arr, childrenTag)
        }
      })
    }, // 设置所有选中和非选中
    setAllDataTreeChecked(treeList, checked) {
      const childrenTag = this.selectOptionsConfig.children
      treeList.forEach((item) => {
        if (!item[this.selectOptionsConfig.disabled]) {
          item.$checked = checked
        }
        if (item[childrenTag] && item[childrenTag].length > 0) {
          this.setAllDataTreeChecked(item[childrenTag], checked)
        } else {
        }
      })
    }, // 清空所有的active状态
    clearAllDataActive(treeList) {
      const childrenTag = this.selectOptionsConfig.children
      treeList.forEach((item) => {
        item.$active = false
        if (item[childrenTag] && item[childrenTag].length > 0) {
          this.clearAllDataActive(item[childrenTag])
        }
      })
    },
    findKeysItem(treeDataList, currentKey, currentItem) {
      const childrenTag = this.selectOptionsConfig.children
      treeDataList.forEach((item) => {
        if (item[this.selectOptionsConfig.value] === currentKey) {
          currentItem.push(item)
        } else {
          if (item[childrenTag] && item[childrenTag].length > 0) {
            this.findKeysItem(item[childrenTag], currentKey, currentItem)
          }
        }
      })
    },
    findAllParentItems(
      currentKey,
      treeDataList,
      originTreeDataList,
      parentItemsList,
      parentItem,
      parentItemsHasKey
    ) {
      if (parentItem && parentItemsHasKey) {
        parentItemsList.unshift(parentItem)
        this.findAllParentItems(
          parentItem[this.selectOptionsConfig.value],
          treeDataList,
          originTreeDataList,
          parentItemsList
        )
      } else {
        this.findKeysAndParentItem(
          currentKey,
          treeDataList,
          originTreeDataList,
          parentItemsList,
          parentItem,
          parentItemsHasKey
        )
      }
    },
    findKeysAndParentItem(
      currentKey,
      treeDataList,
      originTreeDataList,
      parentItemsList,
      parentItem
    ) {
      const childrenTag = this.selectOptionsConfig.children
      treeDataList.forEach((item) => {
        if (item[this.selectOptionsConfig.value] === currentKey && parentItem) {
          this.findAllParentItems(
            parentItem[this.selectOptionsConfig.value],
            originTreeDataList,
            originTreeDataList,
            parentItemsList,
            parentItem,
            true
          )
        } else {
          if (item[childrenTag] && item[childrenTag].length > 0) {
            this.findKeysAndParentItem(
              currentKey,
              item[childrenTag],
              originTreeDataList,
              parentItemsList,
              item
            )
          }
        }
      })
    }, // 将树形结构数据转换成平面结构
    treeListToArrList(treeList) {
      const arrList = []
      if (treeList && treeList[0]) {
        treeList.forEach((item) => {
          arrList.push(item)
        })
        this.treeToList = arrList
      }
    },
    addHideSelectedList(treeList) {
      const treeToListClone = Array.from(this.treeToList)
      treeList.forEach((item) => {
        if (
          !treeToListClone.find(
            (listItem) =>
              listItem[this.selectOptionsConfig.value] ===
              item[this.selectOptionsConfig.value]
          )
        ) {
          treeToListClone.push(item)
        }
      })
      this.treeToList = treeToListClone
    },
    isDisabled(subItem) {
      const childrenTag = this.selectOptionsConfig.children
      const disabledTag = this.selectOptionsConfig.disabled
      let currentChildren = subItem[childrenTag]
      let currentDisabled = subItem[disabledTag]
      if (currentChildren && currentChildren.length > 0) {
        return currentChildren.every((item) => this.isDisabled(item))
      }
      return currentDisabled && currentDisabled == -1 ? true : false
    },
    changeModeValue(value) {
      this.$emit('update:modeValue', value)
    },
    changeTab(value) {
      this.$emit('update:tabsValue', value)
      this.$emit('changeTab', value)
    },
  },
}
</script>
<style lang="scss">
.gb-ant-select-multiple-cascader {
  .input-group-class {
    max-height: 32px;
    display: flex;
    .first-select {
      max-height: 32px;
      width: 28%;
      // color:rgba(0, 0, 0, 0.25)
    }
    .second-select {
      flex: 1;
      .ant-select-selection__rendered {
        height: 30px;
        ul {
          display: inline-flex;
        }
      }
    }
  }
  .ant-select-dropdown-menu-item {
    display: none;
  }
  .cascader-content-item {
    .ant-checkbox-wrapper{
      .ant-checkbox-input{
        height: 21px;
        width: 18px;
        transform: translateY(-2px);
      }
    }
    .checkbox-text {
      padding-left: 8px;
      font-size: 14px;
      line-height: 22px;
      color: #4a4a4a;
      cursor: default;
      flex: 1;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
      text-align: left;
    }
    .item_disabled {
      color: rgba(0, 0, 0, 0.25) !important;
      cursor: not-allowed;
    }
    &.cascader-content-item-active {
      background-color: rgba(66, 70, 86, 0.05);
      .checkbox-text {
        // color: #1d47aa;
        color: rgba(0, 0, 0, 0.65);
        font-weight: 600;
      }
    }
    // .ant-checkbox-wrapper {
    //   //  .ant-checkbox-indeterminate {
    //   //     .ant-checkbox-inner {
    //   //        &:after {
    //   //           background-color: #1d47aa;
    //   //           left: 48%;
    //   //        }
    //   //     }
    //   //  }
    //   //  .ant-checkbox-inner {
    //   //     width: 14px;
    //   //     height: 14px;
    //   //     background-color: #fff;
    //   //     border-color: #7d8292;
    //   //     &:after {
    //   //        border-color: #1d47aa;
    //   //        left: 20%;
    //   //     }
    //   //  }
    // }
  }
  .ant-select-dropdown--multiple {
    min-width: auto !important;
  }
  &.multiple-cascader-outside {
    .ant-select-arrow {
      transform: rotate(180deg);
    }
  }
  .ant-select-arrow {
    margin-top: -8px;
    transition: transform 0.3s;
  }
  .ant-select-dropdown-menu-root {
    display: none;
  }
  .cascader-content-right {
    .right-wrap {
      .ant-checkbox {
        top: 0;
      }
    }
  }
}

.gb-ant-select-multiple-cascader {
  position: relative;
  .cascader-content-wrap {
    height: 100%;
    .cascader-content-container {
      display: flex;
      flex-direction: row;
      .cascader-content-list {
        list-style: none;
        // padding: 0;
        margin: 0;
        width: 153px;
        box-sizing: border-box;
        padding: 4px 0;
        max-height: 250px;
        overflow-y: auto;
        border-right: 1px solid rgba(222, 223, 226, 0.3);
        .cascader-content-item {
          // margin-top: 5px;
          display: flex;
          flex-direction: row;
          align-items: center;
          padding: 5px 12px;
          &:hover {
            background-color: #e6f7ff;
          }
        }
      }
    }
  }
  .cascader-not-content {
    padding: 10px;
  }
}

.tab-warpper {
  .ant-tabs-bar {
    width: 153px;
    margin: 0;
    .ant-tabs-nav {
      width: 100%;
    }
  }
  .ant-tabs-tab-active {
  }
  .ant-tabs-tab {
    padding: 0;
    margin: 0;
    width: 50%;
    text-align: center;
    &::before {
    }
  }
  .ant-tabs-ink-bar {
    // width: 15px!important;
    // left: 25%;
    // transform: translateX(-50%) !important;
  }
}
</style>
