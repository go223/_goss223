css变量：
2017.3微软宣布edge支持css变量

一、变量的声明

声明变量的时候，变量名前面要加两根连词线（--）
body{
	--foo:#7F583F;
	--bar:#F7EFD2;
}
上面代码中，body选择器里面声明了两个变量：--foo和--bar。
他们与color、font-size等正式属性没有什么不同，只是没有默认含义。所以css变量（css variable）又叫做
“css自定义属性”（css custom properties）。因为变量与自定义的css属性其实是一回事

css变量使用连词线--  Sass使用$  Less使用@
:root{
	--main-color: #4d4e53;
	--main-bg: rgb(255,255,255);
	--logo-border-color: rebeccapurple;

	--header-height: 68px;
	--content-padding: 10px 20px;

	--base-line-height: 21px;
	--transition-duration: 0.35s;
	--external-link: "external link";
	--margin-top: calc(2vh + 20px);
}

变量名大小写敏感 --hearder-color 和 --Hearder-Color 是两个不同的变量。

二、var函数
var()函数用于读取变量。
a{
	color:var(--foo);
	text-decoration-color:var(--bar);
}
var()函数还可以使用第二个参数，表示变量的默认值。如果该变量不存在，就会使用这个默认值
color: var(--f00, #7F593F);
第二个参数不处理内部的逗号或空格，都视作参数的一部分。

var(--font-stack, "Roboto","Helvetica");
var(--pad, 10px 15px 20px);

var()函数还可以用在变量的声明
:root{
	--primary-color: red;
	--logo-text: var(--primary-color);
}
需要注意的是：变量值只能作用于属性值，不能作用于属性名

三、变量值得类型
如果变量值是一个字符串，可以与其他字符串拼接

body:after{
	content:'--screen-category: 'var(--screen-cagegory);
}
如果变量是数值，不能与数值单位直接连用
.foo{
	--gap:20;
	/* 无效 */
	margin-top: var(--gap)px;
}
上面代码中，数值与单位直接写在一起，这是无效的。必须使用calc()函数，将它们连接
.foo{
	--gap:20;
	margin-top:calc(var(--gap) * 1px);
}
如果变量值带有单位，就不能写成字符串
.bar{
	--foo:'20px';/* 这种写法是错误的 */
	--f00:20px;/* 这是正确的写法 */
}

四作用域
同一个css变量，可以在多个选择器内声明，读取的时候，优先级最高的声明生效。这与css的“层叠”（cascade）规则是一致的。
<style>
	:root{
		--color:blue;
	}
	div{
		--color:green;
	}
	#alert{
		--color:red;
	}
	*{
		color:var(--color);
	}
</style>
<p>蓝色</p>
<div>绿色</div>
<div id="alert">红色</div>
上面代码中，三个选择器都声明了--color变量。不同元素读取这个变脸的时候，会采用优先级最高的规则，因此三段文字的颜色是不一样的。
也就是说变量的作用域就是它所在的选择器的有效范围。
body{
	--foo:#7F583F;
}
.content{
	--bar:#F7EFD2;
}
上面代码中，变量--foo的作用域是body选择器的生效范围，--bar的作用域是.content选择器的生效范围。
由于这个原因，全局的变量通常放在根元素:root里面，确保任何选择器都可以读取它们。
:root{
	--main-color:#06c;
}

五、响应式布局
css是动态的，页面的任何变化都会导致采用的规则变化，
利用这个特点，可以在响应式布局的media命令里面声明变量，使得不同的屏幕宽度有不同的变量值
body{
	--primary:#7F583F;
	--secondary:#F7EFD2;
}
a{
	color:var(--primary);
	text-decoration-color:var(--secondary);
}
@media screen and (min-width:768px){
	body{
		--primary:#F7EFD2;
		--secondary:#7F583F;
	}
}

六、兼容性处理
对于不支持css变量的浏览器，可以采用下面的写法
a{
	color:#7F583F;
	color:var(--primary)
}
也可以使用@support命令进行检测
@supports((--a:0)){
	/* supported */
}
@supports(not(--a:0)){
	/* not supported */
}

七、JavaScript操作

JavaScript也可以检测浏览器是否支持css变量
<script>
	const isSupported = window.CSS && window.CSS.supports && window.CSS.supports('--a',0);
	if (isSupported) {
		/* supported */
	} else {
		/* not supported */
	}
</script>

JavaScript操作CSS变量的写法如下。

<script>
	//设置变量
	document.body.style.setProperty('--primary','#7F583F');

	//读取变量
	document.body.style.getPropertyValue('--primary').trim();

	//删除变量
	document.body.style.removeProperty('--primary');
</script>
这意味着，JavaScript可以将任意值存入样式表。下面是一个监听事件的例子，事件信息被存入CSS变量。
<script>
	const docStyle = document.documentElement.style;

	document.addEventListener('mousemove',(e) => {
		docStyle.setProperty('--mouse-x',e.clientX);
		docStyle.setProperty('--mouse-y',e.clientY);
	})
</script>
那些对css无用的信息，也可以放入CSS变量。
--foo: if (x > 5) this.width = 10;
上面代码中， --foo的值在css里面是无效语句，但是可以被JavaScript读取。这意味着，可以把样式设置写在css变量中，让JavaScript读取。
所以，CSS变量提供了JavaScript与CSS通信的一种途径。