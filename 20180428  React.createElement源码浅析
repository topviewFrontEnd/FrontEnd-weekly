### JSX
> 定义：JSX是一种JavaScript的语法扩展，可以说是React的一种模板语言，JSX用来声明React当中的元素
- 指定属性
```
//可以使用引号来定义以字符串为值的属性：
const element = <div tabIndex="0"></div>;

//也可以使用大括号来定义以JavaScript表达式为值的属性：
const element = <img src={user.avatarUrl} />;

//JSX使用camelCase（小驼峰）命名来定义属性的名称
```
- 嵌套格式
    - 闭合式
    - 相互嵌套用()括起来
- JSX中可任意使用JavaScript表达式，但要包含在大括号里，另外，JSX本身其实也是一种表达式

```
function getGreeting(user) {
    if (user) {
        return <h1>Hello, {formatName(user)}!</h1>;
    }
    return <h1>Hello, Stranger.</h1>;
}
```
- 防注入攻击
> React DOM在渲染之前默认会过滤所有传入的值(花括号内)。它可以确保应用不会被注入攻击。
#### JSX和 React.createElement的关系
**Babel转译器会把JSX转换成一个名为React.createElement()的方法调用**

```
const element = (
    <h1 className="greeting">
        Hello, world!
    </h1>
);
 
//等价于
const element = React.createElement(
    'h1',
    { className: 'greeting' },
    'Hello, world!'
);
```
注意：自定义组件首字母要大写

```
<div>
    <img src="avatar.png" className="profile"/>
    <Hello/>
</div>;

//将会被 Babel转换为
React.createElement("div", null,React.createElement("img", {
 src: "avatar.png",
 className:"profile"
}),React.createElement(Hello, null));
```

babel在编译时会判断 JSX中组件的首字母，当首字母为**小写**时，其被认定为**原生 DOM标签**， createElement的第一个变量被编译为**字符串**；当首字母为**大写**时，其被认定为**自定义组件**， createElement的第一个变量被编译为**对象**；

**总之，JSX只是为 React.createElement(component,props,...children)方法提供的语法糖。也就是说所有的 JSX代码最后都会转换成 React.createElement(...)这种形式，相当于一个对象变量**

### React.createElement
```
ReactElement.createElement =
function(type,config,children)
```
方法接受三个参数，第一个参数是**组件类型**，第二个参数是**要传递给组件的属性**，第三个参数是**children**。方法最终会返回一个具有以下属性的对象：

```
{
  $$typeof:REACT_ELEMENT_TYPE,
  type:type,
  key:key,
  ref:ref,
  props:props,
  _owner:owner,
  _store:{},
  self:{},
  _source:{}
};
```
- [x] 先来了解一下各个属性代表什么吧

**$$typeof**

被赋值为`REACT_ELEMENT_TYPE`，是一个常量，用来标识该对象是一个ReactElement。
```
var REACT_ELEMENT_TYPE = 
typeof Symbol === 'function' && 
Symbol['for'] &&
 Symbol['for']('react.element') || 
0xeac7;
```
- 如果支持Symbol就会用Symbol.for方法创建一个key为react.element的symbol,否则就会被赋值为0xeac7。

- 当它是一个 Symbol类型的变量，这个变量可以防止 XSS(跨站脚本攻击)，因为JSON中不能存储 Symbol类型的变量。

- 渲染的过滤条件之一

`ReactElement.isValidElement`函数用来判断一个 React组件是否是有效的，下面是它的具体实现。
```
ReactElement.isValidElement = function(object) {
  return (
    typeof object === 'object' &&
    object !== null &&
    object.$$typeof === REACT_ELEMENT_TYPE
  );
};
```
可见 React渲染时会把没有 $$typeof标识，以及规则校验不通过的组件过滤掉。

**type** 元素的类型，可以是原生html类型（字符串），或者自定义组件（函数或 class）

**key**  组件的唯一标识，用于 Diff算法

**ref**  用于访问原生 dom节点

**props**  传入组件的 props

**owner** 当前正在构建的 Component所属的 Component,也就是说父组件
 
**_self**  指定当前位于哪个组件实例

**_source** 指定调试代码来自的文件( fileName)和代码行数( lineNumber)

> 注意： self、 source只有在非生产环境才会被加入对象中

- [x] 具体过程

- 处理props
    - 将特殊属性 ref、 key从 config中取出并赋值
    - 将特殊属性 self、 source从 config中取出并赋值
    - 将除特殊属性的其他属性取出并赋值给 props

```
var propName;
  // Reserved names are extracted
  var props = {};
  var key = null;
  var ref = null;
  var self = null;
  var source = null;
  if (config != null) {
    if (hasValidRef(config)) {
      ref = config.ref;
    }
    if (hasValidKey(config)) {
      key = '' + config.key;
    }
    self = config.__self === undefined ?
    null : config.__self;
    source = config.__source === undefined ?
    null : config.__source;
    for (propName in config){
      if (hasOwnProperty.call(config, propName) &&
          !RESERVED_PROPS.hasOwnProperty(propName)) {
        props[propName] = config[propName];
      }
    }
  }
```

hasValidRef和hasValidKey方法用来校验config中是否存在ref和key属性，有的话就分别赋值给key和ref变量。然后将config.__self和config.__source分别赋值给self和source变量，如果不存在则为null。

```
function hasValidKey(config) {
  if ("development" !== 'production') {
    if (hasOwnProperty.call(config, 'key')) {
      var getter = Object.getOwnPropertyDescriptor(config, 'key').get;
      if (getter && getter.isReactWarning) {
        return false;
      }
    }
  }
  return config.key !== undefined;
}

//首先判断ref是否是config自身的属性，而不是从原型链上继承来的属性，然后再判断此ref是否有效。
```

并通过一个for in循环,**把config中的属性添加的props对象中**，添加的时候会过滤掉config中key、ref、__self、__source这几个值。RESERVED_PROPS就是一个要**过滤的属性的字典对象**：
```
var RESERVED_PROPS = {
  key: true,
  ref: true,
  __self: true,
  __source: true
};
```
- 获取子元素
    - 获取子元素的个数 —— 第二个参数后面的所有参数
    - 若只有一个子元素，赋值给 props.children
    - 若有多个子元素，将子元素填充为一个数组赋值给 props.children
```
var childrenLength = arguments.length - 2;
  if (childrenLength === 1) {
    props.children = children;
  } else if (childrenLength > 1) {
    var childArray = Array(childrenLength);
    for (var i = 0; i < childrenLength; i++) {
      childArray[i] = arguments[i + 2];
    }
    if (process.env.NODE_ENV !== 'production') {
      if (Object.freeze) {
        Object.freeze(childArray);
      }
    }
    props.children = childArray;
  }
```

- 处理默认props
```
if (type && type.defaultProps) {
    var defaultProps = type.defaultProps;
    for (propName in defaultProps) {
      if (props[propName] === undefined) {
        props[propName] = defaultProps[propName];
      }
    }
  }
```
也就是将组件的静态属性 defaultProps定义的默认 props进行赋值

- 最后，将我们获取到的属性传给传递给**ReactElement**方法最终生成一个对象并返回。

ReactElement方法中，首先会创建一个element，


```
ReactElement = function(type, key, ref, self, source, 
ReactCurrentOwner.current, props);
```
ReactCurrentOwner.current是一个运行是赋值的对象，值可能是null或者是一个ComponentWrapper的实例
```
var element = {
    $$typeof: REACT_ELEMENT_TYPE,
    type: type,
    key: key,
    ref: ref,
    props: props,
    _owner: owner
  };
```
之后会判断是否是开发模式，如果是就会添加_store、_source、_self属性进去

```
if (process.env.NODE_ENV !== 'production') {
    element._store = {};
    if (canDefineProperty) {
      Object.defineProperty(element._store, 
    'validated',
    {
        configurable: false,
        enumerable: false,
        writable: true,
        value: false
      });
      Object.defineProperty(element, '_self', 
    {
        configurable: false,
        enumerable: false,
        writable: false,
        value: self
      });
      Object.defineProperty(element, '_source',
    {
        configurable: false,
        enumerable: false,
        writable: false,
        value: source
      });
    } else {
      element._store.validated = false;
      element._self = self;
      element._source = source;
    }
    if (Object.freeze) {
      Object.freeze(element.props);
      Object.freeze(element);
    }
  }
```
### 总结
![image](http://gwjyhs.com/t6/702/1556359104x2728294061.png)

最后就是拿到这个ReactElement对象与之前的进行一些对比以及diff运算等一系列操作渲染到页面成为HTMLElement


参考链接：

https://www.jianshu.com/p/0825218af437

https://www.jianshu.com/p/82de300707df




