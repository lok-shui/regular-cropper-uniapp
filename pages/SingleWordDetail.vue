<template>
	<view class="pageBg">
		<!-- 头部  -->
		<view class="topContent flexBetween">
			<view class="itemStyle">
				<view class="img flexCenter">
					<image :src="img_pred" mode=""></image>
					<!-- <canvas :style="{width:canvasW + 'rpx', height: canvasH+ 'rpx'}" canvas-id="myCanvas" id="myCanvas" class="canvas1"></canvas> -->
					
				</view>
				<view class="lineBg">汉字：{{word}}</view>
			</view>
			
			<view class="itemStyle">
				<view class="img flexCenter">
					<image :src="img_gt" mode=""></image>
					<!-- <canvas :style="{width:canvasW + 'rpx', height: canvasH+ 'rpx'}" canvas-id="myCanvas" id="myCanvas" class="canvas1"></canvas> -->
				</view>
				<view class="lineBg">识别可信度：<text class="bold">{{confidence}}</text></view>
			</view>
			
			<view class="itemStyle third">
				<view>书写评分</view>
				<view class="thirdWord">
					<b class='large'>{{score}}</b>
					<view class="progressBox flexCenter">
						<text class="txt">笔画</text>
						<view  class="prog">
							<progress :percent="score_stroke" activeColor="#6236FF" stroke-width="10" border-radius="10" />
						</view>
						<text class="txt">{{score_stroke}}</text>
						
					</view>
					<view class="progressBox flexCenter">
						<text class="txt">结构</text>
						<view  class="prog">
							<progress  :percent="score_struc" activeColor="#6236FF" stroke-width="10" border-radius="10" />
						</view>
						<text class="txt">{{score_struc}}</text>
					</view>
					 
				</view>
				 
			</view>
		</view>
		
		<!-- 分段器 -->
		<view class="segmented">
			<uni-segmented-control :current="current" :values="items" @clickItem="onClickItem" style-type="text" active-color="#6850A5"></uni-segmented-control>
			<view class="content" style="padding-bottom: 15rpx;">
				<view v-show="current === 0">
					<view class="firstTab flexBetween" v-for="(item, index) in stroke_evaluation" :key="index">
						 
						<view class="itemStyle">
							<view class="img flexCenter">
								<image :src="item.img_vis" mode="widthFix"></image>
								<!-- <image :src="imgPath" mode="" :style="{width:canvasW + 'rpx', height: canvasH+ 'rpx'}"></image> -->
								<!-- <canvas :style="{width:canvasW + 'rpx', height: canvasH+ 'rpx'}" canvas-id="myCanvas1" id="myCanvas1" class="canvas1"></canvas> -->
							</view>
						</view>
						<view class="itemStyle">
							<view class="img flexCenter">
								<image :src="item.img_gt" mode="widthFix"></image>
								<!-- <canvas :style="{width:canvasW + 'rpx', height: canvasH+ 'rpx'}" canvas-id="myCanvas" id="myCanvas" class="canvas1"></canvas -->
							</view>
							<view class="lineBg">标准笔画</view>
						</view>
						<view class="itemStyle thirdItem">
							<view class="stroke"><text>笔画名</text><text>{{item.name_stroke}}</text></view>
							<view class="stroke">
								<text>得分</text><text class="fontSize">{{item.score}}</text>
							</view>
							<view class="review">{{item.review}}</view>
						</view>
					</view>
				</view>
				
				<view v-show="current === 1">
					<view class="secondTab">
						<!-- <view class="left">
							<view class="grap"></view>
							<view class="img">ff</view>
							<view class="item" >
								<view>item</view>
								<view>item</view>
								<view>item</view>
								<view>item</view>
								<view>item</view>
								<view>item</view>
							</view>
						</view> -->
						<view class="left" v-for="(item, index) in struc_evaluation" :key="index">
							<view class="grap"></view>
							<view class="img flexCenter">
								<image :src="item.img_vis" mode="widthFix"></image>
								<!-- <canvas :style="{width:canvasW + 'rpx', height: canvasH+ 'rpx'}" canvas-id="myCanvas2" id="myCanvas2" class="canvas1"></canvas> -->
							</view>
							<view class="item">
								<view class="flexBetween">
									<text><b>{{item.name_strcu}}</b></text>
									<text><b>{{item.score}}</b>分</text>
								</view>
								<view>{{item.review}}</view>
							</view>
							
						</view>
					</view>
				</view>
				
			</view>
		</view>
		
	</view>
</template>

<script>
	import segmentedControl from '../components/uni-segmented-control/uni-segmented-control.vue';
	export default {
		components:{
			segmentedControl
		},
		data() {
			return {
				items: ['看笔画','看结构'],
				current: 0,
				tabItem:[1,2,3],
				single: '山',
				canvasW: 150,
				canvasH: 150,
				imgPath:'',
				wordId: 0,
				
				word: "-",           // 字符类别
				confidence: '-',        // 置信度
				score: 0,             // 文字评测分数
				img_pred: "",
				img_gt: "",          // 标准字的image base64编码
				score_stroke: 0,     // 语法得分
				score_struc: 0,      // 结构得分
				
				stroke_evaluation:[], // 笔画评价
				struc_evaluation:[],  // 结构评价
			}
		},
		methods: {
			onClickItem(e) {
				 let me = this;
			    me.current = e.currentIndex;
			    if(me.current == 1) {
			    	console.log('me.imgPath',me.imgPath)
			    	setTimeout(() => {
			    		me.getImageInfo(me.imgPath)
			    	},1500)
			    }
			 },
			 // 获取图片信息
			 getImageInfo(image) {
			   let me = this;
			   
			 	return new Promise((req, rej) => {
			 		uni.getImageInfo({
			 			src: image,
			 			success: function(res) {
							console.log('5image',image);
			 				if(me.current == 0) {
								const ctx = uni.createCanvasContext('myCanvas');
								ctx.drawImage(image, 0, 0, res.width, res.height);
								ctx.draw(); 
								
								const ctx1 = uni.createCanvasContext('myCanvas1');
								ctx1.drawImage(image, 0, 0, res.width, res.height);
								ctx1.draw();
			 					console.log('切换tab等于1',res)
			 					
			 				} else {
								const ctx2 = uni.createCanvasContext('myCanvas2');
								ctx2.drawImage(image, 0, 0, res.width, res.height);
								ctx2.draw();
							}
			 				
			 				req(res)
			 			},
			 		});
			 	})
			 },
			 
			 requestPort(index){
			 	 let me = this;
				 uni.showLoading({
				     title: '加载中'
				 });
			 	 const header = {
			 	 	'Cache-Control': 'no-cache, no-store, must-revalidate',
			 	 	'dmid': 'xxx-xxxxxx'
			 	 }
				const url = `http://daip-ocr-calligraphy-service.mip.dev.dm-ai.cn/api/v1/word/${index}`;
				
			 	uni.request({
			 		method:'GET',
			 	    url: url,
 
			 	    success: (res) => {
						uni.hideLoading()
						console.log('单字页面返回的res:',res);
						if(res.data.code ===0) {
							let data = res.data.data;
							const {word, confidence, score, score_stroke, score_struc,img_pred, img_gt, stroke_evaluation , struc_evaluation} = res.data.data;
							me.word = word;
							me.confidence = confidence.toFixed(2)*100+'%';
							me.score = score==null?'':score.toFixed(0);
							me.score_stroke = score_stroke == null?'':score_stroke.toFixed(0);
							me.score_struc = score_struc==null?'':score_struc.toFixed(0);
							me.img_pred = `data:image/png;base64,${img_pred}`;
							me.img_gt = `data:image/png;base64,${img_gt}`;
							
							
							for(let i in stroke_evaluation) {
								stroke_evaluation[i].img_vis = `data:image/png;base64,${stroke_evaluation[i].img_vis}`;
								stroke_evaluation[i].img_gt = `data:image/png;base64,${stroke_evaluation[i].img_gt}`;
								stroke_evaluation[i].score = stroke_evaluation[i].score==null?'':stroke_evaluation[i].score.toFixed(0)
								me.stroke_evaluation.push(stroke_evaluation[i]);
							}
							;
							
							for(let st in struc_evaluation) {
								struc_evaluation[st].img_vis = `data:image/png;base64,${struc_evaluation[st].img_vis}`;
								struc_evaluation[st].score = struc_evaluation[st].score==null?'':struc_evaluation[st].score.toFixed(0)
								me.struc_evaluation.push(struc_evaluation[st]);
							}
							
						console.log('单字页面返回的res:',res.data.data)
							 
						}
			 	        
			 	    }
			 	});
			 	 
			 },
			
			// 抽离出来的uni.$on回调方法
			callback(e) {
				let that = this;
				that.wordId = e;
				that.requestPort(e)
				console.log('单字的callback',that.wordId);
			},
		},
		async onLoad(opt) {
			let me = this;
			me.imgPath = opt.imgPath
			console.log('传进来的值',opt);
			uni.$once('wordId',me.callback);
			console.log('单字的callback',me.wordId);
			setTimeout(() => {
				me.getImageInfo(opt.imgPath)
			},500);
			 
		},
		 
	}
</script>

<style lang="scss" scoped>
	$number: 15rpx;
	$height: 220rpx;
	$padLR: 15rpx;
	$marginBot: 15rpx;
	page{background:#FFFFFF;}
	
	/deep/.uni-progress-bar {  
		border-radius: 10px; 
	}  
	
	/deep/.uni-progress-inner-bar {  
		border-radius: 5px;  
	}  
	.topContent {
		width: 100vw;
		height: $height;
		position: relative;
		background: #fff;
		margin-bottom: 2vh;
	}
	.canvas1 {
		// border: 1rpx solid green;
	}
	.itemStyle {
		height: $height;
		flex: 1;
		position: relative;
		top: 0;
		left: 0;
		text-align: center;
		.img {
			
			height: 100%;
			text-align: center;
			display: flex;
			justify-content: center;
			align-items:  center;
		}
			
		image {
			width: 100%;
			max-height: $height;
			// border: 1rpx solid green;
		}
		
		.lineBg {
			position: absolute;
			bottom: 0;
			left: 0;
			height: 40rpx;
			width: 100%;
			font-size: 16rpx;
			line-height: 40rpx;
			text-align: center;
			background:rgba(247,247,247,.75);
			.bold {
				font-size: 21rpx;
				font-weight: bold;
			}
			 
		}
		.thirdWord {
			.large {
				font-size: 60rpx;
				font-style: oblique;
			}
			.progressBox {
				margin-bottom: 10rpx;
				.txt {
					font-size: 16rpx;
				}
				.prog {
					width:61rpx;
					height:10rpx;
					margin-right: 10rpx;
					margin-left: 10rpx;
				}
			}
		}
		.noBg {
			background: none;
		}
		
	}
	.third {
		display: flex;
		flex-direction: column;
		justify-content: center;
	}
	.thirdItem {
		display: flex;
		flex-direction: column;
		justify-content: space-between;
		.stroke {
			font-size: 18rpx;
			border-top: 1rpx solid #EFEFEF;
			flex: 1;
			display: flex;
			justify-content: space-between;
			align-items: center;	
			padding: 0 $number;
			 
			.fontSize {
				font-size: 30rpx;
				font-weight: bold;
			}
		}
		.review {
			text-align: left;
			font-size: 18rpx;
			padding: 0 $number;
			border-top: 1rpx solid #EFEFEF;
			border-bottom: 1rpx solid #EFEFEF;
		}
	}
	
	.segmented {
		height: calc(100vh - 244rpx);
		background: #fff;
		border-radius: 20rpx 20rpx 0 0;
		box-shadow: 0 -5rpx 5rpx 0 #eee;
		padding: 0 10rpx 10rpx 10rpx;
		
		.firstTab {
			margin-bottom: $marginBot;
		}
		.secondTab {
			margin-bottom: $marginBot;
			display: flex;
			flex-wrap: wrap;
			.left{
				width: 50%;
				margin-bottom: 17rpx;
				display: flex;
				position: relative;
				padding-top: 10rpx;
				padding-bottom: 26rpx;
				.img {
					width: 100rpx;
					height: 100rpx;
					z-index: 2;
					image {
						max-width: 100%;
						max-height: 100%;
					}
				}
				.item {
					flex: 1;
					min-height: 100rpx;
					font-size: 18rpx;
					padding: 0 $padLR;
					z-index: 3;
					b {
						font-size: 22rpx;
					}
				}
				.grap {
					position: absolute;
					left: 0;
					bottom: 0;
					width: 100%;
					background: #E3E7EE;
					height: 10rpx;
					z-index: 1;
				}
				 
			}
		}
	}
</style>
