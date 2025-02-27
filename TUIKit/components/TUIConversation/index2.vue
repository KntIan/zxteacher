<template>
	<div class="tui-conversation" @click="handleClickConv" @touchstart="handleTouchStart" @touchend="handleTouchEnd">
		<TUISearch searchType="global" />
		<ConversationHeader v-if="isShowConversationHeader" ref="headerRef" />
		<ConversationNetwork />
		<ConversationList ref="conversationListDomRef" class="tui-conversation-list"
			@handleSwitchConversation="handleSwitchConversation" @getPassingRef="getPassingRef" />
		<!-- <TabBar :tabs="tabs" :currentIndex="currentTabIndex" @tab-selected="onTabSelected" /> -->
	</div>
</template>
<script lang="ts" setup>
	import { TUIStore, StoreName } from '@tencentcloud/chat-uikit-engine'
	import { TUIGlobal } from '@tencentcloud/universal-api'
	import { ref, defineProps, onMounted, watch } from 'vue'
	import TUISearch from '../TUISearch/index.vue'
	import ConversationList from './conversation-list/index.vue'
	import ConversationHeader from './conversation-header/index.vue'
	import ConversationNetwork from './conversation-network/index.vue'
	import TabBar from '../tabbar/tabbar'

	import { onHide } from '@dcloudio/uni-app'
	// #ifdef MP-WEIXIN
	// uniapp packaged mini-programs are integrated by default, and the default initialization entry file is imported here
	// TUIChatKit init needs to be encapsulated because uni vue2 will report an error when compiling H5 directly through conditional compilation
	import './entry.ts'
	// #endif
	// 接收传入的 marker 属性
	const props = defineProps<{
		marker ?: string // 可选的 marker 属性
	}>()
	const emits = defineEmits([
		'handleSwitchConversation',
		'updateConversationListLengthTeachers',
	])

	const totalUnreadCount = ref(0)
	const headerRef = ref<typeof ConversationHeader>()
	const conversationListDomRef = ref<typeof ConversationList>()
	const touchX = ref<number>(0)
	const touchY = ref<number>(0)
	const isShowConversationHeader = ref<boolean>(true)
	const currentTabIndex = ref(0)

	TUIStore.watch(StoreName.CONV, {
		totalUnreadCount: (count : number) => {
			totalUnreadCount.value = count
		},
	})

	TUIStore.watch(StoreName.CUSTOM, {
		isShowConversationHeader: (showStatus : boolean) => {
			isShowConversationHeader.value = showStatus !== false
		},
	})
	const updateParentWithLengthTeachers = () => {
		const length = conversationListDomRef.value?.items?.length || 0 // 获取长度
		emits('updateConversationListLengthTeachers', length) // 触发事件并传递长度
	}
	onMounted(() => {
		updateParentWithLengthTeachers()
	})
	watch(conversationListDomRef, () => {
		updateParentWithLengthTeachers()
	})
	const handleSwitchConversation = (conversationID : string) => {
		TUIGlobal?.navigateTo({
			url: '/TUIKit/components/TUIChat/index',
		})
		emits('handleSwitchConversation', conversationID)
	}

	const closeChildren = () => {
		headerRef?.value?.closeChildren()
		conversationListDomRef?.value?.closeChildren()
	}

	const handleClickConv = () => {
		closeChildren()
	}

	onHide(closeChildren)

	const handleTouchStart = (e : any) => {
		touchX.value = e.changedTouches[0].clientX
		touchY.value = e.changedTouches[0].clientY
	}

	const handleTouchEnd = (e : any) => {
		const x = e.changedTouches[0].clientX
		const y = e.changedTouches[0].clientY
		let turn = ''
		if (x - touchX.value > 50 && Math.abs(y - touchY.value) < 50) {
			// Swipe right
			turn = 'right'
		} else if (x - touchX.value < -50 && Math.abs(y - touchY.value) < 50) {
			// Swipe left
			turn = 'left'
		}
		if (y - touchY.value > 50 && Math.abs(x - touchX.value) < 50) {
			// Swipe down
			turn = 'down'
		} else if (y - touchY.value < -50 && Math.abs(x - touchX.value) < 50) {
			// Swipe up
			turn = 'up'
		}
		// Operate according to the direction
		if (turn === 'down' || turn === 'up') {
			closeChildren()
		}
	}
	// 处理选项卡选择
	const onTabSelected = (index) => {
		currentTabIndex.value = index
		if (index === 1) {
			// 通讯录 Tab 的索引为 1
			TUIGlobal?.navigateTo({
				url: '/TUIKit/components/TUIContact/index',
			})
		}
	}
	const getPassingRef = (ref) => {
		ref.value = conversationListDomRef.value
	}
</script>

<style lang="scss" scoped src="./style/index.scss"></style>