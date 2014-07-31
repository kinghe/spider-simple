#spider-simple

简易的网络爬虫系统

#系统设计思路(System Design)
爬虫系统应该包括如下部分
##任务管理模块task module。

用户和系统交互的中转部分,主要包括UI用户交互模块和任务执行模块。
任务主要包括4个状态`status`。
* `TODO` 由用户新建时预定义，主要用于任务后的状态识别。任务可以立即执行或者定时执行。
* `DOING` 任务开始启动时由任务模块变更任务状态。
* `DONE` 任务完成状态标志。由任务模块在所有任务结束后，负责变更到此任务状态，同时更新任务统计信息`statistics`。
* `UNDO` 任务取消，由用户手工确认后系统变更此标示，注意：
	* 1.任务处于UNDO/DONE时不可变更到此标示,即不可对以及取消或者以及完成的任务进行取消，操作无意义。
	* 2.任务处于`TODO`状态时，直接变更状态即可，无需通知任务执行模块。
	* 3.任务处于`DOING`状态时，不可直接变更状态需通知任务执行系统停止此任务的所有执行后，由任务执行系统变更到此状态，并同时统计任务已执行情况。

###任务运行模块`Task Run Module`
任务运行模块的运行依赖于任务模板信息，见`template@task`。
任务模板信息主要包括
* 子任务并行策略。子任务主要包括资源下载任务`resource down` 和数据分析任务`data parse`。一个用户task由多个子任务构成。子任务之间存在执行依赖，并发控制，分布式调度等，安排子任务执行策略相当的重要。
* 下载会话策略。大部分系统需要经过身份验证后方可获取真正的资源。现运行的主要是经过特殊的子任务(请求登陆页面)获取所需的cookie。
* 资源分组策略。见`group@down`。待下载资源建立时先行建立文件分类索引，
	* `数据分析类` 主要包括`网页HTML`,`XML`。定义为数据分析类的资源用于建立相关联的`数据分析`子任务。
	* `多媒体multi-media` 主要包括图片类jpg/png/gif,视频类mp3/flv/mp4,文档类doc/xls/pdf
	* `其他other` 符合下载策略的资源，但不做任何处理。
* 数据分析策略。主要完成数据的筛选，以进一步的处理。语句见`策略模块`。数据分析工作主要包括二个部分。
	* 资源爬取策略。筛选指定元素，从中获取新的资源url，资源描述，新建下载任务(存储于`down`表)，并调度下载模块完成资源下载。
	* 数据分组策略。筛选指定元素，从中获取有用数据(字符串或者格式化数据)，存储于`data`表(见`数据分析`->`数据模型定义`)，同时提取关键字为本组数据划分标签。
	* 通知策略。主要用于重要信息对用户的通知。筛选指定元素，从中获取有用数据通知用户。详见`系统模块`的`通知模块Notify Module`。

###数据模型定义
* `task` :爬虫任务/spider task。
	* |--`id`:pk
	* |--``status``:任务状态.新建待运行TODO，运行中DOING，完成DONE，任务取消UNDO.
	* |--`url`:资源入口。用于获取的第一个资源
	* |--`depth`:资源爬取深度.
	* |--`start`:任务启动时间(用于定时启动).
	* |--``template``:任务使用的爬虫规则模板(资源爬虫策略，数据分析挖掘，通知系统email或者sms等)
	* |--`description`:任务描述
	* |--`statistics:执行结果(成功success，部分成功?half-success，失败failture)，爬取资源统计(总数,成功，失败)，耗时统计(总耗时，成功资源平均下载速度，失败资源耗时)，最优/差下载速度资源列举

##资源下载模块down module。

###系统信息
* 所属任务`task@down`
* 上一级资源`parent@down`
* 资源深度`depth@down`
* 本地或者局域网存储信息`store@down` 
* 执行下载的爬虫标示`spider@down` 下载任务执行时由执行的爬虫自行标示
* 下载任务状态`status@down`

###下载请求信息
* 资源远程路径`url@down`.
* 用户请求类型`method@down`为空时默认get方法
* 用户认证信息`cookie@down`

###下载统计信息
* 下载开始时间`start@down`
* 时间消耗`spend@down`
* 资源大小`size@down`

###资源信息
* 资源分组`group@down`
* 资源描述标签`tag@down`
* 文本文件字符集`charset@down`
* 资源描述`description@down`


###数据模型定义
* `down`:资源下载信息/resource down info。
	* |--`id`:pk
	* |--`task`:任务号，
	* |--`parent`:来源资源，对于初始化资源此项为空。其他资源为来源资源的父id
	* |--`depth`:爬虫深度，
	* |--`store`:本地存储
	* |--`spider`:执行爬虫(用于分布式系统)，同时也用于指定本地网络存储信息的位置
	* |--`status`:资源状态(通`status@task`)
	* |--`url`:资源路径
	* |--`method`:资源请求类型。默认get
	* |--`cookie`:资源cookie
	* |--`start`:开始时间，
	* |--`spend`:下载所需时间(ms)
	* |--`size`:资源大小。(Byte)
	* |--`group`:资源类型。
	* |--`tag`:资源描述标签，用于索引查询，资源分组
	* |--`charset`:资源字符集。
	* |--`description`:资源描述

##数据分析模块/data parse module。
当数据分析类的资源下载完成后随即启动数据分析子任务。

### 资源解析/Resource Analysis
依赖于`group@down`信息获取资源类型(分组)，调用系统维护的分组映射的资源解析器，完成资源数据解析。以用于下一个的`数据过滤`过程。
资源解析器最终将所有资源解析为一个dom树，用于后续的数据过滤。
资源分组类型

* `HTML`:网页文档，爬虫系统的主分析资源，为获取数据的主要方式。可直接转换为dom树,当js存在部分dom构建暂不考虑。
* `XML`:xml文档，常用于`RSS`，`WebService`等服务的数据交换。可直接转换为dom树.
* `文档`。包括doc，xls，pdf等。一般来说是将其先转为html文档，再进一步转为dom树。
* `多媒体`:包括图片，音频，视频等。生成的dom树格式见下
	
		<media>   
			<image title=" " src=" " length=" " size=" " bit=" " subfix=" ">   
				<pixel x=" " y=" ">0xffffff<pixel>
			</image>   
			<audio title=" " src=" " length=" " size=" " time=" " subfix=" ">   
			</audio>   
			<video>    
			</video>   
		</media>  
		

### 数据过滤/Data Filter
数据过滤采用CSS中的`选择子/Selectors`模型完成数据过滤。
用户可以定义多个`选择子/Selectors`并行运行，相互无关。
每一个`选择子/Selectors`均生成一个元素列表，用于后续的数据挖掘工作。

### 数据挖掘/Data Mining
数据挖掘(Data Mining,DM),又称数据库中的知识发现(Knowledge Discover in Database,KDD).
是指从数据库的大量数据中揭示出隐含的、先前未知的并有潜在价值的信息的非平凡过程。
主要基于人工智能、机器学习、模式识别、统计学、数据库、可视化技术等，高度自动化地分析企业的数据，做出归纳性的推理，从中挖掘出潜在的模式，帮助决策者调整市场策略，减少风险，做出正确的决策。

筛选相关元素用于以下处理。

* 下载繁衍。即查找出新的待下载资源(避免重复下载，符合资源深度，下载url满足一定条件)，新建资源到下一深度，并保持至`down`表，等待下载模块执行。
* 数据汇总/统计。挑选出用户需要的信息并存储至`dm`表，同时向`tag@dm`插入标签，用于信息索引。信息存储分为三种方式。
	* list。用户可提取关键字存储到`tag@dm`中，关键字存在多个时，空格分开。
	* map。tag关键字格式为keyword[attr]。信息为二维表的指定cell的值
	* tree。keyword[attr][attr2].
* 智能机器人，即根据系统现有的知识储备，找到映射的行动执行模块。后续扩展此功能。在挖掘出有用信息后，系统提供的自动处理系统，具有简单智慧，自主学习功能，

###数据模型定义
* `dm`:记录每个资源(url)分析信息
	* |--`id`:pk.
	* |--`down`:资源ID。记录数据挖掘信息的来源。
	* |--`task`:任务号。
	* |--`data`:数据存储信息，主要用于小数据量信息存储，当首字母为@时，则随后的字符串为一文件完整路径，用于挖掘信息的记录。
	* |--`tag`:资源描述标签

##系统模块System Module
系统通用模块
###用户默认设置User Default Config

###定时触发Timer

###通知模块Notify Module
通知包括email，sms，app等。

###用户场景模式
为适应多种用户，提供多种场景操作模式`scene`。
* `最简操作模式`。界面设计要求便捷，易操作。用户仅需提供一个url人口即可，资源爬取策略和数据分析挖掘模块由开发人员按照用户要求设计开发定制，以默认模板的方式提供。
* `定制操作模式`。提供自定义模板管理功能。对用户要求较高，用于更加精细化的任务定制,包括资源爬取策略和数据分析挖掘策略的自定义。

##后续扩展

###智能机器人Intelligent Robot
作为数据分析的一个子部分
