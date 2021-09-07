<template>
	<view>
		<view class="content">
			 <image-cropper :src="tempFilePath" @confirm="confirm" @cancel="cancel"></image-cropper>
			 
			<view class="imgBox">
				<image  src="../static/index-btn-1.png" @tap="upload" class="meslist_img" mode="widthFix"></image>
			</view>
			
			 <!--  注意这里支付宝一定要加下面的东东。我也不明白为什么，明明 在组件里都有了，还用在这里定义顺便也定义了crop-width和crop-height-->
			 <!--  #ifdef MP-ALIPAY -->
			 <canvas  id="myCanvas"  canvas-id="myCanvas" class="meslist_canvas" crop-width="200"  crop-height="200"></canvas>
			 <!--  #endif-->

		</view>
	 
			
	</view>
</template>

<script>
	import ImageCropper from "../components/ling-imgcropper/ling-imgcropper.vue";
	
	export default {
		 components: {
			ImageCropper,
		 },
		 
		 data() {
			return {
			  tempFilePath: "",
			  cropFilePath: "",
			  baseImg:'',
			  param:{
			  	width:500,
			  	height:500
			  },
			   
			};
		 },
		 
		methods: {
			//base64转成blob
			dataURLtoBlob(dataurl) {  
				var arr = dataurl.split(','),  
					mime = arr[0].match(/:(.*?);/)[1],  
					bstr = atob(arr[1]),  
					n = bstr.length,  
					u8arr = new Uint8Array(n);//8位无符号整数，长度1个字节  
				console.log(mime)  
				while (n--) {  
					u8arr[n] = bstr.charCodeAt(n);  
				}  
				// console.log(JSON.stringify(u8arr));  
				return new Blob([u8arr], {  
					type: mime  
				});  
			},  
		  // 获取裁剪图片信息
		   getImageInfo(image) {
				let that = this;
				return new Promise((req, rej) => {
					uni.getImageInfo({
						src: image,
						success: function(res) {
							uni.hideToast();
							console.log('裁剪后的图片信息',res)
							that.param.width = res.width;
							that.param.height = res.height;
							// that.tempFilePaths.transform.rotate(90)
							 
							req(res)
						},
					});
				})
			},
		  
			upload() {
			  uni.chooseImage({
				count: 1, //默认9
				sizeType: ["original"], //可以指定是原图还是压缩图，默认二者都有
				sourceType: ["album","camera"], //从相册选择
				success: (res) => {
				  this.tempFilePath = res.tempFilePaths.shift();
				},
			  });
			},
			
			confirm(e) {
				console.log('e',e)
			  // this.tempFilePath = "";
			  this.cropFilePath = e.detail.tempFilePath;
			  
			  this.getImageInfo(this.cropFilePath)
			  
			  this.baseImg = this.cropFilePath;
			  
			  //#ifdef H5
			  let blob = this.dataURLtoBlob(this.baseImg)
			  let blobUrl = URL.createObjectURL(blob);  
			  this.baseImg = blobUrl;
			  //#endif
			  
			  setTimeout(()=>{
			  	uni.$emit('choosePath',this.param)
			  },1500)
			  
			   // console.log('裁剪后的图片',this.baseImg)
			  
			  uni.navigateTo({
			  	url:'CameraScore?imgPath='+this.baseImg
			  })
		
			  // #ifdef APP-PLUS||MP
		
		
			  // #endif
			},
			cancel() {
			  console.log("canceled");
			  this.tempFilePath = "";
			},
		},
		
		  // onload排在methods后面
		 async onLoad(opt) {
			let that = this;
			console.log('传进来的图片信息',that.tempFilePath)
			setTimeout(() =>{
			  that.tempFilePath = opt.tempFilePaths;
			},1000)
			
		 },
	
	};
</script>

<style scoped>
	page{
		background-color: #FFFFFF;
		padding-top:20rpx;
		border: 1px solid green;
	}
	 
	.uni-modal__btn_primary {
		background-color: pink;
	}
	.uni-app--maxwidth {
		border: 1px solid green;
	}
	.content{
		background-color: #FFFFFF;
		
		height: calc(100vh - 44px);
		/* #ifdef APP-PLUS||MP  */
		height: calc(100vh);
		
		/*  #endif  */
		display: flex;
		align-items: center;
		justify-content: center;
		/* border: 1px solid red; */
		position: absolute;
		top: 0;
		left: 0;
	}	
	.img-result{
		margin-top:20rpx;
	}
	.imgBox {
		display: flex;
		justify-content: center;
		align-items: center;
		width: calc(100vw);
		 
		background-color: #FFFFFF;
	}
	.meslist_img {
		background: #FFFFFF;
	}
	
	.popup {
		position: absolute;
		top: 0;
		left: 0;
		display: flex;
		justify-content: center;
		align-items: center;
		height: 100vh;
		width: 100vw;
		background: rgba(0, 0, 0, .65);
		z-index: 9999;
		background-color: gray;
	}
	.popup .popContent {
		width: 320px;
		height: 288px;
		background-color: #fff;
		padding: 55px 40px 40px 40px;
		border-radius:8px;
		text-align: center;
		margin: auto;
	}
	.popup .popContent .firstLine {
		font-size: 24px;
		height:33px;
		color:rgba(0,0,0,0.85);
		font-weight: 600;
	}
	.popup .popContent .text2 {
		width: 100%;
		height:44px;
		font-size: 16px;
		margin-top: 34px;
		color:rgba(0,0,0,.65);
		line-height: 22px;
	}
	.popup .popContent button {
		width:240px;
		height:56px;
		line-height: 56px;
		background:linear-gradient(193deg,rgba(255,56,95,1) 0%,rgba(219,11,23,1) 100%);
		border-radius:40px;
		margin-top: 26px;
		font-size: 16px;
		color: #fff;
	}
	
</style>
