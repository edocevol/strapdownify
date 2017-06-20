---
title: 通过浏览器加载的方式使用strapdown.js来渲染markdown
---
## 文章导读
1. 如何手动调用`strapdown.js`的渲染方法
2. html下如何将markdown 转换为html
3. 如何拓展strapdown.js的功能
4. 有哪些好用的markdown渲染库

## 本文关联的代码
作者修改后的strapdown.js代码库的github地址如下：
https://github.com/sixtrees/strapdownify

项目中，附有三个demo，详情见文末，demo的启动请使用`tomcat`等web容器进行启动或者使用`webstorm`等IDE进行预览。

## 说明
[strapdown][1]是一个很棒的markdown解析库（当然是基于javascript）的了，但是作者的目的可能是尽可能的适用于所有的场景。
所以，作者在官网给出了如下的使用说明
```html
<!DOCTYPE html>
<html>
<title>Hello Strapdown</title>

<xmp theme="united" style="display:none;">
# Markdown text goes in here

## Chapter 1

Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore
et dolore magna aliqua.

## Chapter 2

Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut
aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse
cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in
culpa qui officia deserunt mollit anim id est laborum.
</xmp>

<script src="http://strapdownjs.com/v/0.2/strapdown.js"></script>
</html>
```

从上面的代码可以看出，我们需要在html中定义一个`xmp`或者`textarea`的`html`元素（当然，如果你不看源码的话，是不会发现也可以用`textarea`来替代`xmp`元素的）。然后，根据源码中的描述，`strapdown`会获取我们的`xmp`中的`markdown`代码，然后利用`document`来解析我们页面中的`link`、`script`、`meta`信息，然后将利用`document.createElement`方法动态的对我们的当前页重新进行渲染。

## 为什么要修改作者的代码库
从上一节的描述中，我们可以了解到，原来的解析方式会替换我们的原始页面，也就是渲染过后的页面只会保留最基本的内容（渲染出来的`markdown`）。因此，我们想有一种方式可以按需进行渲染。

因此，我们对原始的`strapdown.js`进行了如下的修改(替换了原始文件的最后一个自执行函数)：
```javascript

/**
 * 根据传入的$src来获取markdown的内容并转为html显示在$desc元素下
 * 如果$desc元素为undefine，则显示$src元素中
 * @param $src markdown内容的载体
 * @param $desc 渲染过后markdown内容的载体
 */
var markdown = function ($src, $desc) {

    if ($desc == undefined) {
        $desc = $src;
    }
    document.body.style.display = 'none';
    var markdownEl = document.getElementById($src);//markdown载体
    var htmlEl = document.getElementById($desc); //html载体
    var markdown = markdownEl.innerHTML;

    // Generate Markdown
    var html = marked(markdown);
    htmlEl.innerHTML = html;

    // Prettify
    var codeEls = document.getElementsByTagName('code');
    for (var i = 0, ii = codeEls.length; i < ii; i++) {
        var codeEl = codeEls[i];
        var lang = codeEl.className;
        codeEl.className = 'prettyprint lang-' + lang;
    }
    prettyPrint();

    // Style tables
    var tableEls = document.getElementsByTagName('table');
    for (var i = 0, ii = tableEls.length; i < ii; i++) {
        var tableEl = tableEls[i];
        tableEl.className = 'table table-striped table-bordered';
    }
    // All done - show body
    document.body.style.display = '';

};

/**
 * 根据传入的$data作为markdown的内容并转为html显示在$desc元素下
 * @param $data markdown内容
 * @param $desc 渲染过后markdown内容的载体
 */
var markdownFromText = function ($data, $desc) {

    // Generate Markdown
    var html = marked($data);
    document.getElementById($desc).innerHTML = html;

    // Prettify
    var codeEls = document.getElementsByTagName('code');
    for (var i = 0, ii = codeEls.length; i < ii; i++) {
        var codeEl = codeEls[i];
        var lang = codeEl.className;
        codeEl.className = 'prettyprint lang-' + lang;
    }
    prettyPrint();

    // Style tables
    var tableEls = document.getElementsByTagName('table');
    for (var i = 0, ii = tableEls.length; i < ii; i++) {
        var tableEl = tableEls[i];
        tableEl.className = 'table table-striped table-bordered';
    }
};
```

## 使用方式

由于我们对作者的原始库进行了大量的代码精简，因此，样式的选择，我们需要手动的加载`themes`文件夹下的样式主题。

下面给出两个实际的使用案例

#### 案例1
调用`markdown`传递1个参数，可以原始的markdown载体元素作为渲染后的html的载体
```javascript
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="renderer" content="webkit">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <title>strapdown渲染markdown的demo</title>
    <link href="./static/bootstrap-3.3.5/css/bootstrap.min.css" rel="stylesheet">
    <link href="./static/css/reset.css" rel="stylesheet">
    <link href="./static/css/site.css" rel="stylesheet">
    <link href="./static/css/lib.css" rel="stylesheet">
    <!--加载样式-->
    <link href="../strapdown/strapdown.css" rel="stylesheet">
    <link href="../strapdown/themes/readable.min.css" rel="stylesheet">
</head>
<body>
<div class="bt-warp bge6">
    <div id="container" class="container-fluid">
        <div class="sidebar-scroll">
            <div class="sidebar-auto">
                <div id="task" class="task cw" onclick="task()"></div>
                <h3 class="mypcip"><span class="f14 cw" id="serverip">localhost</span></h3>
                <ul class="menu">
                    <li><a class="menu_home" href="index.html">demo1</a></li>
                    <li><a class="menu_folder" href="index2.html">demo2</a></li>
                    <li><a class="menu_fork" href="http://github.com/sixtrees/strapdownify">fork</a></li>
                </ul>
            </div>
        </div>
        <div class="main-content">
            <div class="container-fluid" style="padding-bottom:50px">
                <div id="mdcontent">

                </div>
            </div>
        </div>
    </div>


    <div class="footer bgw">
        markdown内容渲染框架 ©2017-2017
        <a style="margin-left:20px;color:#20a53a;" href="http://github.com/sixtrees/strapdownify"
           target="_blank">strapdownify on github</a>
    </div>
</div>

<script src="./static/js/jquery-1.10.2.min.js"></script>
<script src="./static/js/bootstrap.min.js"></script>
<script src="./static/layer/layer.js"></script>
<script src="../strapdown/strapdown.js"></script>

<script>
    var loadT = layer.msg('正在加载...', {icon: 16, time: 0, shade: [0.3, '#000']});
    $.ajax({
        url: 'readme.md',
        type: "POST",
        dataType: "text",
        success: function (data) {
            markdownFromText(data, "mdcontent");
            layer.close(loadT);
        }
    });
</script>
</body>

</html>

```

#### 案例2
调用`markdown`传递两个参数，可以原始的markdown载体元素需要手动设置为隐藏

```javascript
<!doctype html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
    <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
    <link rel="icon" href="favicon.ico" type="image/x-icon"/>
    <link rel="shortcut icon" href="favicon.ico" type="image/x-icon"/>
    <title>文件管理</title>
    <link rel="stylesheet" type="text/css" href="/static/css/reset.css">
    <link rel="stylesheet" type="text/css" href="/static/css/login.css">
</head>

<body>
<div class="main">
    <div class="login">
        <form class="loginform" method="post" action="/login" onsubmit="return false;">
            <div class="rlogo">系统登录</div>
            <div class="line"><input class="inputtxt" value="" name="userName" datatype="*" nullmsg="请填写账号"
                                     errormsg="格式不对" placeholder="账号" type="text">
                <div class="Validform_checktip"></div>
            </div>
            <div class="line"><input class="inputtxt" name="password" value="" datatype="*" nullmsg="请填写密码"
                                     errormsg="请填写密码" placeholder="密码" type="password">
                <div class="Validform_checktip"></div>
            </div>

            <div class="line yzm">
                <input type="text" class="inputtxt" name="code" nullmsg="请填写验证码" errormsg="验证码不对" datatype="*"
                       placeholder="请填写验证码">
                <div class="Validform_checktip"></div>
                <img width="100" height="40" class="passcode"
                     onClick="this.src=this.src.split('?')[0] + '?'+new Date().getTime()" src="/code"
                     style="border: 1px solid #ccc; float: right;" title="点击换一张">
            </div>

            <div class="login_btn"><input id="login-button" value="登录" type="submit"></div>
            <div class="line">
                <p class="pwinfo">3次以上登录错误将会出现验证码</p>
                <a class="resetpw" href="javascript:void(0);" target="_blank">忘记密码>></a>
            </div>
        </form>
    </div>
</div>
<script type="text/javascript" src="/static/js/jquery-1.10.2.min.js"></script>
<script type="text/javascript" src="/static/layer/layer.js"></script>
<script type="text/javascript" src="/static/js/Validform_v5.3.2_min.js"></script>
<script type="text/javascript">
    $(function () {

        $(".loginform").Validform({
            tiptype: function (msg, o, cssctl) {
                if (!o.obj.is("form")) {
                    var objtip = o.obj.siblings(".Validform_checktip");
                    cssctl(objtip, o.type);
                    objtip.text(msg);
                }
            }
        });
    });

    $('#login-button').click(function () {
        var userName = $("input[name='userName']").val();
        var password = $("input[name='password']").val();
        var code = $("input[name='code']").val();
        if (userName == '' || password == '') {
            layer.msg('表单错误,请重新输入!', {icon: 2});
            return;
        }

        var data = 'userName=' + userName + '&password=' + password + '&code=' + code;
        var loadT = layer.msg('正在登陆...', {icon: 16, time: 0, shade: [0.3, '#000']});
        $.post('/login', data, function (rdata) {
            layer.close(loadT);
            if (rdata.code != "1") {
                $("#errorStr").html(rdata.msg);
                $("input[name='password']").val('');
                num = rdata.remainTimes;
                if (num < 5) {
                    $(".yzm").show();
                    $(".login").css("height", "332px");
                    $("input[name='code']").val('');
                    $(".passcode").click();
                }
                layer.msg(rdata.msg, {icon: 2});
                return;
            }
            layer.msg(rdata.msg, {icon: 6, time: 0, shade: [0.3, '#000']});
            window.location.href = '/';
        });
    });

</script>
</body>
</html>
```

#### 案例3
调用`markdownFromText`
```javascript
<!doctype html>
<html>
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="renderer" content="webkit">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
    <title>strapdown渲染markdown的demo</title>
    <link href="./static/bootstrap-3.3.5/css/bootstrap.min.css" rel="stylesheet">
    <link href="./static/css/reset.css" rel="stylesheet">
    <link href="./static/css/site.css" rel="stylesheet">
    <link href="./static/css/lib.css" rel="stylesheet">
    <!--加载样式-->
    <link href="../strapdown/strapdown.css" rel="stylesheet">
    <link href="../strapdown/themes/readable.min.css" rel="stylesheet">
</head>
<body>
<div class="bt-warp bge6">
    <div id="container" class="container-fluid">
        <div class="sidebar-scroll">
            <div class="sidebar-auto">
                <div id="task" class="task cw" onclick="task()"></div>
                <h3 class="mypcip"><span class="f14 cw" id="serverip">localhost</span></h3>
                <ul class="menu">
                    <li><a class="menu_home" href="index.html">demo1</a></li>
                    <li><a class="menu_folder" href="index2.html">demo2</a></li>
                    <li><a class="menu_fork" href="http://github.com/sixtrees/strapdownify">fork</a></li>
                </ul>
            </div>
        </div>
        <div class="main-content">
            <div class="container-fluid" style="padding-bottom:50px">
                <div id="mdcontent">

                </div>
            </div>
        </div>
    </div>


    <div class="footer bgw">
        markdown内容渲染框架 ©2017-2017
        <a style="margin-left:20px;color:#20a53a;" href="http://github.com/sixtrees/strapdownify"
           target="_blank">strapdownify on github</a>
    </div>
</div>

<script src="./static/js/jquery-1.10.2.min.js"></script>
<script src="./static/js/bootstrap.min.js"></script>
<script src="./static/layer/layer.js"></script>
<script src="../strapdown/strapdown.js"></script>

<script>
    var loadT = layer.msg('正在加载...', {icon: 16, time: 0, shade: [0.3, '#000']});
    $.ajax({
        url: 'readme.md',
        type: "POST",
        dataType: "text",
        success: function (data) {
            markdownFromText(data, "mdcontent");
            layer.close(loadT);
        }
    });
</script>
</body>

</html>

```
  [1]: http://strapdownjs.com/