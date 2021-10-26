<!--  -->
<template>
	<unicloud-db style="padding: 10rpx;" @load="loadData" ref="udb" v-slot:default="{ data, loading, error, options }" collection="questions"
		:page-size="99999">
		<swiper :current="current" class="swiper" @change="swiperChange">
			<swiper-item v-for="(each, swiperIndex) in displayData" :key="swiperIndex">
				<view class="content">{{ swiperIndex + 1 }}.({{ each.source.slice(0,2) }}){{ each.content }}</view>
				<view class="answer-area">
					<!-- <view class="answer-option" v-for="i in 5">{{ each[`answer${i}`] }}</view> -->
					<radio-group @change="radioChange($event, swiperIndex)" v-if="each.answer.length === 1">
						<label v-for="i in 5" :key="i">
							<view v-if="each[`answer${i}`]">
								<radio :value="each[`answer${i}`]"
									:color="each.result === each.answer ? 'green' : 'red'" />
								{{ each[`answer${i}`] }}
							</view>
						</label>
					</radio-group>
					<checkbox-group v-else @change="checkBoxChange($event, swiperIndex)">
						<label v-for="i in 5" :key="i">
							<view v-if="each[`answer${i}`]">
								<!-- 既在正确答案中  又在我选的答案中 那就标绿 -->
								<checkbox :value="each[`answer${i}`]" :color="
										each.result && each.answer.indexOf(each[`answer${i}`].trim().slice(0, 1)) !== -1 && each.result.indexOf(each[`answer${i}`].trim().slice(0, 1)) !== -1
											? 'green'
											: 'red'
									" style="transform:scale(0.7)" />
								{{ each[`answer${i}`] }}
							</view>
						</label>
					</checkbox-group>
				</view>
				<!-- 单选不相等就出来  多选在我选的里面 但是没在正确答案里面也算错-->
				<view style="margin-top: 20rpx;" class="result"
					v-if="each.result && each.result.split('').some(r => !each.answer.includes(r))">
					<view class="">【答案】:{{ each.answer }}</view>
					<view class="parse">【解析】:{{ each.parse }}</view>
				</view>
			</swiper-item>
		</swiper>
	</unicloud-db>
</template>

<script>
	import {
		ref,
		nextTick
	} from 'vue';
	export default {
		name: '',
		setup() {
			let udb = ref('');
			let current = ref(0);
			const pageSize = 3
			let currentPage = 1
			let displayData = ref([])
			let fulldata = []

			function loadData(data, ended) {
				fulldata = data.sort(() => Math.random() > 0.5 ? 1 : -1)
				displayData.value.push(...fulldata.slice(0, currentPage * pageSize))
			}


			function swiperChange({
				detail
			}) {
				current.value = detail.current;
				if (detail.current == displayData.value.length - 1) {
					currentPage += 1
					// udb.value.loadMore();
					displayData.value.push(...fulldata.slice((currentPage - 1) * pageSize, currentPage * pageSize))
				}
			}

			function radioChange({
				detail
			}, index) {
				let target = udb.value.dataList[index];
				target.result = detail.value.slice(0, 1);
			}

			function checkBoxChange({
				detail
			}, index) {
				let target = udb.value.dataList[index];
				target.result = detail.value.map(each => each.trim().slice(0, 1)).join('');
			}
			return {
				udb,
				swiperChange,
				current,
				radioChange,
				checkBoxChange,
				loadData,
				displayData
			};
		}
	};
</script>
<style lang="scss" scoped>
	.swiper {
		height: 100vh;
	}
</style>
