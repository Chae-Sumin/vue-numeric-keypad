
<template>
	<div id="app" :style="styles.app">
		<div class="box" :style="styles.box">
			<h2 :style="styles.title">Test Options</h2>
			<div :style="styles.inner">
				<input
					type="number"
					:value="demo1.value"
					@click.stop="demo1.show = !(demo2.show = false)"
					placeholder="Click for test"
					readonly
				/>
				<VueNumericKeypad
					:value.sync="demo1.value"
					:show.sync="demo1.show"
					:options="demo1.options"
				/>
			</div>
			<div class="options" :style="styles.inner">
				<input type="checkbox" name="keyRandomize" id="keyRandomize" v-model="demo1.options.keyRandomize">&nbsp;
				<label for="keyRandomize">keyRandomize</label>
			</div>
			<div class="options" :style="styles.inner">
				<input type="checkbox" name="randomizeWhenClick" id="randomizeWhenClick" v-model="demo1.options.randomizeWhenClick">&nbsp;
				<label for="randomizeWhenClick">randomizeWhenClick</label>
			</div>
			<div class="options" :style="styles.inner">
				<input type="checkbox" name="fixDeleteKey" id="fixDeleteKey" v-model="demo1.options.fixDeleteKey">&nbsp;
				<label for="fixDeleteKey">fixDeleteKey</label>
			</div>
			<div class="options" :style="styles.inner">
				<input type="checkbox" name="pseudoClick" id="pseudoClick" v-model="demo1.options.pseudoClick">&nbsp;
				<label for="pseudoClick">pseudoClick</label>
			</div>
			<div class="options" :style="styles.inner">
				<input type="checkbox" name="pseudoClickDeleteKey" id="pseudoClickDeleteKey" v-model="demo1.options.pseudoClickDeleteKey">&nbsp;
				<label for="pseudoClickDeleteKey">pseudoClickDeleteKey</label>
			</div>
			<div class="options" :style="styles.inner">
				<input type="checkbox" name="pseudoClickBlankKey" id="pseudoClickBlankKey" v-model="demo1.options.pseudoClickBlankKey">&nbsp;
				<label for="pseudoClickBlankKey">pseudoClickBlankKey</label>
			</div>
			<div class="options" :style="styles.inner">
				<input type="checkbox" name="stopPropagation" id="stopPropagation" v-model="demo1.options.stopPropagation">&nbsp;
				<label for="stopPropagation">stopPropagation</label>
			</div>
			<div class="options" :style="styles.inner">
				<input type="checkbox" name="vibrate" id="vibrate" v-model="demo1.options.vibrate">&nbsp;
				<label for="vibrate">vibrate</label>
			</div>
			<div class="options" :style="styles.inner">
				<input type="number" name="activeButtonDelay" id="activeButtonDelay" v-model="demo1.options.activeButtonDelay" style="width: 50px;">&nbsp;
				<label for="activeButtonDelay">activeButtonDelay</label>
			</div>
			<div class="options" :style="styles.inner">
				<input type="number" name="vibratePattern" id="vibratePattern" v-model="demo1.options.vibratePattern" style="width: 50px;">&nbsp;
				<label for="vibratePattern">vibratePattern</label>
			</div>
			<div class="options" :style="styles.inner">
				<input type="text" name="deleteKeyText" id="deleteKeyText" v-model="demo1.options.deleteKeyText" style="width: 50px;">&nbsp;
				<label for="deleteKeyText">deleteKeyText</label>
			</div>
			<div class="options" :style="styles.inner">
				<input type="text" name="clearKeyText" id="clearKeyText" v-model="demo1.options.clearKeyText" style="width: 50px;">&nbsp;
				<label for="clearKeyText">clearKeyText</label>
			</div>
		</div>
		<div class="box" :style="styles.box">
			<h2 :style="styles.title">Test Encript</h2>
			<div :style="styles.inner">
				Used the encryption function from <a href="http://travistidwell.com/jsencrypt/">http://travistidwell.com/jsencrypt/</a>.
			</div>
			<div :style="styles.inner">
				<input
					type="text"
					:value="demo2.value"
					@click.stop="demo2.show = !(demo1.show = false)"
					placeholder="Click for test"
					readonly
				/>
				<VueNumericKeypad
					:value.sync="demo2.value"
					:encryptedValue.sync="demo2.encryptedValue"
					:show.sync="demo2.show"
					:options="demo2.options"
					:encryptFunc="demo2.encryptFunc"
				/>
			</div>
			<div :style="styles.inner">
				<div v-for="(v, i) in demo2.encryptedValue">{{i + 1}}: <input :value="v" readonly @click="e => copyText(e)" :style="styles.encInput">&ensp;<span class="copy">&lt;- Click for copy</span></div>
			</div>
			<div :style="styles.inner">
				<h3 :style="styles.subtitle">Example public key&ensp;<a href="javascript:;" @click="demo2.toggle1 = !demo2.toggle1" :style="styles.button">{{demo2.toggle1 ? 'hide' : 'show'}}</a></h3>
				<span v-show="demo2.toggle1">{{pubKey}}</span>
			</div>
			<div :style="styles.inner">
				<h3 :style="styles.subtitle">Example private key&ensp;<a href="javascript:;" @click="demo2.toggle2 = !demo2.toggle2" :style="styles.button">{{demo2.toggle2 ? 'hide' : 'show'}}</a></h3>
				<span v-show="demo2.toggle2">*** This is part of the demo. In actual use, the private key should not be exposed. ***<br>{{priKey}}</span>
			</div>
			<br>
			<div :style="styles.inner">
				<h3 :style="styles.subtitle">Test decrypt&ensp;<a href="javascript:;" @click="demo2.encValue = ''" :style="styles.button">Clear</a></h3>
				<input type="text" placeholder="Paste the encrypted string" v-model="demo2.encValue" :style="styles.encInput"> decrypted number is {{decryptedValue || '?'}}
			</div>
		</div>
		&emsp;<a href="https://github.com/Chae-Sumin/vue-numeric-keypad/blob/master/dev/demo.vue" target="_blank" rel="noopener noreferrer">Get Demo Codes</a>
	</div>
</template>
<script>
import VueNumericKeypad from "../src/vue-numeric-keypad";
import JSEncrypt from "jsencrypt";

const pubKey = 'MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDZppsGrlhwX14a05LraK1hgF7HFz2VTZQWhmHwpbc1S6Ui4nFhJWF7AAtehV+bUdapdBY2WCkcccsB0xR4BfKq6IFzXGfDapF/MKXn3Z0VwLITnhV17G4rQk+KYGKoDrZdyfR3qhrBOu03HLJ3jrCWiHwBahxKbQjnYqv0Lfl5KQIDAQAB';
// This is part of the demo. In actual use, the private key should not be exposed.
const priKey = 'MIICXQIBAAKBgQDZppsGrlhwX14a05LraK1hgF7HFz2VTZQWhmHwpbc1S6Ui4nFhJWF7AAtehV+bUdapdBY2WCkcccsB0xR4BfKq6IFzXGfDapF/MKXn3Z0VwLITnhV17G4rQk+KYGKoDrZdyfR3qhrBOu03HLJ3jrCWiHwBahxKbQjnYqv0Lfl5KQIDAQABAoGAKt1YDMHLYx41H11pwvkTpG8uu1EFbOEheCoxIb7RTSq/tBYatEIzZ5EDrDLiOGmuuCLHuR41wodartpOXrD1MHxzKXT/uzfavmtpZLwony2B23ZBcIaUqmlI+WZHZ1w3fdRts/n3WgMUF1+f1M22HPm/scm/v78QP0y8fZU3rh0CQQD9OARUSo9S3Blcfzrw64RRn371JuVz51rAe0fLITRUQ8n4EsgpuM6pslir6XZ8mDrT4468uDvDzYwdh11q9Nz3AkEA3AqUu7qVYzfXg+fWfHHfsnLbRHQr/Yfs5A9uh9TyQhi6Qpa5XZtUCzpiSLxhkri5U1rfOJiv+e/1o9XmezFy3wJAC00tvElbnjoek6dGDSylyjLRKsXipcqknUSjTqibuksQP5cvAdWyu5YvKPURibwNnBli7H9Yg4OwBj1daQGmvwJBALVcAzq3jmk4nWkarK3lLXrnL9I77gYJAjb2gSNzYDkaKGq50A5W9+5JMLjCi6lil10ciN8c+e4G2W8v3cer+gECQQDTCXNbqT/lIELhvCb4w+yitDBzxZRluvDKs18h4oCsSx0x6eKjMLL8SRa086JSZDzexMuSAQjsJRWLZJRuRo1h';
const styles = {
	app: {
		display: 'flex',
		flexWrap: 'wrap',
		alignContent: 'flex-start',
		width: '100%',
		paddingBottom: '500px',
		wordBreak: 'break-all',
	},
	box: {
		width: '100%',
		height: 'fit-content',
		margin: '8px',
		padding: '10px',
		border: '1px solid #222',
		borderRadius: '8px',
		backgroundColor: '#f4f4f4',
	},
	inner: {
		margin: '0 0 4px',
	},
	title: {
		margin: '0 0 10px',
	},
	subtitle: {
		margin: '0 0 4px',
	},
	button: {
		display: 'inline-block',
		minWidth: '35px',
		padding: '2px 4px',
		border: '1px solid #666',
		borderRadius: '4px',
		color: '#222',
		textDecoration: 'none',
		fontSize: '12px',
		textAlign: 'center',
		verticalAlign: 'middle'
	},
	encInput: {
		width: '160px',
	}
};
export default {
	name: "App",
	components: {
		VueNumericKeypad,
	},
	data() {
		return {
			pubKey,
			priKey, // FOR DEMO
			styles,
			codeToggle: false,
			demo1: {
				value: "",
				show: 0,
				options: {
					keyRandomize: true,
					randomizeWhenClick: false,
					fixDeleteKey: true,
					pseudoClick: false,
					pseudoClickDeleteKey: false,
					pseudoClickBlankKey: false,
					stopPropagation: true,
					vibrate: false,
					activeButtonDelay: 300,
					vibratePattern: 200,
					keyArray: [1, 2, 3, 4, 5, 6, 7, 8, 9, 0, -1, -2],
					deleteKeyText: 'del',
					clearKeyText: 'clr',
				},
			},
			demo2: {
				value: "",
				show: 0,
				encryptedValue: [],
				encryptFunc: s => this.demo2.crypt.encrypt(s),
				options: {
					encrypt: true,
					encryptedChar: '@',
					pseudoClick: true,
					vibrate: true,
					keyArray: [1, 2, 3, 4, 5, 6, 7, 8, 9, 0, -1, -2],
				},
				crypt: new JSEncrypt(),
				// FOR DEMO
				encValue: '',
				decrypt: new JSEncrypt(),
				toggle1: false,
				toggle2: false,
			},
		};
	},
	computed: {
		decryptedValue: function () {
			const o = this.demo2;
			return o.decrypt.decrypt(o.encValue);
		},
	},
	methods: {
		copyText: e => {
			e.target.setSelectionRange(0, 999999);
			if (typeof document.execCommand === 'function') {
				document.execCommand('copy');
				e.target.nextElementSibling.innerText = 'Copyed!';
			}
		},
	},
	created() {
		this.demo2.crypt.setKey(pubKey);
		this.demo2.decrypt.setKey(priKey);
		document.addEventListener('click', function () {
			this.demo1.show = false;
			this.demo2.show = false;
		}.bind(this));
	}
};
</script>
