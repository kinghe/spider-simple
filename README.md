#spider-simple

简易的网络爬虫系统

##系统设计思路(System Design)
爬虫系统应该包括如下部分
###1任务管理模块

任务制定完成后，不可再做修改，但可以终止后选择复制到新任务中
* `task` :爬虫任务 spider task<br>
  * |--id:pk
  * |--description:任务描述
  * |--status:任务状态，新建，运行，成功，部分成功，失败.
  * |--init:初始化url，
  * |--regex:爬虫规则，暂未确定是记录到文件还是一个字符串，或是一组合类规则,记录的是爬取深度

###2.资源下载模块。
包括但不限于`html,js,css,jpg/png/gif,mp3/flv/mp4`.<br>
默认保存到本地存储系统，本地存储路径定义在config文件中<br>
每次下载均下载情况存储至sqlite数据表url-map,down-log中<br>
* `url-map`:记录每个资源(url)对应的信息
  * |--url:pk
  * |--path:本地文件相对路径，
  * |--task-id:任务号，
  * |--down-id:对应的下载资源号，
  * |--last:最近更新日期，
  * |--type:资源类型(html,其他)，
  * |--description:资源描述

* `down-log`:记录每一次下载的信息
  * |--id:pk
  * |--url:资源路径
  * |--task-id:任务号，
  * |--task-depth:爬虫深度，
  * |--parent:来源资源，
  * |--status(新建/成功/失败/重复)
  * |--path(成功时为本地资源路径，失败时记录失败原因，重复时，记录待比较文件)
  * |--start:开始时间，
  * |--cost:下载所需时间，
  * |--size:资源大小

###3.html分析模块

分析主要包括二个部分<br>
分析下一个深度需要爬取的资源和待分析的资源<br>
分析本资源内的文本数据，进行初步的数据挖掘工作。


###4.后续模块扩展的考虑

* 系统认证模块(cookies)
* 定时采集
* 定时通知
