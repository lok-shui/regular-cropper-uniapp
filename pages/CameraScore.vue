<template>
	<view class="page">
		
		<!-- 拍照得分页面 -->
		<view class="topContent">
			<view class="totalCount">
				<view class="meCount">我的总分</view>
				<view class="oblique">{{score.toFixed(0)}}</view>
				<view @click="pageTo" class="circleBox">
					<view class="circle flexCenter" >
						<uni-icons type="forward" size="16" color="rgb(109,62,253)"></uni-icons>
					</view>
				</view>
			</view>
		</view>
		
		<!-- 分段 -->
		<view class="piecewise flexCenter">
			<text class="text">分数段：</text>
			<view v-for="(item, index) in piecewise" :key="index" class="flexCenter">
				<view class="dot" :style="{background:item.color}"></view>
				<text class="itemScore">{{item.score}}</text>
			</view>
		</view>
				
		<!-- 大图 -->
		<view class="canvasBox flexCenter" :style="{width:systemw + 'px', height: (systemh-144)+ 'px'}">		
			 
			<canvas :style="{width:canvasW + 'px', height: canvasH+ 'px', left: canvasL +'px', top: canvasT +'px'}" canvas-id="myCanvas" id="myCanvas" v-if="canvasW!==0"  class="canvas1"></canvas>
			
			<view v-if="showRect==true" class="reacCanvas" :style="{width:canvasW + 'px', height: canvasH+ 'px', left: canvasL +'px', top: canvasT +'px'}">
				<view  v-for="(item, index) in canvasVal" :key="index">
					<canvas 
						v-if="item.bbox"
						:style="{
							width:item.bbox[2]-item.bbox[0] + 'px', 
							height: item.bbox[3]-item.bbox[1]+ 'px',
							borderColor: item.color, 
							left:item.bbox[0]+'px',
							top:item.bbox[1]+ 'px'
							}" 
						:canvas-id="item.id"
						class="little"
						:id="item.id"
						@click="thirdA(item, index)"
					>
					</canvas>
					<view v-if="item.bbox">
						<view 
							v-if="item.error_code!==0"
							class="questionIcon"
							:style="{
								left:item.bbox[0]+'px',
								top:item.bbox[1]+ 'px'
								}" 
							 >?
						</view>
					</view>
				</view>
			</view>
			
		</view>
		
	</view>
</template>

<script>
	 
	export default {
		data() {
			return {
				systemw:0,
				systemh:0,
				SystemInfo: {}, // 设备信息
				score: 0,
				// 分段
				piecewise: [
					{
						"color":"#F86767",
						"score":"<60分"
					},
					{
						"color":"#FFDE7F",
						"score":"60-69分"
					},
					{
						"color":"#9465FF",
						"score":"70-79分"
					},
					{
						"color":"#3A99FF",
						"score":"80-89分"
					},
					{
						"color":"#0FCAA1",
						"score":"≥90"
					}
				],
				imgPath:'',
				canvasW: 0,
				canvasH: 0,
				canvasL: 0,
				canvasT: 0,
			 
				show: false,
				rectImg:'',
				littleImg:'',
				
				bbox:[0,0,130,130],
				firstImg:'',
				dataVal:[],
				canvasVal:[],
				tryCanvas:[
					{
						"id": "myCanvas0",
						"bbox": ["172.4", "17.6", "220.1", "76.3"],
						"score": 100
					}
				],
				
				pre: 0,
				pixelRatio:0,
				
				// 前端根据code和message判断，是否显示问号。点击该字，若出提示语，提示语使用message内容
				err_code: 0,
				err_message: '',
				showRect: false,
				percent: 1,
				initImageUrl: ''
					
			}
		},
		 
		watch:{
			canvasW(newVal, oldVal) {
				console.log('最新值是：'+newVal,"原来的值是："+ oldVal);
			}
		},
		methods: {
			// 跳转到总评详情
			pageTo() {
				let me = this;
				const param = {
					width: me.canvasW,
					height: me.canvasH
				}
				setTimeout(()=>{
					uni.$emit('imageVal', param)
				},1500)
				
				uni.navigateTo({
					url:"OverallDetails?imgPath="+me.imgPath
				})
			},
			
			// 单字跳转
			thirdA(item, index){
				let me = this;
				let x = Number(item.bbox[0])
				let y = Number(item.bbox[1])
				let dw = Number(item.bbox[2])
				let dh = Number(item.bbox[3])
				let id = item.id
				console.log('方框跳转：',item.error_code == 0)
				// if(item.error_code == 0) {
				// 	uni.navigateTo({
				// 		url:"./SingleWordDetail"
				// 	})
				// } else {
				// 	uni.showToast({
				// 		icon:'none',
				// 		mask:true,
				// 		title: item.error_message,
				// 	});
				// }
				// 此方法是因为之前要传方框的截图
				const ctx = uni.createCanvasContext(item.id)
				ctx.setFillStyle('#aaa');
				ctx.strokeRect(0,0,dw,dh);
				ctx.drawImage(me.firstImg,x,y, dw,dh, 0,0, dw,dh)
				ctx.draw(true,(ret)=>{ // draw方法 把以上内容画到 canvas 中。
					 
					uni.canvasToTempFilePath({ // 保存canvas为图片
						x: 0,
						y: 0,
						width: dw,
						height: dh,
						destWidth: dw,
						destHeight: dh,
						canvasId: item.id,
						quality: 1,
						success: function(res) {
							console.log('第一个0canvas的返回值：',res)
							setTimeout(()=>{
								uni.$emit('wordId',index)
							},1500)
							
							if(item.error_message !== '') {
								uni.showToast({
									icon:'none',
									mask:true,
									title: item.error_message,
								});
								setTimeout(() =>{
									uni.navigateTo({
										url:"./SingleWordDetail?imgPath="+res.tempFilePath
									})
								},1000)
							} else {
								uni.navigateTo({
									url:"./SingleWordDetail?imgPath="+res.tempFilePath
								})
							}
							
							// if(item.error_code == 0) {
							// 	uni.navigateTo({
							// 		url:"./SingleWordDetail?imgPath="+res.tempFilePath
							// 	})
							// } else {
								
							// 	uni.showToast({
							// 		icon:'none',
							// 		mask:true,
							// 		title: item.error_message,
							// 	});
							// }
							
						}
					});
					 
				})
 				
			},
			
			// 抽离出来的uni.$on回调方法
			callback(e) {
				let that = this;
				that.canvasW = JSON.parse(JSON.stringify(e.width));
				that.canvasH = e.height;
				console.log('callback',that.canvasH);
			},
			
			// 获取设备信息
			getSystemInfo(){
				let me = this;
				return new Promise((req, rej) => {
					uni.getSystemInfo({
						success:function(res){
							me.systemw = res.windowWidth;
							me.systemh = res.windowHeight;
							me.pre = res.windowWidth*0.00065;
							me.pixelRatio = res.pixelRatio;
							
							//#ifdef H5
							if(res.model=='Moto G (4') {
								me.pre = res.windowWidth*0.0009; 
							} else if(res.model == 'Pixel 2') {
								me.pre = res.windowWidth*0.00093; 
							} else if(res.model == 'Pixel 2 XL') {
								me.pre = res.windowWidth*0.00075; 
							} else if(res.model == 'iPhone') {
								me.pre = res.windowWidth*0.002; 
							} else {
								me.pre = me.percent;
								console.log('百分比1',me.pre)
							}
							//#endif
							
							//#ifdef APP-PLUS
							me.pre = me.percent;
							console.log('百分比2',me.pre)
							//#endif
							
							req(res)
						}
					})
				})
			},
			// 获取传进来的图片信息
			getImageInfos(image){
				
				return new Promise((req, rej) => {
					uni.getImageInfo({
						src: image,
						success:function(res){
							 
							req(res)
						}
					})
				})
			},
			
			reponseData(data){
				let me = this;
				me.showRect = false;
				uni.hideLoading();
				
				console.log('接口返回的数据:',data);
				if(data.statusCode&&data.statusCode == 500) {
					uni.hideLoading();
					uni.showToast({
						icon:'none',
						mask:true,
						title: "错误状态码：500",
					});
				} 
				const resData = JSON.parse(data.data);
				console.log('resData数据:',resData.data.err_code);
				if(resData.data.err_code !== 0) {
					uni.hideLoading();
				 	uni.showToast({
				 		icon:'none',
				 		mask:true,
				 		title: `错误状态码${resData.data.error_message}`,
				 	});
				 }  
				if(resData.code !== 0) {
					uni.hideLoading();
				 	uni.showToast({
				 		icon:'none',
				 		mask:true,
				 		title: `错误状态码${resData.code}`,
				 	});
				 }  
				 // ECONNRESET
				 if(resData.code == 0&&resData.data.word_list) {
					 uni.showToast({
					 	icon:'success',
					 	mask:true,
					 	title: '测评完毕！',
					 });
					me.score = resData.data.word_list.total_score;
					let arr = [] ;
					let col = '#F86767';
					const datas = resData.data.word_list;
					 
					for(let a in datas ) {
						for(let b in datas[a].bbox) {
							datas[a].bbox[b] = ((datas[a].bbox[b])*me.percent).toFixed(2);
						}
						const obj = {
							"id":'',
							"bbox":[],
							"score":0,
							"index":0,
							"color": '',
							"error_code":0,
							"error_message":""
						};
						
						obj.id = `myCanvas${a}`;
						obj.bbox = datas[a].bbox;
						obj.error_code = datas[a].error_code;
						obj.error_message = datas[a].error_message;
						if(datas[a].score !==null){
							obj.score = Number(datas[a].score).toFixed(0);
						}else {
							obj.score = datas[a].score
						}
						
						obj.index = Number(a);
						 
						if(60<=obj.score && obj.score<=69){
							col = '#FFDE7F'
						} else if(70<=obj.score && obj.score<=79){
							col = '#9465FF'
						}else if(80<=obj.score && obj.score<=89){
							col = '#3A99FF'
						} else if(90<=obj.score) {
							col = '#0FCAA1'
						}
						obj.color = col;
						arr.push(obj);
					}
					me.canvasVal = arr;
					
					me.showRect = true;
					console.log('接口百分比:', me.percent);
					console.log('处理接口数据data.data.data:',me.canvasVal)
				}
			},
			// 获取图片信息
			getImageInfo(image) {
				let me = this;
				// uni.showLoading({
				//     title: '正在测评'
				// });
				return new Promise((req, rej) => {
					uni.getImageInfo({
						src: image,
						success: function(res) {
							// uni.hideLoading() 
							
							//#ifdef APP-PLUS
							console.log("当前为app模式,将本地图径转为base64");
							
							plus.io.resolveLocalFileSystemURL(res.path, (entry) => {
								
								entry.file( (file) =>{
									let fileReader = new plus.io.FileReader();
									//fileReader.readAsText(file, 'utf-8');
									fileReader.readAsDataURL(file, 'utf-8');
									fileReader.onloadend = (evt) =>{

										const header = {
										 	"Content-Type":"application/x-www-form-urlencoded" ,
										 	'dmid': 'xxx-xxxxxx'
										 }
										let param = {
											"image":res.path
										}
										
										uni.uploadFile({
											url: 'http://daip-ocr-calligraphy-service.mip.dev.dm-ai.cn/api/v1/word',  
											filePath: res.path,
											name: 'image',
											success: (data) => {
											
												me.reponseData(data)
											    
											}
										});
										
										
									}
								} );
							}, ( e ) => {
								console.error( "Resolve file URL failed: " + e.message );
							} );
							
							//#endif
							
							req(res)
						},
					});
				})
			},
			
		},
		
		 
		async onLoad(opt) {
			let that = this;
			
			that.imgPath = opt.imgPath;
			
			//#ifdef H5
			
			uni.showLoading({
			    title: '正在测评!'
			});
			uni.uploadFile({
				url: 'http://daip-ocr-calligraphy-service.mip.dev.dm-ai.cn/api/v1/word',  
				filePath: opt.imgPath,
				name: 'image',
				success: (uploadFileRes) => {
					console.log('接口返回的',uploadFileRes)
					that.reponseData(uploadFileRes)
				}
			});
			 
			//#endif
			
			uni.$once('choosePath',that.callback);
			
			let goodsImg = await that.getImageInfos(opt.imgPath);
			that.SystemInfo = await that.getSystemInfo();
			
			if(goodsImg.errMsg == 'getImageInfo:ok'&& that.SystemInfo.errMsg == 'getSystemInfo:ok'){
				// console.log('11识别页面的图片信息',goodsImg);
				// console.log('11获取设备信息:',that.SystemInfo)
				setTimeout(()=>{
					that.getImageInfo(opt.imgPath);
					if(that.canvasW > that.SystemInfo.windowWidth) {
						const wper = that.SystemInfo.windowWidth / that.canvasW;
						that.percent = wper;
						// goodsImg.width = goodsImg.width * wper;
						// goodsImg.height = goodsImg.height * wper;
						that.canvasW = that.canvasW * wper;
						that.canvasH = wper*that.canvasH;
					}
					console.log('画布大小:', goodsImg.width,goodsImg.height);
					console.log('图片大小:', that.canvasW, that.canvasH);
					console.log('画布百分比that.percent:', that.percent);
					
					that.canvasL = (that.SystemInfo.windowWidth - that.canvasW) / 2;
					that.canvasT = (that.SystemInfo.windowHeight - 132 - that.canvasH) / 2;
					console.log('左右:', that.canvasL, that.canvasT);
					  
					const ctx = uni.createCanvasContext('myCanvas')
					ctx.setFillStyle('#aaa');
					ctx.fillRect(0, 0, that.canvasW, that.canvasH);
					ctx.drawImage(opt.imgPath, 0, 0, that.canvasW, that.canvasH)
					ctx.draw(true,(ret)=>{ // draw方法 把以上内容画到 canvas 中。
						// console.log(ret) 
						
						 console.log('22识别页面的图片信息',that.canvasW, that.canvasH);
						uni.canvasToTempFilePath({ // 保存canvas为图片
							x: 0,
							y: 0,
							width: that.canvasW,
							height: that.canvasH,
							destWidth: that.canvasW,
							destHeight: that.canvasH,
							canvasId: 'myCanvas',
							quality: 1,
							success: function(res) {
								console.log('第一个canvas的返回值：',res)
								that.firstImg = res.tempFilePath;
								
								// const ctx1 = uni.createCanvasContext('myCanvas1');
								// ctx1.setStrokeStyle('#F86767');
								// ctx1.strokeRect(172.4,17.6,220.1,76.3);
								// // ctx2.drawImage(res.tempFilePath,0,0,100,100, 0,0, 100,120)
								// ctx1.draw();
								
								// const ctx2 = uni.createCanvasContext('myCanvas2');
								// ctx2.setStrokeStyle('#0FCAA1');
								// ctx2.strokeRect(0,0,100,130);
								// // ctx2.drawImage(res.tempFilePath,130,130,100,120, 0,0, 100,120)
								// ctx2.draw();
								
							}
						});
						
					})
					
				},1500)
			} else {
				console.log('error')
			}
		},
		
	}
</script>

<style lang="scss" scoped>
	.little {
		//border: 1rpx solid red;
		position: absolute;
		padding: 0;
		border-width: 2rpx;
		border-style: solid;
		
		padding: 0;
		margin: 0;
	}
	.questionIcon {
		width: 12rpx;
		height: 15rpx;
		position: absolute;
		border: 1rpx solid blue;
		font-size: 14rpx;
	}
	.img1 {
		position: relative;
		top: 400rpx;
	}
	$number: 22rpx;
	$scoreH: 78rpx;
	.page {
	  /* #ifdef APP-PLUS */
	  position: absolute;
	  width: 100%;	   
	  height: 100%;
	 /* border: 1px solid red; */
	  box-sizing: border-box;
	  background: #000;
	   /* #endif */
	}
	.topContent {
		background: #fff;
		border-bottom: 1rpx solid #fff;
		display: flex;
		justify-content: flex-end;
	}
	/*总分*/
	.totalCount {
		// width: 240rpx;
		height: $scoreH;
		background:rgba(240,242,246,1);
		border-radius:38rpx 0 0 38rpx;
		display: flex;
		align-items: center;
		justify-content: flex-end;
		padding-left: 20rpx;
		.meCount {
			font-size: 16rpx;
			// width:64rpx;
			height:24rpx;
 			font-family:SourceHanSansSC-Regular,SourceHanSansSC;
			font-weight:bold;
			color:rgba(0,0,0,1);
			line-height:24rpx;
		}
		.oblique {
			width:72rpx;
			height:$scoreH;
			line-height: $scoreH;
			font-style: oblique;
			font-weight: bold;
			font-size: 64rpx;
			font-family:Roboto-BoldItalic,Roboto;
			color:rgba(0,0,0,0.85);
			margin-right: 11rpx;
		}
		.circleBox {
			width: 50rpx;
			height: $scoreH;
			line-height: $scoreH;
			display: flex;
			align-items: center;
			padding-left:8rpx;
		}
		.circle {
			width: $number;
			height: $number;;
			border-radius:11rpx;
			line-height:$number;
			text-align: center;
			font-size: 14rpx;
			color: rgb(109,62,253);
			border:3rpx solid rgb(109,62,253);
		}
	}
	.piecewise {
		height: 64rpx;
		background: #fff;
		border-radius:0 0 12rpx 12rpx;
		text {
			font-size: 16rpx;
			color:rgba(0,0,0,.65);
		}
		.text {
			margin-right: 24rpx;
		}
		.dot {
			width: 14rpx;
			height: 14rpx;
			margin-right: 5rpx;
			border-radius: 7rpx;
		}
		.itemScore {
			margin-right: 24rpx;
		}
	}
	.canvasBox {
		position: relative;
		overflow: auto;
		top: 0;
		left: 0;
		.canvas1, #myCanvas {
			position: absolute;
			top: 0;
			left: 0;
			// border: 2rpx solid #0FCAA1;
		}
		.reacCanvas {
			position: absolute;
			top: 0;
			left: 0;
		}
		 
	 
	}
	
	.try {
		position: absolute;
		top: 600rpx;
		border: 1rpx solid red;
	}

</style>
