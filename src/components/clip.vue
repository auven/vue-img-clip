<template>
    <div>
        <div class="clip-img" id="clipImg">
            <div class="msg">请选择一张图片用于合成</div>
            <img :src="clipUrl" alt="" v-if="clipUrl">
            <div class="textBox" id="textBox">
                <p class="tab">{{ text1 == '' ? '全场最佳' : text1 }}</p>
                <p class="name">{{ text2 == '' ? '玩家名称' : text2 }}</p>
                <p class="hero">{{ text3 == '' ? '所选英雄' : text3 }}</p>
            </div>
        </div>

        <div class="input">
            <input type="text" class="text" placeholder="全场最佳" v-model="text1">
            <input type="text" class="text" placeholder="玩家名称" v-model="text2">
            <input type="text" class="text" placeholder="所选英雄" v-model="text3">
        </div>
        <div class="clip-wp" id="clip-wp" v-show="isClip">
            <div id="container"></div>
            <button id="save-img" @click="saveImg">完成</button>
        </div>
        <div class="upload-wp">

            <ul>
                <li @click="openImg">选择图片</li>
                <li @click="addText">合成</li>
            </ul>
            <input type="file" class="file" id="file" accept="image/*;capture=camera" name="img" @change="clipImg($event)">
        </div>
        <div class="preview-wp" v-if="isPreview">
            <img class="preview-img" :src="saveUrl">
            <div class="msg">长按保存到手机</div>
            <button class="back" @click="cancelPreview">返回</button>
        </div>
    </div>
</template>

<script>
import clip from '../clip/clip';

export default {
    data() {
        return {
            isClip: false,
            isPreview: false,
            isUpload: false,
            clipUrl: '',
            noScoll: function(evt) {
                this.isClip && evt.preventDefault();
            },
            clip: {},
            text1: '',
            text2: '',
            text3: '',
            saveUrl: ''
        }
    },
    methods: {
        cancelPreview() {
            this.isPreview = false;
        },
        clipImg(event) {
            console.log('图片改变，图片路径为', event.target.value);
            if (event.target.value) {
                this.clip = new clip('container', this);
                this.clip.init(event.target.files[0]);
                this.isClip = true;
                document.body.addEventListener('touchmove', this.noScoll, false);
            }
        },
        saveImg() {
            this.isClip = false;
            this.clip.save();
            document.body.removeEventListener('touchmove', this.noScoll, false);

        },
        uploadImg() {
            //上传到服务器或进行相关操作
            this.isUpload = true;
        },
        addText() {
            if (!this.clip.save) {
                alert('请选择图片');
                return;
            }
            if (this.text1 == '' && this.text2 == '' && this.text3 == '') {
                alert('请输入内容！');
            } else {
                this.clip.save(this.text1, this.text2, this.text3);
                this.isPreview = true;
            }
        },
        save2png() {
            this.clip.save2png(this.clipUrl);
        },
        openImg() {
            document.getElementById("file").click();
        }
    },
    mounted: function() {
        //do something

        let textBox = document.getElementById("textBox");
        let draging = false;

        //记录初始点击点的pageX，pageY。用于记录位移
        //            let pageX = 0;
        //            let pageY = 0;
        let pageX = 0;
        let pageY = 0;

        //初始textBox的坐标
        let startX = 0;
        let startY = 0;

        textBox.addEventListener('touchmove', function(ev) {
            let e = ev.touches[0];

            //偏移量
            let offsetX = e.pageX - pageX;
            let offsetY = e.pageY - pageY;
            if (draging) {
                let sX = startX + offsetX;
                let sY = startY + offsetY;

                let maxX = document.getElementById('clipImg').clientWidth - textBox.offsetWidth;
                let maxY = document.getElementById('clipImg').clientHeight - textBox.offsetHeight;

                textBox.style.left = Math.min(Math.max(sX, 0), maxX) + "px";
                textBox.style.top = Math.min(Math.max(sY, 0), maxY) + "px";
            }
        });
        textBox.addEventListener('touchstart', function(ev) {
            let e = ev.touches[0];
            draging = true;

            pageX = e.pageX;
            pageY = e.pageY;

            startX = textBox.offsetLeft;
            startY = textBox.offsetTop;
        });
        textBox.addEventListener('touchend', function() {
            draging = false;
        })

        let setFS = function() {
            let boxArr = textBox.getElementsByTagName("p");
            let w = document.getElementById('clipImg').clientWidth;
            textBox.style.textShadow = (2 / 1280) * w + 'px ' + (2 / 1280) * w + 'px ' + (5 / 1280) * w + 'px rgba(0,0,0,0.2)';
            boxArr[0].style.fontSize = (45 / 1280) * w + 'px';
            boxArr[1].style.fontSize = (70 / 1280) * w + 'px';
            boxArr[1].style.paddingLeft = (25 / 1280) * w + 'px';
            boxArr[1].style.textShadow = (2 / 1280) * w + 'px ' + (2 / 1280) * w + 'px ' + (5 / 1280) * w + 'px rgb(239, 156, 51)';
            boxArr[2].style.fontSize = (33 / 1280) * w + 'px';
            boxArr[2].style.paddingLeft = (50 / 1280) * w + 'px';
        };

        window.onresize = setFS();
        setFS();
    }
}
</script>

<style lang="less">
body {
    padding: 0;
    margin: 0;
    background: rgba(165, 167, 171, .3);
}

.clip-img {
    position: relative;
    width: 100%;
    padding-bottom: 56.25%;
    height: 0;
    background: #fff;
    overflow: hidden;

    .msg {
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        color: #a5a7ab
    }

    img {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
    }
    div.textBox {
        position: absolute;
        top: 30%;
        font-style: italic;
        padding: 0.2rem;
        p {
            padding: 0;
            color: #ffffff;
            font-size: 1rem;
            margin: 0;
            text-shadow: 0.05rem 0.05rem 0 rgba(0, 0, 0, 0.2);
            font-weight: bold;
        }
        p:nth-of-type(2) {
            color: rgb(239, 179, 52);
            text-shadow: 0.05rem 0.05rem 0 rgb(239, 156, 51);
            font-size: 1.5rem;
            padding-left: 0.5rem;
        }
        p:last-of-type {
            font-size: 0.8rem;
            padding-left: 1rem
        }
    }
}

.upload-wp {
    position: fixed;
    text-align: center;
    width: 100%;
    height: 50px;
    line-height: 50px;
    background: #ffffff;
    overflow: hidden;
    bottom: 0;
    color: #222;
    ul {
        margin: 0;
        padding: 0;
        li {
            width: 50%;
            box-sizing: border-box;
            float: left;
            text-align: center;
            list-style: none;
            border-right: 1px solid rgba(165, 167, 171, .3);
        }
        li:last-of-type {
            border-right: none;
        }
    }
    .file {
        display: none;
    }
}

.clip-wp {
    position: fixed;
    width: 100%;
    top: 0;
    bottom: 0;
    z-index: 11;
    background-color: #000;
    text-align: center;
    #container {
        background-color: #000;
        text-align: center;
        width: 100%;
        left: 0;
        right: 0;
        top: 20px;
        bottom: 80px;
        margin: 0 auto;
        position: absolute;
    }
    #save-img {
        position: absolute;
        bottom: 20px;
        width: 90%;
        left: 5%;
        height: 42px;
        line-height: 42px;
        color: #fff;
        background-color: #32c47c;
        border-radius: 20px;
    }
    #image-box {
        position: absolute;
        left: 0px;
        right: 0px;
        bottom: 0px;
        top: 0px;
        margin: auto;
    }
    #cover-box {
        position: absolute;
        z-index: 9999;
        display: none;
        left: 0px;
        right: 0px;
        bottom: 0px;
        top: 0px;
        margin: auto;
    }
}

.preview-wp {
    text-align: left;
    position: fixed;
    top: 0;
    bottom: 0;
    width: 100%;
    background-color: #000;
    overflow: auto;
    .preview-img {
        position: absolute;
        top: 50%;
        width: 100%;
        transform: translate(0, -50%);
        -webkit-transform: translate(0, -50%);
        border: none;
    }
    .msg {
        position: absolute;
        bottom: 82px;
        width: 90%;
        left: 5%;
        height: 42px;
        line-height: 42px;
        color: #fff;
        text-align: center;
    }
    .back {
        position: absolute;
        bottom: 20px;
        width: 90%;
        left: 5%;
        height: 42px;
        line-height: 42px;
        color: #fff;
        background-color: #32c47c;
        border-radius: 20px;
        text-align: center;
    }
}

.input {
    width: 95%;
    margin: 0 auto;
    .text {
        display: block;
        width: 100%;
        border: none;
        border-bottom: 1px solid rgba(165, 167, 171, 1);
        margin-top: 0.5rem;
        padding: 0.6rem 0;
        background: none;
    }
}
</style>