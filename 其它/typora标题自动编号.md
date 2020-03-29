## 说明

## 目录

## 方法

> - 文件-->偏好设置-->打开主题文件夹
>
> - 新建一个css文件base.user.css，内容如下
>
>   ```css
>   /* 正文标题区: #write */
>   /* [TOC]目录树区: .md-toc-content */
>   /* 侧边栏的目录大纲区: .sidebar-content */
>   
>   /** 
>    * 说明：
>    *     Typora的标题共有6级，从h1到h6。
>    *     我个人觉得h1级的标题太大，所以我的标题都是从h2级开始。
>    *     个人习惯每篇文章都有一个总标题，有一个目录，所以h2级的标题前两个都不会计数。
>    *     一般情况下，我虽然不使用h1级的标题，但是为了以防万一，h1级的标题前两个也都不会计数。
>    *     若想启用h1级标题，就取消包含“content: counter(h1) "."”项的注释，然后将包含“content: counter(h2) "."”的项注释掉即可。
>    */ 
>   /** initialize css counter */
>   #write, .sidebar-content,.md-toc-content{
>   	/* 设置全局计数器的基准 */
>   	/* 因为我喜欢从h2级标题用起，所以这里设置为h2 */
>       counter-reset: h2
>   }
>   
>   #write h1, .outline-h1, .md-toc-item.md-toc-h1 {
>       counter-reset: h2
>   }
>   
>   #write h2, .outline-h2, .md-toc-item.md-toc-h2 {
>       counter-reset: h3
>   }
>   
>   #write h3, .outline-h3, .md-toc-item.md-toc-h3 {
>       counter-reset: h4
>   }
>   
>   #write h4, .outline-h4, .md-toc-item.md-toc-h4 {
>       counter-reset: h5
>   }
>   
>   #write h5, .outline-h5, .md-toc-item.md-toc-h5 {
>       counter-reset: h6
>   }
>   
>   /** put counter result into headings */
>   #write h1:before,
>   .outline-h1>.outline-item>.outline-label:before,
>   .md-toc-item.md-toc-h1>.md-toc-inner:before {
>       counter-increment: h1;
>       content: counter(h1) ". "
>   }
>   
>   /* 使用h1标题时，去掉前两个h1标题的序号，包括正文标题、目录树和大纲 */
>   /* nth-of-type中的数字表示获取第几个h1元素，请根据情况自行修改。 */
>   #write h1:nth-of-type(1):before,
>   .outline-h1:nth-of-type(1)>.outline-item>.outline-label:before,
>   .md-toc-item.md-toc-h1:nth-of-type(1)>.md-toc-inner:before,
>   #write h1:nth-of-type(2):before,
>   .outline-h1:nth-of-type(2)>.outline-item>.outline-label:before,
>   .md-toc-item.md-toc-h1:nth-of-type(2)>.md-toc-inner:before{
>   	counter-reset: h1;
>   	content: ""
>   }
>   
>   #write h2:before,
>   .outline-h2>.outline-item>.outline-label:before,
>   .md-toc-item.md-toc-h2>.md-toc-inner:before {
>       counter-increment: h2;
>       content: counter(h2) ". "
>   }
>   
>   /* 使用h2标题时，去掉前两个h2标题的序号，包括正文标题、目录树和大纲 */
>   /* nth-of-type中的数字表示获取第几个h2元素，请根据情况自行修改。 */
>   #write h2:nth-of-type(1):before,
>   .outline-h2:nth-of-type(1)>.outline-item>.outline-label:before,
>   .md-toc-item.md-toc-h2:nth-of-type(1)>.md-toc-inner:before,
>   #write h2:nth-of-type(2):before,
>   .outline-h2:nth-of-type(2)>.outline-item>.outline-label:before,
>   .md-toc-item.md-toc-h2:nth-of-type(2)>.md-toc-inner:before{
>   	counter-reset: h2;
>   	content: ""
>   }
>   
>   #write h3:before,
>   h3.md-focus.md-heading:before, /** override the default style for focused headings */
>   .outline-h3>.outline-item>.outline-label:before,
>   .md-toc-item.md-toc-h3>.md-toc-inner:before {
>   	text-decoration: none;
>       counter-increment: h3;
>       /* content: counter(h1) "." counter(h2) "." counter(h3) ". " */
>       content: counter(h2) "." counter(h3) ". "
>   }
>   
>   #write h4:before,
>   h4.md-focus.md-heading:before,
>   .outline-h4>.outline-item>.outline-label:before,
>   .md-toc-item.md-toc-h4>.md-toc-inner:before {
>   	text-decoration: none;
>       counter-increment: h4;
>       /* content: counter(h1) "." counter(h2) "." counter(h3) "." counter(h4) ". " */
>       content: counter(h2) "." counter(h3) "." counter(h4) ". "
>   }
>   
>   #write h5:before,
>   h5.md-focus.md-heading:before,
>   .outline-h5>.outline-item>.outline-label:before,
>   .md-toc-item.md-toc-h5>.md-toc-inner:before {
>   	text-decoration: none;
>       counter-increment: h5;
>       /* content: counter(h1) "." counter(h2) "." counter(h3) "." counter(h4) "." counter(h5) ". " */
>       content: counter(h2) "." counter(h3) "." counter(h4) "." counter(h5) ". "
>   }
>   
>   #write h6:before,
>   h6.md-focus.md-heading:before,
>   .outline-h6>.outline-item>.outline-label:before,
>   .md-toc-item.md-toc-h6>.md-toc-inner:before {
>   	text-decoration: none;
>       counter-increment: h6;
>       /* content: counter(h1) "." counter(h2) "." counter(h3) "." counter(h4) "." counter(h5) "." counter(h6) ". " */
>       content: counter(h2) "." counter(h3) "." counter(h4) "." counter(h5) "." counter(h6) ". "
>   }
>   
>   /** override the default style for focused headings */
>   #write>h3.md-focus:before,
>   #write>h4.md-focus:before,
>   #write>h5.md-focus:before,
>   #write>h6.md-focus:before,
>   h3.md-focus:before,
>   h4.md-focus:before,
>   h5.md-focus:before,
>   h6.md-focus:before {
>       color: inherit;
>       border: inherit;
>       border-radius: inherit;
>       position: inherit;
>       left:initial;
>       float: none;
>       top:initial;
>       font-size: inherit;
>       padding-left: inherit;
>       padding-right: inherit;
>       vertical-align: inherit;
>       font-weight: inherit;
>       line-height: inherit;
>   }
>   ```
>
> - 