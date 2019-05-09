---
layout: post
title: 域环境下管理员通过配置组策略管理client端(win10系统)
---

【导读】
使用Windows系统的大的团体、组织中，IT管理员有多种方式管理client机器，其中包括组策略(group policy)。
本文描述在域环境下，管理员如何通过配置组策略中的admx管理模板对客户端进行管理。

--------------
任意一台Windows 10计算机上，在`%windir%\PolicyDefinitions`下存在大量ADMX文件，本地组策略中管理模板的内容就来源于此。在win10(1809)上有206个不同的管理模板，并且还会有额外的ADMX文件没有随Win10一起安装到用户机器中。那么这些管理模板是什么东西呢？

### ADMX 模板的参考
从用户角度来讲，需要从两方面了解这些模板文件：这些文件的内容什么，如何使用这些文件配置系统。因此，了解这些ADMX文件的功能非常重要。在此提供两个参考链接(本文重点在于如何配置域控下的组策略，所以不具体描述Admx模板的内容)：

[Group Policy Settings Reference for Windows and Windows Server](http://www.microsoft.com/en-us/download/details.aspx?id=25250)
![group_policy_ref]({{ site.baseurl }}/images\in_post\post_domain_controller_group_policy\group_policy_settings_reference.png)

[Group Policy Search Tool](http://gpsearch.azurewebsites.net/)
![group_policy_tool]({{ site.baseurl }}/images\in_post\post_domain_controller_group_policy\group_policy_tool.png)


Group policy settings reference是一个excel表格，列出了win10(包括win8.1, vista, server等)中可用的组策略设置和对应的功能描述。同时，该表格中还给出了每一条组策略所对应的注册表信息。(这个对应关系非常有用，可以通过设置组策略影响注册表，相关内容在另外一篇文章中描述)

Search Tool中的内容从本质上讲和Group Policy Settings Reference是一样的，只是它以树状结构提供对组策略的view/filter/copy等功能，从查询的角度来讲，更加方便、清晰。

### 部署admx模板
对ADMX模板文件了解后，基本知道它能够做什么了，接下来就是我们如何在**域环境**下使用它们对client进行管理了。

#### 配置域控制器
以Windows Server 2016为例，通过“服务器管理器”->“工具”，安装AD 域控制器(domain controller)。基本上一路“下一步”就能够完成安装。

#### domain上创建Central Store
Window domain controller使用Central Store来存储管理模板文件。默认情况下，windows domain controller中没有存储ADMX文件。需要手动在windows domain controller的SYSVOL文件夹下创建一个Central Store。在创建GPO的时候，Group Policy工具会检查这里并使用其中的ADMX文件做配置，这些配置信息在稍后会复制到所有的域成员中。

创建Central Store步骤如下：

1. 在`\\domain.com\SYSVOL\domain.com\policies`下创建一个central store，命名为PolicyDefinitions 
2. 从一台Windows 10机器中拷贝`C:\Windows\PolicyDefinitions`的内容到`\\domain.com\SYSVOL\domain.com\policies\PolicyDefinitions`
![create_central_store]({{ site.baseurl }}/images\in_post\post_domain_controller_group_policy\create_central_store.jpg)

![inside_policy_definitions]({{ site.baseurl }}/images\in_post\post_domain_controller_group_policy\inside_policydefinitions.jpg)

#### domain上创建并链接GPO
完成了以上步骤，现在就可以创建一个GPO来管理域环境下的Windows10客户端了。

运行gpmc.msc或者通过“服务器管理器”->“工具”->“组策略管理”打开组策略管理界面。

默认的domain policy中仅包含Windows设置中的包含设置。
![default_domain_policy]({{ site.baseurl }}/images\in_post\post_domain_controller_group_policy\default_domain_policy.jpg)

新建"组策略对象"并"链接现有GPO"
![search_policy_from_local]({{ site.baseurl }}/images\in_post\post_domain_controller_group_policy\search_policy_from_local.jpg)

![search_policy_from_central_store]({{ site.baseurl }}/images\in_post\post_domain_controller_group_policy\search_policy_from_central_store.jpg)


此处的新建"组策略对象"有两种方式：从中央存储检索到的ADMX创建和从本地计算机中检索到的ADMX创建。从中央存储检索到的ADMX创建，就是根据前面创建的central store中的admx文件创建gpo。而从本地计算机中检索到的ADMX创建，是会根据当前server中的policydefinitions中admx文件创建gpo的。

这两者所包含的策略(对应admx文件)存在差异，既然domain controller中配置的组策略会应用到client端，建议以"中央存储检索ADMX创建"的方式创建组策略对象。

补充两点，central store中的内容来源于client端完全能够满足要求。如果client端没有policydefinitions，可以通过
[微软官网](https://www.microsoft.com/en-us/download/confirmation.aspx?id=48257) 下载相关文件安装后，将它的内容拷贝到domain controller的central store中。当采用"中央存储检索ADMX创建"的方式时候，之前本地检索的方式也会自动变成"中央存储检索ADMX创建"GPO。

然后，在域名(如图中的testroot.com)上右键选择“链接现有GPO”，选择刚刚创建好的GPO。至此，我们已经可以像操作本地组策略一样来编辑domain controller中的GPO。

### 验证domain controller中的配置是否生效

可以通过以下方式验证域控组策略是否生效。

编辑domain controller中新建的"组策略对象"，如下图，启用"隐藏windows功能"。
![server_verify]({{ site.baseurl }}/images\in_post\post_domain_controller_group_policy\verify_domain_controller_server.jpg)
在client端更新组策略，执行gpupdate /force。
此时，点开控制面板->程序和功能->启用或关闭windows功能，提示系统管理员已经禁用该功能。
![client_verify]({{ site.baseurl }}/images\in_post\post_domain_controller_group_policy\verify_domain_controller_client.jpg)

打开client端本地组策略，查看"隐藏windows功能"这一条组策略，其处于"未配置"的状态。说明，controller的设置已经生效；

组策略中选择禁用"隐藏windows功能"，点击"启用或关闭windows功能"，仍提示管理员已禁用该功能。说明，在controller端的设置会覆盖本地client端设置。

