<template>
	<view class="page">
		<view class="box1">
			<image class="image_2" referrerpolicy="no-referrer"
				src="https://lanhu.oss-cn-beijing.aliyuncs.com/SketchPng4253829e1a377894ca42c94d7edf9cdfd758b5bf890e937604ce7635bfc36c30" />
			<text class="text-group_2">练习书法 习得一手好字</text>
		</view>
		<view class="box2">
			<view class="box_2 flex-col"></view>
			<text class="text2">泽熙书画</text>
		</view>
	</view>

</template>
<script>
	export default {
		data() {
			return {
				constants: {},
			};
		},
		onLoad() {
			setTimeout(() => {
				uni.navigateTo({
					url: "/pages/login/login"
				})
			}, 3000)
		},

		methods: {

		},
	};
</script>
<style>
	.page {
		background-color: rgba(255, 255, 255, 1);
		position: absolute;
		width: 100%;
		height: 100%;
		overflow: hidden;
	}

	.box1 {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		position: relative;
		top: 10%;
	}

	.image_2 {
		width: 174px;
		height: 164px;
	}

	.box2 {
		display: flex;
		align-items: center;
		position: fixed;
		bottom: 10%;
		left: 50%;
		transform: translate(-50%);
	}

	.box_2 {
		border-radius: 4px;
		background-image: url(https://lanhu-dds-backend.oss-cn-beijing.aliyuncs.com/merge_image/imgs/236b4a513e3f4f5fb13b40084948436c_mergeImage.png);
		width: 44px;
		height: 44px;

	}

	.text2 {
		font-size: 14px;
		font-weight: 400;
		margin-left: 10px;
		font-family: PangMenZhengDao;
	}
</style>