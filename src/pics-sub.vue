<template >
	<div class="container">
		<waterfall column-count="2" style="width: 750px;">
			<cell v-for="item in datalist" @click="click" :item="item">
				<div class='item'>
					<div v-if="item.imgs">
						<image resize="cover" style="background-color: #e3e3e3; width: 335px; height: 430px;" :src="item.imgs[0]"></image>
						<div v-if="item.isGIF" style="position: absolute; top: 0; left: 0;right: 0; bottom: 0; background-color: #000000; opacity: 0.4; justify-content: center; align-items: center;">
							<image :src="gif" style="width: 50px; height: 50px; margin-top: 215px;"></image>
						</div>
					</div>
					<div v-if="!item.imgs">
						<image resize="cover" style="background-color: #e3e3e3; width: 335px; height: 430px;"></image>
					</div>
					<text>{{item.title}}</text>
					<div style="justify-content:space-between; align-items: center; margin-top: 20px; flex-direction: row;">
						<text>{{item.author}}</text>
						<text v-if="item.imgs">({{item.imgs.length}})</text>
					</div>
					<div style="align-items: center;">
						<image v-if="item.isLike" ref='like' class="like-item" :src="config.image('like.png')"></image>
					</div>
				</div>
			</cell>
			<cell style="height: 80px;"></cell>
			<loading class="loading" @loading="onloading" :display="showLoading">
		      <text class="indicator">{{loadingTips}}</text>
		    </loading>
		</waterfall>
		<div class="like-conver" v-if="showLike">
			<image ref='like' class="like" :src="config.image('like.png')"></image>
		</div>
	</div>
</template>
<style>
	.container{padding-left: 10px;background-color: #f3f3f3;}
	.item{border-radius: 10px;background-color: #ffffff;padding: 10px;margin-top: 40px;}
	.loading{width: 750px; padding-top: 40px; padding-bottom: 40px; align-items: center;justify-content: center;}
</style>
<script>
	import config from './config'
	import jandan from './services/jandan'
	import animation from './animation/animation'
	const browser = weex.requireModule('browser')
	export default {
		data: {
			config:config,
			datalist:[1,1,1,1,1,1,1,1,1,1,1],
			maxPage:0,
			showLoading:'hide',
			lastClickObj:{item:null,timestamp:0},
			isSingleClick:true,
			gif:config.image('gif.png'),
			type:'boring'
		},
		computed:{
			loadingTips(){
				if(!this.datalist || this.datalist.length == 0){
					return '加载中...'
				}
				return this.showLoading == 'hide' ? '上拉加载更多' : '加载中...'
			}
		},
		created() {
			this.type = config.params('type')
			jandan.list(this.type).then((response)=>{
				this.datalist = response.datalist
				if(response.maxPage) {
					this.maxPage = response.maxPage
				}
			})
			config.event('boring','无聊图')
		},
		methods: {
			getUrlParam (key) {
				var reg = new RegExp('[?|&]' + key + '=([^&]+)')
				var match = weex.config.bundleUrl.match(reg)
				return match && match[1]
			},
			onloading(){
		        this.showLoading = 'show'
		        this.maxPage -= 1
		        jandan.list(this.type,this.maxPage).then(response =>{
		        		this.showLoading = 'hide'
		        		setTimeout(()=>{
		        			this.datalist = this.datalist.concat(response.datalist)
		        		},100)
		        })
			},
			click(e){
				const item = e.target.attr.item
				if(item.originImages && item.originImages.length > 0) {
					var originImages = [];
					var currentIndex = 0;
					var find = false
					this.datalist.forEach((value) => {
						if(value.originImages && value.originImages.length>0) {
							originImages = originImages.concat(value.originImages);
							if(!find && value != item) {
								currentIndex += value.originImages.length;
							} else {
								find = true
							}
						}
					})
					browser.browserImages(originImages, currentIndex)
				} else if(item.imgs && item.imgs.length > 0) {
					var imgs = [];
					var currentIndex = 0;
					var find = false
					this.datalist.forEach((value) => {
						if(value.imgs && value.imgs.length>0) {
							imgs = imgs.concat(value.imgs);
							if(!find && value != item) {
								currentIndex += value.imgs.length;
							} else {
								find = true
							}
						}
					})
					browser.browserImages(imgs, currentIndex)
				}
			}
		}
	}
</script>