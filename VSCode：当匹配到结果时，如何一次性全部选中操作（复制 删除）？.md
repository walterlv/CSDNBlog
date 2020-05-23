

最近需要处理几十万行的文字，然后提取出数千行（嗯，我在做输入法词库）。在 VSCode 里我用正则匹配到了想要的结果后，如何能够快速把这些行提取出来呢？

---

其实非常简单，Alt + Enter 即可选中所有已经匹配到的文字。

来，我们看这个具体的例子：

这里有一个几十万行的词库，我需要将其中的英文部分提取出来做成单独的词库。于是我使用正则表达式，匹配到所有英文词。

![匹配文字](https://blog.walterlv.com/static/posts/2020-04-07-11-04-56.png)

接着，按下 Alt + Enter 我就可以复制出所有的已匹配的词。将其粘贴出来即形成新的纯英文词库。

![已选中文字](https://blog.walterlv.com/static/posts/2020-04-07-11-08-48.png)

![新的词库文件](https://blog.walterlv.com/static/posts/2020-04-07-11-09-39.png)

---

我的博客会首发于 <https://blog.walterlv.com/>，而 CSDN 会从其中精选发布，但是一旦发布了就很少更新。

如果在博客看到有任何不懂的内容，欢迎交流。我搭建了 [dotnet 职业技术学院](https://t.me/dotnet_campus) 欢迎大家加入。

![知识共享许可协议](https://img-blog.csdnimg.cn/20190406094629787.png)

本作品采用[知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议](http://creativecommons.org/licenses/by-nc-sa/4.0/)进行许可。欢迎转载、使用、重新发布，但务必保留文章署名吕毅（包含链接：<https://walterlv.blog.csdn.net/>），不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请[与我联系](mailto:walter.lv@qq.com)。
