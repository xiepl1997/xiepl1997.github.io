---
layout : post
title : springboot上传文件
date : 2020-05-31
author : xiepl1997
tags : springboot
---

最近有个需求是上传文件到服务器，使用到的框架是springboot，查询资料后记录如下。  

#### 1.添加基本依赖
这是第一步，但一般建立springboot项目的时候能够勾选该启动依赖。
```java
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

#### 前端form
这是我项目中提交文件的前端代码，这个就因人而异了。（使用的thymeleaf）
```java
<div class="card">
	<div class="card-header">
		文件上传
	</div>
	<div class="card-body card-block">
		<form th:action="'/upload?projectid='+${project.projectid}" method="post" class="form-horizontal" id="fileform" enctype="multipart/form-data">
			<div class="row form-group">
				<div class="col col-md-3"><label for="file-multiple-input" class=" form-control-label">项目申报书上传</label></div>
				<div class="col-12 col-md-9"><input type="file" id="file-multiple-input" name="file-multiple" multiple="multiple" class="form-control-file" required="required"></div>
			</div>
		</form>
	</div>
	<div class="card-footer">
		<button class="btn btn-primary btn-sm" id="fileupload" onclick="tijiao()">
			<i class="fa fa-dot-circle-o"></i> 提交
		</button>
	</div>
</div>
```
或者简单点，可以这样。
```java
<form th:action="'/upload?projectid='+${project.projectid}" method="post" enctype="multipart/form-data">
	<input type="file" name="file-multiple" multiple="multiple">
	<button type="submit">提交</button>
</form>
```

#### controller代码
前端提交请求传到控制层，控制层代码如下。
```java
@Autowired
private ProjectService projectService = null;

@RequestMapping("/upload")
@ResponseBody
public String upload(@RequestParam("file-multiple") MultipartFile file,
                         @RequestParam("projectid") int projectid){
	if(file.isEmpty()){
		return "请选择文件！";
	}
	String fileName = file.getOriginalFilename();
	//设置文件上传位置
	String filePath = "C:\\Users\\xiepl\\Desktop\\testfiles\\";
	File dest = new File(filePath + fileName);
	try{
		file.transferTo(dest);
		projectService.updatefile(projectid, filePath);
		return "上传成功！";
	}catch (IOException e){
		;
	}
	return "上传失败！";
}
```
transferTo方法就将文件上传到指定的路径了。