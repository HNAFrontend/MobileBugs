# HNA前端团队官方文档
------
好记性不如烂笔头，团队秉承脚踏实地的实干精神，从工作和学习中不断积累成长的力量，坚信终有一天能够成为受人尊敬的前端团队！

本文的主题是移动端Bug记录，旨在记录工作中的移动端功能与体验的坑，让自己成长。

## 1.flex下文本溢出隐藏失效的bug

### 描述：
* 父元素设置display:-webkit-flex和display:flex布局下的子元素（设置了flex:n）是自动适应，使用text-overflow:ellipsis和white-space:nowrap以及overflow:hidden无法实现多余内容的加省略号的效果

### 原因：
* 暂时未知

### 解决方案：
* 子元素加上width：0%

### 方案原理：
* 暂时未知

## 2.iOS下元素添加click事件后元素的样式问题

### 描述：
* iOS下元素被点击，该点击是由click触发，则元素底色会有一个灰底，在很多时候这个是不需要的，会给页面的体验带来误导

### 原因：
* iOS对tap有个默认的高亮反馈

### 解决方案：
* 使用touch事件来替换click事件
* 通过设置高亮的样式为透明，-webkit-tap-highlight-color: rgba(0, 0, 0, 0)

### 方案原理：
* 不触发tap
* 将tap高亮的样式设置为透明，则用户就无法感知到

## 3.通过window.innerWidth获取屏幕宽度出错

### 描述：
* 红米note在网页上第一次打开页面，并执行window.innerWidth获取的值是960，但是刷新后几次获取的值为360

### 原因：
* 红米是神机，一切皆有可能

### 解决方案：
* 在延时方法里面获取，如下

```javascript
    setTimeout(function(){
        var screenWidth = window.innerWidth;
    },0);
```

### 方案原理：
* 移动端的取值异常，君不要着急，先用setTimeout来试试

## 4.iOS & Android下表单控件表现的不一致

### 描述：
* android & ios & else系统的浏览器下表单的默认样式各不相同

### 原因：
* 群魔乱舞，各显神通

### 解决方案：
* 在reset中将表单进行重置和统一

```css
    input[type=button]{
        -webkit-appearance:none;
        outline:none
    }
```

### 方案原理：
* 一统江湖

## 5.window.innerHeight首次获取总是为0

### 描述：
* android下把qq浏览器x5内核打包进行webview解析，当渲染页面是通过window.innerHeight获取视口高度时首次总是为0。

### 原因：
* 未知，待排查

### 解决方案：
*通过定时器一直获取直到获取到视口的高度为止

```javascript
    var h = 0;
    var timer = setInterval(function(){
        h = window.innerHeight;
        if(h > 0){
            clearInterval(timer);
        }
    },10);
```

### 方案原理：
* 通过多次测试发现，使用window.innerheight并不是获取不到值，只是在首次获取的时候总是为0，那意味着我们是否可以通过轮询的方式获取该值，于是使用定时器的方式来处理，解决了window.innerheight获取失败的问题。
* 

## 6.iOS8.1.2以下服务器时间如 “2016-08-12 22：00：00”，前端解析时会出错

### 描述：
* iOS8.1.1使用 new Date('xxx')的时候，然后通过getFullyear（）等api时获取到的值是NAN

### 原因：
* 猜测是webview上js的内置date对象内部实现的差异

### 解决方案：
*为了避免这个问题，只能避免使用new date的方法来转换，使用字符串操作的处理，以兼容所有的系统版本。

```javascript
    var date = '2016-08-31 23:59:59';

    var sDate = date.split(' ')[0].split('-');

    var tmp = sDate[0] + '年' + sDate[1] + '月' + sDate[2] + '日'
    tmp 最终的值是 "2016年08月31日"
```

### 方案原理：
* 版本的探测太过繁琐且不靠谱儿，所以将日期的处理使用自己的字符串的处理来进行兼容多系统版本。

## 7.Object.assign在一些浏览器中的支持性不太好，导致使用时产生的问题，但其实不算是bug
### 描述：
* android部分机器的微信webview中都无法识别Object.assign方法

### 原因：
* 内置的浏览器内核不支持这个ES6的特性

### 解决方案：
*对这个系统方法进行兼容处理，自定义的方法暂时只支持2个对象间的浅拷贝

```javascript
     function assign() {
            var args = arguments;
            if(Object.assign){
                return Object.assign(args);
            }
            if(args.length <= 0){
                return {};
            }else if(args.length === 1){
                return args[0];
            }else{
                var originObj = args[0];
                var sourceObj = args[1];
                function copy(targetData,sourceData) {
                    for(var i in sourceData){
                        //如果目标的属性是个空或者数值，则直接整个子树拷贝过去
                        if(typeof targetData[i] !== 'object'){
                            targetData[i] = sourceData[i];
                        }else{
                            if(typeof sourceData[i] === 'object'){
                                copy(targetData[i],sourceData[i]);
                            }else{
                                targetData[i] = sourceData[i];
                            }
                        }
                    }
                }
                copy(originObj,sourceObj);
                return originObj;
            }
        }

```

### 方案原理：
* 如果系统支持Object.assign对象拷贝方法则使用系统的方法，否则使用自定义的兼容方法

## 8.微信分享给朋友，缩略图无法显示的bug

### 描述：
* 在微信里面将一个连接分享给微信好友时，分享人可以可以看到分享的缩略图，但是被分享人看不到分享的缩略图，但是标题和描述是正常显示的
### 原因：
* 分享的标题、描述存在“红包”、“领取”这样的敏感词

### 解决方案：
* 修改文案，对传播文案进行适当调整。

## 9.Vuejs数据渲染偶发性失败的bug
### 描述：
* 使用Vuejs渲染页面的DOM时，在mouted事件里面使用ajax获取数据然后赋值给页面的数据Model，会偶发性地失败
### 原因：
* 页面上有部分动画是使用setInterval开发的动画，去掉这个部分动画渲染没有问题，推测是setInterval重复修改某个dom的属性导致Vuejs渲染的失败

### 解决方案：
* 谨慎使用消耗性能的一些操作
