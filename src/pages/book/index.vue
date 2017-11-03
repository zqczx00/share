<template>
	<div class="book-index-page">
		<v-top :scrollTop="scrollTop"></v-top>
		<!-- scroll 外层容器 -->
		<div ref="wrapper" class="wrapper" :style="{ height: winHeight }">
			<!-- 内容 -->
			<section ref="content" class="content">
				<v-banner :style="{ height: bannerHeight }" :listImg="listImg" class="banner" @chooseItem="chooseItem"></v-banner>
				<!-- 图书种类 -->
				<ul class="type-row">
					<li v-for="(item, index) in bookTypeList" @click="openSearch(item.id)" class="type-col">{{ item.name }}</li>
					<li @click="openSearch()" class="type-col">更多...</li>
				</ul>
				<!-- 推荐列表 -->
				<ul class="recommend-list">
					<!-- 优惠 -->
					<li class="list-item">
						<section class="discounts top">
							<div class="title discount-title">
								<div class="top-line"></div>
								<div class="bottom-line"></div>
								<div class="special-name">
									今日优惠
								</div>
							</div>
							<div class="more" @click="openSingle('1', '1', '今日特惠')">
								<span>更多</span>
								<img src="./more_icon.png">
							</div>
						</section>
						<ul class="item-body recommend-row">
							<li v-for="item in promotions" class="recommend-col" @click="openDetail(item.id)">
								<div class="img" :style="imgStyleObj">
									<img v-lazy="item.logo">
									<div class="bottom">
										<span class="text"><span class="big">{{ item.discount }}</span>/折</span>
									</div>
								</div>
								<p class="name">{{ item.name }}</p>
							</li>
						</ul>
					</li>
					<!-- 最新上架 -->
					<li class="list-item">
						<section class="special top">
							<div class="title">
								<img src="./new_icon.png">
								<span class="new-name">
									最新上架
								</span>
							</div>
							<div class="more" @click="openSingle('1', '2', '最新上架')">
								<span>更多</span>
								<img src="./more_icon.png">
							</div>
						</section>
						<ul class="item-body recommend-row">
							<li v-for="item in newests" class="recommend-col" @click="openDetail(item.id)">
								<div class="img" :style="imgStyleObj">
									<img v-lazy="item.logo">
								</div>
								<p class="name">{{ item.name }}</p>
							</li>
						</ul>
					</li>
					<!-- 动态栏目 -->
					<li v-for="column in columns" v-if="column.books.length >= 4" class="list-item">
						<section class="special top">
							<div class="title">
								<span class="name">
									{{ column.name }}
								</span>
							</div>
							<div @click="openSingle('2', column.id, column.name)" class="more">
								<span>更多</span>
								<img src="./more_icon.png">
							</div>
						</section>
						<ul class="item-body recommend-row">
							<li v-for="item in column.books" class="recommend-col" @click="openDetail(item.id)">
								<div class="img" :style="imgStyleObj">
									<img v-lazy="item.logo">
								</div>
								<p class="name">{{ item.name }}</p>
							</li>
						</ul>
					</li>
				</ul>
				<!-- 猜你喜欢 -->
				<v-recommend v-if="recommendList && recommendList.length" :recommendList="recommendList" @freshBScroll="initBetterScroll"></v-recommend>
			</section>
		</div>
	</div>
</template>

<script>
	import banner from '@/components/common/banner/banner'
	import top from './top/top'
	import BScroll from 'better-scroll'
	import recommend from './recommend/recommend'
	export default {
		name: 'book-index-page',
		data () {
			return {
				bannerHeight: window.innerWidth / 3 + 'px',
				// 屏幕高度
				winHeight: window.innerHeight - 50 + 'px',
				scroller: '',
				scrollTop: 0,
				bookTypeList: [],
				promotions: [],
				newests: [],
				columns: [],
				recommendList: [],
				adverts: []
			}
		},
		computed: {
			listImg () {
				let list = []
				if (this.adverts.length) {
					this.adverts.forEach(item => {
						list.push({
							url: item.logo
						})
					})
				}
				return list
			},
			imgStyleObj () {
				let resize = (window.innerWidth - 16 * 2 - 5 * 4) / 4
				return {
					height: resize + 'px'
				}
			}
		},
		components: {
			'v-banner': banner,
			'v-top': top,
			'v-recommend': recommend
		},
		created () {
			// 加载数据
			this.loadData()
		},
		mounted () {
		},
		methods: {
			loadData () {
				// 图书种类列表
				this.$ajax.bookConstant().then(res => {
					console.log(res)
					let bookList = res.data.bookTypeList
					if (bookList.length >= 16) {
						bookList = bookList.slice(0, 15)
					}
					this.bookTypeList = bookList
				}, err => {
					console.log(err)
				})
				// 首页数据
				this.$ajax.bookHomepage().then(res => {
					console.log(res)
					this.promotions = res.data.promotions
					this.columns = res.data.columns
					this.newests = res.data.newests
					this.recommendList = res.data.recommends
					this.adverts = res.data.adverts
					this.$nextTick(() => {
						// 初始化 better-scroll
						this.initBetterScroll()
					})
				}, err => {
					console.log(err)
				})
			},
			initBetterScroll () {
				console.log(this.$refs.content.offsetHeight)
				if (!this.scroller) {
					this.scroller = new BScroll(this.$refs.wrapper, {
						probeType: 3,
						click: true
					})
					// 监听滚动条
					this.listenScroll()
				} else {
					this.scroller.refresh()
				}
			},
			listenScroll () {
				this.scroller.on('scroll', (pos) => {
					this.scrollTop = -pos.y
				})
				this.scroller.on('touchend', (pos) => {
					if (pos.y > 50) {
						// 刷新
						// this.loadData()
					}
				})
			},
			// 打开单项列表
			openSingle (type, id, title) {
				this.$router.push({
					path: 'book/single',
					query: {
						type: type,
						id: id,
						title: title
					}
				})
			},
			openSearch (id) {
				if (id) {
					this.$router.push({
						path: 'book/search',
						query: {
							id: id
						}
					})
				}
			},
			// 选择 banner
			chooseItem (index) {
				let id = this.adverts[index].srcid
				this.$router.push({
					path: 'book/detail',
					query: {
						id: id
					}
				})
			},
			// 打开详情
			openDetail (id) {
				this.$router.push({
					path: 'book/detail',
					query: {
						id: id
					}
				})
			}
		}
	}
</script>

<style lang="stylus" rel="stylesheet/stylus">
	@import './index.styl'
</style>