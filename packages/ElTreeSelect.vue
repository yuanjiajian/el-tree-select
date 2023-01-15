<!-- eslint-disable vue/no-mutating-props -->
<template>
  <el-select
    v-model="labelName"
    :multiple="isShowCheckbox"
    placeholder="请选择"
  >
    <template #empty>
      <el-tree
        v-if="!treeLazy"
        ref="tree"
        :data="treeData"
        :node-key="treeNodeKey"
        :props="treeProps"
        :show-checkbox="isShowCheckbox"
        :check-strictly="isCheckStrictly"
      />
      <el-tree
        v-if="treeLazy"
        ref="lazyTree"
        :node-key="treeNodeKey"
        :props="treeProps"
        :load="treeLoad"
        :lazy="treeLazy"
        :show-checkbox="isShowCheckbox"
        :check-strictly="isCheckStrictly"
      />
    </template>
  </el-select>
</template>

<script>

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
          resolve([
            {
              label: `测试${parseInt(Math.random() * 100)}`,
              id: '1'
            },
            {
              label: `测试${parseInt(Math.random() * 100)}`,
              id: `${Math.random() * 100}`,
              isLeaf: true
            }
          ])
        }, 1000)
      }
    },
    treeLazy: {
      type: Boolean,
      default: true
    },
    isCheckStrictly: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      isShowCheckbox: false,
      labelName: ''
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
    }
  },
  mounted() {
    this.updateLabelName()
  },
  methods: {
    updateLabelName() {
      this.$watch(() => {
        return this.treeLazy ? this.$refs.lazyTree.store : this.$refs.tree.store
      }, (n) => {
        const nodesMap = n.nodesMap
        if (this.isShowCheckbox) {
          const labelNameList = []
          Object.keys(nodesMap).map(key => {
            if (this.value.includes(key)) {
              labelNameList.push(nodesMap[key].data[this.treeProps.label])
            }
          })
          this.labelName = labelNameList
        } else {
          Object.keys(nodesMap).map(key => {
            if (this.value === key) {
              this.labelName = nodesMap[key].data[this.treeProps.label]
            }
          })
        }
      }, { immediate: true, deep: true })
    }
  }
}
</script>

<style></style>
