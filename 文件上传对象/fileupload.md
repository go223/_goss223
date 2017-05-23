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
	<script>
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
	</script>
上面一段代码完成了最基本的上传功能，为了更好的用户体验，我们还需要上传进度和图片预览功能

上传进度
因为是XNLHttpRequest Level 2，所以很容易就可以支持对上传进度的监听。chrome的developer tools的console里new一个XHR对象，调用点运算符就可以看到智能提示出来一个onprogress事件监听器，那是不是我们只要绑定XHR对象的progress事件就可以了呢？
但是XHR对象的直属progress事件并不是用来监听上传资源的进度的，XHR对象还有一个属性upload，它返回一个XMLHttpRequestUpload对象，这个对象拥有下列方法：
	onloadstart
	onprogress
	onabort
	onerror
	onload
	ontimeout
	onloadend
这些方法在XHR对象中都存在同名版本，区别是后者用于加载资源时，而前者用于资源上传时。其中onprogress事件回调方法可用于跟踪资源上传的进度，它的event参数对象包含两个重要的属性loaded和total。分别代表当前已上传的字节数（number of bytes）和文件的总字节数。比如我们可以这样计算进度百分比：
	xhr.upload.onprogress = function (event) {
		if (event.lengthCompultable) {
			var percentComplete = (event.load / event.total) * 100;
			//对进度进行处理
		}
	}
其中事件的lengthComplutable属性代表文件总大小是否可知。如果lengthComputable属性是false，那么意味着总字节数是未知并且total的值为零。
如果是现代浏览器，可以直接配合HTML5提供的元素使用，方法快捷的显示进度条。
<progress id = "myProgress" value = "50" max = "100"></progress>
其value属性绑定上面代码中的percentComplete的值即可。再进一步我们还可以对<progress>的样式统一调整，实现优雅降级方案。

图片预览
普通青年的图片预览方式是待文件上传成功后，后台返回上传文件的url，然后把预览图片的img元素的src指向该url。这其实达不到预览的效果和目的。
使用H5的FileReader API
	<script>
		function handleImageFile (file) {
			var previewArea = document.getElementById('previewArea');
			var img = document.createElement('img');
			var fileInput = document.getElementById("myFile");
			var file = fileInput.files[0];
			img.file = file;
			previewArea.appendChild(img);

			var reader = new FileReader();
			reader.onload = (function(aImg) {
				return function (e) {
					aImg.src = e.target.result;
				}
			})(img);
			reader.readAsDataURL(file);
		}
	</script>
这里我们使用FileReader来处理图片的异步加载。在创建的FileReader对象之后，我们建立了onload函数，然后调用readerAsDataURL()开始在后台进行读取操作，当图像文件加载后，转换成一个data:URL,并传递到onload回调函数中设置给img的src。
另外我们还可以通过使用对象URL来实现预览
	<script>
		var img = document.createElement("img");
		img.src = window.URL.createObjectURL(file);
		img.onload = function () {
			//明确地通过调用释放
			window.URL.revokeObjectURL(this.src);
		}
		previewArea.appendChild(img);
	</script>

多文件上传
FileUpload对象有个multiple属性
<input id = "myFile" type = "file" multiple />
这样我们就能在打开的文件选择对话框中选中多个文件了。然后你再代码里拿到的FileUpload对象的files属性就是一个选中的多文件的数组了。
	<script>
		var fileInput = document.getElementById("myFile");
		var files = fileInput.files;
		var formData = new FormData();
		for (var i = 0; i < files.length; i++) {
			var file = files[i];
			formData.append('files[]', file, file.name);
		}
	</script>
FormData的append方法提供第三个可选参数用于指定文件名，这样就可以使用同一个表单项名，然后用文件名区分上传的多个文件。这样也方便前后台的循环操作。

二进制上传
有了FileReader，其实我们还有一种上传途径，读取文件内容后直接以二进制格式上传。
	<script>
		var reader = new FileReader();
		reader.onload = function() {
			xhr.sendAsBinary(this.result);
		}
		//把从input里读取的文件内容，放到FileReader的result字段里
		reader.readAsBinaryString(file);
		//不过chrome已经把XMLHttpRequest的sendAsBinar方法移除了，所以以后可能得自行实现一个
		XMLHttpRequest.prototype.senfAsBinary = function (text) {
			var data = new ArrayBuffer(text.length);
			var ui8a = new Uint8Array(data,0);
			for (var i = 0; i < text.length; i++) {
				ui8a[i] = (text.charCodeAt(i) & 0xff);
			}
			this.send(ui8a);
		}
	</script>
这段代码将字符串转成8位无符号整型，然后存放到一个8位无符号整型数组里面，再把整个数组发送出去。

拖拽的支持
利用HTML的都让 & drop事件，我们可以很快实现对拖拽的支持，首先我们可能需要确定一个允许拖放的区域，然后绑定相应的事件进行处理
	<script>
		var dropArea;

		dropArea = document.getElementById("dropArea");
		dropArea.addEventListener("dragenter", handleDragenter, false);
		dropArea.addEventListener("dragover", handDragover, false);
		dropArea.addEventListener("drop", handleDrop, false);
		//阻止dragenter和dragover的默认行为，这样才能使drop事件被触发
		function handleDragenter (e) {
			e.stopPropagation();
			e.preventDefault();
		}

		function handleDragover (e) {
			e.stopPropagation();
			e.preventDefault();
		}

		function handleDrop (e) {
			e.stopPropagation();
			e.preventDefault();

			var dt = e.dataTransfer;
			var files = dt.files;

			// handle files ...
		}
	</script>
这里可以把通过事件对象的dataTransfer拿到的files数组和之前相同处理，以实际预览上传等功能。有了这些事件回调，我们也可以在不同的事件给我们UI元素添加不同的class来实现更好交互效果。

IE10以下的浏览器如何实现无刷新上传
借用iframe
现代浏览器中我们可以用XMLHttpRequest Level 2来支持二进制数据，异步文件上传，并且动态创建FormData。而低版本的IE里的XMLHttpRequest是Level 1。所以我们通过XHR异步向服务器发上传请求的路走不通了。只能用form的submit。
form的submit会导致页面刷新。我们不能让form的submit不刷新整个页面，可以利用iframe。把form的target指定到一个看不见的iframe，那么返回的数据就会被这个iframe接受，于是乎就只有这个iframe会刷新。
	<script type="text/javascript">
		window.__iframeCount = 0;
		var hiddenframe = document.createElement("iframe");
		var frameName = "upload-iframe" + ++window.__iframeCount;
		hiddenframe.name = frameName;
		hiddenframe.id = frameName;
		hiddenframe.setAttribute("style","width:0;height:0;dispaly:none");
		document.body.appendChild(hiddenframe);

		var form = document.getElementById("myForm");
		form.target = frameName;

		//然后响应iframe的onload事件，获取response
		hiddenframe,onload = function () {
			//获取iframe的内容，即服务器返回的数据
			var resData = this.contentDocument.body.textContent || this.contentWindow.document.body.textContent;
			//处理数据

			//删除iframe
			setTimeout(function () {
				var _frame = document.getElementById(frameName);
				_frame.parentNode.removeChild(_frame);
			},100);
		}
	</script>
iframe的实现大致如此，但是如果文件上传的地址与当前页面不在同一个域下就会出现跨域问题。导致iframe的onload回到里的访问服务返回的数据失败。
这时我们再祭出JSONP这把利剑，来解决跨域问题。首先在上传之前注册一个全局的函数，把函数名发给服务器。服务器需要配合在response里让浏览器直接调用这个函数。
	<script type="text/javascript">
	//生成全局函数名，避免冲突
		var CALLBACK_NAME = 'CALLBACK_NAME';
		var genCallbackName = (function () {
			var i = 0;
			return function () {
				return CALLBACK_NAME + ++i;
			}
		})();

		var curCallbackName = genCallbackName();
		window[curCallbackName] = function (res) {
			//处理response...

			//删除iframe
			var _frame = document.getElementById(frameName);
			_frame.parentNode.removeChild(_frame);
			//删除全局函数本身
			window[curCallbackName] = undefined;
		}
		//如果已有其他参数，这里需要判断一下，改为拼接&callback=
		form.action = form.action + '?callback=' + curCallbackName;
	</script>
