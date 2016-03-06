# 我从Bad Code中学到了什么

前端开发最核心的三要素众所周知: html, css, javascript
但Web 2.0年代的前端开发与现在的移动前端的开发, 有着很多不同

前端开发噩梦之一: 各种浏览器的兼容性, 明星代表IE系列
前端开发噩梦之二: 原生javascript语言又臭又长, 易读性忒差
前端开发噩梦之三: 品种繁多的 html/css/javascript 封装解决方案过多(很多同质化严重/本身设计也有缺陷), 首用或者做接盘侠维护/二次开发的学习成本实在太高

### Bad Code 示例伪代码

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<head><meta ...../><meta ..../></head>

<style>
#a {position: absolute;color: #3d34df;}
.b {
background-color: #dddddd;
    margin: 0px 0px;
}
</style>

<body><div style="font-family:arial;...."><span>badcode</span></span></div>
<div class="header_main_left "><a href="http://www.appcan.cn" style="float:left;margin-top:20px;"><img src="static/image/newimg/logo_03.png"/></a>
    <ul>
        	<li><a href="http://appcan.cn/" class="indexlink link1">首页</a></li>
            <li><a href="forum.php" class="link2" onclick="webstatfun('page=nav|do=forum')">论坛</a></li>
            <li><a href="home.php?mod=space&amp;do=blog" class="link3" onclick="webstatfun('page=nav|do=blog')" >博客</a></li>
            <li><a href="http://edu.appcan.cn/" target="_blank">学院</a></li>
            <li><a href="http://edu.appcan.cn/Wanted.html" class="noline" target="_blank">招聘</a></li>
        	</ul>
    	</div>
     <div class="header_main_right ">
                     <form method="post" autocomplete="off" id="lsform" action="member.php?mod=logging&amp;action=login&amp;loginsubmit=yes&amp;infloat=yes&amp;lssubmit=yes&amp;resurl=http%3A%2F%2Fbbs.appcan.cn%2Fforum.php%3Fmod%3Dviewthread%26tid%3D30323%26extra%3D%26page%3D1" onSubmit="return lsSubmit();">
     	<div class="login" id="login">
        Hi~ &nbsp;&nbsp;[
        <button type="submit" class="submitbtn" >登录</button>]&nbsp; 
        [&nbsp;&nbsp;<a href="http://dashboard.appcan.cn/register">注册</a>&nbsp;&nbsp;]
        </div>
        </form>
    .....
    .....
    .....
    .....
    
<div >
<div>adfasdfasd....</div>
</div>

<script>
new lazyload();
</script>

</body>


<script type="text/javascript">
initSearchmenu('scbar', '');
</script>

<script type="text/javascript">document.onkeyup = function(e){keyPageScroll(e, 0, 1, 'forum.php?mod=viewthread&tid=30323', 1);}</script>



### 以上代码是典型前端开发/维护99.99%概率会见到过的一团糟页面之一

### Bad No.1:
[style] tag: 个人不建议把大量的css属性写在html的style 标签里主体里, 某些特殊个别用到的, 仅限于在当前页面才需要处理的css属性, 可以适当包含一些, 如果量大的话, 强烈建议单独出一份css样式表文件, 在当前页引用

<link href="template/bbs2015/common/bootstrap.min.css" rel="stylesheet" type="text/css">
<!-- <link href="http://appcan.cn/css/appcanfooter.css"  rel="stylesheet"/> -->

样式表可以同时引用多个, href 可以是相对路径, 或者绝对路径, 甚至可以是链接, 这样既能保持源码的精简, 也便于以后的维护和扩展

### Bad No.2:
[script] tag: 同样, js的语言本来就比较啰嗦, 操作dom元素或者做一些数据上的逻辑处理, 代码量会随业务的复杂型直线上升, 如果像以上代码一样, 多处内嵌及直接在页面里处理逻辑, 长久积攒, 日后几乎无法维护, 所以单独出一份js文件来进行管理是非常必要的


<script src="http://bbs.appcan.cn//template/bbs2015/js/jquery1.min.js" type="text/javascript"></script>
<script src="data/cache/common.js" type="text/javascript"></script>

同样, js脚本也支持引用多个, 但脚本之间有依赖关系的话, 顺序上是有要求的, 以上面为例, 如果common.js中的方法或者任何语言使用到jQuery的话, jQuery.min.js脚本必须在common.js之前先引用, 这样在加载页面的时候不会报错.

以下为示例参考 -> [] 中括号意为文件夹

[utils]
 - DataAuthentication.js (例如是否为email, 手机, 密码强度核验, 敏感字过滤等, 必填信息是否为空 etc.)
 - DateFormatter.js (时间格式的处理, 很多网页显示时间的格式都会不一样, 这个js中可以专门处理)
 - Animations.js (处理动画的公用类)
 - Constants.js (定义一些静态数据, 方便开发中调用)
 ....
 ...
[network]
 - account_api.js (专门处理登录登出, 注册, 找回密码, 资料更新, 上传头像 etc.)
 - 

### Bad No.3
上例中body主体里的元素, 在编程规范中是非常忌讳的, 和其他开发语言一样, 良好的编码规范, 让开发者在阅读和理解上花的时间会大大减少, 以下为格式规范后的参考, 可读性大大提高

<body>
    <table id="info_table">
        <thead>
            <tr>
                <td>
                    <div>
                        <img src="../resources/icon/home/banner.png" />
                        <span>
                            我的主页
                        </span>
                    </div>
                </td>
            </tr>
        </thead>
        
        <tboyd>
            <tr>
                <td>
                    <div>
                        关于我~
                    </div>
                </td>
                
                <td>
                    <div>
                        联系我~
                    </div>
                </td>
            </tr>
        </tboyd>
    </table>
    
    <div id="info_form_section">
        <form method="post" autocomplete="on" id="info_form" action="member.php" onSubmit="infoSubmit();">
        
        <input id="name" />
        <input id="email" />
        
        </form>
    </div>
</body>



### html 5 语言新扩充的Tag

