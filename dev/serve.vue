
<template>
	<div id="app">
		<input
			type="number"
			:value="value"
			@click.stop="show = true"
			readonly
		/>
		<VueNumericKeypad
			:value.sync="value"
			:show.sync="show"
			:options="options"
		/>
	</div>
</template>

<script>
import  VueNumericKeypad  from "@/vue-numeric-keypad";

export  default {
	name:  "App",
	components: {
		VueNumericKeypad,
	},
	data() {
		return {
			value:  "",
			show:  0,
			options: {
				keyRandomize:  true,
				randomizeClick:  true,
				fixDeleteKey:  false,
			},
		};
	},
	created() {
		document.addEventListener('click', function () {
			this.show = 0;
		}.bind(this));
	}
};
</script>
