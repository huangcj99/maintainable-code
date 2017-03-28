###可维护与松耦合代码的编写
这几个月看了不少书啊，像设计模式，编写高性能js对我编写代码的风格和理念上有非常大的帮助，
里面有不少编写程序的时候需要注意的一些细节和一些原则，啊不，是建议。

那在这里我就简述一下我现在写代码的过程：
首先，编写可维护的代码，我个人觉得应该先关注我这个页面所需要的'表层逻辑'，就拿一个注册页来说，
点击注册按钮，触发表单验证，格式错误拒绝表单提交，反之用ajax异步提交到后台服务器中进行验证，后台
匹配后，注册成功则进入相应登录页面，失败则插入失败提示。

上面所说的就是我之前所说的'表层逻辑'，这里我们不关注表层逻辑的内部实现，要做到代码的可维护，这一层逻辑
首先得让别人看懂，因为项目可能以后会交给别人负责，如果你不想看你代码的人骂你- -。。

例：

```js
//点击注册按钮，触发表单的提交
$register_btn.on('click',() => {
    submitForm();
});

//下面的代码是为了将验证表单和提交表单后的操作进行分离
//包装函数
//这里我们改写了原型函数，以便之后我们可以用这个before函数对提交表单的函数进行前置包装
//让其能逻辑分离的同时并带有一定依赖关系
Function.prototype.before = function () {
	let _self = this;
	return function () {
		if ( !validataFunc.apply(this,arguments) ) {
			console.log("有错误,不允许提交表单");
		        //提交表单若失败，则return不进行表单提交			
		        return;
		}
		return _self.apply(this,arguments);
        }
};

//验证表单函数,部分代码省略
let validataFunc = function () {
	//创建验证器的实例
	let validator = new Validator();

	//给验证器添加验证方法
	validator.addMethod($usr[0],"isNonEmpty","请填写账户名");
  	...
  
  	//执行加入到验证器的方法并接收返回的msg
	let msg = validator.formValidated();

	//判断是否存在msg错误信息，没有则执行表单提交将表单数据post上服务器
	if (msg) {
		//账户名
		if (msg === "请填写账户名" || msg ==="请填写正确的手机号或邮箱") {
			$usr.val("");
			$usr_msg.html(msg);
		}
    		...
    		return false;
	}
	else {
		return true;
	}
};

//表单提交
let submitForm = function () {
	console.log("没有错误提交表单");
    let usr = $usr.val(),
        pwd = $pwd.val();
    let url = `http://localhost:8080/register?usr=${usr}&pwd=${pwd}`;
    
    //post数据到服务器
    $.post(url,(res) => {
        let resText = JSON.parse(res),
            $register_success = $('.register_success');
        if (resText.status){
            //插入注册成功并跳转页面
            ...
        }
        else {
            //插入注册失败的info并清空表单
            ...
        }
    })
};

//运用了我们前面在Funtion的原型内部添加的before对表单提交进行了包装
//让表单提交与表单验证逻辑分离
submitForm = submitForm.before(validataFunc);

```

上面提到的关注表层逻辑，即onclick => validataFunc => submitForm()，点击，验证，提交，通过可读的函数名将功能进行封装，达到表层逻辑的可读
如果以后有其他人维护你的代码，至少不需要花太多时间去阅读乱糟糟的代码。

那么表层逻辑之后就是各个模块的实现了，对于深层逻辑的实现，我也给个不成熟的小建议把，

####不！要！含！有！过！多！的！if！else！嵌！套！！！由于个人之前也是经常写一些嵌套的if-else代码，导致后期阅读起来那是相当的蛋疼！我不想再自我伤害了，也不希望以后工作了被别人伤害！= =。。。
