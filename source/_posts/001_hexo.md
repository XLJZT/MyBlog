---
title: 记录一下安装hexo的过程
date: 2022-1-10
top_image: https://cdn.jsdelivr.net/gh/XLJZT/img@main/blog/001.svg
cover: https://cdn.jsdelivr.net/gh/XLJZT/img@main/blog/001.svg
categories: Hexo
---



### 记录一下安装hexo的过程

（防止以后遗忘 不知道咋弄）

#### 安装hexo

首先安装好git 和 node js

```
$ npm install -g hexo-cli
or
$ npm install hexo
```

完成之后用 **`hexo -v`** 查看版本

接下来初始化 `hexo`

```
$ hexo init 文件名
$ cd 文件名
$ npm install
```

新建完成之后会有如下文件夹

![image-20220110162452660](https://cdn.jsdelivr.net/gh/XLJZT/img@main/blog/image-20220110162452660.png)

```
$ hexo g
$ hexo s
```

之后便可以进入到初始博客

![image-20220110162645465](https://cdn.jsdelivr.net/gh/XLJZT/img@main/blog/image-20220110162645465.png)

#### GitHub或Gitee 创建新库

TIP: 在GitHub上注册的仓库名为  `相同的用户名.github.io`

​		Gitee上为`相同的用户名`



#### 将hexo部署到GitHub或Gitee上

先安装deploy-git

```
$ npm install hexo-deployer-git --save
```

在**`_config.yml`**文件中最后修改

```yaml
deploy:
- type: 'git'
  repo: https://github.com/username/username.github.io.git
  branch: blog
- type: 'git'
  repo: https://gitee.com/username/username.git
  branch: blog
```

然后执行命令

```
$ hexo clean
$ hexo g
$ hexo d
```

其中 `hexo clean`清除了你之前生成的东西，也可以不加。
`hexo generate` 顾名思义，生成静态文章，可以用 `hexo g`缩写
`hexo deploy` 部署文章，可以用`hexo d`缩写

部署结束

#### 修改主题

我选择的是 [Hexo Theme Cupertino](https://cdn.jsdelivr.net/gh/XLJZT/img@main/blog/https://github.com/MrWillCom/hexo-theme-cupertino)

![image-20220110164357577](https://cdn.jsdelivr.net/gh/XLJZT/img@main/blog/image-20220110164357577.png)

完全符合我的审美

进入主题文件

```
$ cd themes
$ git clone https://github.com/MrWillCom/hexo-theme-cupertino.git
```

修改全局配置文件 `_config.yml`

```yaml
theme: hexo-theme-cupertino
```

修改原作者的链接

![image-20220110165134907](https://cdn.jsdelivr.net/gh/XLJZT/img@main/blog/image-20220110165134907.png)

`添加浏览器图标png放到 文件名/source 下`

修改全局配置文件

```yaml
subtitle:''
description:''
```

这是现实在主页的两行字体 作者源码解释在此

![image-20220110165549516](https://cdn.jsdelivr.net/gh/XLJZT/img@main/blog/image-20220110165549516.png)

至此主题部分配置已结束

#### 关于图片插入的问题

我看到一个插件非常中意 但是安装的时候出现问题就一直没能装上

**[hexo-easy-images](https://github.com/boboidream/hexo-easy-images#hexo-easy-images)**

然后对比使用了**[hexo-filter-image](https://github.com/JoeyBling/hexo-filter-image)** 和 **[hexo-asset-image](https://github.com/xcodebuild/hexo-asset-image)** 之后我选择了使用后者

接下来是我的使用方案

#### 安装  **[hexo-asset-image](https://github.com/xcodebuild/hexo-asset-image)**

```
npm install hexo-asset-image --save
```

源文件`/node_modules/hexo-asset-image/index.js`里的代码有错误 需要进行修改为以下代码

```js
'use strict';
var cheerio = require('cheerio');

// http://stackoverflow.com/questions/14480345/how-to-get-the-nth-occurrence-in-a-string
function getPosition(str, m, i) {
  return str.split(m, i).join(m).length;
}

var version = String(hexo.version).split('.');
hexo.extend.filter.register('after_post_render', function(data){
  var config = hexo.config;
  if(config.post_asset_folder){
        var link = data.permalink;
    if(version.length > 0 && Number(version[0]) == 3)
       var beginPos = getPosition(link, '/', 1) + 1;
    else
       var beginPos = getPosition(link, '/', 3) + 1;
    // In hexo 3.1.1, the permalink of "about" page is like ".../about/index.html".
    var endPos = link.lastIndexOf('/') + 1;
    link = link.substring(beginPos, endPos);

    var toprocess = ['excerpt', 'more', 'content'];
    for(var i = 0; i < toprocess.length; i++){
      var key = toprocess[i];
 
      var $ = cheerio.load(data[key], {
        ignoreWhitespace: false,
        xmlMode: false,
        lowerCaseTags: false,
        decodeEntities: false
      });

      $('img').each(function(){
        if ($(this).attr('src')){
            // For windows style path, we replace '\' to '/'.
            var src = $(this).attr('src').replace('\\', '/');
            if(!/http[s]*.*|\/\/.*/.test(src) &&
               !/^\s*\//.test(src)) {
              // For "about" page, the first part of "src" can't be removed.
              // In addition, to support multi-level local directory.
              var linkArray = link.split('/').filter(function(elem){
                return elem != '';
              });
              var srcArray = src.split('/').filter(function(elem){
                return elem != '' && elem != '.';
              });
              if(srcArray.length > 1)
                srcArray.shift();
              src = srcArray.join('/');
              $(this).attr('src', config.root + link + src);
              console.info&&console.info("update link as:-->"+config.root + link + src);
            }
        }else{
            console.info&&console.info("no src attr, skipped...");
            console.info&&console.info($(this));
        }
      });
      data[key] = $.html();
    }
  }
});
```



#### 食用方法

在写markdown文件时设置好图像的问题，我用的是typora更改图片设置如下图

![image-20220111114426389](https://cdn.jsdelivr.net/gh/XLJZT/img@main/blog/image-20220111114426389.png)

这样就会将图片和文件放到一个文件夹下

![image-20220111114517951](https://cdn.jsdelivr.net/gh/XLJZT/img@main/blog/image-20220111114517951.png)

然后使用命令

```yaml
hexo new 记录一下安装hexo的过程
```

或者手动在  `文件/source/_post/ `下创建相同名字的文件

然后将写好的markdown文件的图片放入文件夹里 将md文件放到外面

![image-20220111115043380](https://cdn.jsdelivr.net/gh/XLJZT/img@main/blog/image-20220111115043380.png)

本人建议直接将编辑好的markdown文件夹复制到 `文件/source/_post/ ` 然后将`.md`文件给拉到和文件夹同一层级，形成上图的层级格式 

#### TIP

> 文件夹里只有图片

#### 修改url

`_Config.yml`

```yaml
# URL
## Set your site url here. For example, if you use GitHub Page, set url as 'https://username.github.io/project'

#修改url是因为github不加载css和js
url: https://username.github.io
root: /

```

> 记得上传源文件到GitHub或Gitee上
>
> 将themes的主题里的`.git`删掉 因为github不能嵌套上传
>
> 建议将page放入一个分支 源码放入一个分支



 

**如果我的看不懂可以建议看一下[这个链接](https://blog.csdn.net/sinat_37781304/article/details/82729029) 我做的时候学习的这个作者才做出来的给了我不少启发**

第一篇文章到此结束！！！🥳🥳🥳



# 2022.5.13 Update

选择了新的主题，主题相较于之前的更加丰富，并且从github page迁移到了vercel
