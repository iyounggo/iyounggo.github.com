---
layout: page
description: "To Do List 2017-12-03"  
---


> Usage：
> 一级和二级标题作为可以设置"标记"的单元，用来追踪需要完成的条目。如有再细分的级别，仅用来描述完成该条目需要考虑(注意)的内容及步骤。
>
> keywords： depends，
>
> Items:

		1. 工具制作
		a. 证书安装小工具
			i. 内容：CMIT证书在client的安装；WPP上证书在client上安装；PK.cer的安装
			ii. 实现方式： powershell
		b. CSV文件处理
		c. 通用log功能
		d. LGPO的功能实现
		
	2. Daily matter
		a. 理清消费类贷款偿付理财的最佳方式
		
		
	3. 需要总结(小结)的内容
		a. COM组件使用的术语及相关用法小结
		b. 为WUA的查找、download写个小结
		c. 同步、异步是怎么回事儿
		d. com接口实现及调用的三种方式：本地、DLL导出、标准com DLL
		e. C++调用约定、DLL调用、写给DLL的code如何本地调用
		f. 导出函数被本地文件使用的方法，#include的最佳实践
		g. 了解证书、密钥体系
			
	4. 验证CI对package的控制(下载、安装)
		
	5. 更新包管理website制作
		a. 更新包的显示，增删改查
		b. 以webservice形式提供服务及在线工具
			
	6. iyounggo更新内容上线
		a. 基本网站结构确定并上传   ---> depends: 图片制作
		b. 思考并确定网站内容大类、主题及文章列表
			
	7. 个人图书馆里程序开发
		a. PC端
		b. 小程序
		c. Android端
			
	8. 了解一下工厂模式(55条建议中提到了，com组件也提到了类厂)
		
	9. COM调用异步下载windows update
		a. 接口实现，生成dll；
		b. 研究在一个DLL中添加多个coclass的用法
		c. Debug  begindownload（）目前不知道哪里出现了问题！！
		d. 调用com接口实现可以不用做成DLL，可以直接new出来指针就可以实现，需验证
	10. 了解网站发布后，如何使用ssh登录(流程是什么？)

>  
>
>  
>
> COM程序
>
> 为原有的interface再继承一层interface，然后为其指定guid等
