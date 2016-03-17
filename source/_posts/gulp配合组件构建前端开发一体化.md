---
title: gulp配合组件构建前端开发一体化
date: 2016-03-09 20:05:25
tags: [gulp,前端构建]
---

### 1、What’s is gulp?
gulp是一款基于任务的设计模式自动化工具，通过插件的配合解决全套前端解决方案，如静态页面的压缩、图片压缩、js合并、sass同步编译并压缩css、服务端控制客户端同步刷新等。

Gulp是一个构建系统，开发者可以使用它在网站开发过程中自动执行常见任务。Gulp是基于Node.js构建的，因此Gulp源文件和你用来定义任务的Gulp文件都被写进了JavaScript（或者CoffeeScript）里。前端开发工程师还可以用自己熟悉的语言来编写任务去lint JavaScript和CSS、解析模板以及在文件变动时编译LESS文件.

Gulp是一个可以在[GitHub](https://github.com/gulpjs/gulp/)上找到的开源项目。

<!--more-->

### 2、安装gulp

	1、安装nodejs环境
	2、安装gulp模块（全局安装）
		npm install -g gulp
	3、在项目中安装gulp（局部安装）
        npm install --save-dev gulp

### 3、使用gulp
##### 前需要的功能有：

- 图片（压缩图片支持jpg、png、gif）

- 样式 （支持sass 同时支持合并、压缩、重命名）

- javascript （检查、合并、压缩、重命名）

- html （压缩）

- 客户端同步刷新显示修改

- 构建项目前清除发布环境下的文件（保持发布环境的清洁）

##### 寻找我们所需要的组件（[gulp plugins](http://gulpjs.com/plugins/)）：
- gulp-imagemin: 压缩图片
- gulp-ruby-sass: 支持sass
- gulp-minify-css: 压缩css
- gulp-jshint: 检查js
- gulp-uglify: 压缩js
- gulp-concat: 合并文件
- gulp-rename: 重命名文件
- gulp-htmlmin: 压缩html
- gulp-clean: 清空文件夹
- gulp-livereload: 服务器控制客户端同步刷新（需配合chrome插件LiveReload及tiny-lr）

##### 安装所需组件
	npm install gulp-util gulp-imagemin gulp-ruby-sass gulp-minify-css gulp-jshint gulp-uglify gulp-rename gulp-concat gulp-clean gulp-livereload tiny-lr gulp-htmlmin --save-dev

##### gulp基础语法
- gulp通过gulpfile文件来完成相关任务，因此项目中必须包含gulpfile.js
- gulp有五个方法：src、dest、task、run、watch
- src和dest：指定源文件和处理后文件的路径
- watch：用来监听文件的变化
- task：指定任务
- run：执行任务

##### 项目目录结构 

	project(项目名称)
	|–.git 通过git管理项目会生成这个文件夹
	|-.sass-cache 使用了sass之后的缓存目录
	|–node_modules 组件目录
	|–dist 发布环境
	    |–css 样式文件
			|-online
				|-wbd
				|-wfc
 				|-wzp
				|-...
	    |–img 图片文件(压缩图片)
			|-wbd
			|-...
	    |–js js文件夹
			|-conf boot配置
			|-lib  库文件夹
			|-pkg  pkg打包文件夹
				|-wbd 
				|-...
	    |–html 静态文件夹
	|–src 生产环境
	    |–css css文件
			|-mod 模块文件夹(里边的模块可以使用sass也可以直接写css)
				|-wzp
				|-wbd
				|-...
	    |–img 图片文件
			|-wzp
			|-wbd
			|-...
	    |–js js文件
			|-conf
			|-mod
				|-wzp
				|-wbd
				|-...
	    |–html 静态文件夹
			|-wzp
			|-wbd
			|-...
	|–.jshintrc jshint配置文件
	|-config.js 项目配置文件
	|–gulpfile.js gulp任务文件


##### 编写gulp任务
 引入 gulp及组件
 
```
var gulp = require('gulp'),                 //基础库
imagemin = require('gulp-imagemin'),    	    //图片压缩
sass = require('gulp-ruby-sass'),           //sass支持
minifycss = require('gulp-minify-css'),     //css压缩
jshint = require('gulp-jshint'),            //js检查
uglify = require('gulp-uglify'),            //js压缩
rename = require('gulp-rename'),            //重命名
concat = require('gulp-concat'),            //合并文件
clean = require('gulp-clean'),              //清空文件夹
htmlmin = require('gulp-htmlmin'),          //压缩html
tinylr = require('tiny-lr'),                //livereload,服务器控制客户端同步刷新（需配合chrome插件LiveReload及tiny-lr）
server = tinylr(),
port = 35729,
livereload = require('gulp-livereload'),    //livereload
glob = require('glob'),                     //匹配文件
config = require('./config');         		//项目配置文件
```

项目配置文件 config.js

```
	module.exports = {
		"workDirName":"wbd",
		"pkgFileName":"index.js"
	}
```

自定义开发项目文件夹名字

```
	var workDirName = config.workDirName, 			//wbd、wfc、wzp等等...
    htmlName = config.htmlName,  				//纯静态页，不需要后端rd开发模板的静态文件
    bootFileName = config.jsBootFileName; 		//js的boot配置文件,主要用于设置版本号
    pkgFileName = config.pkgFileName;
```
	
路径配置对象

```
	var conf = {
		htmlSrc:'./src/html/'+workDirName+'/*.html', //html开发目录
		htmlDst:'./dist/html/',						 //html发布目录
	
		cssSrc:'./src/css/mod/'+workDirName+'/*.css',   //没使用sass的css
		sassSrc:'./src/css/mod/'+workDirName+'/*.scss', //sass目录
		cssDst:'./dist/css/online/'+workDirName+'/',    //css发布目录
	
		imgSrc:'./src/img/'+workDirName+'/*',
		imgDst:'./dist/img/'+workDirName+'/',
	
		confSrc:'./src/js/conf/'+bootFileName,
		confDst:'./dist/js/conf/',
	
		jsSrc:'./src/js/mod/'+workDirName+'/*.js',
		commonSrc:'./src/js/mod/common/*.js',
		jsDst:'./dist/js/pkg/'+workDirName+'/'
	}
```

html处理任务

```
	gulp.task('html',function(){
	
		glob(conf.htmlSrc,{},function(err,files){
			if(err){
				return console.log(err);
			}
			for(var i=0,len=files.length;i<len;i++){
				gulp.src(files[i])
					.pipe(livereload(server))
					.pipe(gulp.dest(conf.htmlDst));
			}
		});

	});
```
	
样式处理，不使用sass任务

```
	gulp.task('css',function(){

		glob(conf.cssSrc,{},function(err,files){
			if(err){
				return console.log(err);
			}
			for(var i=0,len=files.length;i<len;i++){
				gulp.src(files[i])
		        	.pipe(gulp.dest(cssDst))
			    	.pipe(rename({suffix:'.min'}))
			    	.pipe(minifycss())
			    	.pipe(livereload(server))
			    	.pipe(gulp.dest(cssDst));
			}
		});

	});
```

样式处理 使用sass任务

```
	gulp.task('scss',function(){

		glob(conf.sassSrc,{},function(err,files){
			if(err){
				return console.log(err);
			}
			for(var i=0,len=files.length;i<len;i++){
				sass(files[i],{style:'expanded'})
		        	.pipe(gulp.dest(conf.cssDst))
			    	.pipe(rename({suffix:'.min'}))
			    	.pipe(minifycss())
			    	.pipe(livereload(server))
			    	.pipe(gulp.dest(conf.cssDst));
			}
		});
	});
```

图片处理任务

```
	gulp.task('images',function(){
	
		glob(conf.imgSrc,{},function(err,files){
			if(err){
				return console.log(err);
			}
			for(var i=0,len=files.length;i<len;i++){
				gulp.src(files[i])
					.pipe(imagemin())
					.pipe(livereload(server))
					.pipe(gulp.dest(conf.imgDst));
			}
		});

	});
```

js处理任务(这里的任务只能是简单的实现文件的合并，对于require引入模块的方式时，则没办法处理依赖关系，这就需要别的打包工具来配合，如webpack、gulp-requirejs-optimize等模块)

```
	gulp.task('js',function(){

		glob(conf.jsSrc, {}, function(err, files) {
			if (err) {
				return console.log(err);
			}
			gulp.src([conf.jsSrc, conf.commonSrc])
				.pipe(concat(pkgFileName))
				.pipe(gulp.dest(conf.jsDst))
				.pipe(rename({
					suffix: '.min'
				}))
				.pipe(uglify())
				.pipe(livereload(server))
				.pipe(gulp.dest(conf.jsDst));
		});
	
	});
```

清空图片、样式、js、html

```
	gulp.task('clean',function(){
		gulp.src([
			'./css/online/'+workDirName,  //css
			'./js/pkg/'+workDirName,      //js
		],{read:false}).pipe(clean());
	});
```

监听任务 运行语句 gulp watch

```
	gulp.task('watch',function(){
		server.listen(port,function(err){
			if(err){
				return console.log(err);
			}
	
			gulp.watch('../mod/base/*.css', ['css']);
	  		gulp.watch('../mod/common/*.css', ['css']);
	  		gulp.watch('../dev/common/*.css', ['css']);
	
			//监听scss
			gulp.watch(conf.sassSrc,['scss']);
			gulp.watch(conf.sassMod,['scss']);
	
			//监听css
			gulp.watch(conf.cssSrc,['css']);
			gulp.watch(conf.cssMod,['css']);
	
	
			//监听js
			gulp.watch(conf.jsSrc,['js']);
			gulp.watch(conf.commonSrc,['js']);
	
		})
	});
```

默认任务：清空图片、样式、js并重建 运行语句 gulp 监听文件变化

```
	gulp.task('default',['clean'],function(){
		gulp.start('css','scss','js');
		gulp.start('watch');
	});
```

##### LiveReload配置
1. 安装Chrome LiveReload
2. 通过npm安装http-server ，快速建立http服务

```
npm install http-server -g
```
	
3. 通过cd找到发布环境目录dist
4. 运行http-server，默认端口是8080
5. 访问路径localhost:8080
6. 再打开一个cmd，通过cd找到项目路径执行gulp，清空发布环境并初始化
7. 执行监控 gulp
8. 点击chrome上的LiveReload插件，空心变成实心即关联上，你可以修改css、js、html即时会显示到页面中。

