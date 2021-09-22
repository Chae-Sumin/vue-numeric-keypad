# Vue Numeric Keypad

The virtual numeric keypad that can be used on Vue. 
It was produced in NPM v6.14.13 and Vue v2.6.14 environments.
By default, keys are randomly arranged and you can specify various settings using options.

## Installation

```bash
npm i vue-numeric-keypad
```

## Usage

Simple Usage
```html
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
import  VueNumericKeypad  from "vue-numeric-keypad";

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
```

## Demo

https://chae-sumin.github.io/vue-numeric-keypad/

## Props and Options

The props have to deliver changing values or objects, so bind them with `v-bind:` or `:`. 
In addition, `value` and `show` require two-way binding, so add the `.sync` modifier.
### props
|property|Description|Required|type|
|-|-|-|-|
|`id`|ID of the keypad.|False|String|
|`:value.sync`|The value to change when entering the keypad.|True|String variable|
|`:show.sync`|Bind to the v-show on the keypad. And adjust the font size and randomize the key arrangement.|True|Boolean or Number variable|
|`:options`|Set several values.<br>(details can be found below)|False|Object|
### options
|option|Description|type|default|
|-|-|-|-|
|`keyRandomize`|If the value is true, randomize the key array whenever the `show` prop changes.|Boolean|True|
|`randomizeWhenClick`|If the value is true and `keyRandomize` is true, randomize the key array whenever you press the key.|Boolean|False|
|`fixDeleteKey`|If the value is true, the delete key is fixed at the end when the key array is randomized.|Boolean|True|
|`keypadStyles`|Set the style of the keypad.|Object<br>(Style Object)|Check below.|
|`buttonWrapStyles`|Set the style of wrapping the button.|Object<br>(Style Object)|Check below.|
|`buttonStyles`|Set the style of the button.|Object<br>(Style Object)|Check below.|
|`deleteButtonStyles`|Set the style of the delete button.|Object<br>(Style Object)|`buttonStyles`|
|`blankButtonStyles`|Set the style of the blank button.|Object<br>(Style Object)|`buttonStyles`|
|`zIndex`|Sets the z-index value.<br>Valid when `keypadStyles` is the default.|Number|1|
|`rows`|Sets the number of rows in the key array.<br>Valid when `buttonWrapStyles` is the default.|Number|4|
|`columns`|Sets the number of columns in the key array.<br>Valid when `buttonWrapStyles` is the default.|Number|3|
#### styles defaults
`keypadStyles` :
```js
const  fontSize = Math.min(this.cellWidth, this.cellHeight) * 0.3;
{
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
	fontSize: fontSize + 'px'
}
```
`buttonWrapStyles` :
```js
// columns = button columns
// rows = button rows
{
	display: 'grid',
	width: '100%',
	height: '100%',
	gridTemplateColumns: `repeat(${columns}, 1fr)`,
	gridTemplateRows: `repeat(${rows}, 1fr)`,
	gridGap: '5px'
}
```
`buttonStyles` : 
```js
{
	display: 'flex',
	justifyContent: 'center',
	alignItems: 'center',
	backgroundColor: '#fff',
	border: '1px solid #000',
	borderRadius: '5px',
	fontSize: 'inherit'
}
```
## Tip

 - You can bind only one value to one keypad, or you can bind multiple values.
 - You can always keep the value of the show true and adjust the position of the keypad so that the keypad can always be located in a fixed layout.
 - You don't have to always bind the visible value.
 -  If you want to customize a style using css, initialize the style by putting an empty object in the style-related option.
 
## License
[MIT license](https://tldrlegal.com/license/mit-license).

## Contact

cotnmin@gmail.com