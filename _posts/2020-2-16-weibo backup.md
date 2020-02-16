---
layout:     post
title:      使用Python备份微博的内容
subtitle:   十年大号两万微博毁于一旦
date:       2020-2-16
author:     CescMessi
header-img: img/weiboban.png
catalog: true
tags:
    - Blog
    - Python
---

# 前情提要
2月1日的晚上，当我看完西班牙人的比赛，正准备为武磊出场时间过少发一条微博时，发现我的微博被封号了，具体表现就是，无法发送微博，无法转评赞，无法私信，无法关注，只能自己浏览，在别的账户的视角里，这个账号就像从未存在过，所有的微博、评论都会消失……

我的微博是2010年注册的，也算是最早的一批用户了，至今也有近两万条微博，从初中到现在，和很多朋友的交流也都是在微博上。本着对新浪的极度不信任，我决定自己备份我的微博。

其实网上已经有现有的微博备份工具了，比如我关注的[@Easy](https://weibo.com/easy) 大大之前就做过一款叫[微贝](https://weibo.com/1088413295/F9a9fevuX?mid=4122510561558416&ouid=1088413295&type=comment)的Chrome扩展。不过免费额度是2000条，不太适合我。反正闲着也是闲着，不如自己用Python写一个。

# 备份过程
**FBI WARNING！！！**

**操作有风险，且有时效性，请确保你能明白自己每一步的操作，如果因本文所述操作造成封号或限制，本人概不负责！！！**
## 下载全部微博json文件
在查找资料的时候，发现了LeadroyaL的博客里的一篇文章[看我如何让被封掉的微博秽土转生](https://www.leadroyal.cn/?p=724)，博主的情况和我一样，也是被炸号，然后博主开始备份微博数据，然后用新号重新关注、发送微博。不过这篇文章的重点是在“转生”，而我想要的是“备份”，文章里也没有提到评论之类的内容。

但是有几点非常有帮助：
1. 使用手机版网页`https://m.weibo.cn`抓取数据，这里得到的数据都是json文件，比电脑版的好处理。
2. 作者提供了[GitHub链接](https://github.com/LeadroyaL/weibo_repeater)，里面是秽土转生的全部代码。

由于需求不同，所以还是得稍微改一下。备份微博我直接使用了`web/content.py`文件，文件内容如下：

```Python
# -*-coding:utf8-*-

import time
import requests

user_id = 12345678

cookie = {
    "Cookie": "ALF=1544707706; SCF=Ak7h6raU-MOVDMV3RDOAWlZKt0DWVwcjJ13fKETjSF6u-vs0nu_VAOJocKAOI-LM_9K5yyiJuTWm8pOvASeXZt0.; SUB=_2A2527qHpDeRhGeVH6lAQ8CvOzDqIHXVSEM-hrDV6PUJbktANLRnakW1NT2fsMmZ0BDPDk1Ao3-fpQ2lkz1RfF9uZ; SUBP=0033WrSXqPxfM725Ws9jqgMF55529P9D9WWPCrQQRo3CG6es4qqEDNqL5JpX5K-hUgL.Foe4eKzpeh-ES0q2dJLoI09e9g8.MJv4q-7LxKqL1KMLBoqLxKqL1KMLBK-LxKBLBonL12BLxK-LBKBL1-2LxKBLB.2L1hqt; SUHB=0ftp9n7k2d59rv; SSOLoginState=1542115769; _T_WM=5059053e1fe585297cba2fceba393ef4; MLOGIN=1; WEIBOCN_FROM=1110006030; M_WEIBOCN_PARAMS=luicode%3D10000011%26lfid%3D2304133912105276_-_WEIBO_SECOND_PROFILE_WEIBO%26fid%3D102803%26uicode%3D20000174"}

headers = {
    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8",
    "Referer": "https://m.weibo.cn/p/2304133912105276_-_WEIBO_SECOND_PROFILE_WEIBO",
    "Accept-Language": "zh-CN,zh;q=0.9",
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.102 Safari/537.36",
    "Cache-Control": "max-age=0",
    "Upgrade-Insecure-Requests": "1",
    "Host": "m.weibo.cn",
    "Connection": "close",
    "MWeibo-Pwa": "1",
    "X-Requested-With": "XMLHttpRequest",

}

for i in range(70):
    fd = open("D:\\content_.json" % i, 'wb')
    url = 'https://m.weibo.cn/api/container/getIndex?containerid=2304133912105276_-_WEIBO_SECOND_PROFILE_WEIBO&page_type=03&page=%d' % i
    lxml = requests.get(url, cookies=cookie, headers=headers).content
    print(lxml)
    print("-------")
    fd.write(lxml)
    fd.close()
    time.sleep(5)
```
这里面，主要修改`cookie`、`header`和`url`三项，`user_id`在这个文件里并没有被使用，其实改不改无所谓。

那么怎么改呢？登录你的微博，进入手机版页面，点进你的微博列表，按下F12，点到network里，刷新一下界面，找到一个名字为`getIndex?containerid=xxx_-_WEIBO_SECOND_PROFILE_WEIBO`的文件，Chrome的话会看到两个名字一样的，前面有齿轮的会有cookie，在`Headers`一栏里找到`Request Headers`，里面的cookie替换到代码里，其他项替换到代码里的`headers`，有些项在浏览器里有而代码里没有，如果出现问题可以考虑添加进去，我使用的时候没有什么影响。

![image.png](https://i.loli.net/2020/02/16/hO9gxCQ6vKml1jM.png)

至于`url`，很明显，观察一下刚刚选中的文件，将containerid换成你自己的即可。

当然还有一件重要的事，那就是循环次数。原博主选择的是70，那是因为他的微博有70页。观察一下刚刚选中的那个json文件，每个文件里是有10条微博的，根据你的微博总数算一下就能得到循环次数了。**但是！！！这里有一个坑！！！原作者并没有发现，循环是从0开始的，而第0页和第1页是一样的数据，因此你还需要将url那里的`i`改成`(i+1)`才能完整获取你的全部微博。**

## 分析微博的json文件
其实微博手机版的json文件写的非常清晰，稍微懂一点的人（指我自己）都能看明白。最外层有两个key，一个是状态码，一个是数据`data`，里面有一个`cards`的数组，里面的成员都是一个个的对象，除了第一页第一个是搜索框，其他的都是一条单独的微博。而微博内容就存储在`mblog`中，里面有很多信息，这里把重要的列个表展示一下：

|   Key  |  Value |
| :----: | :----: |
| created_at | 创建时间 |
| mid | 微博的ID* |
| text | 微博内容 |
| source | 微博来源 |
| user | 微博发送者信息 |
| reposts_count | 转发数量 |
| comments_count | 评论数量 |
| attitudes_count | 点赞数量 |
| pic_num | 图片数量 |
| reads_count | 阅读数量* |
| pics | 图片信息* |
| retweeted_status | 转发内容* |

*有三个带id的参数`id`、`idstr`和`mid`，在近几年的微博里这三个id是一样的，但在九年前，这三个参数是不一样的。通过这个id是可以访问单条微博的，形式如`https://m.weibo.cn/detail/`+`mid`，好像微博官方也没有注意到这个问题，后面获取评论时这有坑。

*阅读数量和上面id一样，也是有历史遗留问题，只有近几年的微博有这个参数，早期的微博是获取不到这个参数的

*图片信息里有原图的链接，可以用这个链接获取图片，如果要在外站访问注意新浪的防盗链。

*转发内容里的形式和单个微博是一样的

还有一点值得注意，上面列的参数有些微博可能是没有的，比如图片信息如果没有发图是没有的，转发内容也是一样，而且转发内容和图片信息应该不会同时出现。

## 下载微博的评论
现在微博手机版的评论json已经改版了，比较难爬了，当然网上也有解决办法(见参考资料）。但是老版的api依然可以使用，那还是直接用老版比较好。形式是`https://m.weibo.cn/api/comments/show?id=%s&page=%d`，其中`id`是微博的id，也就是上面提到的mid，`page`就是评论的页数。下面放出我写的下载评论的函数代码，请求头和cookie之类的可以自己用浏览器访问以下这个api然后复制：

```Python
def downloadComments(wbid, flag=0):
    

    try:
        os.mkdir('./comments/'+wbid)
    except Exception as identifier:
        print('fail to make dir')
    
    print('下载评论中……')
    for i in range(100):
        url = 'https://m.weibo.cn/api/comments/show?id=%s&page=%d'%(wbid,i+1)
        
        print(url)
        time.sleep(5)
        comdata = requests.get(url, cookies=cookie, headers=headers)
        comcontent = comdata.content
        comjson = comdata.json()
        # print(comjson['ok'])
        
        
        print(comjson['msg'])
        if comjson['ok'] == 0:
            if i == 0:
                if flag == 5:
                    logf = open('log.txt','a',encoding='utf-8')
                    logf.write(url+'\n')
                    logf.close()
                    print('下载失败')
                    break
                print('第%d次重试下载'%(flag+1))
                downloadComments(wbid, flag=flag+1)
                break
            else:
                break
        fd = open("./comments/%s/comments_%d.json"%(wbid,i) , 'wb')
        fd.write(comcontent)
        fd.close()
        time.sleep(2)
```

整个操作流程就是，先建立一个名字为微博id的文件夹，然后将评论json文件放到这个文件夹里。至于微博id怎么获取，之前下载的那一堆json文件就有用了，直接在json文件里读取`mid`即可。注意有时候微博的json文件里的评论数量和你能获取到的数据是不一样的，因为有的评论时被和谐了的。所以单纯依靠数量去判断有多少页是不合适的，我这里是检测状态码成功就下载。循环次数看个人需求，我是没有100页的评论了……延迟时间我这里设置的比较大，如果想快点可以自己修改，但是更容易触发新浪的反爬虫机制。

另外之前说到的关于微博id的坑就在这里了，手机版微博新的评论api里面有两个参数`id`和`mid`，用手机版打开九年前的微博，评论都不会显示，观察api的url会发现，`id`和`mid`取值是一样的……渣浪程序员出来挨打！不过电脑网页版没有出现这种情况。

获取到的评论json文件和微博的json文件类似，多了`reply_text`回复内容，还是很好看懂的。不过旧api获取到的评论时间只能精确到日，而新api是可以精确到秒的。所以如果想精确一点，还是建议研究一下新的api。

# 后续工作
微博和评论的json文件都下载完了，基本上可以说是备份完成了，你也可以继续下载微博里的图片。当然直接看json文件是不现实的，如果你有相关知识的话，可以自己做一个前端，数据就用你下载到的json文件。我就嫌麻烦，用Python直接写html文件，每个文件100条微博，也能凑合着看，只是不够美观。

# 参考资料
[微贝介绍](https://www.appinn.com/weibei-for-chrome/)

[看我如何让被封掉的微博秽土转生](https://www.leadroyal.cn/?p=724)

[如何科学地蹭热点：用python爬虫获取热门微博评论并进行情感分析](https://www.jianshu.com/p/92de44f0376a)

[爬虫之最新版微博评论爬取方法（非url中拼接page=num方法）](https://www.jianshu.com/p/8dc04794e35f)