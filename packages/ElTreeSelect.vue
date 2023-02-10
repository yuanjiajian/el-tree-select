<!-- eslint-disable vue/no-mutating-props -->
<template>
  <div>
    <el-select
      v-if="isEdit"
      ref="select"
      v-model="labelName"
      :multiple="isShowCheckbox"
      :clearable="isClearable"
      :disabled="isDisabled"
      placeholder="请选择"
      @remove-tag="removeTag"
      @clear="clear"
    >
      <template #empty>
        <el-tree
          v-if="!treeLazy"
          ref="tree"
          :data="treeData"
          :node-key="treeNodeKey"
          :props="treeProps"
          :show-checkbox="isShowCheckbox"
          check-strictly
          :expand-on-click-node="false"
          :default-expanded-keys="defaultExpandedKeys"
          highlight-current
          @node-click="handleNodeClick"
          @check-change="handleCheckChange"
        />
        <el-tree
          v-if="treeLazy"
          ref="lazyTree"
          :node-key="treeNodeKey"
          :props="treeProps"
          :load="treeLoadFun"
          :lazy="treeLazy"
          :show-checkbox="isShowCheckbox"
          check-strictly
          :expand-on-click-node="false"
          :default-expanded-keys="defaultExpandedKeys"
          highlight-current
          @node-click="handleNodeClick"
          @check-change="handleCheckChange"
        />
      </template>
    </el-select>
    <div v-else class="filter_form " style="height: 36px">
      <div v-if="isEllipsis" class="ellipsis">
        <el-tooltip class="item" effect="dark" :content="labelName.toString()+''" placement="top-start">
          <span style="font-size: 12px;">{{ labelName.toString() }}</span>
        </el-tooltip>
      </div>
      <div v-if="!isEllipsis" style="font-size: 12px;">
        {{ labelName.toString() }}
      </div>
    </div>
  </div>

</template>

<script>
import XEUtils from 'xe-utils'

export default {
  model: {
    prop: 'value',
    event: 'change'
  },
  props: {
    value: {
      type: [String, Array],
      default: ''
    },
    isClearable: {
      type: Boolean,
      default: true
    },
    isDisabled: {
      type: Boolean,
      default: false
    },
    isEdit: {
      type: Boolean,
      default: true
    },
    isEllipsis: {
      type: Boolean,
      default: true
    },
    treeData: {
      type: Array,
      default() {
        return [
          {
            label: '测试',
            id: '1',
            children: [
              {
                id: '1-1',
                label: '测试-1'
              },
              {
                id: '1-2',
                label: '测试-2'
              }
            ]
          },
          {
            label: '测试2',
            id: '2',
            isLeaf: true
          }
        ]
      }
    },
    treeNodeKey: {
      type: String,
      default: 'id'
    },
    treeProps: {
      type: Object,
      default() {
        return {
          label: 'label',
          children: 'children',
          disabled: 'disabled',
          isLeaf: 'isLeaf'
        }
      }
    },
    treeLoad: {
      type: Function,
      default: (node, resolve) => {
        setTimeout(() => {
          resolve(dealNavLsit([
            { id: '1', label: '测试', parentId: '0' },
            { id: '1-1', parentId: '1', label: '测试-1', isLeaf: true },
            { id: '1-2', parentId: '1', label: '测试-2', isLeaf: true },
            { id: '2', parentId: '0', label: '测试2', isLeaf: true }
          ], node.data ? node.data.id : '0'))
        }, 1000)
      }
    },
    treeLazy: {
      type: Boolean,
      default: false
    },
    defaultExpandedKeys: {
      type: Array,
      default: () => []
    }
  },
  data() {
    return {
      isShowCheckbox: false,
      labelName: '',
      treeDataList: []
    }
  },
  watch: {
    value: {
      handler(n) {
        if (Array.isArray(n)) {
          this.isShowCheckbox = true
        } else {
          this.isShowCheckbox = false
        }
      },
      immediate: true,
      deep: true
    },
    treeDataList: {
      handler() {
        this.updateLabelName()
        this.updateCheckedNode()
      },
      deep: true
    }
  },
  mounted() {
    if (!this.treeLazy) {
      this.treeDataList = this.treeData
    }
  },
  methods: {
    treeLoadFun(node, resolve) {
      new Promise((resolve) => {
        this.treeLoad(node, resolve)
      }).then((data) => {
        const _data = XEUtils.clone(data, true)
        if (node.data) {
          const treeDataList = XEUtils.clone(this.treeDataList, true)
          XEUtils.mapTree(treeDataList, item => {
            if (item[this.treeNodeKey] === node.data[this.treeNodeKey]) {
              item[this.treeProps.children] = _data
              return item
            } else {
              return item
            }
          }, { children: this.treeProps.children, mapChildren: this.treeProps.children })
          this.treeDataList = treeDataList
        } else {
          this.treeDataList = _data
        }
        resolve(data)
      })
    },
    updateLabelName() {
      if (this.isShowCheckbox) {
        const labelName = []
        this.value.forEach(val => {
          const data = XEUtils.findTree(this.treeDataList, node => node[this.treeNodeKey] === val, { children: this.treeProps.children })
          if (data && data.item) {
            labelName.push(data.item[this.treeProps.label])
          }
        })
        this.labelName = labelName
      } else {
        let labelName = ''
        const data = XEUtils.findTree(this.treeDataList, node => node[this.treeNodeKey] === this.value, { children: this.treeProps.children })
        if (data && data.item) {
          labelName = data.item[this.treeProps.label]
        }
        this.labelName = labelName
      }
    },
    updateCheckedNode() {
      const tree = this.treeLazy ? this.$refs.lazyTree : this.$refs.tree
      this.isShowCheckbox ? tree.setCheckedKeys(this.value) : tree.setCurrentKey(this.value)
    },
    removeTag(tag) {
      const data = XEUtils.findTree(this.treeDataList, node => node[this.treeProps.label] === tag, { children: this.treeProps.children })
      if (data && data.item) {
        const value = XEUtils.clone(this.value, true)
        value.splice(value.indexOf(data.item[this.treeNodeKey]), 1)
        this.$emit('change', value)
        this.$nextTick(() => {
          this.updateCheckedNode()
        })
      }
    },
    clear() {
      const nodes = Object.values(this.treeLazy ? this.$refs.lazyTree.store.nodesMap : this.$refs.tree.store.nodesMap)
      nodes.forEach(node => {
        node.checked = false
        node.isCurrent = false
      })
      this.$emit('change', this.isShowCheckbox ? [] : '')
    },
    handleNodeClick(node) {
      if (!this.isShowCheckbox) {
        this.$emit('change', node[this.treeNodeKey])
        this.$nextTick(() => {
          this.updateLabelName()
        })
        this.$refs.select.blur()
      }
    },
    handleCheckChange() {
      if (this.isShowCheckbox) {
        const data = this.treeLazy ? this.$refs.lazyTree.getCheckedNodes() : this.$refs.tree.getCheckedNodes()
        this.$emit('change', data.map(item => item[this.treeNodeKey]))
        this.$nextTick(() => {
          this.updateLabelName()
        })
      }
    }
  }
}

function dealNavLsit(data, parentId) {
  var tree = []
  for (var i = 0; i < data.length; i++) {
    if (data[i].parentId === parentId) {
      var obj = data[i]
      tree.push(obj)
    }
  }
  return tree
}
</script>
<style></style>
