<template>
	<view class="sku" v-if="isV3 ? modelValue : value" @click="isMaskClose ? close : ''">
		<view class="shopSpecsPopup">
			<image class="close" src="../../img/close.png" @click="close"></image>
			<view class="content">
				<view class="info flex">
					<view class="cover" :style="{backgroundImage: `url(${selectSku.logo || defaultCover})`}"></view>
					<view class="right">
						<view class="title ellipsis t-w">
							{{selectSku.title || defaultTitle}}
						</view>
						<view class="price flex f-y-c" :style="{color: themeRGB}">
							<view class="uity">￥</view>{{selectSku.id ? selectSku.price : `${showAreaPrice[0]}-${showAreaPrice[1]}`}}
						</view>
						<view class="stock">
							库存: {{selectSku.id ? selectSku.stock : `${showAreaStock[0]}-${showAreaStock[1]}`}}
						</view>
					</view>
				</view>
				<view class="number flex f-y-c f-x-b">
					<view class="key">数量</view>
					<numBox :width="200" integer v-model="num" :minValue="selectSku.stock == 0 ? 0 : 1" :maxValue="selectSku.id ? selectSku.stock : showAreaStock[1]"></numBox>
				</view>
				<view class="specsList">
					<view class="item" v-for="(skuArr, skuArrKey) in r.result" :key="'skuArrKey' + skuArrKey">
						<view class="title">{{skuArrKey}}</view>
						<view class="specsValueList flex f-w">
							<template v-for="(sku, skuKey) in skuArr">
								<view class="specs act" v-if="sku.active"
									:key="'sku' + skuKey"
									:style="{
										backgroundColor: themeRGBA,
										color: themeRGB,
										border: `2rpx dashed ${themeRGB}`
									}"
									@click="bindEvent(sku, skuArr, skuArrKey)"
								>
									{{sku.value}}
								</view>
								<view class="specs" v-else
									:key="'sku' + skuKey"
									:class="{ disabled: sku.disabled }"
									@click="bindEvent(sku, skuArr, skuArrKey)"
								>
									{{sku.value}}
								</view>
							</template>
						</view>
					</view>
				</view>
			</view>
			<view class="btnBox">
				<view
					class="confirm" 
					:class="{
						disabled: selectSku.id && selectSku.stock * 1 < 1
					}" 
					:style="{
						background: themeRGB
					}"
					@click="confirm"
				>
					{{ selectSku.id && selectSku.stock * 1 < 1 ? notStockText : btnConfirmText}}
				</view>
			</view>
		</view>
	</view>
</template>

<script>
	/**
	 * geek-sku
	 * @description 商品sku组件。
	 * @tutorial https://ext.dcloud.net.cn/plugin?id=10470
	 * @property {Array} data 源数据。
	 * @property {Boolean} modelValue 是否显示sku组件(默认值: false, v3显示)。
	 * @property {Boolean} value 是否显示sku组件(默认值: false, v2显示)。
	 * @property {Boolean} isMaskClose 是否可以点击遮罩层关闭(默认值: false)。
	 * @property {Boolean} isSelectMinPriceSku 是否默认选中最低价格的sku(默认值: true)。
	 * @property {String} defaultTitle 默认标题，用于没有选中完整的sku时展示(默认值: '商品')。
	 * @property {String} defaultCover 默认封面图，用于没有选中完整的sku时展示。
	 * @property {Array} themeColor 主题色，需要传入一个数组长度为3的数组，分别对应rgb三个颜色的值，例如: [84, 164, 255]。
	 * @property {String} btnConfirmText 确认按钮文字(默认值: '确认')。
	 * @property {String} notStockText 库存不足文字(默认值: '库存不足')。
	 * @property {String} notSelectSku 未选择完整的sku时的文字提示(默认值: '请选择完整的sku属性')。
	 * @event {Function} confirm 点击确认按钮时触发的事件，会返回e， e = { sku, skuText , num }，分别对应选中的sku值 、sku属性名 、输入框内的数量。
	 * @event {Function} skuChange sku发生变化时出发的事件，如果有选中完整的sku则会返回该sku，否则会返回{}。
	 * @event {Function} close 关闭sku组件触发事件。
	 * @event {Function} open 打开sku组件触发事件。
	 * @event {Function} numChange 输入框内数量发生改变时触发事件。
	 * @example <geek-sku 
					v-model="skuShow" 
					:data="skus"
					:defaultTitle="title" 
					:defaultCover="logo"
					@skuChange="skuChange"
					@confirm="buyShop"
				>
				</geek-sku>
	 */
	export default {
		props: {
			// 源数据
			data: {
				default: [],
				type: Array
			},
			// 是否显示sku组件
			// #ifdef VUE3
			modelValue: {
				default: false,
				type: Boolean
			},
			// #endif
			// #ifndef VUE3
			value: {
				default: false,
				type: Boolean
			},
			// #endif
			// 是否可以点击关闭
			isMaskClose: {
				default: false,
				type: Boolean
			},
			// 是否默认选中最低价格sku
			isSelectMinPriceSku: {
				default: true,
				type: Boolean
			},
			// 默认标题
			defaultTitle: {
				default: "商品标题",
				type: String
			},
			// 默认封面图
			defaultCover: {
				default: "",
				type: String
			},
			// 主题色
			themeColor: {
				default: ()=>[84, 164, 255],
				type: Array
			},
			// 确认按钮文字
			btnConfirmText: {
				default: "确定",
				type: String
			},
			// 库存不足文字
			notStockText: {
				default: "库存不足",
				type: String
			},
			// 未选择完整的sku时的文字提示
			notSelectSku: {
				default: "请选择完整的sku属性",
				type: String
			}
		},
		// data() 返回的属性将会成为响应式的状态
		// 并且暴露在 `this` 上
		data() {
			return {
				// sku所有属性得可能性集合
				res: {},
				// 拼接得字符
				spliter: '\u2299',
				// 组合数据
				r: {},
				// sku名
				keys: [],
				// 选中的sku的副本
				selectedCache: [],
				// 选中的sku
				selectSku: {},
				// 展示的区间价格
				showAreaPrice: [0, 0],
				// 展示的区间库存
				showAreaStock: [0 , 0],
				// 购买的数量
				num: 1,
				// 主题色RGB
				themeRGB: '',
				// 主题色RGBA
				themeRGBA: '',
				// 是否v3
				// #ifdef VUE3
				isV3: true,
				// #endif
				// #ifndef VUE3
				isV3: false,
				// #endif
			}
		},
		// methods 是一些用来更改状态与触发更新的函数
		methods: {
			/**
			 * 计算组合数据
			 */
			combineAttr(data, keys) {
				var allKeys = []
				var result = {}
					
				for (var i = 0; i < data.length; i++) {
					var item = data[i].sku_attrs;
					var values = []
					
					for (var j = 0; j < keys.length; j++) {
						var key = keys[j]
						if (!result[key]) result[key] = []
						if (result[key].indexOf(item[key]) < 0) result[key].push(item[key])
						values.push(item[key])
					}
					
					allKeys.push({
						path: values.join(this.spliter),
						sku: data[i].id
					})
				}
				
				for(var key in result) {
					let obj = {};
					result[key].forEach(item=>{
						obj[item] = {
							value: item,
							disabled: false,
							active: false
						}
					})
					result[key] = obj;
				}
				
				return {
					result: result,
					items: allKeys
				}
			},
			getAllKeys(arr) {
				var result = []
				for (var i = 0; i < arr.length; i++) {
					result.push(arr[i].path)
				}
				return result
			},
			/**
			 * 取得集合的所有子集「幂集」
			 arr = [1,2,3]
			
			     i = 0, ps = [[]]:
			         j = 0; j < ps.length => j < 1:
			             i=0, j=0 ps.push(ps[0].concat(arr[0])) => ps.push([].concat(1)) => [1]
			                      ps = [[], [1]]
			
			     i = 1, ps = [[], [1]] :
			         j = 0; j < ps.length => j < 2
			             i=1, j=0 ps.push(ps[0].concat(arr[1])) => ps.push([].concat(2))  => [2]
			             i=1, j=1 ps.push(ps[1].concat(arr[1])) => ps.push([1].concat(2)) => [1,2]
			                      ps = [[], [1], [2], [1,2]]
			
			     i = 2, ps = [[], [1], [2], [1,2]]
			         j = 0; j < ps.length => j < 4
			             i=2, j=0 ps.push(ps[0].concat(arr[2])) => ps.push([3])    => [3]
			             i=2, j=1 ps.push(ps[1].concat(arr[2])) => ps.push([1, 3]) => [1, 3]
			             i=2, j=2 ps.push(ps[2].concat(arr[2])) => ps.push([2, 3]) => [2, 3]
			             i=2, j=3 ps.push(ps[3].concat(arr[2])) => ps.push([2, 3]) => [1, 2, 3]
			                      ps = [[], [1], [2], [1,2], [3], [1, 3], [2, 3], [1, 2, 3]]
			 */
			powerset(arr) {
				var ps = [
					[]
				];
				for (var i = 0; i < arr.length; i++) {
					for (var j = 0, len = ps.length; j < len; j++) {
						ps.push(ps[j].concat(arr[i]));
					}
				}
				return ps;
			},
			/**
			 * 生成所有子集是否可选、库存状态 map
			 */
			buildResult(items) {
				var allKeys = this.getAllKeys(items);
			
				for (var i = 0; i < allKeys.length; i++) {
					var curr = allKeys[i]
					var sku = items[i].sku
					var values = curr.split(this.spliter)
			
					var allSets = this.powerset(values)
			
					// 每个组合的子集
					for (var j = 0; j < allSets.length; j++) {
						var set = allSets[j]
						var key = set.join(this.spliter)
			
						if (this.res[key]) {
							this.res[key].skus.push(sku)
						} else {
							this.res[key] = {
								skus: [sku]
							}
						}
					}
				}
			},
			trimSpliter(str, spliter) {
				// ⊙abc⊙ => abc
				// ⊙a⊙⊙b⊙c⊙ => a⊙b⊙c
				var reLeft = new RegExp('^' + spliter + '+', 'g');
				var reRight = new RegExp(spliter + '+$', 'g');
				var reSpliter = new RegExp(spliter + '+', 'g');
				return str.replace(reLeft, '')
					.replace(reRight, '')
					.replace(reSpliter, spliter)
			},
			/**
			 * 获取当前选中的属性
			 */
			getSelectedItem() {
				var result = [];
				
				let resObj = this.r.result;
				
				Object.keys(resObj).forEach((key1, index)=>{
					result[index] = "";
					Object.keys(resObj[key1]).forEach(key2=>{
						// 查找选中的属性
						let item = resObj[key1][key2];
						item.active ? result[index] = item.value : '';
					})
				})
				
				return result
			},
			/**
			 * 无效属性点击
			 */
			handleDisableClick(sku, skuArrKey, index) {
				this.selectedCache[index] = sku;
				
				// 清空所有sku选中属性
				let resObj = this.r.result;
				Object.keys(resObj).forEach((key1, index)=>{
					Object.keys(resObj[key1]).forEach(key2=>{
						// 查找选中的属性
						let item = resObj[key1][key2];
						item.active = false;
					})
				})
				
				// 删除无效属性的禁止状态 并 选中
				sku.active = true;
				sku.disabled = false;
				
				this.updateStatus(this.getSelectedItem());
			
				/**
				 * 恢复原来已选属性
				 * 遍历所有非当前属性行
				 *   1. 与 selectedCache 对比
				 *   2. 如果要恢复的属性存在（非 disable）且 和当前*未高亮行*已选择属性的*可组合*），高亮原来已选择的属性且更新
				 *   3. 否则什么也不做
				 */
				for (var i = 0; i < this.keys.length; i++) {
					var item = this.keys[i];
					if (item == skuArrKey) continue
			
					// 没有被禁用的属性 (可以被选择)
					if (this.selectedCache[i] && !this.selectedCache[i].disabled) {
						this.selectedCache[i].active = true;
						this.updateStatus(this.getSelectedItem())
					}
				}
			
			},
			
			/**
			 * 更新所有属性状态
			 */
			updateStatus(selected) {
				for (var i = 0; i < this.keys.length; i++) {
					var key = this.keys[i];
					var data = this.r.result[key];
					var hasActive = !!selected[i];
					var copy = selected.slice();
					
					Object.keys(data).forEach(dataKey=>{
						var item = data[dataKey];
						if (selected[i] == item.value) return
						copy[i] = item.value;
						var curr = this.trimSpliter(copy.join(this.spliter), this.spliter);
						if (this.res[curr]) {
							item.disabled = false;
						} else {
							item.disabled = true;
							item.active = false;
						}
					})
				}
				
				// 获取当前选中的属性
				var result = selected.slice();
				// 如果所有属性都已选中
				if(result.every(item=>item)) {
					// 将属性合成r.items中的sku名称
					let name = this.trimSpliter(result.join(this.spliter), this.spliter);
					// 查找sku
					let sku = this.r.items.find(item=>item.path == name);
					// 如果找到该sku
					if(sku) {
						// 根据sku的sku属性(唯一标识，通常来说会是id)找到源数据中的匹配的那一项并选中;
						this.selectSku = JSON.parse(JSON.stringify(this.data.find(item=> item.id == sku.sku)))
					}
				} else {
					this.selectSku = {};
				}
			},
			bindEvent(sku, skuArr, skuArrKey) {
				if (!sku.active) {
					// 清空当前行的所有sku选中属性
					Object.keys(skuArr).forEach(key=>{
						skuArr[key].active = false;
					})
					
					sku.active = true;
					
					// 当前选中的keys的index
					let index = this.keys.findIndex(item=> item == skuArrKey);
					
					if (sku.disabled) {
						this.handleDisableClick(sku, skuArrKey, index);
					} else {
						this.selectedCache[index] = sku;
					}
				} else {
					sku.active = false;
					// 清空所有sku禁用状态
					let resObj = this.r.result;
					Object.keys(resObj).forEach((key1, index)=>{
						Object.keys(resObj[key1]).forEach(key2=>{
							// 查找选中的属性
							let item = resObj[key1][key2];
							item.disabled = false;
						})
					})
				}
				
				this.updateStatus(this.getSelectedItem());
			},
			
			/**
			 * 选中最便宜价格的sku
			 */
			selectMinPriceSku() {
				let minPriceSku = null;
				this.data.forEach(sku=>{
					// 如果为空则直接赋值
					if(minPriceSku === null) {
						minPriceSku = sku
					} else if(minPriceSku.price > sku.price) { // 如果比最小价格低 就赋值
						minPriceSku = sku
					}
				})
				for(var key in minPriceSku.sku_attrs) {
					// 找出对应项并选中
					this.r.result[key][minPriceSku.sku_attrs[key]].active = true;
				}
				this.updateStatus(this.getSelectedItem());
			},
			
			/**
			 * 找出区间数据
			 */
			findAreaData() {
				let minPrice = null;
				let maxPrice = 0;
				let minStock = null;
				let maxStock = 0;
				this.data.forEach(sku=>{
					// 如果最小价格为空则直接赋值
					if(minPrice === null) {
						minPrice = sku.price;
					} else if(minPrice > sku.price) { // 如果比最小价格低 就赋值
						minPrice = sku.price;
					}
					// 如果大于最大价格则赋值
					if(maxPrice < sku.price) {
						maxPrice = sku.price;
					}
					
					// 如果最小库存为空则直接赋值
					if(minStock === null) {
						minStock = sku.stock;
					} else if(minStock > sku.stock) { // 如果比最小库存少 就赋值
						minStock = sku.stock;
					}
					// 如果大于最大库存则赋值
					if(maxStock < sku.stock) {
						maxStock = sku.stock;
					}
				})
				// 赋值区间
				this.showAreaPrice = [minPrice, maxPrice];
				this.showAreaStock = [minStock, maxStock];
			},
			
			// 初始化
			init(data) {
				this.res = {}
				this.r = {};
				this.keys = [];
				this.selectedCache = [];
				
				// 拼接主题色
				this.joinThemColor(this.themeColor);
				
				// 找出sku的可选属性的标题
				for (var attr_key in data[0].sku_attrs) {
					if (!data[0].sku_attrs.hasOwnProperty(attr_key)) continue;
					this.keys.push(attr_key);
				}
				
				//计算组合数据
				this.r = this.combineAttr(data, this.keys);
				// 生成所有子集是否可选、库存状态 map
				this.buildResult(this.r.items);
				
				// 找到区间数据
				this.findAreaData();
				
				// 如果需要选中默认最便宜的sku
				if(this.isSelectMinPriceSku) {
					this.selectMinPriceSku();
				}
			},
			
			// 确认事件
			confirm() {
				// 如果已有选中的完整sku
				if(this.selectSku.id) {
					// 如果有库存
					if(this.selectSku.stock * 1 > 0) {
						// 按钮确认事件
						this.$emit('confirm', {
							sku: this.selectSku,
							skuText: this.getSelectedItem(),
							num: this.num * 1
						})
					}
				} else {
					uni.showToast({
						title: this.notSelectSku,
						icon: 'none',
						duration: 1500
					})
				}
			},
			
			// 拼接主题色
			joinThemColor(n) {
				this.themeRGB = `rgb(${n[0]}, ${n[1]}, ${n[2]})`;
				this.themeRGBA = `rgba(${n[0]}, ${n[1]}, ${n[2]}, 0.1)`;
			},
			
			// 关闭sku组件
			close() {
				// #ifdef VUE3
				this.$emit('update:modelValue', false);
				// #endif
				// #ifndef VUE3
				this.$emit('input', false);
				// #endif
				this.$emit('close');
			},
			
			// 打开sku组件
			open() {
				// #ifdef VUE3
				this.$emit('update:modelValue', true);
				// #endif
				// #ifndef VUE3
				this.$emit('input', true);
				// #endif
				this.$emit('open');
			}
		},
		// 监听
		watch: {
			data(n) {
				this.init(n);
			},
			// #ifdef VUE3
			modelValue(n) {
				n ? this.open() : this.close();
			},
			// #endif
			// #ifndef VUE3
			value(n) {
				n ? this.open() : this.close();
			},
			// #endif
			num(n) {
				this.$emit('numChange', n);
			},
			selectSku(n) {
				this.$emit('skuChange', n);
			},
			themeColor(n) {
				this.joinThemColor(n);
			}
		},
		// 挂载时
		mounted() {
			this.init(this.data);
		}
	}
	import numBox from "../numBox/numBox.vue";
</script>

<style lang="less" scoped>
	@import url("../common.less");
	.sku {
		position: fixed;
		left: 0;
		right: 0;
		bottom: 0;
		top: 0;
		background-color: rgba(0, 0, 0, 0.4);
		z-index: 999;
		.shopSpecsPopup {
			background: #FFFFFF;
			border-radius: 24rpx 24rpx 0 0;
			padding: 45rpx 30rpx 30rpx;
			position: fixed;
			box-sizing: border-box;
			left: 0;
			right: 0;
			bottom: 0;
			
			.close {
				width: 40rpx;
				height: 40rpx;
				position: absolute;
				top: 33rpx;
				right: 30rpx;
			}
		
			.content {
				bottom: 0;
			}
		
			.btnBox {
				background-color: #fff;
				border: none;
				margin-top: 40rpx;
				.confirm {
					border-radius: 50rpx;
					font-size: 32rpx;
					font-weight: 500;
					text-align: center;
					box-sizing: border-box;
					height: 86rpx;
					line-height: 1;
					display: flex;
					justify-content: center;
					align-items: center;
					color: #FFFFFF;
					&.disabled {
						background-color: #b4b4b4 !important;
						color: #fff;
					}
				}
			}
		
			.info {
				.cover {
					width: 200rpx;
					height: 200rpx;
					flex-shrink: 0;
					margin-right: 20rpx;
					border-radius: 20rpx;
					background-size: cover;
					background-repeat: no-repeat;
					background-position: center;
				}
		
				.right {
					width: calc(100% - 220rpx);
		
					.title {
						font-size: 32rpx;
						font-weight: 500;
						color: #333333;
						margin-bottom: 20rpx;
						padding-right: 120rpx;
					}
		
					.price {
						font-size: 36rpx;
						font-weight: bold;
						margin-bottom: 20rpx;
		
						.uity {
							font-size: 26rpx;
						}
					}
					
					.stock {
						font-size: 30rpx;
						color: #999999;
					}
				}
			}
		
			.number {
				margin-top: 30rpx;
				margin-bottom: 22rpx;
		
				.key {
					font-size: 32rpx;
					font-weight: 500;
					color: #333333;
				}
			}
		
			.specsList {
				max-height: 350rpx;
				overflow-y: auto;
				margin-bottom: 30rpx;
				.item {
					.title {
						font-size: 32rpx;
						font-weight: 500;
						color: #333333;
						margin-bottom: 30rpx;
					}
				
					.specsValueList {
						.specs {
							padding: 14rpx 40rpx;
							background: #fff;
							border-radius: 10rpx;
							font-size: 28rpx;
							font-weight: 500;
							color: #999999;
							margin-bottom: 30rpx;
							margin-right: 30rpx;
							border: 2rpx solid #e4e4e4;
							
							&.disabled {
								background-color: #f3f3f3;
								border: 2rpx solid transparent;
							}
						}
					}
				}
			}
		}
	}
</style>
