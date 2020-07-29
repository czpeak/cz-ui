<template>
  <li
    @click.stop="selectOptionClick"
    class="el-select-dropdown__item"
    v-show="visible"
    :class="{
      'is-disabled': disabled || groupDisabled || limitReached,
    }">
    <slot>{{ currentLabel }}</slot>
  </li>
</template>

<script>
  import Emitter from './../../common/mixins/emitter';
  export default {
    mixins: [Emitter],

    name: 'ClOption',

    componentName: 'ClOption',

    inject: ['select'],

    props: {
      value: {
        required: true
      },
      label: [String, Number],
      disabled: {
        type: Boolean,
        default: false
      }
    },

    data() {
      return {
        groupDisabled: false,
        visible: true
      }
    },

    computed: {
      isObject() {
        return Object.prototype.toString.call(this.value).toLowerCase() === '[object object]';
      },

      currentLabel() {
        return this.label || (this.isObject ? '' : this.value)
      },

      currentValue() {
        return this.value || this.label || '';
      },

      limitReached() {
        if (this.select.multiple) {
          return (!this.itemSelected && (this.select.value || []).length >= this.select.maxMultipleLimit &&
            this.select.maxMultipleLimit > 0) || (this.itemSelected && (this.select.value || []).length <= this.select.minMultipleLimit && this.select.minMultipleLimit > 0)
        } else {
          return false
        }
      }
    },

    methods: {
      selectOptionClick() {
        if (this.disabled !== true && this.groupDisabled !== true) {
          console.log('222')
          this.dispatch('ClSelect', 'handleOptionClick', [this, true])
        }
      }
    },

    created() {
      this.select.options.push(this);
      this.select.cachedOptions.push(this);
      this.select.optionsCount++;
      this.select.filteredOptionsCount++;

      // this.$on('queryChange', this.queryChange);
      // this.$on('handleGroupDisabled', this.handleGroupDisabled);
    },

    beforeDestory() {
      const { selected, multiple } = this.select;
      let selectedOptions = multiple ? selected : [selected];
      let index = this.select.cachedOptions.indexOf(this);
      let selectedIndex = selectedOptions.indexOf(this);

      // if option is not selected, remove it from cache
      if (index > -1 && selectedIndex < 0) {
        this.select.cachedOptions.splice(index, 1);
      }
      this.select.onOptionDestroy(this.select.options.indexOf(this));
    }
  }
</script>

<style lang="scss" scoped>
@import './../theme-chalk/option.scss'
</style>
