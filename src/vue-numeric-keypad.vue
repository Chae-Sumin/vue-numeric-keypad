<template>
  <div :id="id" :class="keypadClass" :style="keypadStyles" v-show="show" @click="stopPropagation && $event.stopPropagation()">
    <div :class="buttonWrapClass" :style="buttonWrapStyles">
      <button
        v-for="(val, idx) in keyArray"
        :key="idx"
        ref="button"
        type="button"
        :class="buttonClass + ' ' + (val === -1 ? deleteButtonClass : val === '' ? blankButtonClass : '')"
        @click="click(val)"
        :style="btnStyle(val)"
      >
        {{ showKey(val) }}
      </button>
    </div>
  </div>
</template>

<script>
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
    options: {
      type: Object,
      default: function () {
        return {};
      },
      required: false,
    },
  },
  data() {
    const columns = Number(this.options.columns) || 3;
    return {
      keyArray: columns === 3 ? [1, 2, 3, 4, 5, 6, 7, 8, 9, "", 0, -1] : [1, 2, 3, 4, 5, 6, 7, 8, 9, 0, "", -1],
      keyRandomize: this.options.keyRandomize === undefined ? true : Boolean(this.options.keyRandomize),
      randomizeWhenClick: this.options.randomizeWhenClick === undefined ? false : Boolean(this.options.randomizeWhenClick),
      fixDeleteKey: this.options.fixDeleteKey === undefined ? true : Boolean(this.options.fixDeleteKey),
      stopPropagation: this.options.stopPropagation === undefined ? true : Boolean(this.options.stopPropagation),
      keypadClass: this.options.keypadClass === undefined ? 'numeric-keypad' : String(this.options.keypadClass),
      buttonWrapClass: this.options.buttonWrapClass === undefined ? 'numeric-keypad__button-wrap' : String(this.options.buttonWrapClass),
      buttonClass: this.options.buttonClass === undefined ? 'numeric-keypad__button' : String(this.options.buttonClass),
      deleteButtonClass: this.options.deleteButtonClass === undefined ? 'numeric-keypad__button--delete' : String(this.options.deleteButtonClass),
      blankButtonClass: this.options.blankButtonClass === undefined ? 'numeric-keypad__button--blank' : String(this.options.blankButtonClass),
      rows: Number(this.options.rows) || 4,
      columns: columns,
      cellWidth: 0,
      cellHeight: 0,
    };
  },
  watch: {
    show: function () {
      this.$nextTick(function () {
        if (this.show) {
          this.cellWidth = this.$refs.button[0].offsetWidth;
          this.cellHeight = this.$refs.button[0].offsetHeight;
        }
      }.bind(this));
      if (this.keyRandomize) {
        this.randomize(this.fixDeleteKey);
      }
    },
  },
  computed: {
    keypadStyles: function () {
      const fontSize = Math.min(this.cellWidth, this.cellHeight) * 0.3;
      return this.options.keypadStyles || {
        position: 'fixed',
        bottom: 0,
        left: 0,
        right: 0,
        height: '40vh',
        padding: '10px',
        backgroundColor: '#fff',
        borderRadius: '10px 10px 0 0',
        boxShadow: '0 -4px 4px rgba(0, 0, 0, 0.1)',
        color: '#000',
        overflow: 'hidden',
        fontSize: fontSize + 'px',
      }
    },
    buttonWrapStyles: function () {
      return this.options.buttonWrapStyles || {
        display: 'grid',
        width: '100%',
        height: '100%',
        gridTemplateColumns: `repeat(${this.columns}, 1fr)`,
        gridTemplateRows: `repeat(${this.rows}, 1fr)`,
        gridGap: '5px',
      }
    },
    buttonStyles: function () {
      return this.options.buttonStyles || {
        display: 'flex',
        justifyContent: 'center',
        alignItems: 'center',
        backgroundColor: '#fff',
        border: '1px solid #000',
        borderRadius: '5px',
        fontSize: 'inherit',
      }
    },
    deleteButtonStyles: function () {
      return this.options.deleteButtonStyles || this.options.buttonStyles || this.buttonStyles;
    },
    blankButtonStyles: function () {
      return this.options.blankButtonStyles || this.options.buttonStyles || this.buttonStyles;
    },
  },
  methods: {
    click(key) {
      var newVal = "";
      if (key === -1) {
        newVal = this.value.slice(0, -1);
      } else {
        newVal = this.value + key;
      }
      this.$emit("update:value", String(newVal));
      if (this.keyRandomize && this.randomizeWhenClick) {
        this.randomize(this.fixDeleteKey);
      }
    },
    randomize(fixDeleteKey) {
      var newKeyArray = [];
      for (var i = 0; i < this.keyArray.length; i++) {
        var r = Math.random();
        if (fixDeleteKey && this.keyArray[i] == -1) continue;
        if (r < 0.5) newKeyArray.push(this.keyArray[i]);
        else newKeyArray.unshift(this.keyArray[i]);
      }
      if (fixDeleteKey) newKeyArray.push(-1);
      this.keyArray = newKeyArray;
    },
    showKey(key) {
      if (key === -1) {
        return "del";
      } else return key;
    },
    btnStyle(key) {
      if (key === -1) {
        return this.deleteButtonStyles;
      } else if (typeof key === 'number') {
        return this.buttonStyles;
      } else return this.blankButtonStyles;
    },
    resize() {
      this.cellWidth = this.$refs.button[0].offsetWidth;
      this.cellHeight = this.$refs.button[0].offsetHeight;
    }
  },
  mounted() {
    window.addEventListener('resize', this.resize);
    this.cellWidth = this.$refs.button[0].offsetWidth;
    this.cellHeight = this.$refs.button[0].offsetHeight;
    if (this.keyRandomize) {
      this.randomize(this.fixDeleteKey);
    }
  },
  beforeDestroy() {
    window.removeEventListener('resize', this.resize);
  }
};
</script>