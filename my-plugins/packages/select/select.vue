<template>
  <div class="el-select" v-clickoutside="handleClose">
    <div class="el-select-box" @click.stop="toggleMenu">
      <input
        type="text"
        ref="reference"
        v-model="selectedLabel"
        :readonly="readonly"
        :placeholder="currentPlaceholder"
      >
    </div>
    <transition
      name="el-zoom-in-top">
      <div class="el-select-dropdown el-popper">
        <div class="el-scrollbar">
          <div class="el-select-dropdown__wrap">
            <ul class="el-select-dropdown__list"
              v-show="visible">
              <slot></slot>
            </ul>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
  import { valueEquals, getValueByPath } from '../../common/utils/util'
  import Clickoutside from '../../common/utils/clickoutside';
  // import ClOption from '../option/option.vue';
  export default {
    name: 'ClSelect',

    componentName: 'ClSelect',

    inject: {

    },

    provide() {
      return {
        'select': this
      }
    },

    directives: { Clickoutside },

    data() {
      return {
        options: [],
        cachedOptions: [],
        selected: this.multiple ? [] : {},
        optionsCount: 0,
        filteredOptionsCount: 0,
        selectedLabel: '',
        visible: false,
        currentPlaceholder: '',
        cachedPlaceHolder: '',
      }
    },

    props: {
      value: {
        required: true
      },
      multiple: Boolean,
      maxMultipleLimit: {
        type: Number,
        default: 0
      },
      minMultipleLimit: {
        type: Number,
        default: 0
      },
      placeholder: {
        type: String,
        default: 'Please Select'
      },
    },

    components: {
      // ClOption
    },
    filters: {

    },

    computed: {
      readonly() {
        return true
      }
    },

    watch: {
      placeholder(val) {
        this.cachedPlaceHolder = this.currentPlaceholder = val;
      },

      value(val, oldVal) {
        if (this.multiple) {
          if ((val && val.length > 0) || (this.$refs.input && this.query !== '')) {
            this.currentPlaceholder = '';
          } else {
            this.currentPlaceholder = this.cachedPlaceHolder;
          }
          if (this.filterable && !this.reserveKeyword) {
            this.query = '';
            this.handleQueryChange(this.query);
          }
        }
        this.setSelected();

        // if (!valueEquals(val, oldVal)) {
        //   this.dispatch('ElFormItem', 'el.form.change', val);
        // }
      },

      visible(val) {
        if(!val) {
          this.selectedLabel = '';
          if (!this.multiple) {
            if (this.selected) {
              this.selectedLabel = this.selected.currentLabel;
            }
          }
        } else {
          console.log('显示下拉选项')
        }
        this.$emit('visible-change', val);
      },

      options() {
        if (this.$isServer) return;
        let inputs = this.$el.querySelectorAll('input');  //查找获取焦点的input
        if ([].indexOf.call(inputs, document.activeElement) === -1) {
          this.setSelected();
        }
        // if (this.defaultFirstOption && (this.filterable || this.remote) && this.filteredOptionsCount) {
        //   this.checkDefaultFirstOption();
        // }
      }
    },

    methods: {
      handleClose() {
        this.visible = false;
      },
      toggleMenu() {
        this.visible = !this.visible
      },
      emitChange(val) {
        if (!valueEquals(this.value, val)) {
          this.$emit('change', val);
        }
      },
      handleOptionSelect(option, byClick) {
        console.log(option, byClick)
        if (this.multiple) {
          console.log('multiple')
        } else {
          this.$emit('input', option.value);  // 实现双向绑定
          this.emitChange(option.value);  // chang事件钩子
          this.selected = option
          this.visible = false;
        }
      },

      getOption(value) {
        let option;
        const isObject = Object.prototype.toString.call(value).toLowerCase() === '[object object]';
        const isNull = Object.prototype.toString.call(value).toLowerCase() === '[object null]';
        const isUndefined = Object.prototype.toString.call(value).toLowerCase() === '[object undefined]';

        for (let i = this.cachedOptions.length - 1; i >= 0; i--) {
          const cachedOption = this.cachedOptions[i];
          const isEqual = isObject
            ? getValueByPath(cachedOption.value, this.valueKey) === getValueByPath(value, this.valueKey)
            : cachedOption.value === value;
          if (isEqual) {
            option = cachedOption;
            break;
          }
        }
        if (option) return option;
        const label = (!isObject && !isNull && !isUndefined)
          ? value : '';
        let newOption = {
          value: value,
          currentLabel: label
        };
        if (this.multiple) {
          newOption.hitState = false;
        }
        return newOption;
      },

      setSelected() {
        if (!this.multiple) {
          let option = this.getOption(this.value);
          if (option.created) {
            this.createdLabel = option.currentLabel;
            this.createdSelected = true;
          } else {
            this.createdSelected = false;
          }
          this.selectedLabel = option.currentLabel;
          this.selected = option;
          if (this.filterable) this.query = this.selectedLabel;
          return;
        }
        let result = [];
        if (Array.isArray(this.value)) {
          this.value.forEach(value => {
            result.push(this.getOption(value));
          });
        }
        this.selected = result;
      }
    },

    created() {
      this.cachedPlaceHolder = this.currentPlaceholder = this.placeholder;
      if (this.multiple && !Array.isArray(this.value)) {
        this.$emit('input', []);
      }
      if (!this.multiple && Array.isArray(this.value)) {
        this.$emit('input', '');
      }


      this.$on('handleOptionClick', this.handleOptionSelect)
    },

    mounted() {
      if (this.multiple && Array.isArray(this.value) && this.value.length > 0) {
        this.currentPlaceholder = '';
      }

      this.setSelected();
    },
  }
</script>

<style lang="scss" scoped>
@import './../theme-chalk/select.scss';
@import './../theme-chalk/input.scss'
</style>
