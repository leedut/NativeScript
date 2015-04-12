# NativeScript:数据绑定

翻译自[nativescript.org](http://docs.nativescript.org/bindings) 有误、不妥之处还望指出

### 使用数据绑定

数据绑定是NativeScript中用户界面和业务模型之间的连接方式。通过设置数据绑定可以使得数据源更新时，用户界面的显示数据同时改变。 

依赖于设置和模块加载，用户界面的数据改变也可以更新数据源。

	数据源可以是任何业务逻辑对象，而绑定目标可以是任意UI控件

### 基本概念
几乎所有的UI控件都可以绑定数据源，但是数据绑定还是有一些约束条件：

 * 绑定目标（指UI控件等）必须是可绑定的
 * 绑定目标必须含有相关属性来接受数据源的绑定；绑定可以是单向的也可以是双向的；如果不需要双向数据绑定，仅使用一个普通属性就可以
 * 数据源对象必须添加propertyChange时间来监听属性值修改

### 数据流的方向

数据绑定时需要指定数据流的方向。NativeScript支持以下两种数据绑定方式：

	1.单向	指数据源的修改会影响被绑定对象，而被绑定对象不会影响数据源
	2.双向	指数据源和被绑定对象间会互相影响，任意端的数据修改都会影响另一端
    
### 创建数据绑定
创建数据源：上面绑定概念中提到，数据源必须添加属性修改的监听事件。NativeScrip内置的一个类（Observable类）能实现这种监听需求。

```javascript
//Javascript
var observableModule = require("data/observable");
var source = new observableModule.Observable();
```

创建绑定目标：
我们添加一个Bindable类的实例（所有UI控件都是继承于这个类）。

```javascript
//Javascript
var textFieldModule = require("ui/text-field");
var targetTextField = new textFieldModule.TextField();
```

添加绑定：

```javascript
//Javascript
var bindingOptions = {
    sourceProperty: "textSource",
    targetProperty: "text",
    twoWay: true
};
targetTextField.bind(bindingOptions, source);
//set & get
source.set("textSource", "Text set via binding");
source.get("textSource");
```

在XML布局中添加绑定：

* 1.创建数据源,同上
* 2.在XML中使用mustache语法插入绑定属性
* 3.XML中添加的数据绑定默认都是双向的

```XML
<Page>
    <StackLayout>
        <TextField text= {{ textSource }} />
        </StackLayout>
</Page>
```

在XML中添加数据绑定我们只声明被绑定对象的text属性和添加数据源textSource，却没有添加绑定，所以还需要通过函数把声明的textSource和text属性值绑定起来。

### 绑定数据源

绑定操作可调用bind函数实现，例如targetTextField.bind(bindOptions,[source])。

对于Bindable类（所有的UI控件等）来说，被绑定的数据源对象会成为控件的bindingContext属性。而且这种属性还会沿着节点树向下传递，即可以直接把数据源绑定在最上层的控件Page或StackLayout等上，内部的textfield也会拿到其对应的数据绑定。（但是textfield必须在xml内部写明所绑属性值text，或调用bind函数在bindOptions中声明）。

```javascript
//javascript
page.bindingContext = source;
//or
stackLayout.bindingContext = source;
```

甚至可以通过把某属性设定为函数的方式实现控件的操作事件绑定（tap等）。

```javascript
//javascript
page.bindingContext = source;
source.set("onTap", function(eventData) {
        console.log("button is tapped!");
});
```

```xml
//xml
<Page>
    <StackLayout>
        <Button text="Test Button For Binding" tap="{{ onTap }}" />
    </StackLayout>
</Page>
```

### 取消数据绑定

实际上没有必要取消数据绑定，NativeScript在这里使用弱引用来避免内存泄露。但是有的时候可能业务逻辑需要我们来取消控件上的数据绑定，调用unbind(propertyName)函数即可。

```javascript
//javascript
targetTextField.unbind("text");
```