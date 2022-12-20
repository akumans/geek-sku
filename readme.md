# geek-sku
一款轻量化、强大、简洁、可以根据主题色属性自动生成相应主题色的v3版本的商品多规格sku，仅需要按照指定格式传入sku数组便可以直接使用。

## 使用方法
从插件时长直接使用HBuilderX 导入插件复制下方的示例即可直接快速使用，然后再根据业务需求进行调整即可。

```html
	<template>
		<view class="test">
			<geek-sku
				v-model="skuShow"
				:data="skus"
				defaultTitle="iPhone14 Pro"
				defaultCover="https://inews.gtimg.com/newsapp_bt/0/15259986145/1000"
				btnConfirmText="购买"
				notSelectSku="请选择完整的商品信息"
				@skuChange="skuChange"
				@confirm="skuConfirm"
			></geek-sku>
			<!-- 打开sku组件 -->
			<button @click="skuShow = true">打开sku组件</button>
		</view>
	</template>
	
	<script setup>
		import { ref, reactive, inject, toRefs } from "vue";
		import { onLoad } from '@dcloudio/uni-app';
		
		let skuShow = ref(false);
		// sku列表
		let skus = ref([
			{
				id: 1,
				price: 7500,
				stock: 30,
				sku_attrs: {
					'机身颜色': '暗紫色',
					'储存容量': '128G',
				}
			},
			{
				id: 2,
				price: 8500,
				stock: 10,
				sku_attrs: {
					'机身颜色': '暗紫色',
					'储存容量': '256G',
				}
			},
			{
				id: 3,
				price: 9500,
				stock: 0,
				sku_attrs: {
					'机身颜色': '暗紫色',
					'储存容量': '512G',
				}
			},
			{
				id: 4,
				price: 9200,
				stock: 60,
				sku_attrs: {
					'机身颜色': '银色',
					'储存容量': '512G',
				}
			},
			{
				id: 5,
				price: 9200,
				stock: 80,
				sku_attrs: {
					'机身颜色': '金色',
					'储存容量': '512G',
				}
			}
		])
		
		// sku发生改变事件, 选择完整的sku则回返回 否则sku的值为{}
		let skuChange = (sku)=>{
			console.log(sku);
		}
		// sku确认事件
		let skuConfirm = (e)=>{
			console.log(e);
		}
	</script>
	
	<style lang="less" scoped>
		.test {
			padding-top: 3rem;
		}
	</style>
```
在sku列表数组中的每一项都可以有title、logo属性，这样在选择完整的sku后就会自动展示本项sku的title、logo。

例如: 
``` javascript
	[
		...
		{
			id: 5,
			price: 9200,
			title: 'iphone14 pro max 金色 512G',
			logo: '图片路径',
			stock: 80,
			sku_attrs: {
				'机身颜色': '金色',
				'储存容量': '512G',
			}
		}
		...
	]
```

## API

### Props
| 参数 | 说明 | 类型 | 默认值 | 可选值 |
| --- | --- | -- | -- | -- |
| data | 源数据(sku列表) | Array | [] | - |
| modelValue | 是否显示sku组件 | Boolean | false | true |
| isMaskClose | 是否可以点击遮罩层关闭 | Boolean | false | true |
| isSelectMinPriceSku | 是否默认选中最低价格的sku | Boolean | true | false |
| defaultTitle |  默认标题，用于没有选中完整的sku时展示 | String | '商品' | - |
| defaultCover | 默认封面图，用于没有选中完整的sku时展示 | String | '' | - |
| themeColor | 主题色，需要传入一个数组长度为3的数组，分别对应rgb三个颜色的值，例如: [84, 164, 255] | Array | [84, 164, 255] | - |
| btnConfirmText | 确认按钮文字 | String | '确认' | - |
| notStockText | 库存不足文字 | String | '库存不足' | - |
| notSelectSku | 未选择完整的sku时的文字提示 | String | '请选择完整的sku属性' | - |

### Events
| 事件名 | 说明 | 回调参数 |
| --- | --- | -- |
| confirm | 点击确认按钮时触发的事件 | e = { sku, skuText , num }。分别对应选中的sku值 、sku属性名数组 、输入框内的数量。 |
| skuChange | sku发生变化时出发的事件，如果有选中完整的sku则会返回该sku，否则会返回{} | e = {} |
| numChange | 输入框内数量发生改变时触发事件 | e = num, 返回输入框数量 |
| close | 关闭sku组件触发事件 | - |
| open | 打开sku组件触发事件 | - |
