<template>
  <div :id="id" :class="keypadClass" v-show="show" @click="stopPropagation && $event.stopPropagation()">
    <div :class="buttonWrapClass">
      <button
        v-for="(val, idx) in keyArray"
        :key="idx"
        ref="button"
        type="button"
        :class="setClass(val, idx)"
        @click="click(val, idx)"
      >
        {{ showKey(val) }}
      </button>
    </div>
    <slot></slot>
  </div>
</template>

<script>
if (!Object.entries) {
  Object.entries = function( obj ){
    var ownProps = Object.keys( obj ),
        i = ownProps.length,
        resArray = new Array(i); // preallocate the Array
    while (i--)
      resArray[i] = [ownProps[i], obj[ownProps[i]]];

    return resArray;
  };
}

export default {
  name: "VueNumericKeypad",
  props: {
    id: {
      type: String,
      required: false,
    },
    value: {
      type: String,
      default: "",
      required: true,
    },
    show: {
      type: [Boolean, Number],
      default: false,
      required: true,
    },
    encryptFunc: {
      type: Function,
      default: c => c,
    },
    encryptedValue: {
      type: Array,
      default: () => [],
    },
    options: {
      type: Object,
      default: () => ({}),
      validator: function (value) {
        const keyArrayDisable = (value.keyArray || []).some(key => {
          switch (typeof key) {
            case 'number':
              return (!Number.isInteger(key) || key < -1 || key > 9);
            case 'string':
              return key;
            default:
              return false;
          }
        });
        if (keyArrayDisable) {
          console.error("KeyArray can contain only an integer 'number' between -1 and 9 and an empty 'string'.");
          return false;
        }
        const classDisable = Object.keys(value).some(key => /Class/.test(key) && /[^0-9A-z\-_ ]/.test(value[key]));
        if (classDisable) {
          console.error("Class name can contain only 'a-z' and 'A-Z', '0-9', '_', '-', ' '.");
          return false;
        }
        return true;
      },
      required: false,
    },
  },
  data() {
    const columns = Number(this.options.columns) || 3;
    const arr1 = [1, 2, 3, 4, 5, 6, 7, 8, 9, "", 0, -1];
    const arr2 = [1, 2, 3, 4, 5, 6, 7, 8, 9, 0, "", -1];
    return {
      encryptedChar: typeof this.options.encryptedChar === 'string' ? this.options.encryptedChar.substring(0, 1) : "0",
      onEncrypt: Boolean(this.options.onEncrypt),
      keyArray: this.options.keyArray || (columns === 3 ? arr1 : arr2),
      keyRandomize: Boolean(this.options.keyRandomize ?? true),
      randomizeWhenClick: Boolean(this.options.randomizeWhenClick),
      fixDeleteKey: Boolean(this.options.fixDeleteKey ?? true),
      stopPropagation: Boolean(this.options.stopPropagation ?? true),
      keypadClass: this.options.keypadClass || 'numeric-keypad',
      buttonWrapClass: this.options.buttonWrapClass || 'numeric-keypad__button-wrap',
      buttonClass: this.options.buttonClass || 'numeric-keypad__button',
      deleteButtonClass: this.options.deleteButtonClass || 'numeric-keypad__button--delete',
      blankButtonClass: this.options.blankButtonClass || 'numeric-keypad__button--blank',
      activeButtonClass: this.options.activeButtonClass || 'numeric-keypad__button--active',
      activeButtonIndexes: {},
      activeButtonDelay: Number(this.options.activeButtonDelay) || 300,
      pseudoClick: Boolean(this.options.pseudoClick),
      rows: Number(this.options.rows) || 4,
      columns,
      zIndex: Number(this.options.zIndex) || 1,
      cellWidth: 0,
      cellHeight: 0,
      defaultStyleSheet: document.createElement('style'),
      defaultStyle: ['all', 'button', 'wrap', 'none'].find(s => s === this.options.setDefaultStyle) || 'all',
      keypadStylesIndex: null,
    };
  },
  watch: {
    show: function () {
      this.$nextTick(function () {
        if (this.show) this.resize();
      });
      if (this.keyRandomize) this.randomize();
    },
  },
  computed: {
    keypadStyles: function () {
      const fontSize = Math.round(Math.min(this.cellWidth, this.cellHeight) * 0.3);
      return `
        position: fixed;
        z-index: ${this.zIndex};
        bottom: 0;
        left: 0;
        right: 0;
        height: 40vh;
        padding: 10px;
        background-color: #fff;
        border-radius: 12px 12px 0 0;
        box-shadow: 0 -4px 4px rgba(0, 0, 0, 0.15);
        color: #000;
        overflow: hidden;
        font-size: ${fontSize}px;
      `;
    },
    buttonWrapStyles: function () {
      return `
        display: flex;
        witdth: 100%;
        height: 100%;
        justify-content: space-between;
        align-content: space-between;
        flex-wrap: wrap;
        gridGap: 5px;
      `;
    },
    buttonStyles: function () {
      const width = `calc(${Math.round(1000 / this.columns) / 10}% - ${Math.ceil(5 * (this.columns - 1) / this.columns)}px)`;
      const height = `calc(${Math.round(1000 / this.rows) / 10}% - ${Math.ceil(5 * (this.rows - 1) / this.rows)}px)`;
      return `
        flex: 0 0 auto;
        display: flex;
        width: ${width};
        height: ${height};
        justify-content: center;
        align-items: center;
        background-color: #fff;
        border: 1px solid #000;
        border-radius: 8px;
        font-size: inherit;
      `;
    }
  },
  methods: {
    click(key, idx) {
      this.activeButton(idx);
      if (this.pseudoClick) {
        const l = this.keyArray.length;
        const pIdx = Math.floor((Math.random() * (l - 1)) + idx + 1) % l;
        this.activeButton(pIdx);
      }
      let newVal = this.value;
      const encryptedValue = [...this.encryptedValue];
      if (this.onEncrypt) {
        if (key === -1) {
          newVal = this.value.slice(0, -1);
          encryptedValue.pop();
        } else if (key !== '') {
          newVal += this.encryptedChar;
          encryptedValue.push(this.encryptFunc(key.toString()));
        }
      } else {
        if (key === -1) {
          newVal = this.value.slice(0, -1);
        } else {
          newVal += key;
        }
      }
      this.$emit("update:value", String(newVal));
      this.$emit("update:encryptedValue", encryptedValue);
      if (this.keyRandomize && this.randomizeWhenClick) this.randomize();
    },
    randomize() {
      let newkeyArray = [];
      let delKeyCnt = 0;
      for (let i = 0; i < this.keyArray.length; i++) {
        let r = Math.random();
        if (this.fixDeleteKey && this.keyArray[i] == -1) {
          delKeyCnt++;
          continue;
        }
        if (r < 0.5) newkeyArray.push(this.keyArray[i]);
        else newkeyArray.unshift(this.keyArray[i]);
      }
      if (delKeyCnt) {
        for (let i = 0; i < delKeyCnt; i++) newkeyArray.push(-1);
      };
      this.keyArray = newkeyArray;
    },
    showKey(key) {
      if (key === -1) {
        return "del";
      } else return key;
    },
    resize() {
      this.cellWidth = this.$refs.button[0].offsetWidth;
      this.cellHeight = this.$refs.button[0].offsetHeight;
      const sheet = this.defaultStyleSheet.sheet;
      if (sheet && this.keypadStylesIndex !== null) {
        sheet.deleteRule(0);
        sheet.insertRule(`.${this.keypadClass.trim().split(' ')[0]} {${this.keypadStyles}}`, 0);
      }
    },
    setClass(key, idx) {
      const classArr = [this.buttonClass];
      if (key === -1) {
        classArr.push(this.deleteButtonClass);
      }
      if (key === '') {
        classArr.push(this.blankButtonClass);
      }
      if (this.activeButtonIndexes[idx]) {
        classArr.push(this.activeButtonClass);
      }
      return classArr;
    },
    activeButton(idx) {
      if (this.activeButtonIndexes[idx]) {
        clearTimeout(this.activeButtonIndexes[idx]);
      } else {
        this.$refs.button[idx].classList.add(this.activeButtonClass);
      }
      this.activeButtonIndexes[idx] = setTimeout(() => {
        this.$refs.button[idx].classList.remove(this.activeButtonClass);
        clearTimeout(this.activeButtonIndexes[idx]);
        delete this.activeButtonIndexes[idx];
      }, this.activeButtonDelay);
    },
    initDefaultStyles(sheet) {
      const test = /[^0-9A-z\-_ ]/;
      let padIndex = 0;
      if (this.setDefaultStyle !== 'button') {
        if (!test.test(this.keypadClass)) {
          this.keypadStylesIndex = padIndex;
          sheet.insertRule(`.${this.keypadClass.trim().split(' ')[0]} {${this.keypadStyles}}`, padIndex++);
        }
        if (!test.test(this.buttonWrapClass)) {
          sheet.insertRule(`.${this.buttonWrapClass.trim().split(' ')[0]} {${this.buttonWrapStyles}}`, padIndex++);
        }
      }
      if (this.setDefaultStyle !== 'wrap') {
        if (!test.test(this.buttonClass)) {
          sheet.insertRule(`.${this.buttonClass.trim().split(' ')[0]} {${this.buttonStyles}}`, padIndex++);
          if (!test.test(this.activeButtonClass)) {
            sheet.insertRule(`.${this.buttonClass.trim().split(' ')[0]}.${this.activeButtonClass.trim().split(' ')[0]} {background-color: #eaeaea;}`, padIndex++);
          }
        }
      }
    },
  },
  mounted() {
    window.addEventListener('resize', this.resize);
    if (this.setDefaultStyle !== 'none') {
      document.head.appendChild(this.defaultStyleSheet);
      this.initDefaultStyles(this.defaultStyleSheet.sheet);
    }
    if (this.keyRandomize) this.randomize();
    this.resize();
  },
  beforeDestroy() {
    window.removeEventListener('resize', this.resize);
  }
};
</script>
