<template>
	<view>
		<!-- "总评详细"页面 -->
		<view class="topContent flexCenter">
			<view class="leftBg ">
				<view class="leftImg flexCenter">
					<image v-if="imgPath!==''" :src="imgPath" mode="aspectFit"></image>
				</view>
			</view>
			
			<view class="rightProgress">
				<view class="qiun-charts3">
					<canvas canvas-id="canvasArcbar1" id="canvasArcbar1" class="charts3"></canvas>
				</view>
			</view>
		</view>
		
		
		<view class="qiun-columns">
			<view class="qiun-title">文字得分分布</view>
			<view class="qiun-charts flexCenter" >
				<canvas canvas-id="canvasPie" id="canvasPie" class="chartsPie" @touchstart="touchPie" ></canvas>
				<!-- <canvas canvas-id="canvasRing" id="canvasRing" class="charts" @touchstart="touchRing"></canvas> -->
			</view>
		</view>
		
		<view class="wordLevel">
			<view class="wordTitle">写得最差的字</view>
			<view class="imgBox">
				<view class="imgItem"  v-for="(item, s) in worstWord" :key="s" >
					<image :src="item.img_pred" mode="aspectFit"></image>
					<!-- <canvas :style="{width:150 + 'rpx', height: 150+ 'rpx'}" canvas-id="myCanvas1" id="myCanvas1" ></canvas> -->
					<view class="opacity">文字总分：<text class="bolder">{{item.score.toFixed(0)}}</text>分</view>
				</view> 
			</view>
		</view>
		
		<view class="wordLevel">
			<view class="wordTitle">写得最好的字</view>
			<view class="imgBox">
				<view class="imgItem"  v-for="(item, s) in bestWord" :key="s" >
					<image :src="item.img_pred" mode="aspectFit"></image>
					<view class="opacity">文字总分：<text class="bolder">{{item.score.toFixed(0)}}</text>分</view>
				</view> 
			</view>
		</view>
		<!-- :src="item.img_vis" -->
		<view class="wordLevel">
			<view class="wordTitle">写得最差的笔画</view>
			<view class="imgBox">
				<view class="imgItem"  v-for="(item, s) in worstStroke" :key="s" >
					<image :src="item.img_vis" mode="aspectFit"></image>
					<view class="opacity">文字总分：<text class="bolder">{{item.score.toFixed(0)}}</text>分</view>
				</view> 
			</view>
		</view>
		
	</view>
	
</template>

<script>
	import uCharts from '@/components/u-charts/u-charts.js';
	var _self;
	var canvaRing1=null;
    var canvaArcbar1;
	var canvaRing2=null;
	var canvaArcbar2;
	var canvaPie=null;
	export default {
		data() {
			return {
				cWidth:'',
				cHeight:'',
				pixelRatio:1,
				serverData:'',
				cWidth3:'',//圆弧进度图
				cHeight3:'',//圆弧进度图
				arcbarWidth:'',//圆弧进度图，进度条宽度,此设置可使各端宽度一致
				
				Ring: {
					"series": [{
						"name": "<60分",
						"data": 50
					}, {
						"name": "60-69分",
						"data": 30
					}, {
						"name": "70-79分",
						"data": 20
					}, {
						"name": "80-89分",
						"data": 18
					}, {
						"name": "≥90",
						"data": 8
					}]
				},
				 
				progress:[{
					"name": "我的总分",
					"data": 0,
					"color": "#6236FF"
				}],
				wordLevel:[
					{
						title: '写得最好的字',
						score: 90
					},
					{
						title: '写得最差的字',
						score: 90
					},
					{
						title: '写得最差的笔画',
						score: 90
					}
				],
				imgPath:'',
				regularImg:'',
				 // 最差3个字
				"worstWord": [],
				 // 最好3个字
				"bestWord":[],
				// 最差3个笔画
				"worstStroke":[],
				imagew:0,
				imageh:0,
				screenw:0,
			}
		},
		
		
		async onLoad(opt) {
			
			let me = this;
			//#ifdef H5
			me.screenw = window.screen.availWidth
			console.log('h5宽度',window.screen.availWidth);
			//#endif
			
			//#ifdef APP-PLUS
			const system = await me.getSystemInfo();
			me.screenw = system.screenWidth
			console.log('app设备宽度',system.screenWidth)
			//#endif
			
			me.imgPath = opt.imgPath
			console.log('传进来的值00',opt);
			 uni.$once('imageVal',me.callback);
			me.requestPort()
			 
		 
			//this.cWidth=uni.upx2px(750);
			me.cWidth=uni.upx2px(600);
			// this.cHeight=uni.upx2px(500);
			me.cHeight=uni.upx2px(400);
			// me.getServerData();
			 
			me.cWidth3=uni.upx2px(220);//这里要与样式的宽高对应
			me.cHeight3=uni.upx2px(220);//这里要与样式的宽高对应
			me.arcbarWidth=uni.upx2px(14);//进度条的宽
			me.getServerData2();
			
			
						 
		},
		methods: {
			// 获取设备信息
			getSystemInfo(){
				return new Promise((req, rej) => {
					uni.getSystemInfo({
						success:function(res){
							console.log('详情页设备信息:',res)
							req(res)
						}
					})
				})
			},
			
			callback(e) {
				let me = this;
				me.imagew = e.width;
				me.imageh = e.height;
				// console.log('传进来的callback',e);
				
			},
			// 环形图
			 
			
			getServerData(){
				_self = this;
				let Ring={series:[]};
				Ring.series = _self.Ring.series;
				_self.showPie("canvasPie",Ring);
			},
			showPie(canvasId,chartData){
				canvaPie=new uCharts({
					$this:_self,
					canvasId: canvasId,
					type: 'pie',
					fontSize:11,
					legend:{show:true},
					background:'#FFFFFF',
					pixelRatio:_self.pixelRatio,
					series: chartData.series,
					animation: true,
					// width: _self.cWidth*_self.pixelRatio,
					width: _self.screenw,
					height: _self.cHeight*_self.pixelRatio,
					dataLabel: true,
					extra: {
						pie: {
						  lableWidth:15
						}
					},
				});
			},
			touchPie(e){
				canvaPie.showToolTip(e, {
					format: function (item) {
						return item.name + '：' + item.data +'个字'
					}
				});
			},
			showRing(canvasId,chartData){
				console.log('')
				canvaRing1=new uCharts({
					$this:_self,
					canvasId: canvasId,
					type: 'ring',
					fontSize:18,
					// legend:true,
					legend:{
						show:true,
						position:'right',
						float:'bottom',
					 	lineHeight:40,
					},
					title: {
						name: '35',
						color: '#000000',
						//fontSize: 25*_self.pixelRatio,
						fontSize: 80*_self.pixelRatio,
						offsetY:-20*_self.pixelRatio,
					},
					subtitle: {
						name: '平均分',
						color: 'rgba(0,0,0,0.65)',
						fontSize: 15*_self.pixelRatio,
						offsetY:30*_self.pixelRatio,
					},
					extra: {
						pie: {
						  offsetAngle: -45,
						  ringWidth: 50*_self.pixelRatio,
						  labelWidth:15
						}
					},
					background:'#FFFFFF',
					pixelRatio:_self.pixelRatio,
					series: chartData.series,
					animation: true,
					width: _self.cWidth*_self.pixelRatio,
					height: _self.cHeight*_self.pixelRatio,
					disablePieStroke: true,
					dataLabel: true,
				});
			},
			touchRing(e){
				canvaRing1.showToolTip(e, {
					format: function (item) {
						return item.name + ':' + item.data 
					}
				});
			},
			
			
			getServerData22(){
				_self = this;
				uni.request({
					url: 'https://www.ucharts.cn/data.json',
					data:{
					},
					success: function(res) {
						console.log('进度条',res.data.data.Arcbar1.series)
						let Arcbar1={series:[]};
						//这里我后台返回的是数组，所以用等于，如果您后台返回的是单条数据，需要push进去
						// Arcbar1.series=res.data.data.Arcbar1.series;
						Arcbar1.series= _self.progress;
						_self.showArcbar("canvasArcbar1",Arcbar1);
					},
					fail: () => {
						_self.tips="网络错误，小程序端请检查合法域名";
					},
				});
			},
			// 进度条
			getServerData2(){
				_self = this;
				let Arcbar1={series:[]};
				Arcbar1.series= _self.progress;
				//_self.showArcbar("canvasArcbar1",Arcbar1);
			},
			showArcbar(canvasId,chartData){
				canvaArcbar2=new uCharts({
					$this:_self,
					canvasId: canvasId,
					type: 'arcbar',
					fontSize:11,
					legend:{show:false},
					background:'#FFFFFF',
					pixelRatio:_self.pixelRatio,
					series: chartData.series,
					animation: true,
					width: _self.cWidth3*_self.pixelRatio,
					height: _self.cHeight3*_self.pixelRatio,
					dataLabel: true,
					title: {
						//name: Math.round(chartData.series[0].data*100)+'%',//这里我自动计算了，您可以直接给任意字符串
						//color: chartData.series[0].color,
						name: Math.round(chartData.series[0].data*100),
						color:'#000000',
						// fontSize: 25*_self.pixelRatio
						fontSize: 80*_self.pixelRatio,
						fontStyle:'oblique',
						fontFamily:'Roboto-BoldItalic,Roboto',
						fontWeight:'800'
					},
					subtitle: {
						name: chartData.series[0].name,//这里您可以直接给任意字符串
						//name:'我的总分',
						color: '#666666',
						fontSize: 15*_self.pixelRatio
					},
					extra: {
						arcbar:{
							type:'circle',//整圆类型进度条图
							width: _self.arcbarWidth*_self.pixelRatio,//圆弧的宽度
							startAngle:0.5//整圆类型只需配置起始角度即可
						}
					}
				});
				
			},
			
			requestPort(){
				 let me = this;
				uni.showLoading({
				    title: 'loading'
				});
				 const header = {
				 	'Cache-Control': 'no-cache, no-store, must-revalidate',
				 	'dmid': 'xxx-xxxxxx'
				 }
				uni.request({
					method:'GET',
				    url: 'http://daip-ocr-calligraphy-service.mip.dev.dm-ai.cn/api/v1/word/general', 
				    success: (res) => {
				        
						if(res.data.code == 0) {
							 
							const score = res.data.data.score*0.01;
							const dist =res.data.data.distribute;
							 me.progress = [{
							 	"name": "我的总分",
							 	"data":score,
							 	"color": "#6236FF"
							 }]
							 let Arcbar1={series:[]};
							 Arcbar1.series= me.progress;
							 me.showArcbar("canvasArcbar1",Arcbar1);
							 
							 me.Ring = {
								"series": [{
									"name": "≥90",
									"data": dist['90'],
									"color": "#0FCAA1"
								}, {
									"name": "80-89分",
									"data": dist['80'],
									"color": "#6236FF",
								}, {
									"name": "70-79分",
									"data": dist['70'],
									"color": "#9465FF"
								}, {
									"name": "60-69分",
									"data": dist['60'],
									"color": "#F7C739"
								}, {
									"name": "<60分",
									"data": dist['0'],
									"color": "#F86767"
								}]
							}
							let Ring={series:[]};
							Ring.series = me.Ring.series;
							me.showPie("canvasPie",Ring);
							 
							const {worst_word, best_word, worst_stroke } = res.data.data;
							// worstWord
							for(let a in worst_word) {
								worst_word[a].img_pred = `data:image/png;base64,${worst_word[a].img_pred}`;
								me.worstWord.push(worst_word[a])
							}
							// best_word 
							for(let best in best_word) {
								best_word[best].img_pred = `data:image/png;base64,${best_word[best].img_pred}`;
								me.bestWord.push(best_word[best])
							} 
							// worst_stroke
							for(let st in worst_stroke) {
								worst_stroke[st].img_vis = `data:image/png;base64,${worst_stroke[st].img_vis}`;
								me.worstStroke.push(worst_stroke[st])
							}
							// 从大到小排序
							me.worstWord.sort((x, y) =>{
								if (x.score < y.score) {
									return -1;
								}
								if (x.score > y.score) {
									return 1;
								}
								return 0;
							});
							me.bestWord.sort((x, y) =>{
								if (x.score < y.score) {
									return 1;
								}
								if (x.score > y.score) {
									return -1;
								}
								return 0;
							});
							
							me.worstStroke.sort((x, y) =>{
								if (x.score < y.score) {
									return -1;
								}
								if (x.score > y.score) {
									return 1;
								}
								return 0;
							})
							   console.log('详情页面返回的me.worstStroke:',me.worstStroke);
							 const ctx1 = uni.createCanvasContext('myCanvas1');
							 ctx1.setStrokeStyle('#F86767');
							 ctx1.strokeRect(172.4,17.6,220.1,76.3);
							 // ctx2.drawImage(res.tempFilePath,0,0,100,100, 0,0, 100,120)
							 ctx1.draw();
						 }
						uni.hideLoading();	
				    }
				});
				 
			},
			
		},
		 
	}
</script>

<style scoped lang="scss">
$bottom: 25rpx;
page{background:#F2F2F2;width: 750upx;overflow-x: hidden;}
.topContent {
	height: 301rpx;
	margin-bottom: $bottom;
	background: #FFFFFF;
	.leftBg {
		width: 50%;
		background: #000;
		height: 301rpx;
		.leftImg {
			width: 100%;
			height: 100%;
			margin: 0 auto;
		}
	}
	
	.rightProgress {
		flex: 1;
		
	}
}
.qiun-padding{padding:2%; width:96%;}
.qiun-wrap{display:flex; flex-wrap:wrap;}
.qiun-rows{display:flex; flex-direction:row !important;}
.qiun-columns{ 
	height: 488rpx;
	width: 100vw;
	padding-top: 25rpx;
	background: #FFFFFF;
	margin-bottom: $bottom;
	// border: 1px solid green;
}
.qiun-common-mt{margin-top:10upx;}
.qiun-title-bar{width:96%; padding:10upx 2%; flex-wrap:nowrap;}
.qiun-title{ 
	padding-left: 10upx; 
	line-height: 26rpx;
	font-weight: bold;
	font-size: 18upx;
	color: #000000;
}
.qiun-charts{
	width: 100vw;
}
.chartsPie{
	width: 100vw;
	height: 405upx;
	background-color: #fff;
	display: flex;
	align-items: center;
	justify-content: center;
}

/* 进度条 */
.qiun-charts3 {
	height: 250upx;
}
.charts3 {
	width: 250upx;
	height: 250upx;
	margin: 0 auto;
	background-color: #FFFFFF;
}
//文字级别
.wordLevel {
	background: #FFFFFF;
	padding: 19rpx 10rpx 17rpx 10rpx;
	.wordTitle {
		font-size: 18rpx;
		line-height: 26rpx;
		margin-bottom: 17rpx;
		font-weight: bold;
	}
	.imgBox {
		display: flex;
		.imgItem {
			width: (100%/3);
			height: 200rpx;
			text-align: center;
			position: relative;
			image {
				max-width: 100%;
				max-height: 100%;
			}
			.opacity {
				width: 100%;
				position: absolute;
				bottom: 0;
				height: 32rpx;
				line-height: 32rpx;
				font-size: 16rpx;
				background:rgba(247,247,247,1);
				opacity:0.79;
				 
				.bolder {
					font-size: 23rpx;
					font-weight: bold;
				}
			}
			&:nth-child(2) {
				
				margin-left: 16rpx;
				margin-right: 16rpx;
			}
		}
	}
}
</style>