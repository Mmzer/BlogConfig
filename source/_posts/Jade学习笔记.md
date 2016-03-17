---
title: Jade学习笔记
date: 2016-03-15 17:36:45
tags: [Jade,模板引擎]
---

### 安装
使用npm：

```
npm install jade
```

### 使用

#### options

- **filename** 异常发生时候使用，includes 和 extends时必须使用
- **doctype** html页面的DOCTYPE
- **pretty** 给输出的代码加上缩进，默认为false
- **self** 使用 self 命名空间来持有本地变量. (默认为 false)
- **debug** 开启调试模式，会输出 token 和翻译后的函数体
- **compileDebug** 调试的时候，调试结构会被输出
- **cache** 如果设置为true，编译函数会被缓存，此时，filename必须被设置为缓存的key
- **compiler** 重写jade默认的编译器
- **parser** 重写默认的parser
- **globals** 添加一组可以在模板中使用的全局变量

<!--more-->

#### jade.compile(source, options)
编译一些jade资源，并返回一个自定义设置的可以使用多次的函数。

- **source** jade将要编译的资源
- **options** 一些自定义的配置
- **@return** 返回一个包含自定义设置的生成html文件的函数

例如：

```
var jade = require('jade');

// Compile a function
var fn = jade.compile('string of jade', options);

// Render the function
var html = fn(locals);
// => '<string>of jade</string>'
```

#### jade.compileFile(path, options)

根据路径编译一些jade资源文件，并返回一个自定义设置的可以使用多次的函数。

- **path** 待编译文件的路径
- **options** 一些自定义的配置
- **@return** 返回一个包含自定义设置的生成html文件的函数

```
var jade = require('jade');

// Compile a function
var fn = jade.compileFile('path to jade file', options);

// Render the function
var html = fn(locals);
// => '<string>of jade</string>'
```

#### jade.compileClient(source, options)
编译一些jade资源为JavaScript字符串，在jade runtime时被用户使用

- **source** 待编译文件的路径
- **options** 一些自定义的配置
- **@return** 返回一个字符串形式的JavaScript函数体

```
var jade = require('jade');

// Compile a function
var fn = jade.compileClient('string of jade', options);

// Render the function
var html = fn(locals);
// => 'function template(locals) { return "<string>of jade</string>"; }'
```

#### jade.compileClientWithDependenciesTracked(source, options)
参照**jade.compileClient**方法，期待返回一个JavaScript对象。

```
{
  'body': 'function (locals) {...}',
  'dependencies': ['filename.jade']
}
```
在你需要实现watch某些jade文件变化的时候应该使用这个方法。

#### jade.compileFileClient(path, options)
根据路径编译一些jade资源文件为JavaScript字符串，在jade runtime时被用户使用

- **path** 待编译文件的路径
- **options** 一些自定义的配置
- **options.name** 设置了这个属性作为返回的额函数名字
- **@return** 返回一个字符串形式的JavaScript函数体

jade文件

```
h1 This is a Jade template
h2 By #{author}
```

编译jade文件

```
var fs   = require('fs');
var jade = require('jade');

// Compile the template to a function string
var jsFunctionString = jade.compileFileClient('/path/to/jadeFile.jade', {name: "fancyTemplateFun"});

// Maybe you want to compile all of your templates to a templates.js file and serve it to the client
fs.writeFileSync("templates.js", jsFunctionString);
```

输出的文件（写入templates.js)

```
function fancyTemplateFun(locals) {
  var buf = [];
  var jade_mixins = {};
  var jade_interp;

  var locals_for_with = (locals || {});

  (function (author) {
    buf.push("<h1>This is a Jade template</h1><h2>By "
      + (jade.escape((jade_interp = author) == null ? '' : jade_interp))
      + "</h2>");
  }.call(this, "author" in locals_for_with ?
    locals_for_with.author : typeof author !== "undefined" ?
      author : undefined)
  );

  return buf.join("");
}

```

确认发送除你刚刚编译的模板之外的函数到Jade runtime **(node_modules/jade/runtime.js) **

```
<!DOCTYPE html>
<html>
  <head>
    <script src="/runtime.js"></script>
    <script src="/templates.js"></script>
  </head>

  <body>
    <h1>This is one fancy template.</h1>

    <script type="text/javascript">
      var html = window.fancyTemplateFun({author: "enlore"});
      var div = document.createElement("div");
      div.innerHTML = html;
      document.body.appendChild(div);
    </script>
  </body>
</html>
```

#### jade.render(source, options)

- **source** 待编译资源
- **options** 一些自定义的配置
- **@return** 返回html的结果字符串

```
var jade = require('jade');

var html = jade.render('string of jade', options);
// => '<string>of jade</string>'
```

#### jade.renderFile(filename, options)

- **filename** 待编译jade文件的路径
- **options** 一些自定义的配置
- **@return** 返回html的结果字符串

```
var jade = require('jade');

var html = jade.renderFile('path/to/file.jade', options);
// ...
```


