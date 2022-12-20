<template>
	<view class="numBox" :style="{color}">
		<view class="reduce" @click="change('-')" :class="{ disable: inputValue == minValue }">-</view>
		<input type="number" :maxlength="-1" :value="inputValue" @input="input"/>
		<view class="add" @click="change('+')" :class="{ disable: inputValue ==  maxValue }">+</view>
	</view>
</template>

<script setup>
	/**
	 * numBox 步进器
	 * @description 步进器，一般用于选择商品数量的场景。
	 * @property {String} width 宽度(默认值: 180rpx)。
	 * @property {String} height 高度(默认值: 64rpx)。
	 * @property {Nunber} maxValue 最大值(默认值: -1), -1则为无限大
	 * @property {Nunber} minValue 最小值(默认值: 1)
	 * @property {Nunber} modelValue 值
	 * @property {Boolean} integer 是否只可输入整数(默认值: false)
	 * @property {Number} step 每次步进的范围(默认值: 1)
	 * @property {Number} decimalLength 显示的小数位数(默认值: -1)，-1代表不限制小数位数
	 * @property {String} bgColor 步进器背景色(默认值: #EBECEE)
	 * @property {String} color 文字颜色(默认值: #323233)
	 * @property {String} disabledBgColor 按钮禁用背景色(默认值: #F7F8Fa)
	 * @property {String} disabledColor 按钮禁用的文字颜色(默认值: #C8C9CC)
	 * @property {String} borderColor 边框背景色(默认值:EEEFF1)
	 * @property {String} size -、+大小(默认值: 45rpx)
	 * @event {Function} change 数量发生变化时触发。
	 * @event {Function} input 输入数量时发生触发。
	 * @example <numBox :modelValue="1"></numBox>
	 */
	import {
		ref,
		reactive,
		inject,
		toRefs,
		watch,
		onMounted,
		nextTick
	} from "vue";
	import {
		onLoad
	} from '@dcloudio/uni-app';
	// 获取全局对象
	const global = inject('global');
	// 解构需要使用的部分
	const {
		$Common,
		$api
	} = global;

	let props = defineProps({
		// 宽
		width: {
			default: "180rpx",
		},
		// 高
		height: {
			default: "64rpx",
		},
		// 最大值为-1则可最大值为无限
		maxValue: {
			default: -1,
		},
		// 最小值
		minValue: {
			default: 1,
		},
		// 值
		modelValue: {
			default: 1
		},
		// 是否只可输入整数类型
		integer: {
			default: false,
			type: Boolean
		},
		// 步进
		step: {
			default: 1,
			type: Number
		},
		// 显示的小数位数 -1代表不限制小数位数
		decimalLength: {
			default: -1,
			type: Number
		},
		// 步进器背景色
		bgColor: {
			default: '#EBECEE',
			type: String
		},
		// 文字颜色
		color: {
			default: "#323233",
			type: String
		},
		// 按钮禁用背景色
		disabledBgColor: {
			default: '#F7F8Fa',
			type: String
		},
		// 按钮禁用文字颜色
		disabledColor: {
			default: '#C8C9CC',
			type: String
		},
		// +、-大小
		size: {
			default: "45rpx",
			type: String
		}
	})
	let emit = defineEmits(['update:modelValue', 'max', 'min', 'change', 'input']);
	
	let inputValue = ref(null);
	
	onMounted(()=>{
		inputValue.value = props.modelValue;
	})

	// 更改数量
	let change = (type) => {
		// 看步进是否有小数
		let valueArr = (props.step + '').split('.');
		// 保留小数长度
		let decimal = props.decimalLength != -1 ? props.decimalLength : valueArr.length > 1 ? valueArr[1].length : 0;
		// 值
		let value = inputValue.value * 1;
		if (type == "+") {
			if (props.maxValue == -1 || inputValue.value < props.maxValue) {
				value += props.step;
				inputValue.value = value.toFixed(decimal);
			}

		} else if (type == "-" && inputValue.value > props.minValue) {
			value -= props.step;
			inputValue.value = value.toFixed(decimal);
		}
		emit('update:modelValue', inputValue.value);
		emit('change', inputValue.value);
	}

	// 输入事件
	let input = (e)=>{
		inputValue.value = e.detail.value;
		correctingNum(inputValue.value);
		setTimeout(()=>{
			emit('update:modelValue', inputValue.value);
			emit('input', inputValue.value);
		}, 10)
	}
	
	// 校正数量
	let correctingNum = (value)=>{
		// 看步进是否有小数
		let valueArr = (props.step + '').split('.');
		// 保留小数长度
		let decimal = props.decimalLength != -1 ? props.decimalLength : valueArr.length > 1 ? valueArr[1].length : 0;
		if (value < props.minValue) {
			nextTick(()=>{
				inputValue.value = props.minValue.toFixed(decimal);
			})
		} else if (props.maxValue !== -1 && value > props.maxValue) {
			nextTick(()=>{
				inputValue.value = props.maxValue.toFixed(decimal);
			})
		} else if(props.integer){
			// // 检测是否包含.
			let index = (value + '').indexOf('.');
			if(index != -1) {
				nextTick(()=>{
					inputValue.value = value.slice(0, index);
				})
			}
		}
	}
	
	watch(()=>props.modelValue, v1=>{
		inputValue.value = v1;
		correctingNum(inputValue.value);
	})

	watch(()=>props.maxValue, ()=>{
		correctingNum(inputValue.value);
	})
	
	watch(()=>props.minValue, ()=>{
		correctingNum(inputValue.value);
	})
</script>

<style lang="less" scoped>
	.numBox {
		width: v-bind(width);
		height: v-bind(height);
		border-radius: 10rpx;
		overflow: hidden;
		display: flex;
		align-content: center;
		text-align: center;
		font-size: v-bind(size);

		.reduce,
		.add {
			width: 33%;
			line-height: calc(v-bind(height) * 0.95);
			background-color: v-bind(bgColor);

			&.disable {
				color: v-bind(disabledColor);
				background-color: v-bind(disabledBgColor);
			}
		}

		input {
			background-color: v-bind(bgColor);
			height: 100%;
			width: 33%;
			border-left: 3rpx solid #EEEFF1;
			border-right: 3rpx solid #EEEFF1;
			margin: 0 5rpx;
		}
	}
</style>
