<template>
    <div class="wrapper">
        <div class="sentence" v-html="sentenceText"></div>
        <div class="canvas-wrapper">
            <div class="canvas-bg"
                v-bind:style="{width: canvasWidth + 'px', height: canvasHeight + 'px', 'line-height': canvasHeight + 'px'}">{{currentWord}}</div>
            <canvas id="hwCanvas"
                v-bind:width="canvasWidth"
                v-bind:height="canvasHeight"
                v-bind:style="{'margin-top': '-' + canvasWidth + 'px'}"
                v-on:touchstart="touchstartHandler($event)"
                v-on:touchmove="touchmoveHandler($event)"
                v-on:touchend="touchendHandler($event)"></canvas>
        </div>
        <div class="btn-group">
            <a @click="clear">清除</a>
            <a @click="next">{{this.isFinished ? '完成' : '下一个'}}</a>
        </div>
    </div>
</template>
<script>
module.exports = {
    name: 'handwriting',
    props: {
        sentence: String
    },
    data: function() {
        return {
            canvasWidth: '',
            canvasHeight: '',
            sentenceText: '',
            currentWord: '',
            currentWordIndex: 0,
            isFinished: false,
            down: false,
            canvasObj: '',
            canvasContext: '',
            canvasOffsetLeft: 0,
            canvasOffsetTop: 0,
            BH: '',//笔迹坐标
        }
    },
    mounted() {
        // 初始化设置canvas宽高
        this.canvasWidth = document.documentElement.clientWidth;
        this.canvasHeight = document.documentElement.clientWidth;

        this.canvasObj = document.getElementById('hwCanvas');
        this.canvasContext = this.canvasObj.getContext('2d');
        this.canvasOffsetLeft = this.canvasObj.offsetLeft;
        this.canvasOffsetTop = this.canvasObj.offsetTop;

        this.sentenceText = this.sentence ? this.sentence : '';
        this.showWroteWords(this.currentWordIndex);
    },
    methods: {
        showWroteWords: function(index) {
            if(index == this.sentence.length - 1){
                this.isFinished = true;
            }
            this.sentenceText = this.sentence;
            let currWord = this.sentenceText[index];
            this.sentenceText = `${this.sentenceText.substring(0, index)}<span class="wroted">${this.sentenceText[index]}</span>${this.sentenceText.substring(index + 1)}`;
            this.currentWord = currWord;
        },
        touchstartHandler: function(e) {
            this.down = true;
            this.canvasContext.lineCap = 'round';
            this.canvasContext.lineWidth = 5;
            this.canvasContext.beginPath();
            this.canvasContext.moveTo(this.getPosition(e).x, this.getPosition(e).y);
        },
        touchmoveHandler: function(e) {
            if(this.down){
                var x = Math.ceil(e.changedTouches[0].screenX) < 0 ? 0 : Math.ceil(e.changedTouches[0].screenX);
                var y = Math.ceil(e.changedTouches[0].screenY) < 0 ? 0 : Math.ceil(e.changedTouches[0].screenY);
                x = x > Number(e.target.getAttribute('width')) ? Number(e.target.getAttribute('width')) : x;
                y = y > Number(e.target.getAttribute('height')) ? Number(e.target.getAttribute('height')) : y;
                this.BH += x + ',' + y + ',';
                this.canvasContext.lineTo(this.getPosition(e).x, this.getPosition(e).y);
                this.canvasContext.stroke();
            }
        },
        touchendHandler: function(e) {
            this.canvasContext.closePath();
            this.BH += '-1,0,';
            this.down = false;
        },
        getPosition: function(e) {
            var canvasRect = e.target.getBoundingClientRect();
            var offsetLeft = canvasRect.left;
            var offsetTop = canvasRect.top;
            return {
                x: e.changedTouches[0].pageX - offsetLeft,
                y: e.changedTouches[0].pageY - offsetTop
            }
        },
        getBase64: function() {
            return this.canvasObj.toDataURL();
        },
        isCanvasBlank(canvas) {
            var blank = document.createElement('canvas');
            blank.width = canvas.width;
            blank.height = canvas.height;
            return this.canvasObj.toDataURL() == blank.toDataURL();
        },
        clear: function() {
            this.canvasContext.clearRect(0, 0, this.canvasObj.width, this.canvasObj.height);
            this.BH = '';
        },
        next: function() {
            // if(this.isCanvasBlank(this.canvasObj)){
            //     alert('请按水印进行抄写');
            //     return;
            // }
            console.log('next...');
            if(this.currentWordIndex < this.sentence.length - 1){
                this.currentWordIndex++;
            }
            this.showWroteWords(this.currentWordIndex);
            // this.hwRecognition();
        },
        hwRecognition: function() {
            this.BH += '-1,-1';
            console.log(this.BH);
            // 发送识别请求

        }
    }
}
</script>
<style scoped>
.sentence {
    line-height: 24px;
    padding: 5px;
}
.wroted {
    color: #0079ba;
}
.btn-group {
    clear: both;
}
.canvas-bg {
    float: left;
    text-align: center;
    font-size: 200px;
    color: #aaaaaa;
    background-size: 100%;
    background-image: url(data:image/png;base64,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);
}
#hwCanvas {
    float: left;
}
.btn-group {
    height: 50px;
    display: flex;
    justify-content: space-evenly;
    align-items: center;
}
.btn-group > a {
    width: 80px;
    height: 30px;
    line-height: 30px;
    border: 1px solid #999;
    border-radius: 5px;
    text-align: center;
}
</style>