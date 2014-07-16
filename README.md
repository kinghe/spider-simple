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
每次下载均下载情况存储至sqlite数据表`down-res`中<br>

* `down-res`:记录每一次下载的信息
	* |--id:pk
	* |--url:资源路径
	* |--taskid:任务号，
	* |--depth:爬虫深度，
	* |--parent:来源资源，
	* |--status(新建/成功/失败/重复)
	* |--path(成功时为本地资源路径，失败时记录失败原因，重复时，记录待比较文件)
	* |--start:开始时间，
	* |--cost:下载所需时间，
	* |--size:资源大小。

###3.数据分析模块
资源下载完成后随即进行数据分析过程。<br>

* 根据规则文件定义，启动对指定资源类型的数据分析工作。并将数据分析结果反馈至`data-parse`表中。
	* `HTML文档`。爬虫系统的核心部分之一，系统将根据`HTML文档`解析为`DOM树`,使用`CSS`的方式分析指定的元素，挖掘出
		* 新待下载资源。新建下一深度的资源列表，并保持至`down-res`表。`description@data-parse`记录资源id列表，个数。
		* 数据汇总/统计。为数据挖掘部分，挑选出用户需要的信息并输出至指定文档中。
	* `XML文档`。如`RSS`，`WebService`等服务。
	* `图片文档`。图片的基本信息，如类型，大小，长宽等。后续深入图形学研究后进一步挖掘有用信息。

* `data-parse`:记录每个资源(url)分析信息
	* |--url:pk
	* |--path:本地文件相对路径，
	* |--task-id:任务号，
	* |--down-id:对应的下载资源号，
	* |--last:最近更新日期，
	* |--type:资源类型(html,其他)，
	* |--description:资源描述。

###4.规则管理模块
根据资源类型，名称选择相关的数据解析模块。若无法找到资源解析器，不予解析.其中，HTML解析为核心模块。规则管理模块主要也是针对的html的解析。<br>

* 规则定义(syntax)。
	* 基础语法结构B:`选择器selectors`{`标签处理tag handle`}。采用css语法获取待分析元素。
		* 选择器selector.采用CSS2语法。
		* 标签处理tag handle。与css2相异。CSS2是对符合选择器的所有元素属性值的统一设置，而本规则是对符合选择器的所有元素属性值的迭代处理。
			* down url.新下载资源`url`。
			* select data into gbl_list。将数据append指定的全局资源列表`gbl_list`中。其中`gbl_list`为数组，data为字符串string，数组list，或者字典map。当data为list|map时,常用于单个资源，或整个task完成后的统计分析工作
			* group gbl_list [by index/key].为`gbl_list`分组
			* template template_name.引用规则模板完成数据处理。
	* 复合语法结构C。`选择器selectors`[(B|C)+]
* 规则模板(template)。对常用的规则完成模板化。

###后续模块扩展的考虑

* 系统认证模块(cookies)
* 定时采集
* 定时通知
