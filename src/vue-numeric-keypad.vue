<template>
  <div :id="id" :class="keypadClass" v-show="show" @click="stopPropagation && $event.stopPropagation()">
    <div :class="buttonWrapClass">
      <button
        v-for="(val, idx) in keyArray"
        :key="idx"
        ref="button"
        type="button"
        :class="setClass(val, idx)"
        @pointerup="click(val, idx, $event)"
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

const arr1 = [1, 2, 3, 4, 5, 6, 7, 8, 9, "", 0, -1];
const arr2 = [1, 2, 3, 4, 5, 6, 7, 8, 9, 0, "", -1];
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
              return (!Number.isInteger(key) || key < -2 || key > 9);
            case 'string':
              return key;
            default:
              return false;
          }
        });
        if (keyArrayDisable) {
          console.error("KeyArray can contain only an integer 'number' between -2 and 9 and an empty 'string'.");
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
    return {
      encrypt: Boolean(this.options.encrypt),
      encryptedChar: typeof this.options.encryptedChar === 'string' ? this.options.encryptedChar.substring(0, 1) : "0",
      keyArray: this.options.keyArray || (columns === 3 ? arr1 : arr2),
      keyRandomize: Boolean(this.options.keyRandomize === undefined || this.options.keyRandomize),
      randomizeWhenClick: Boolean(this.options.randomizeWhenClick),
      fixDeleteKey: Boolean(this.options.fixDeleteKey === undefined || this.options.fixDeleteKey),
      fixBlankKey: Boolean(this.options.fixBlankKey === undefined || this.options.fixBlankKey),
      stopPropagation: Boolean(this.options.stopPropagation === undefined || this.options.stopPropagation),
      keypadClass: this.options.keypadClass || 'numeric-keypad',
      buttonWrapClass: this.options.buttonWrapClass || 'numeric-keypad__button-wrap',
      buttonClass: this.options.buttonClass || 'numeric-keypad__button',
      deleteButtonClass: this.options.deleteButtonClass || 'numeric-keypad__button--delete',
      clearButtonClass: this.options.clearButtonClass || 'numeric-keypad__button--clear',
      blankButtonClass: this.options.blankButtonClass || 'numeric-keypad__button--blank',
      activeButtonClass: this.options.activeButtonClass || 'numeric-keypad__button--active',
      activeButtonIndexes: {},
      activeButtonDelay: Number(this.options.activeButtonDelay) || 300,
      pseudoClick: Boolean(this.options.pseudoClick),
      pseudoClickDeleteKey: this.options.pseudoClickDeleteKey === undefined ? Boolean(this.options.pseudoClick) : Boolean(this.options.pseudoClickDeleteKey),
      pseudoClickBlankKey: this.options.pseudoClickBlankKey === undefined ? Boolean(this.options.pseudoClick) : Boolean(this.options.pseudoClickBlankKey),
      vibrate: Boolean(this.options.vibrate),
      vibratePattern: this.options.vibratePattern || 200,
      rows: Number(this.options.rows) || 4,
      columns,
      zIndex: Number(this.options.zIndex) || 1,
      cellWidth: 0,
      cellHeight: 0,
      defaultStyleSheet: document.createElement('style'),
      defaultStyle: ['all', 'button', 'wrap', 'none'].find(s => s === this.options.defaultStyle) || 'all',
      keypadStylesIndex: null,
      deleteKeyText: this.options.deleteKeyText === undefined ? 'del' : this.options.deleteKeyText,
      clearKeyText: this.options.clearKeyText === undefined ? 'clr' : this.options.clearKeyText,
    };
  },
  watch: {
    show: function () {
      this.$nextTick(function () {
        if (this.show) this.resize();
      });
      if (this.keyRandomize) this.randomize();
    },
    options: {
      deep: true,
      handler(options) {
        if (typeof options !== 'object') return;
        const columns = Number(options.columns) || 3;
        this.encryptedChar = typeof options.encryptedChar === 'string' ? options.encryptedChar.substring(0, 1) : "0";
        this.keyArray = options.keyArray || (columns === 3 ? arr1 : arr2);
        this.keyRandomize = Boolean(options.keyRandomize === undefined || options.keyRandomize);
        this.randomizeWhenClick = Boolean(options.randomizeWhenClick);
        this.fixDeleteKey = Boolean(options.fixDeleteKey === undefined || options.fixDeleteKey);
        this.fixBlankKey = Boolean(options.fixBlankKey === undefined || options.fixBlankKey);
        this.stopPropagation = Boolean(options.stopPropagation === undefined || options.stopPropagation);
        this.activeButtonDelay = Number(options.activeButtonDelay) || 300;
        this.pseudoClick = Boolean(options.pseudoClick);
        this.pseudoClickDeleteKey = options.pseudoClickDeleteKey === undefined ? Boolean(options.pseudoClick) : Boolean(options.pseudoClickDeleteKey);
        this.pseudoClickBlankKey = options.pseudoClickBlankKey === undefined ? Boolean(options.pseudoClick) : Boolean(options.pseudoClickBlankKey);
        this.vibrate = Boolean(options.vibrate);
        this.vibratePattern = this.vibratePattern || 200
        this.rows = Number(options.rows) || 4;
        this.zIndex = Number(options.zIndex) || 1;
        this.deleteKeyText = options.deleteKeyText === undefined ? 'del' : options.deleteKeyText;
        this.clearKeyText = options.clearKeyText === undefined ? 'clr' : options.clearKeyText;
        const defaultStyle = ['all', 'button', 'wrap', 'none'].find(s => s === options.defaultStyle) || 'all';
        if (this.defaultStyle !== defaultStyle) {
          this.defaultStyle = defaultStyle;
          document.head.removeChild(this.defaultStyleSheet);
          this.defaultStyleSheet = document.createElement('style');
          if (this.defaultStyle !== 'none') {
            document.head.appendChild(this.defaultStyleSheet);
            this.initDefaultStyles(this.defaultStyleSheet.sheet);
          }
        }
        if (this.keyRandomize) this.randomize();
    }}
  },
  computed: {
    keypadStyles: function () {
      const fontSize = Math.round(Math.min(this.cellWidth, this.cellHeight) * 0.3);
      const style = `
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
      `;
      return fontSize ? style + `\nfont-size: ${fontSize}px;` : style;
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
    click(key, idx, event) {
      if (!event.isTrusted) return;
      if (this.pseudoClick) {
        if (!(((key == -1 || key == -2) && !this.pseudoClickDeleteKey) || (key == '' && !this.pseudoClickBlankKey))) {
          const l = this.keyArray.length;
          const pIdx = Math.floor((Math.random() * (l - 1)) + idx + 1) % l;
          this.activeButton(pIdx);
        }
      }
      this.activeButton(idx);
      if (this.vibrate && window.navigator.vibrate) window.navigator.vibrate(this.vibratePattern);
      switch (key) {
        case -1:
          this.del();
          break;
        case -2:
          this.clear();
        case "":
          break;
        default:
          this.add(key);
          break;
      }
      if (this.keyRandomize && this.randomizeWhenClick) this.randomize();
    },
    del() {
      const encryptedValue = [...this.encryptedValue];
      const newVal = this.value.slice(0, -1);
      this.$emit("update:value", String(newVal));
      if (this.encrypt) {
        encryptedValue.pop();
        this.$emit("update:encryptedValue", encryptedValue);
      }
    },
    clear() {
      this.$emit("update:value", '');
      if (this.encrypt) this.$emit("update:encryptedValue", []);
    },
    add(key) {
      const encryptedValue = [...this.encryptedValue];
      let newVal = this.value;
      if (this.encrypt) {
        newVal += this.encryptedChar;
        encryptedValue.push(this.encryptFunc(key.toString()));
        this.$emit("update:encryptedValue", encryptedValue);
      } else {
        newVal += key;
      }
      this.$emit("update:value", String(newVal));
    },
    randomize() {
      const l = this.keyArray.length;
      const newkeyArray = [...this.keyArray];
      for (let i = 0; i < l; i++) {
        const key = newkeyArray[i];
        if (this.checkFix(key)) continue;

        let r = Math.floor(Math.random() * l);
        if (r == i) r = (r + 1) % l;
        let count = 0;
        while ((this.checkFix(newkeyArray[r]) || newkeyArray[r] === newkeyArray[i]) && count < l) {
          r = (r + 1) % l;
          count++;
        }
        if (count >= l) continue;
        newkeyArray[i] = newkeyArray[r];
        newkeyArray[r] = key;
      }
      this.keyArray = newkeyArray;
    },
    checkFix(key) {
      if (key === '' && this.fixBlankKey) return true;
      if (key === -1 && this.fixDeleteKey) return true;
      if (key === -2 && this.fixDeleteKey) return true;
      return false;
    },
    showKey(key) {
      if (key === -1) {
        return this.deleteKeyText;
      } else if (key === -2) {
        return this.clearKeyText;
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
      if (key === -2) {
        classArr.push(this.clearButtonClass);
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
      if (this.defaultStyle === 'none') return;
      const test = /[^0-9A-z\-_ ]/;
      let padIndex = 0;
      if (this.defaultStyle !== 'button') {
        if (!test.test(this.keypadClass)) {
          this.keypadStylesIndex = padIndex;
          sheet.insertRule(`.${this.keypadClass.trim().split(' ')[0]} {${this.keypadStyles}}`, padIndex++);
        }
        if (!test.test(this.buttonWrapClass)) {
          sheet.insertRule(`.${this.buttonWrapClass.trim().split(' ')[0]} {${this.buttonWrapStyles}}`, padIndex++);
        }
      }
      if (this.defaultStyle !== 'wrap') {
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
    if (this.defaultStyle !== 'none') {
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
