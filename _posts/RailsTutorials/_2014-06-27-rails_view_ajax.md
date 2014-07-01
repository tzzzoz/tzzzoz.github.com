*view里用ajax具体怎么调用？*

有几种方式

第一种 比较纯净的json

你只返回json数据 然后前台用handlebars这样的template engine去渲染

第二种 返回纯html 然后$('#side').html覆盖掉

第三种 用类似pjax的技术

	def index
	  if request.headers['X-PJAX']
	    render :layout => false
	  end
	end

不渲染layout