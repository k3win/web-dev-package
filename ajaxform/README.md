# ajax form。
#jquery异步上传表单插件

##字段：

###type
（默认为表单的method属性值，若未设置取GET）
请求的类型，例如：POST、GET、PUT及PROPFIND。大小写不敏感。

###url
（默认取表单的action属性值，若未设置默认取window.location.href）请求的URL地址，可以为绝对地址也可以为相对地址。

###data
（对象成员必须包含name和value属性）提供额外数据对象，通过$.param()函数返回序列化后的字符串，稍后会拼接到表单元素序列化的字符串之后。

###extraData
（此参数无需外部提供，由内部处理）
此参数是data在进行序列化成字符串之前的一个拷贝，只用于在表单包含<input type=”file” />并且是老浏览器。
因为在老浏览器中文件上传文件我们需要通过<iframe>来模拟异步提交，此时extraData会转变为<input type=”hidden” />元素包含在表单中，被一起提交到服务器。

###dataType
一般不需自己设置。参数作用请看：《jQuery.ajax()-dataType》

###traditional
如果你想要用传统的方式来序列化数据，那么就设置为true。请参考$.param()深度递归详解。

###delegation
（适用于ajaxForm）ajaxForm支持Jquery插件的委托方式（需要Jquery v1.7+），所以当你调用ajaxForm的时候其表单form不一定存在，但动态构建的form会在适当的时候调用ajaxSubmit。Eg：
$('#myForm').ajaxForm({ 
    delegation: true,
    target: '#output'
});

###replaceTarget
（默认：false）与target参数共同起作用，True则执行replaceWirh()函数，false则执行html()函数

###target
提供一个Html元素，在请求“成功”并且未设置dataType参数，则将返回的数据replaceWith()或html()掉对象原来的内容，再遍历对象调用success回调函数。
if (!options.dataType && options.target) {
    var oldSuccess = options.success || function(){};
    callbacks.push(function(data) {
        var fn = options.replaceTarget ? 'replaceWith' : 'html';
        $(options.target)[fn](data).each(oldSuccess, arguments);
    });
}

###includeHidden
在请求成功后，若设置执行clearForm()函数清空表单元素则会根据includeHidden设置决定如何清空隐藏域元素。
1)传递true，表示清空表单的所有隐藏域元素。
2)传递字符串，表示清空特殊匹配的隐藏域表单元素，eg： $('#myForm').clearForm('.special:hidden')，清空class属性包含special值的隐藏域

###clearForm
请求成功时触发（同success），并用options. includeHidden做为回调函数参数。
回调函数：$form.clearForm(options.includeHidden);

###resetForm
请求成功时触发（同success）。
回调函数：$form.resetForm()

###semantic
布尔值，指示表单元素序列化时是否严格按照表单元素定义顺序。
在序列化只有<input type=”image” />元素会放在序列化字符串的最后，若semantic=true，则会按照它的定义顺序进行序列化。
若你服务器严格要求表单序列化字符串的顺序，则使用此参数进行控制。

###iframe
（默认：false）若有文件上传'input[type=file]:enabled[value!=""]'，指示是否应该使用<iframe>标签（在支持html5文件上传新特性的浏览器中不会使用iframe模式）
iframeTarget
指定一个现有的<iframe>元素，否则将自动生成一个<iframe>元素以及name属性值。若现有的<iframe>元素没有设置name属性，则会自动生成一个name值
iframeSrc
为<iframe>元素设定src属性值
 
##回调函数：

###beforeSerialize
提供在将表单元素序列化为字符串之前，处理表单元素的回调函数。
签名：function(form,options)
函数说明：当前表单对象、options参数集合
返回值：返回false，表示终止表单提交操作。

###beforeSubmit
提供在执行表单提交之前，处理数据的回调函数。
签名：function(a,form,options)
函数说明：通过formToArray(options.semantic, elements)返回的表单元素数组、当前表单对象、options参数集合
返回值：返回false，表示终止表单提交操作。

###ajaxSubmit函数处理流程：

1)         根据<form action=”” method=””>处理url、type参数以及success、iframeSrc等参数。
2)         触发beforeSerialize()回调函数
3)         序列化data参数和表单元素
4)         触发beforeSubmit()回调函数
5)         根据type参数处理options.data和options.url参数
6)         注册resetForm()和clearForm()回调函数
7)         注册将返回数据加载到options.target指定的元素上的回调函数
8)         注册success回调函数，若有options.target则循环该元素，并为每个子元素注册success回调函数
9)         处理<input type=”file” />文件上传元素
	a)         不包含文件元素，直接调用jQuery.ajax()函数。
	b)         包含文件元素，并且不支持XMLHttpRequest Level 2提供的文件上传新特性window.FormData。则通过IFrame模拟表单异步提交。
	                   i.              调用fileUploadIframe()函数。
	                   ii.              根据options. iframeTarget设置决定是创建一个<iframe>元素还是使用现有的<iframe>元素
	                   iii.              模拟xhr对象以及jQuery.ajax()过程，以支持xhr对象返回和ajax事件触发
	                   iv.              设置<form>的target指向<iframe>元素、encoding和enctypedata”、method为”post”值等等
	                   v.              处理options.extraData为<input type=”hidden” />元素并添加到<form>元素中。
	                   vi.              调用<form>的submit()事件。（同步提交，但因为<form>的target指向<iframe>标签，所以刷新的是<iframe>中的内容，以此模拟异步提交）
	c)         包含文件元素，并且支持XMLHttpRequest Level 2提供的新特性，则调用fileUploadXhr()函数，通过FormData()对象将数据传递给options.data参数，再调用jQuery.ajax(options)函数异步提交表单。并且XMLHttpRequest Level 2的新特性还支持进度条提示功能。（更多新特性请看：《XMLHttpRequest Level 2 使用指南》）
10)     将内部jqxhr缓存起来，以供访问。$form.removeData('jqxhr').data('jqxhr', jqxhr);
11)     返回表单元素本身，以便符合jQuery的链式操作模式。

##方法：

###$(“form1”).ajaxForm(options)
是对$(“any”).ajaxSubmit(options)函数的一个封装，适用于表单提交的方式（注意，主体对象是<form>），会帮你管理表单的submit和提交元素（[type=submit],[type=image]）的click事件。在出发表单的submit事件时：阻止submit()事件的默认行为（同步提交的行为）并且调用$(this).ajaxSubmit(options)函数。
ajaxForm支持Jquery插件的委托方式（需要Jquery v1.7+），所以当你调用ajaxForm的时候其表单form不一定存在，ajaxSubmit将在适当的时候调用。

###$(“form1”).ajaxFormUnbind()
取消$(“”).ajaxForm(options)函数对指定表单绑定的submit和click事件。

###$(“form1”).formToArray(semantic,elements)
个数组元素都是包含name和value属性的对象。返回值是内部构件的一个数组元素，而elements参数将包含除<input type=”image”>以外的所有表单元素。

###$(“form1”).formSerialize(semantic)
将表当前单元素序列化为字符串形式。

###$(“form1”).fieldSerialize(successful)  
序列化包含name属性的表单元素为一个字符串。Successful参数标识是否获取type为reset、button、checkbox、radio、submit、image值得元素以及<select>的值。返回$(el).val()。
###
$(“form1”).fieldValue(successful) 或 $.fieldValue(element, successful)
获取指定表单中的表单元素或指定表单元素的值。Successful参数标识是否获取type为reset、button、checkbox、radio、submit、image值得元素以及<select>的值。返回$(el).val()。

###$(“form1”).clearForm(includeHidden)
清空当前表单中input、select、textarea元素的值。includeHidden设置决定如何清空隐藏域元素。
a)         传递true，表示清空表单的所有隐藏域元素。
b)         传递字符串，表示清空特殊匹配的隐藏域表单元素，eg： $('#myForm').clearForm('.special:hidden')，清空class属性包含special值的隐藏域

###$.(“form1”).clearFields(includeHidden) 和 $.(“form1”).clearInputs(includeHidden)
作用相同，清空当前表单中所有表单元素的指。includeHidden设置决定如何清空隐藏域元素。
a)         传递true，表示清空表单的所有隐藏域元素。
b)         传递字符串，表示清空特殊匹配的隐藏域表单元素，eg： $('#myForm').clearForm('.special:hidden')，清空class属性包含special值的隐藏域

###$(“form1”).resetForm()
重置当前表单元素，导致所有表单元素重置到它的初始值。

###$(“form1”).selected(select)
将当前表单元素中所有checkbox、radio设置为select。select参数为布尔值。