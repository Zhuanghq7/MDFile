应对多种浏览器(移动)
-

- transform:translate(200px,100px);
- -webkit-transform:translate(200px,100px);/\***safari** **chrome***/
- -ms-transform:translate(200px.100px);/\***IE***/
- -o-transform:translate(200px,100px)/\***opera***/
- -moz-transform:translate(200px,100px);/\***firefox***/


过渡动画
-
在要使用过渡动画的原件上使用**transition**标签<br/>
栗子：

    -webkit-transition:width 2s,height 2s,-webkit-transform 2s;

然后在使用过渡动画的状态中对设置过的改变值进行改变
栗子:

    div:hover{	
		width:200px;
		height:200px;
		transform:rotate(360deg);
		-webkit_transform:rotate(360deg);
	}
	

动画
-
在要使用动画的原件上使用**animation标签**

    animation:anim 5s infinite linear;

然后调用

	@keyframes anim{
		0%{transform:rotate(0deg);
		100%{transform:rotate(-360deg);
	}	
 注意对应的浏览器支持还要支持keyframes例如**@-webkit-keyframes**

事件|描述
---|-----
onClick|单击事件
onMouseOver|鼠标经过事件
onMouseOut|鼠标移出事件
onChange|文本内容改变事件
onSelect|文本框选中事件
onFoucs|光标聚集事件
onBlur|移开光标事件
onLoad|网页加载事件
onUnload|关闭网页事件
