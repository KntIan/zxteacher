<template>
	<div class="custom">
		<template v-if="isCustom.businessID === CHAT_MSG_CUSTOM_TYPE.SERVICE">
			<div>
				<h1>
					<label>{{ extension.title }}</label>
					<a v-if="extension.hyperlinks_text" :href="extension.hyperlinks_text.value"
						target="view_window">{{ extension.hyperlinks_text.key }}</a>
				</h1>
				<ul v-if="extension.item && extension.item.length > 0">
					<li v-for="(item, index) in extension.item" :key="index">
						<a v-if="isUrl(item.value)" :href="item.value" target="view_window">{{ item.key }}</a>
						<p v-else>
							{{ item.key }}
						</p>
					</li>
				</ul>
				<article>{{ extension.description }}</article>
			</div>
		</template>
		<template v-else-if="isCustom.businessID === CHAT_MSG_CUSTOM_TYPE.EVALUATE">
			<div class="evaluate">
				<h1>{{ TUITranslateService.t("message.custom.对本次服务评价") }}</h1>
				<ul class="evaluate-list">
					<li v-for="(item, index) in Math.max(isCustom.score, 0)" :key="index" class="evaluate-list-item">
						<Icon :file="star" class="file-icon" />
					</li>
				</ul>
				<article>{{ isCustom.comment }}</article>
			</div>
		</template>
		<template v-else-if="isCustom.businessID === CHAT_MSG_CUSTOM_TYPE.ORDER">
			<div class="order" @click="openLink(isCustom.link)">
				<img :src="isCustom.imageUrl">
				<main>
					<h1>{{ isCustom.title }}</h1>
					<p>{{ isCustom.description }}</p>
					<span>{{ isCustom.price }}</span>
				</main>
			</div>
		</template>
		<template v-else-if="isCustom.businessID === CHAT_MSG_CUSTOM_TYPE.LINK">
			<div class="textLink">
				<p>{{ isCustom.text }}</p>
				<a :href="isCustom.link" target="view_window">{{
          TUITranslateService.t("message.custom.查看详情>>")
        }}</a>
			</div>
		</template>
		<template v-else>
			<span v-html="content.custom" />
		</template>
	</div>
</template>

<script lang="ts" setup>
	import { watchEffect, ref } from 'vue';
	import { TUITranslateService, IMessageModel } from '@tencentcloud/chat-uikit-engine';
	import { isUrl, JSONToObject } from '../../../../utils/index';
	import { CHAT_MSG_CUSTOM_TYPE } from '../../../../constant';
	import { ICustomMessagePayload } from '../../../../interface';
	import Icon from '../../../common/Icon.vue';
	import star from '../../../../assets/icon/star-light.png';
	interface Props {
		messageItem : IMessageModel;
		content : any;
	}

	const props = withDefaults(defineProps<Props>(), {
		messageItem: undefined,
		content: undefined,
	});

	const custom = ref();
	const message = ref<IMessageModel>();
	const extension = ref();
	const isCustom = ref<ICustomMessagePayload>({
		businessID: '',
	});

	watchEffect(() => {
		custom.value = props.content;
		message.value = props.messageItem;
		const { payload } = props.messageItem;
		isCustom.value = payload.data || '';
		isCustom.value = JSONToObject(payload.data);
		if (payload.data === CHAT_MSG_CUSTOM_TYPE.SERVICE) {
			extension.value = JSONToObject(payload.extension);
		}
	});
	const openLink = (url : any) => {
		window.open(url);
	};
</script>
<style lang="scss" scoped>
	@import "../../../../assets/styles/common";

	a {
		color: #679ce1;
	}

	.custom {
		font-size: 14px;

		h1 {
			font-size: 14px;
			color: #000;
		}

		h1,
		a,
		p {
			font-size: 14px;
		}

		.evaluate {
			ul {
				display: flex;
				padding: 10px 0;
			}

			&-list {
				display: flex;
				flex-direction: row;

				&-item {
					padding: 0 2px;
				}
			}
		}

		.order {
			display: flex;

			main {
				padding-left: 5px;

				p {
					font-family: PingFangSC-Regular;
					width: 145px;
					line-height: 17px;
					font-size: 14px;
					color: #999;
					letter-spacing: 0;
					margin-bottom: 6px;
					word-break: break-word;
				}

				span {
					font-family: PingFangSC-Regular;
					line-height: 25px;
					color: #ff7201;
				}
			}

			img {
				width: 67px;
				height: 67px;
			}
		}
	}
</style>