<template>
  <div class="vue-list-picker">
    <div class="list-picker-container list-picker-left" v-bind:class="{'disabled': disabled}">
      <div class="list-picker-title" :class="getTitleClasses">
        {{ titleLeft | textSubstr(titleSubstr) }}
      </div>
      <div class="list-picker-panel" ref="moverleft">
        <div class="list-picker-item"
             v-for="item in unselectedItems"
             :key="item[contentKey]"
             :class="[getContentClasses, {'list-picker-selected': item.isSelected}]"
             @click="selectUnselectItem(item, unselectedItems)"
             @mousemove="selectItem(item, selectedItems)"
             @mousedown="startDrag"
        >
          {{ item[contentAttr] | textSubstr(contentSubstr) }}
        </div>
      </div>
    </div>
    <div class="list-picker-actions" v-bind:class="{'disabled': disabled}">
      <a @click="moveAllRight" :class="buttonClass">
        <img v-if="!this.$slots.moveAllRight" src="../assets/chevrons-right.svg" alt="Chevrons right">
        <slot name="moveAllRight"/>
      </a>
      <a @click="moveRight" class="mb-25" :class="buttonClass">
        <img v-if="!this.$slots.moveRight" src="../assets/chevron-right.svg" alt="Chevron right">
        <slot name="moveRight"/>
      </a>
      <a @click="moveLeft" :class="buttonClass">
        <img v-if="!this.$slots.moveLeft" src="../assets/chevron-left.svg" alt="Chevron left">
        <slot name="moveLeft"/>
      </a>
      <a @click="moveAllLeft" class="mb-25" :class="buttonClass">
        <img v-if="!this.$slots.moveAllLeft" src="../assets/chevrons-left.svg" alt="Chevrons left">
        <slot name="moveAllLeft"/>
      </a>
      <a @click="unselectAll" :class="buttonClass">
        <img v-if="!this.$slots.unselectAll" src="../assets/x.svg" alt="X icon">
        <slot name="unselectAll"/>
      </a>
    </div>
    <div class="list-picker-container list-picker-right" v-bind:class="{'disabled': disabled}">
      <div class="list-picker-title" :class="getTitleClasses">
        {{ titleRight | textSubstr(titleSubstr) }}
      </div>
      <div class="list-picker-panel" ref="moverright">
        <div class="list-picker-item"
             v-for="item in selectedItems"
             :key="item[contentKey]"
             :class="[getContentClasses, {'list-picker-selected': item.isSelected}]"
             @click="selectUnselectItem(item, selectedItems)"
             @mousemove="selectItem(item, selectedItems)"
             @mousedown="startDrag"
        >
          {{ item[contentAttr] | textSubstr(contentSubstr) }}
        </div>
      </div>
    </div>
  </div>
</template>
<script>
  export default {
    name: 'VueListPicker',
    props: {
      leftItems: {
        type: Array,
        required: true
      },
      rightItems: {
        type: Array,
        required: true
      },
      movedItemLocation: {
        type: String,
        default: 'bottom'
      },
      titleLeft: {
        type: String,
        default: 'Selecione o tipo de campo'
      },
      disabled: {
        type: Boolean,
        default: false
      },
      titleRight: {
        type: String,
        default: 'Tipo de campo selecionado'
      },
      titleCentered: {
        type: Boolean,
        default: true
      },
      titleClass: {
        type: String,
        default: ''
      },
      titleSubstr: {
        type: Number,
        default: 40
      },
      buttonClass: {
        type: String,
        default: ''
      },
      contentKey: {
        type: String,
        default: 'id'
      },
      contentAttr: {
        type: String,
        default: 'content'
      },
      contentCentered: {
        type: Boolean,
        default: false
      },
      contentClass: {
        type: String,
        default: ''
      },
      contentSubstr: {
        type: Number,
        default: 23
      },
      minHeight: {
        type: String,
        default: '450px'
      },
      height: {
        type: String,
        default: ''
      },
      minWidth: {
        type: String,
        default: '220px'
      },
      width: {
        type: String,
        default: ''
      }
    },
    data: () => ({
      loading: false,
      dragging: false,
      lastMovedItem: null
    }),
    filters: {
      textSubstr(value, qtd = 250, mask = '...') {
        return value && value.length > qtd ? `${value.substring(0, qtd)}${mask}` : value
      }
    },
    computed: {
      unselectedItems: {
        get() {
          return this.leftItems
        },
        set(val) {
          this.$emit('leftItems:update', val)
        }
      },
      selectedItems: {
        get() {
          return this.rightItems
        },
        set(val) {
          this.$emit('rightItems:update', val)
        }
      },
      getTitleClasses() {
        const {titleClass, titleCentered} = this;

        return titleClass || {'text-center': titleCentered}
      },
      getContentClasses() {
        const {contentClass, contentCentered} = this;

        return contentClass || {'text-center': contentCentered}
      },
      getStyles() {
        const {height, minHeight, minWidth, width} = this;

        return {
          height,
          minHeight,
          minWidth,
          width
        }
      }
    },
    created() {
      this.unselectedItems = this.unselectedItems && this.unselectedItems.length
        ? this.unselectedItems.map(it => ({...it, isSelected: false}))
        : []
      this.selectedItems = this.selectedItems && this.selectedItems.length
        ? this.selectedItems.map(it => ({...it, isSelected: false}))
        : []
    },
    mounted() {
      document.addEventListener('mouseup', this.stopDrag)
    },
    beforeDestroy() {
      document.removeEventListener('mouseup', this.stopDrag)
    },
    methods: {
      startDrag() {
        if (this.disabled) return;

        this.dragging = true
      },
      stopDrag() {
        if (this.disabled) return;

        this.dragging = false
      },
      selectUnselectItem(item, items) {
        if (this.disabled) return;

        this.setItem(item, items, !item.isSelected)
      },
      setItem(item, items, val) {
        let itemBase = item

        if (items && items.length && !itemBase) {
          itemBase = items[0]
        }

        if (!itemBase) return

        itemBase.isSelected = val
        this.$forceUpdate()
      },
      selectItem(item, items) {
        if (this.disabled) return;

        if (this.dragging) {
          const VALUE_IS_SET_TO_TRUE = true
          this.setItem(item, items, VALUE_IS_SET_TO_TRUE)
        }
      },
      moveRight() {
        if (this.disabled) return;

        this.moveOne(this.unselectedItems, this.selectedItems, 'moverleft', 'move-right')
      },
      moveLeft() {
        if (this.disabled) return;

        this.moveOne(this.selectedItems, this.unselectedItems, 'moverright', 'move-left')
      },
      unselect(items) {
        if (!items.length) return

        items.forEach(it => {
          it.isSelected = false
        })
        this.$forceUpdate()
      },
      unselectAll() {
        if (this.disabled) return;

        this.unselect(this.unselectedItems)
        this.unselect(this.selectedItems)
        this.$emit('unselect-all')
      },
      moveOne(firstArray, secondArray, refsName, event) {
        if (this.disabled) return;

        const items = firstArray.filter(itm => itm.isSelected)
        if (!items || !items.length) return

        items.forEach(item => {
          firstArray.forEach((it, idx) => {
            if (it[this.contentKey] === item[this.contentKey]) {
              firstArray.splice(idx, 1)
              this.$emit(event, it)

              if (this.movedItemLocation === 'top') {
                secondArray.unshift(it)
                this.$refs[refsName].scrollTop = 0
              } else {
                secondArray.push(it)
              }
            }
          })
        })

        this.unselectAll()
      },
      moveAllRight() {
        if (this.disabled) return;

        this.moveAll(this.unselectedItems, this.selectedItems)
        this.$emit('move-all-right', this.selectedItems)
      },
      moveAllLeft() {
        if (this.disabled) return;

        this.moveAll(this.selectedItems, this.unselectedItems)
        this.$emit('move-all-left', this.unselectedItems)
      },
      moveAll(firstArray, secondArray) {
        if (this.disabled) return;

        for (let i = firstArray.length - 1; i >= 0; i--) {
          let item = firstArray[i]
          firstArray.splice(i, 1)
          secondArray.push(item)
        }

        this.unselectAll()
      }
    }
  }
</script>
<style scoped>
  .vue-list-picker {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Oxygen", "Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Helvetica Neue", sans-serif;
    -webkit-font-smoothing: antialiased;
    text-rendering: optimizeLegibility;
    display: flex;
    flex-flow: row nowrap;
    cursor: default;
  }

  .vue-list-picker .list-picker-container {
    flex: 1 1 auto;
    border-radius: 3px;
    overflow: auto;
    display: flex;
    flex-direction: column
  }

  .vue-list-picker .list-picker-panel {
    min-height: 400px;
    overflow-y: auto;
    border: 1px solid #dedede;
    background: #fefefe;
    margin-top: 11px;
    border-radius: 2px;
  }

  .vue-list-picker .list-picker-item::selection {
    background: unset;
  }

  .vue-list-picker .list-picker-actions {
    flex: 0 0 auto;
    align-self: center;
    padding: 0 15px;
  }

  .vue-list-picker .list-picker-actions > a {
    min-width: 50px;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 8px 0;
    border: 1px solid #dedede;
    margin-top: -1px;
  }

  .vue-list-picker .list-picker-item, .vue-list-picker .list-picker-selected, .vue-list-picker .list-picker-title {
    padding: 8px 10px;
    border-bottom: 1px solid #ccc;
    font-size: 13px;
  }

  .vue-list-picker .list-picker-title {
    background: #f0f0f0;
    border: 1px solid #dcdcdc;
    color: #595959;
  }

  .vue-list-picker .list-picker-selected {
    background: #13CB93;
    color: #F0F0F0;
    font-weight: 600;
  }

  .vue-list-picker .mb-25 {
    margin-bottom: 25px;
  }

  .vue-list-picker .text-center {
    text-align: center;
  }
</style>
