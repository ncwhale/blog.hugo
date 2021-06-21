---
title: "Chrome 下快捷分享当前网页链接到微博的方法"
date: 2011-06-01T11:20:14+08:00
lastmod: 2011-06-01T11:20:14+08:00
draft: false
isCJKLanguage: true
---

www,其实好久米写博客了各种发现自己犯懒啊喵~(殴，快写正文啊喵！！

其实很早就想有那么一个快捷键，当咱按下去的时候，就会自动弹出一个小窗口，然后将咱滴评论放进去后就可以直接发送到微博去，最早在饭否见到了这么一个方法觉得挺不错滴：它素直接将Javascript调用做成了书签方式，然后可以点击运行，效果如下：

<a href="#" onclick="javascript:var d=document,w=window,f='http://fanfou.com/share',l=d.location,e=encodeURIComponent,p='?u='+e(l.href)+'&t='+e(d.title)+'&d='+e(w.getSelection?w.getSelection().toString():d.getSelection?d.getSelection():d.selection.createRange().text)+'&s=bm';a=function(){if(!w.open(f+'r'+p,'sharer','toolbar=0,status=0,resizable=0,width=600,height=400'))l.href=f+'.new'+p};if(/Firefox/.test(navigator.userAgent))setTimeout(a,0);else{a()}void(0); return false;"><img src="http://static.fanfou.com/img/share_button.gif" alt="分享到饭否" /></a>

推特也有类似的分享按钮而且更加复杂喵：<a href="http://twitter.com/share" class="twitter-share-button" data-count="horizontal" data-via="Kouga_">Tweet</a><script type="text/javascript" src="http://platform.twitter.com/widgets.js"></script>

但是咱想要的是快捷键而不是一个个的分享按钮啊喵！这时候就需要Chrome的扩展 "<a href="https://chrome.google.com/webstore/detail/mgjjeipcdnnjhgodgjpfkffcejoljijf">Shortcut Manager</a>"来帮忙了喵！

安装该插件后，进入设置页面，新建一个Shortcut,
1、设置它的启动快捷按键(Shortcut key)到你喜欢的而且不和浏览器冲突的一个喵~
* 注意如果不分配就会默认执行后面的动作，到时候会变得很糟糕滴喵！如果不想启用脚本就点上面那个Disable禁用了喵~
2、URL patterns:这里填写适合的URL匹配，我们想要分享任何一个页面就直接填写 * 进去喵~
3、Action: 这里是重点，如果选择 Browser action 则会执行浏览器的一些指令，我们可以在这里设置前进后退刷新等等但是和主题无关所以不要继续跑题了喵~(呼，喝口水继续喵~
我们直接选择 Execute Javascript 作为我们的目标即可喵~
(1) URL 里面可以填入需要预先载入的.js文件，不过在有GFW的情况下那样做会很卡，所以推荐不填写，然后直接使用下面推荐的Javascript脚本喵~
(2) Then execute Javascript below:
这里才是我们的重点哦喵~例如饭否的就可以直接将它提供的代码填写进去就行了，twitter滴需要稍微自己写一下但是也不复杂喵~(或者可以复制咱后面提供滴代码喵~

Description:	 这里就填入“分享到饭否”，“分享到推特”等等就好了喵~当然也可以发到某些广播服务来一本满足喵~

嗯嗯，这里顺道写下饭否和推特的共享Javascript:

分享到饭否:
<blockquote><code>javascript:var d=document,w=window,f='http://fanfou.com/share',l=d.location,e=encodeURIComponent,p='?u='+e(l.href)+'&t='+e(d.title)+'&d='+e(w.getSelection?w.getSelection().toString():d.getSelection?d.getSelection():d.selection.createRange().text)+'&s=bm';a=function(){if(!w.open(f+'r'+p,'sharer','toolbar=0,status=0,resizable=0,width=600,height=400'))l.href=f+'.new'+p};if(/Firefox/.test(navigator.userAgent))setTimeout(a,0);else{a()}void(0)</code>
</blockquote>

分享到推特:
<blockquote><code>u=location.href;t=document.title;
window.open('http://twitter.com/share?url='+encodeURIComponent(u)+'&text='+encodeURIComponent(t),
'分享 '+ t,'toolbar=0,status=0,width=526,height=236');</code></blockquote>

嗯嗯，将来有闲心的话还会继续加入分享到非死不可等Javascript吧喵？(大概