---
title: 基于NexT主题的Hexo博客美化
categories:
  - null
mathjax: false
katex: true
date: 2024-05-08 23:22:01
tags:
---


## 主题安装

通过`git`安装主题

```sh
$ cd [Blogroot]
$ git clone https://github.com/next-theme/hexo-theme-next themes/next
```
<!--more-->

修改博客主配置文件

```yml [Blogroot]/_config.yml
theme: next
```

备份NexT主题的配置文件

```sh
$ cp [Blogroot]/theme/next/_config.yml [Blogroot]/_config.next.yml
$ mv [Blogroot]/theme/next/_config.yml [Blogroot]/theme/next/_config.yml.template
```

安装完成，重新生成博客就能直接用了

当然NexT提供了四种主题，可以在`_config.next.yml`文件中进行设置

```yml [Blogroot]/_config.next.yml
scheme: Gemini
```

头像

```yml [Blogroot]/_config.next.yml
avatar:
  # Replace the default image and set the url here.
  url: https://static.wixstatic.com/media/7ac599_2e792bfa812140c898f6c3a78e4ab78f~mv2.png/v1/fill/w_951,h_1046,al_c,q_90,usm_0.66_1.00_0.01,enc_auto/7ac599_2e792bfa812140c898f6c3a78e4ab78f~mv2.png #/images/avatar.gif
  # If true, the avatar will be displayed in circle.
  rounded: true
  # If true, the avatar will be rotated with the cursor.
  rotated: true
```

社交媒体

```yml [Blogroot]/_config.next.yml
social:
  GitHub: https://github.com/Jccc-l || fab fa-github
  E-Mail: mailto:1216550215Jc@gmail.com || fa fa-envelope
```

## 主题修改美化

NexT主题支持黑暗~~黑化~~模式

```yml [Blogroot]/_config.next.yml
darkmode: true
```

设置头像

```yml [Blogroot]/_config.next.yml
avatar:
  url: https://static.wixstatic.com/media/7ac599_2e792bfa812140c898f6c3a78e4ab78f~mv2.png/v1/fill/w_951,h_1046,al_c,q_90,usm_0.66_1.00_0.01,enc_auto/7ac599_2e792bfa812140c898f6c3a78e4ab78f~mv2.png
```

设置社交媒体

```yml [Blogroot]/_config.next.yml
social:
  GitHub: https://github.com/yourname || fab fa-github
  #E-Mail: mailto:yourname@gmail.com || fa fa-envelope
```

## 布局调整

### 调整内容宽度

Gemini主题默认的文章宽度太宽了

打开`[Blogroot]/themes/next/source/css/_variables/Pisces.styl`文件，调整三个变量

```styl [Blogroot]/themes/next/source/css/_variables/Pisces.styl
$content-desktop              = 'calc(100% - %s)' % unit($content-desktop-padding / 2, 'px');
$content-desktop-large        = 1160px;
$content-desktop-largest      = 45%;
```

这个文件还能调整标题、侧栏等的布局

## 页面设置

修改主题配置文件，增加`标签`、`分类`、`归档`等页面

```yml [Blogroot]/_config.next.yml
menu:
  home: / || fa fa-home
  about: /about/ || fa fa-user
  tags: /tags/ || fa fa-tags
  categories: /categories/ || fa fa-th
  archives: /archives/ || fa fa-archive
```

建立页面

```sh
$ hexo n page categories
$ hexo n page about
$ hexo n page tags
```

编辑添加的页面

```md [Blogroot]/source/categories/index.md
---
title: categories
date: 2024-05-07 09:04:59
type: "categories"
---
```

```md [Blogroot]/source/tags/index.md
---
title: tags
date: 2024-05-07 08:53:57
type: "tags"
---
```

```md [Blogroot]/source/about/index.md
---
title: about
date: 2024-05-07 08:53:57
type: "about"
---
```

## 侧栏内容设置

## 添加网站运行时间

修改NexT主题配置，自定义footer

```yml [Blogroot]/_config.next.yml
custom_file_path:
  footer: source/_data/footer.njk
  style: source/_data/styles.styl
```

编辑`footer.njk`文件

```njk [Blogroot]/source/_data/footer.njk
<div>
<span id="timeDate">载入天数...</span><span id="times">载入时分秒...</span>
<script>
    var now = new Date();
    function createtime() {
        var grt= new Date("05/04/2023 00:00:00");
        now.setTime(now.getTime()+250);
        days = (now - grt ) / 1000 / 60 / 60 / 24; dnum = Math.floor(days);
        hours = (now - grt ) / 1000 / 60 / 60 - (24 * dnum); hnum = Math.floor(hours);
        if(String(hnum).length ==1 ){hnum = "0" + hnum;} minutes = (now - grt ) / 1000 /60 - (24 * 60 * dnum) - (60 * hnum);
        mnum = Math.floor(minutes); if(String(mnum).length ==1 ){mnum = "0" + mnum;}
        seconds = (now - grt ) / 1000 - (24 * 60 * 60 * dnum) - (60 * 60 * hnum) - (60 * mnum);
        snum = Math.round(seconds); if(String(snum).length ==1 ){snum = "0" + snum;}
        document.getElementById("timeDate").innerHTML = "本站已安全运行 "+dnum+" 天 ";
        document.getElementById("times").innerHTML = hnum + " 小时 " + mnum + " 分 " + snum + " 秒";
    }
setInterval("createtime()",250);
</script>
</div>
```

时间改为网站的起始时间

## 添加版权信息

```njk [Blogroot]/source/_data/footer.njk
<!-- 版权信息与联系信息部分 -->
<br>
<span id="copyright">&copy; 2010-<span id="currentYearPlaceholder">YYYY</span> 路过的即是风景. All rights reserved. Non-commercial use allowed under <a href="https://creativecommons.org/licenses/by-nc-sa/4.0/" target="_blank">CC BY-NC-SA 4.0</a>.</span>
<br>
<span id="disclaimer">Disclaimer: Content provided for reference only. Accuracy, completeness, suitability not guaranteed. Not liable for any loss or damage.</span>
<br>
<!--联系信息部分-->
<span id="Contact">Contact Us: <a href="mailto:1216550215Jc@gmail.com">GMail</a> | Address: Guangdong, China</span>
<script>
    // 使用JavaScript获取当前年份并更新到页面上
    document.getElementById('currentYearPlaceholder').innerText = new Date().getFullYear();
</script>
```

## Preference

[【个人网站搭建】hexo框架Next主题下添加网站运行时间](https://blog.csdn.net/wangqingchuan92/article/details/126346205)
