## weibospider

[![](https://img.shields.io/badge/python-3-brightgreen.svg)](https://www.python.org/downloads/)
[![](https://travis-ci.org/SpiderClub/weibospider.svg?branch=master)](https://travis-ci.org/SpiderClub/weibospider)
[![codecov](https://codecov.io/gh/SpiderClub/weibospider/branch/master/graph/badge.svg)](https://codecov.io/gh/SpiderClub/weibospider)
[![GitHub issues](https://img.shields.io/github/issues/SpiderClub/weibospider.svg?style=plastic)](https://github.com/SpiderClub/weibospider/issues)
[![](https://img.shields.io/badge/Say%20Thanks-!-1EAEDB.svg)](https://saythanks.io/to/ResolveWang)
[![](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

## Features :star:
- 支持多种类型的微博数据抓取
- 抓取数据的信息内容非常全面
- **项目可以长期稳定运行**
  - 为了保证程序能长期稳定运行，数据所有的网络请求都是通过抓包手动分析的
  - 通过合理的阈值设定，账号可以保证安全。**但是不推荐用户使用自己的常用账号**
  - 即使账号不可用或者登陆失败，项目都对其做了处理（智能冻结账号，出错重试等），以保证每次请求都是有效的，并及时把错误反馈给用户
  - 通过大量的异常检测和处理，几乎捕获了所有的解析和抓取异常。编写了大量的解析代码来获取足够全面的信息
- 复用性和扩展性好
- 项目支持长期更新
- 丰富的开发者文档支持

## Quick Start :octocat:

1.阅读[项目环境配置](https://github.com/ResolveWang/WeiboSpider/wiki/%E5%88%86%E5%B8%83%E5%BC%8F%E7%88%AC%E8%99%AB%E7%8E%AF%E5%A2%83%E9%85%8D%E7%BD%AE)以配置项目所需的环境。

2.到[release](https://github.com/ResolveWang/weibospider/releases)页面下载稳定版本的应用程序

3.解压你所下载的程序，并且`cd`到它的目录

4.快速安装所需依赖，如果你想使用虚拟环境管理依赖，那么使用`source env.sh`即可，如果你想使用系统的Python环境，那么使用`pip3 install -r requirements.txt`安装所有依赖

5.使用编辑器编辑[配置文件spider.yml](config/spider.yaml)，设置MySQL、Redis连接信息、云打码（需要进行注册并充值）登录信息和邮箱报警信息。另外也可以对抓取间隔等进行配置，具体请阅读相关注释。

6.先通过手动创建一个名为`weibo`的数据库，然后使用`python config/create_all.py`来创建爬虫所需要的表，如果是v1.7.2及之前的版本，输入`python create_all.py`即可。

7.(*可选*，v1.7.3新增)如果你想通过*Web UI*来进行爬虫关键词等信息的配置，那么还需要修改[admin/weibo_admin/settings.py](./admin/weibo_admin/settings.py)中`DATABSES`一栏的数据库连接信息。
然后在项目根目录下运行
```shell
python admin/manage.py makemigrations
python admin/manage.py migrate
python admin/manage.py createsuperuser
```
以生成`django admin`所需要的一些数据表，在执行`python admin/manage.py createsuperuser`的时候，会让你输入django后台的超级管理员用户名、邮箱和密码，比如我依次输入为`test`、`resolvewang@foxmail.com`、`weibospider2017`，然后便成功创建了超级管理员。

8.我们在爬虫程序启动之前，需要预插入微博账号和密码以及一些种子数据。比如你想抓取一个用户，那么就需要在`seed_ids`表中插入他的`uid`，`uid`可以通过打开该用户主页，点击**查看页面源代码**搜索`oid`获取到。如果你想通过通过微博的搜索接口搜索一个关键词，那么需要在`keywords`表中插入你想搜索的关键词。如果你完成了步骤7，那么可以通过*Web UI*来进行配置。通过运行
> python admin/manage.py runserver 0.0.0.0:8000

来启动爬虫配置后台。然后再在你的浏览器输入`http://127.0.0.1:8000/admin`来访问爬虫配置程序。在登录界面输入刚才创建的用户名`test`和密码`weibospider2017`即可，然后在*微博配置*一栏中进行配置。注意，django自带的web server**无法达到生产级别的稳定性**，如果需要
在生产环境中使用，建议使用[gunicorn](http://gunicorn.org/)或者[uwsgi](https://github.com/unbit/uwsgi)作为web server,并且使用supervisor作为进程管理器。

9.配置完成后，通过
> celery -A tasks.workers -Q login_queue,user_crawler,fans_followers,search_crawler,home_crawler worker -l info -c 1

启动worker。注意这里`-Q`表示在本机上可以接收哪些任务执行，详细请阅读[weibospider中所有任务及其说明](https://github.com/ResolveWang/WeiboSpider/wiki/WeibSpider%E4%B8%AD%E6%89%80%E6%9C%89%E4%BB%BB%E5%8A%A1%E5%8F%8A%E5%85%B6%E4%BD%9C%E7%94%A8%E8%AF%B4%E6%98%8E)。`-c`表示并发数，`-l`表示日志等级。

上述命令可以在多台机器上执行，以达到分布式抓取的目的。我们需要做的仅仅是在别的机器上装好项目所需依赖（通过`source env.sh`或者`pip3 install -r requirements.txt`），是不是很简单？


10.到这个时候，我们已经做好所有准备了。现在我们需要发送任务给worker。有两种方式：1）通过执行`python first_task_execution/login_first.py`来进行登录，其他任务发送操作也类似。2）由于我们采用定时的机制来应对**微博Cookie24小时失效**的问题和达到**不间断抓取**的目的，那么我们可以在任何一台节点执行
> celery beat -A tasks.workers -l info

以启动一个celery beater，它会定时将任务发送给Celery Worker进行执行，注意**beater只能有一个**，否则任务可能重复执行。定时设置在[tasks/workers.py](./tasks/workers.py)这个文件。

到这里所有配置已经结束了，如果大家在上述过程中遇到了问题，**请耐心浏览[项目所有文档](https://github.com/ResolveWang/weibospider/wiki)**，实在还是不懂或者使用过程中有任何问题可以提issue。

## Donation :thumbsup:

如果项目对你有用或者对你有启发，不妨通过微信或者支付宝进行*小额捐赠*，以支持该项目的持续维护和发展。

- 通过微信捐赠作者

 ![](http://opqm8qbph.bkt.clouddn.com/8371514638056_.pic.jpg?imageMogr2/thumbnail/!19p)

- 通过支付宝捐赠作者

 ![](http://opqm8qbph.bkt.clouddn.com/alipay.png?imageMogr2/thumbnail/!32p)


## Developer docs :heavy_exclamation_mark:
点击 [wiki](https://github.com/SpiderClub/weibospider/wiki) 查看开发者文档

## Acknowledgements :heart:
- 感谢　[celery](https://github.com/celery/celery)　分布式任务调度框架和　[requests](https://github.com/kennethreitz/requests) 网络库
- 感谢为项目贡献源码的朋友，点击查看[贡献者列表](./AUTHORS.rst)
- 感谢所有捐赠本项目的朋友，点击查看[捐赠者列表](https://github.com/ResolveWang/WeiboSpider/wiki/%E6%8D%90%E8%B5%A0%E8%AF%A5%E9%A1%B9%E7%9B%AE)
- 感谢`star`支持的网友和在使用过程中提issue或者给出宝贵建议的朋友
