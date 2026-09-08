内部js抽取成外部文件：
再通过src属性引入。


分段逻辑提取：抽取到工具类js文件中，其他js文件需要时再import导入

导出：
注：要被外部使用，必须要export导出暴露给其他文件：
![[Pasted image 20260416151627.png]]

导入：按需模块化导入：
![[Pasted image 20260416151714.png]]

如果HTML中script标签引入的js是模块化的（使用了import、export的模块化关键字），**script**标签需加上 type属性：`type = "module"`

这样js控制的逻辑和HTML控制的样式就解耦了，修改样式在HTML文件中，需要修改交互则直接在引入的js文件中修改，便于维护项目。