<script>
import Vue from 'vue';
import VueNumericKeypad from '@/vue-numeric-keypad.vue';


export default {
  name: "ServeDev",
  components: {
    VueNumericKeypad,
  },
  data() {
    return {
      values: ["", ""],
      inputValue: "",
      divValue: "",
      hiddenValue: "",
      showKeypad1: 0,
      showKeypad2: false,
      showKeypad3: false,
      showKeypad4: 1,
      valuesIndex: 0,
      options: {
        keyRandomize: true,
        randomizeWhenClick: true,
        fixDeleteKey: true,
      },
      keypad2Options: {
        keyRandomize: false,
        randomizeWhenClick: true, //keyRandomize is false, so it's ignored.
        buttonStyles: {
          display: 'flex',
          justifyContent: 'center',
          alignItems: 'center',
          backgroundColor: '#faa',
          border: 'none',
          borderRadius: '10px',
          fontSize: 'inherit',
        }
      },
      keypad3Options: {
        randomizeWhenClick: true,
        columns: 4,
        rows: 3,
      },
      keypad4Options: {
        fixDeleteKey: false,
        keypadClass: 'keypad',
        buttonWrapClass: 'button-wrap',
        buttonClass: 'button',
        deleteButtonClass: 'delete-button',
        blankButtonClass: 'blank-button',
        keypadStyles: {},
        buttonWrapStyles: {},
        buttonStyles: {},
      },
    };
  },
  methods: {
    openKeypad1: function (valuesIndex) {
      this.showKeypad1++;
      this.showKeypad2 = false;
      this.showKeypad3 = false;
      this.valuesIndex = valuesIndex;
    },
    openKeypad2: function () {
      this.showKeypad1 = 0;
      this.showKeypad2 = true;
      this.showKeypad3 = false;
    },
    openKeypad3: function () {
      this.showKeypad1 = 0;
      this.showKeypad2 = false;
      this.showKeypad3 = true;
    },
    changeKeypad4: function () {
      this.showKeypad1 = 0;
      this.showKeypad2 = false;
      this.showKeypad3 = false;
      this.showKeypad4++;
    },
  },
  created() {
    document.addEventListener('click', function () {
      this.showKeypad1 = 0;
      this.showKeypad2 = false;
      this.showKeypad3 = false;
    }.bind(this));
  }
};
</script>

<template>
  <div id="app">
    <div>
      <h2>Binding multiple element to one keypad.</h2>
      <p>The options for keypad 1 are default.</p>
      <!--Input 1 bound to keypad 1-->
      <label>input: number&emsp;</label>
      <input
        type="number"
        name="input1"
        readonly
        @click.stop="openKeypad1(0)"
        :value="values[0]"
      />
      <br>
      <!--Input 2 bound to keypad 1-->
      <label>input: password&nbsp;</label>
      <input
        type="password"
        name="input2"
        readonly
        @click.stop="openKeypad1(1)"
        :value="values[1]"
      />
      <!--keypad 1-->
      <VueNumericKeypad
        :value.sync="values[valuesIndex]"
        :show.sync="showKeypad1"
      />
    </div>
    <div>
      <h2>Binding one element per keypad.</h2>
      <!--Input 3 bound to keypad 2-->
      <label>keypad 2 that's not randomized.&ensp;</label>
      <input
        type="number"
        name="input3"
        readonly
        @click.stop="openKeypad2()"
        :value="inputValue"
      />
      <!--keypad 2-->
      <VueNumericKeypad
        :value.sync="inputValue"
        :show.sync="showKeypad2"
        :options="keypad2Options"
      />
      <br><br>
      <!--div element bound to keypad 3-->
      <label>Keypad 3 is columns 4, rows 3. Randomize every time you press a button.</label>
      <div 
        class="div_element"  
        @click.stop="openKeypad3()"
        style="width: 120px; height: 18px; border: 1px solid #0e0; white-space: nowrap; overflow: hidden;"
      >
      {{divValue}}
      </div>
      <br>
      <!--keypad 3-->
      <VueNumericKeypad
        :value.sync="divValue"
        :show.sync="showKeypad3"
        :options="keypad3Options"
      />
    </div>
    <div>
      <h2>The keypad that's always exposed.</h2>
      <!--hidden value bound to keypad 4-->
      <!-- Non-essential  <input type="hidden" name="input_hidden" :value="hiddenValue" /> -->
      <label>Keypad 4 applied style to class and arranged delelte keys randomly.</label>
      <br>
      <button type="button" @click.stop="changeKeypad4()">
      Changing the arrangement of keypad 4
      </button>
      <span>&emsp;Input length: {{hiddenValue.length}}</span>
      <!--keypad 4-->
      <VueNumericKeypad
        :value.sync="hiddenValue"
        :show.sync="showKeypad4"
        :options="keypad4Options"
      />
    </div>
  </div>
</template>

<style>
.keypad {
  width: 100%;
  max-width: 360px;
  padding: 5px;
  margin-top: 30px;
  background-color: #eaeaea;
}
.button-wrap {
  display: flex;
  flex-wrap: wrap;
}
.button {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 50px;
  height: 50px;
  margin: 5px;
  border: none;
  background: rgba(0, 100, 0, 0.5);
  font-size: 20px;
}
.delete-button {
  font-size: 0;
  text-indent: -999px;
  background: rgba(0, 100, 0, 0.5) url('assets/delete.png') 50% 50% no-repeat;
}
.blank-button {
  background: rgba(0, 100, 0, 0.5) url('assets/logo.png') 50% 50% / 30px auto no-repeat;
}
</style>
