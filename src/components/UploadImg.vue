<template>
    <div id="app">
        <div class="model" v-show="model" @click="model = false">
            <div class="model-show">
                <img :src="modelSrc" alt="">
            </div>
        </div>
        <p>例子</p>
        <div class="cut">
                 <vue-cropper ref="cropper"  :img="option.img" :output-size="option.size" :output-type="option.outputType"
                         :info="true" :full="option.full"
                         :can-move="option.canMove" :can-move-box="option.canMoveBox" :fixed-box="option.fixedBox"
                         :original="option.original"
                         :auto-crop="option.autoCrop" :auto-crop-width="option.autoCropWidth"
                         :auto-crop-height="option.autoCropHeight" :center-box="option.centerBox"
                         @real-time="realTime" :high="option.high" @img-load="imgLoad"></vue-cropper>

                         
        </div>
        <div class="test-button">
            <button @click="changeImg" class="btn">changeImg</button>
            <label class="btn" for="uploads">upload</label>
            <input type="file" id="uploads" style="position:absolute; clip:rect(0 0 0 0);"
                   accept="image/png, image/jpeg, image/gif, image/jpg"
                   @change="uploadImg($event, 1)">
            <button @click="startCrop" v-if="!crap" class="btn">start</button>
            <button @click="stopCrop" v-else class="btn">stop</button>
            <button @click="clearCrop" class="btn">clear</button>
            <button @click="refreshCrop" class="btn">refresh</button>
            <button @click="changeScale(1)" class="btn">+</button>
            <button @click="changeScale(-1)" class="btn">-</button>
            <button @click="rotateLeft" class="btn">rotateLeft</button>
            <button @click="rotateRight" class="btn">rotateRight</button>
            <button @click="finish('base64')" class="btn">preview(base64)</button>
            <button @click="finish('blob')" class="btn">preview(blob)</button>
            <a @click="down('base64')" class="btn">download(base64)</a>
            <a @click="down('blob')" class="btn">download(blob)</a>
            <!-- <button @click="uploadImgOss" class="btn">uploadOss</button> -->
            <div style="display:block; width: 100%;">
                <label class="c-item">
                    <span>上传图片是否显示原始宽高 (针对大图 可以铺满)</span>
                    <input type="checkbox" v-model="option.original">
                    <span>original: {{ option.original}}</span>
                </label>
                <label class="c-item">
                    <span>能否拖动图片</span>
                    <input type="checkbox" v-model="option.canMove">
                </label>
                <label class="c-item">
                    <span>能否拖动截图框</span>
                    <input type="checkbox" v-model="option.canMoveBox">
                    <span>canMoveBox: {{ option.canMoveBox}}</span>
                </label>
                <label class="c-item">
                    <span>截图框固定大小</span>
                    <input type="checkbox" v-model="option.fixedBox">
                    <span>fixedBox: {{ option.fixedBox}}</span>
                </label>
                <label class="c-item">
                    <span>是否输出原图比例的截图</span>
                    <input type="checkbox" v-model="option.full">
                    <span>full: {{ option.full}}</span>
                </label>
                <label class="c-item">
                    <span>是否自动生成截图框</span>
                    <input type="checkbox" v-model="option.autoCrop">
                    <span>autoCrop: {{ option.autoCrop}}</span>
                </label>
                <label class="c-item">
                    <span>是否根据dpr生成适合屏幕的高清图片</span>
                    <input type="checkbox" v-model="option.high">
                    <span>high: {{ option.high}}</span>
                </label>
                <label class="c-item">
                    <span>截图框是否限制在图片里(只有在自动生成截图框时才能生效)</span>
                    <input type="checkbox" v-model="option.centerBox">
                    <span>centerBox: {{ option.centerBox}}</span>
                </label>
                <label class="c-item">
                    <p>输出图片格式</p>
                    <label>jpg
                        <input type="radio" name="type" value="jpeg" v-model="option.outputType">
                    </label>
                    <label>png
                        <input type="radio" name="type" value="png" v-model="option.outputType">
                    </label>
                    <label>webp
                        <input type="radio" name="type" value="webp" v-model="option.outputType">
                    </label>
                </label>
            </div>
        </div>
        <div class="show-preview"
             :style="{'width': previews.w + 'px', 'height': previews.h + 'px',  'overflow': 'hidden', 'margin': '5px'
             ,background:' #FFFFFF',    border:'5px solid red'}">
            <div :style="previews.div">
                <img :src="previews.url" :style="previews.img">
            </div>
        </div>
    </div>

</template>
<script crossOrigin="*">
   import vueCropper  from 'vue-cropper'
    import  { uploadOSS } from '@/common/oss';
    export default {
        name: '',
        data() {
            return {
                model: false,
                modelSrc: '',
                crap: false,
                previews: {},
                lists: [
                   {
                         img: 'http://oss-product-test.ybm100.com/G1/M00/1C/1E/Cgoz1GCkaCCACj4DAACE92RF5cA537.jpg?x-oss-process=image/resize,p_100/rotate,0/bright,0'
                         }
                ],
                option: {
                    img: 'http://upload.ybm100.com/G4/M00/0C/FF/CiIDh2CrXAaAIabQAAOHLIlp9cY15.jpeg',//剪切图片地址
                    size: 1,//输出图片压缩比默认为1
                    outputSize:1,
                    outputType: 'png',//输出图片格式
                    info:true,// 裁剪框的大小信息
                    canScale:true,//图片是否允许滚轮缩放
                    autoCrop:true,//是否默认生成截图框
                    // 只有自动截图开启 宽度高度才生效
                   autoCropWidth: 800,//截图框宽
                    autoCropHeight: 800,//截图框高
                    fixed:false,//是否开启截图框宽高固定比例
                    //fixedNumber:[1,1],//        截图框的宽高比例
                    full: false,//输出截图是否缩放
                    canMove: true,//是否可以拖动图片
                    fixedBox: true,//是否固定截图大小
                    original: false,//上传图片按原始比例
                    canMoveBox: true,//是否可以拖动截图框
                    centerBox: false,//截图框能否超过图片
                    high: false,//是否根据dpr输出高清图片
                    infoTrue:true
                    
                },
                show: true,
                imgUrl: '',
            }
        },
        components: {
          //  'vue-cropper': cropper
        },
        methods: {
          	// 将网络图片转换成base64格式
	transBase64FromImage(image) {
		let canvas = document.createElement("canvas");
		canvas.width = image.width;
		canvas.height = image.height;
		let ctx = canvas.getContext("2d");
		ctx.drawImage(image, 0, 0, image.width, image.height);
		// 可选其他值 image/jpeg
		return canvas.toDataURL("image/png");
	},
            //更换图片
            changeImg() {
            let _this = this;
        //   this.option.img = this.lists[0].img
           let mag = this.lists[0].img
              let image = new Image();
              image.crossOrigin = "*";
		// 处理缓存
		image.src = mag;
		// 支持跨域图片
        console.info(image)
		image.onload = function () {
      console.info( _this.option)
        let base64 =   _this.transBase64FromImage(image);
           console.info(base64)
           _this.option.img = base64
		}
        
            },
            // 将网络图片转换成base64格式
                transBase64FromImage(image) {
                let canvas = document.createElement("canvas");
                canvas.width = image.width;
                canvas.height = image.height;
                let ctx = canvas.getContext("2d");
                ctx.drawImage(image, 0, 0, image.width, image.height);
                // 可选其他值 image/jpeg
                return canvas.toDataURL("image/png");
                }
                   ,
            // 自由截图
            startCrop() {
                // start
                this.crap = true
                this.$refs.cropper.startCrop()
            },
            //禁止自由截图
            stopCrop() {
                //  stop
                this.crap = false
                this.$refs.cropper.stopCrop()
            },
            //清除截图框
            clearCrop() {
                // clear
                this.$refs.cropper.clearCrop()
            },
            //重新加载图片
            refreshCrop() {
                // clear
                this.$refs.cropper.refresh()
            },
            //图片放大缩小 1 放大 | -1 缩小
            changeScale(num) {
                num = num || 1
                this.$refs.cropper.changeScale(num)
            },
            //图片向左旋转
            rotateLeft() {
                this.$refs.cropper.rotateLeft()
            },
            //图片向右旋转
            rotateRight() {
                this.$refs.cropper.rotateRight()
            },
            //转变图片格式
            finish(type) {
                // 输出
                // var test = window.open('about:blank')
                // test.document.body.innerHTML = '图片生成中..'
                //blob图片
               
                if (type === 'blob') {
                    this.$refs.cropper.getCropBlob((data) => {
                       console.info(data)
                        var img = window.URL.createObjectURL(data)
                        this.model = true
                        this.modelSrc = img
                    })
                } else {
                    //base64格式图片
                    this.$refs.cropper.getCropData((data) => {
                       console.info(data)
                        this.model = true
                        this.modelSrc = data
                    })
                }
            },
            // 实时预览函数
            realTime(data) {
                this.previews = data
                console.log(data)
            },
            //下载
            down(type) {
                // event.preventDefault()
                var aLink = document.createElement('a')
                aLink.download = 'demo'
                // 输出
                if (type === 'blob') {
                    this.$refs.cropper.getCropBlob((data) => {
                        this.downImg = window.URL.createObjectURL(data)
                        aLink.href = window.URL.createObjectURL(data)
                        aLink.click()
                    })
                } else {
                    this.$refs.cropper.getCropData((data) => {
                        this.downImg = data
                        aLink.href = data
                        aLink.click()
                    })
                }
            },
            //图片上传浏览器
            uploadImg(e, num) {
                //上传图片
                // this.option.img
                var file = e.target.files[0]
                if (!/\.(gif|jpg|jpeg|png|bmp|GIF|JPG|PNG)$/.test(e.target.value)) {
                    alert('图片类型必须是.gif,jpeg,jpg,png,bmp中的一种')
                    return false
                }
                var reader = new FileReader()
                reader.onload = (e) => {
                    let data
                    if (typeof e.target.result === 'object') {
                        // 把Array Buffer转化为blob 如果是base64不需要
                        data = window.URL.createObjectURL(new Blob([e.target.result]))
                    } else {
                        data = e.target.result
                    }
                    if (num === 1) {
                        this.option.img = data
                    } else if (num === 2) {
                        this.example2.img = data
                    }
                    console.info(num)
                     console.info(e)
                }
                // 转化为base64
                // reader.readAsDataURL(file)
                // 转化为blob
                reader.readAsArrayBuffer(file)
            },
            imgLoad(msg) {


                 console.info("加载失败："+msg)
                if(msg!=''&&msg!='success'){
                     console.info("重新加载")

            

                 let _this = this;
               let image = new Image();
               image.crossOrigin = "*";
                // 处理缓存
                image.src = "http://oss-product-test.ybm100.com/G2/M00/16/B2/Cgo0017WKVSAM4KSAABEiHCVB08381.jpg?x-oss-process=image/resize,p_69/rotate,30/bright,32";
                // 支持跨域图片
                console.info(image)
                image.onload = function () {
            console.info( _this.option)
                let base64 =   _this.transBase64FromImage(image);
                console.info(base64)
                _this.option.img = base64
                }
                }
            },
            //截图上传OSS
            uploadImgOss() {
                this.$refs.cropper.getCropData(data => {
                    //设置图片名称
                    let name = Math.floor(Math.random()*1000000)
                    //将显示图片地址情况，防止重复上传
                    this.option.img = ''
                    //将裁剪的base64图片转换成file格式
                    let file = this.dataURLtoFile(data, name)
                    //将图片上传到OSS
                    this.uploadImage(file)
                })
            },
            //将base64转换为file格式
            dataURLtoFile(dataurl, filename) {
                //将base64转换为文件
                var arr = dataurl.split(','),
                    mime = arr[0].match(/:(.*?);/)[1],
                    bstr = atob(arr[1]),
                    n = bstr.length,
                    u8arr = new Uint8Array(n)
                while (n--) {
                    u8arr[n] = bstr.charCodeAt(n)
                }
                return new File([u8arr], filename, {type: mime})
            },
            // 将裁剪完图片上传oss
            async uploadImage(event) {
                this.imgUrl = await uploadOSS(event, 0)
                console.log(this.imgUrl)
            },
        },
        mounted: function () {
            console.log(this.$refs.cropper)
        }
    }
</script>

<style>
    * {
        margin: 0;
        padding: 0;
    }

    .cut {
        width:1000px;
        height: 1000px;
        margin: 30px auto;
      /**
      background-image: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQAQMAAAAlPW0iAAAAA3NCSVQICAjb4U/gAAAABlBMVEXMzMz////TjRV2AAAACXBIWXMAAArrAAAK6wGCiw1aAAAAHHRFWHRTb2Z0d2FyZQBBZG9iZSBGaXJld29ya3MgQ1M26LyyjAAAABFJREFUCJlj+M/AgBVhF/0PAH6/D/HkDxOGAAAAAElFTkSuQmCC);
    */  
    }

    .c-item {
        max-width: 800px;
        margin: 10px auto;
        margin-top: 20px;
    }

    .content {
        margin: auto;
        max-width: 1200px;
        margin-bottom: 100px;
    }

    .test-button {
        display: flex;
        flex-wrap: wrap;
        align-content: center;
        justify-content: center;
    }

    .btn {
        display: inline-block;
        line-height: 1;
        white-space: nowrap;
        cursor: pointer;
        background: #fff;
        border: 1px solid #c0ccda;
        color: #1f2d3d;
        text-align: center;
        box-sizing: border-box;
        outline: none;
        margin: 20px 10px 0px 0px;
        padding: 9px 15px;
        font-size: 14px;
        border-radius: 4px;
        color: #fff;
        background-color: #50bfff;
        border-color: #50bfff;
        transition: all .2s ease;
        text-decoration: none;
        user-select: none;
    }

    .des {
        line-height: 30px;
    }

    code.language-html {
        padding: 10px 20px;
        margin: 10px 0px;
        display: block;
        background-color: #333;
        color: #fff;
        overflow-x: auto;
        font-family: Consolas, Monaco, Droid, Sans, Mono, Source, Code, Pro, Menlo, Lucida, Sans, Type, Writer, Ubuntu, Mono;
        border-radius: 5px;
        white-space: pre;
    }

    .show-info {
        margin-bottom: 50px;
    }
   
    .show-info h2 {
        line-height: 50px;
    }

    /*.title, .title:hover, .title-focus, .title:visited {
      color: black;
    }*/

    .title {
        display: block;
        text-decoration: none;
        text-align: center;
        line-height: 1.5;
        margin: 20px 0px;
        background-image: -webkit-linear-gradient(left, #3498db, #f47920 10%, #d71345 20%, #f7acbc 30%, #ffd400 40%, #3498db 50%, #f47920 60%, #d71345 70%, #f7acbc 80%, #ffd400 90%, #3498db);
        color: transparent;
        -webkit-background-clip: text;
        background-size: 200% 100%;
        animation: slide 5s infinite linear;
        font-size: 40px;
    }

    .test {
        height: 500px;
    }

    .model {
        position: fixed;
        z-index: 10;
        width: 100vw;
        height: 100vh;
        overflow: auto;
        top: 0;
        left: 0;
        background: rgba(0, 0, 0, 0.8);
    }

    .model-show {
        display: flex;
        justify-content: center;
        align-items: center;
        width: 100vw;
        height: 100vh;
    }

    .model img {
        display: block;
        margin: auto;
        max-width: 80%;
        user-select: none;
        background-position: 0px 0px, 10px 10px;
        background-size: 20px 20px;
    /**-moz-animation: ;  background-image: linear-gradient(45deg, #eee 25%, transparent 25%, transparent 75%, #eee 75%, #eee 100%), linear-gradient(45deg, #eee 25%, white 25%, white 75%, #eee 75%, #eee 100%);*/
        background: #ffffff;
    }

    .c-item {
        display: block;
        user-select: none;
    }

    @keyframes slide {
        0% {
            background-position: 0 0;
        }
        100% {
            background-position: -100% 0;
        }
    }

</style>