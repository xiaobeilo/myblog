---
title: '机智如我:利用canvas画布批处理图片'
date: 2016-10-16 12:49:04
tags: [机智如我,canvas,自动下载]
thumbnail: http://pic5.photophoto.cn/20071029/0010023968667557_b.jpg

---

放假在家中休息的时候,嫂子给了我一个好差事:`将她公司的照片文件按序号标注并且两张两张的形式合并为一张.`我大概数了一下,大概是6个文件夹,每个文件夹约有七八十张照片,这大概有四五百张的图片,如果每张按照ps来处理一下,得,我这假期不用过了.光处理这图片这假期都不够用的呢.
好吧,既然是我嫂子的要求,我先来观察一下这堆文件：

>1. 文件夹内的照片命名混乱;
>2. 每张照片图片大小尺寸不一样,并且有些尺寸相差巨大;

那么我现在需要做的是:
>1. 将图片按照数字顺序重新命名;
>2. 将图片右下角打上序号标志,并且与图片文件名称一致;
>3. 将每个文件夹内的图片两两合并,并且文件夹名为 `文件名1-文件名2.xxx`;

好,那么思考一下,我们可以这么做:
>1. 图片重命名可以用软件进行批处理,像好压压缩包就有自带.
>2. 图片右下角打上序号..这个..其实也是有软件解决的.(ImageTuner)
>3. 图片两两合并..这个还真没有软件帮我了!

好,那么问题就出在图片两两合并!怎么解决呢?怎么解决呢??
有了!我们可以利用canvas的画布功能!
### 首先,我们创建一个画布,然后为这个画布添加dragover和drop事件:
```html
<button id="start">开始!</button>
<canvas id="canvas" width="500" height="800"></canvas>
<!-- 假设我们需要输出500x??高度的图片,高度暂定800 -->
<div id="download">
    <!-- 这里存放完成处理后的a标签 -->
</div>
```
习惯对象字面量的写法了:
```javascript
var batchImg = {
    canvas:document.getElementById('canvas'),
    download:document.getElementById('download'),
    ctx:this.canvas.getContext('2d'),
    cWidth:this.canvas.width,//保存canvas的宽度
    fileList:null,//保存所有的图片文件
    fileNameList:[],//只保存文件名,方便输出 文件名1-文件名2.jpg 这样的文件
    curInx:0,//当前处理第几张图片
    allH:0, //记录上一张图片画的高度,下一张从这开始画
    init:function(){
        var me = this;
        this.canvas.addEventListener('dragover',function(e){
            e.preventDefault();
            e.stopPropagation();
            e.dataTransfer.dropEffect = 'copy';
        },false);
        this.canvas.addEventListener('drop',function(e){
            e.preventDefault();
            e.stopPropagation();
            me.fileList = e.dataTransfer.files;
            for(let i=0;i<me.fileList.length;i++){
                me.fileNameList.push((me.fileList[i].name+'').replace(/\.\w+/,''));
            }
        },false);
        document.getElementById('start').onclick = this.addImg.bind(this);
    }
```
好的,初始化完毕,我们来看看文件拖拽到canvas内是否可以读取到文件:

![图片链接](/images/file-list.png)

ok!成功!

### 我们再来为batchImg添加一个addImg()的方法:
```javascript
addImg:function(){
    var me = this;
    if(!this.fileNameList.join('')){
        alert('请先将2张以上的图片拖入黑框内!');return false;
    }else if(this.fileNameList.length-this.curInx === 0){
        alert('处理完毕!');return false;
    }else if(this.fileNameList.length-this.curInx === 1){
        alert('还有一张没有处理'+this.fileNameList[this.fileNameList.length-1]);
        return false;
    }
    /*this.ctx.fillStyle= '#fff';
    这里其实本来想用canvas的fillText方法来给图片右下角打标记,但是fillText的参数只能用px,而px是相对于原图片大小尺寸计算的.难以做到每张图片序号大小一致.
    */
    this.ctx.clearRect(0,0,this.cWidth,this.height);//每次重新画的时候都要清除一下画布
    
    var count = 0;//记录图片数量,当等于2时,触发output方法
    var imgs = [];
    for(let i=this.curInx;i<this.curInx+2;i++){
        var file = this.fileList[i];
        var reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = function(e){
            let img = new Image();
            let imgH = 0;
            img.src = this.result;
            img.onload = function(){
                count += 1;
                imgs.push(this);
                count === 2 && me.output(imgs);
            }
        }
    }
    this.curInx+=2;
}
```
**这里有个坑:canvas如果给height属性重新赋值,便会清除画布!!**
**所以,这里不能画完一张图片就去定一次canvas高度,而要在所有图片开始画之前就计算好所需要的高度!!**
**所以的我解决方案是创建一个imgs数组,保存所有需要画的图片,然后在下一次的方法中绘制.**
```javascript
output:function(imgs){
    var me = this;
    var imgH0 = me.cWidth*imgs[0].height/imgs[0].width;
    var imgH1 = me.cWidth*imgs[1].height/imgs[1].width;
    //按照原图尺寸转到500宽度的比例缩放高度.
    this.canvas.height = imgH0 + imgH1;
    this.ctx.drawImage(imgs[0],0,0,me.cWidth,imgH0);
    this.ctx.drawImage(imgs[1],0,imgH0,me.cWidth,imgH1)
    this.canvas.toBlob(function(blob){
        var aLink = document.createElement('a');
        url = URL.createObjectURL(blob);
        aLink.href = url;
        aLink.innerText += me.fileNameList[me.curInx-2] + '-' + me.fileNameList[me.curInx-1] + '.jpeg';
        // var evt = document.createEvent('HTMLEvents');
        // evt.initEvent('click',false,false);
        var evt = new MouseEvent('click')
        aLink.download = me.fileNameList[me.curInx-2] + "-" + me.fileNameList[me.curInx-1];
        aLink.dispatchEvent(evt);

        me.download.appendChild(aLink);
        me.addImg();
    },'images/jpeg');
}
```
**14行注释部分也是一个坑!!仅仅一个月的时间,谷歌升级到53版本的时候,initEvent方法已经失效了!!**
具体的情况是:在创建a链接的时候无法让浏览器实现自动下载.
**具体可以参考[MDN](https://developer.mozilla.org/en-US/docs/Web/API/Event/initEvent)链接**
推荐使用new MouseEvent()的方法.可以实现浏览器的自动下载.

{% asset_img GIF.gif 大功告成 %}
![](/images/canvas-gif.gif)

### ok!大功搞成,我们来看看最终效果!



如果遗漏了哪张,是可以点击对应的a标签重新下载的.
耶,又过了一个愉快的假期~

