FileUpload对象
在网页上传文件，最核心元素就是这个HTML DOM的FileUpload对象
	<input type="file">
其实在HTML文档中该标签没出现一次，一个FileUpload对象就会被创建。该标签包含一个按钮，用来打开文件选择对话框，以及一段文字显示选中的文件命名或提示没有文件被选中。
把这个标签放在<form>标签内，设置form的action为服务器目标上传地址，并点击submit按钮或通过JS调用form的submit()方法就可以实现最简单的文件上传了。
	<form id="uploadForm" method="post" action="upload.php" enctype="multipart/form-data>">
		<input type="file" id = "myFile" name = "file" />
		<input type="submit" value="提交" />
	</form>
上面这段代码就是最简单的文件上传了

更优雅的上传
现代网页通过----XMLHttpRequest来实现用户与服务器的无刷新交互，如果使用现代浏览器开发，那么上传文件就比较容易，针对支持XMLHttpReauest Level2的浏览器
XMLHttpRequest Level 1的限制：
	仅支持文本数据传输，无法传输二进制数据
	传输数据时，没有进度信息提示，只能提示是否完成
	受浏览器同源策略限制，只能请求同源资源
	没有超时机制，不方便掌控ajax请求节奏
XMLHttpRequest Level 2 的改进
	支持二进制数据，可以上传文件，可以使用FormData对象管理表单
	提供进度提示，可以通过xhr.upload.onprogress事件回调方法获取传输进度
	依然受同源策略限制，这个安全机制不会变，xhr2新提供Access-Control-Allow-Origin等header，设置为*时表示允许任何域名请求，从而实现跨域CORS访问
	可以设置timeout及ontimeout，方便设置超时时长和超时后续处理。
目前主流浏览器基本都支持XHR2，除了IE系列需要IE10及更高版本。IE10以下是不支持XHR2的。
上述中的FormData就是我们最常用的一种方式。通过在脚本里新建FormData对象，把File对象设置到表单项中，然后利用XMLHttpRequest异步上传到服务器
	<scrip>
		var xhr = new XMLHttpRequest();
		var formdata = new FormData();
		var fileInput = document.getElementById("myFile");
		var file = fileInput.files[0];

		formData.append('myFile', file);
		xhr.open('POST','/upload.php');
		xhr.onload = function () {
			if (this.status === 200) {
				//对请求成功的处理
			}
		}
		xhr.send(formData);
		xhr = null;
	</scrip>
上面一段代码完成了最基本的上传功能，为了更好的用户体验，我们还需要上传进度和图片预览功能

上传进度




