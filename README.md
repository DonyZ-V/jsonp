# 利用jsonp实现百度搜索

各位同好，这篇文章是关于如何在自己的网站中实现百度搜索框的效果，所利用的是jsonp相关的知识点，所以在代码实现之前先普及一下jsonp是用来干嘛的。
> 首先，我们知道jsonp是用来跨域的，那么，为什么要跨域呢，这里就是因为浏览器有一个同源策略的机制，同源策略阻止从一个源加载的文档或脚本获取或设置另一个源加载的文档的属性。

这里可能用朋友就会疑问，什么叫做同源，同源嘛，字面意思理解就是同一个源，所以，关键是在“源”上面。
###一个**源**的定义 [(详情)](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)
> 如果协议，端口（如果指定了一个）和主机对于两个页面是相同的，则两个页面具有相同的**源**。

因为这个机制，所以，今天我们要做的在你的个人网站上实现百度搜索的阻碍就是这个，那么jsonp就是来解决这个问题的。
知道了为什么要使用jsonp，那它到底是如何实现的呢？
> 早在很久以前，为了解决这个问题，机智的前辈们就发现，Web页面上调用js文件时则不受是否跨域的影响（不仅如此，我们还发现凡是拥有"src"这个属性的标签都拥有跨域的能力，比如script、img、iframe）

首先我们来看一下，百度本身是怎么做的。
打开chrome，在chrome的调试窗口下看看百度搜索发出的请求。当输入关键字“LOL”，如下图：


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3333527-90d27bcd5111f97d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这里产生了一条类型为**script**的请求，这就是我们需要的东西，可以点开看一下里面是什么

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3333527-ea6d8b54d22403fc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们可以看到一个名为“JQuery110204094770724259329_1479977887752”的对象，这个对象里包含的，就是我们所需要的数据，比如图片上对象下方的，g , p , q , s 这些，那这个对象是怎么被请求过来的呢？


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/3333527-f75a7fc6f6f54f59.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我们分析这个地址，我把它复制下来

> https://sp0.baidu.com/5a1Fazu8AA54nxGko9WTAnF6hhy/su?wd=lol&sugmode=2&json=1&p=3&sid=1448_19036_18240_17944_21084_18560_17001_20593_21455_21409_21553_21378_21399&req=2&bs=lol&pbs=lol&csor=3&pwd=lol&cb=jQuery110204094770724259329_1479977887752&_=1479977887766

可以在链接的末尾看到我们熟悉的名字，所以我们可以知道，服务器是通过这个字段```&cb=jQuery110204094770724259329_1479977887752```将我们所需要的数据打包过来的，那我们如何接收呢，就是通过script标签的src
例如：
```<script src="http://www.baidu.com/su?wd='+关键字+'&cb=方法名';">```
> 补充：经测试链接中间的那些可有可无，所以在这里将他们都删除了，只留下最重要的不能缺少的，还有就是方法名是可以随自己的喜好随意取名的。关键字就是你在输入框里所输入的内容，我们会用this.value代替。

那么现在，这个数据就被我们加载到了自己的页面中，就在这个方法名里面，接下来我们只需要将它解析出来，就可以为我们所用了。

弄清楚了上面这些，下面就开始实现我们今天要实现的百度搜索智能提示功能。
首先是HTML+CSS部分
```
<style>
	#keywords {width: 300px; height: 30px; padding: 5px; border:1px solid #f90; font-size: 16px;}
	#relative {border:1px solid #f90; width: 310px; margin: 0;padding: 0; display: none;}
	li a { line-height: 30px; padding: 5px; text-decoration: none; color: black; display: block;}
	li a:hover{ background: #f90; color: white; }
</style>
body>
	<input type="text" id="keywords" />
	<ul id="relative"></ul>
</body>
```
然后是js部分，做一下数据解析并在页面渲染出来的工作。
```
<script>
//在这里通过myNeed函数解析数据
function myNeed(data) {
	var relative = document.getElementById('relative');
	var html = '';
	//由上文知道，我们所需要的字段都在s里面存放着
	if (data.s.length) {
		relative.style.display = 'block';
		//这里可以限制一下所需要联想词的个数，太多了也没必要
		for (var i=0; i<data.s.length-5; i++) {
			html += '<li><a target="_blank" href="http://www.baidu.com/s?wd='+data.s[i]+'">'+ data.s[i] +'</a></li>';
		}
		//渲染到页面里
		relative.innerHTML = html;
	} else {
		relative.style.display = 'none';
	}
}
window.onload = function() {
	var keywords = document.getElementById('keywords');
	var relative = document.getElementById('relative');

	//每当有键盘事件的时候都要有相对应的联想词
	keywords.onkeyup = function() {
		//如果有输入内容的时候，就生成一个script标签获取数据
		if ( this.value != '' ) {
			var oScript = document.createElement('script');
			oScript.src = 'http://www.baidu.com/su?wd='+this.value+'&cb=myNeed';
			document.body.appendChild(oScript);
		} else {
			relative.style.display = 'none';
		}
	}
	
}
</script>
```

以上就是今天我们所要讲的内容了，重点我们总结一下，首先知道如何获取数据(通过script标签)，然后再通过解析出对象中所包含的数据并渲染到页面中来达到我们的目的，实现手法多种多样，但原理都是一样的，所以弄清楚了原理，随便你想怎么玩都可以。
上面所用的demo在我的GitHub里，欢迎路过的同学star，O(∩_∩)O~







