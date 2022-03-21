#<center>js学习</center>

## ES6特征

### Arrows箭头
箭头是使用`=>`语法的函数简写.它既支持语句块,也支持表达式块返回表达式的值.不同于函数的是箭头和它当前代码作用域都可以访问的是同一个'this':

```js
// 表达式块
var odds=evens.map(v=>v+1);
var nums=evens.map((v,i)=>v+i);
var pairs=evens.map(v=>({even:v,odd:v+1}))

// 语句块
nums.forEach(v=>{
    if(v%5==0)
        fives.push(v);
});

// this
var bob={
    _name:"Bob",
    _friends:[],
    printFriends() {
        this._friends.forEach(f=>console.log(this._name+"knows"+f));
    }
}
```
### Classes类

ES6中的类是基于原型的面向对象模式的简单补充.具有单个方便的声明形式使类模式更易于使用,并提倡互操作性.类支持基于原型的继承, super call, 实例以及静态方法和构造函数.
```js
class SkinnedMesh extends THREE.Mesh {
    constructor(geometry, materials)
    {
        super(geometry, materials);

        this.idMatrix=SkinnedMesh.defaultMatrix();
        this.bones=[];
        this.boneMatrices=[];
        //..
    }

    update(camera)
    {
        //...
        super.update();
    }
    get boneCount()
    {
        return this.bones.length;
    }

    set matrixType(matrixType)
    {
        this.idMatrix = SkinnedMesh[matrixType]();
    }

    static defaultMatrix()
    {
        return new THREE.Matrix4();
    }
}
```

### Enhanced Object Literals增强对象文字
对象文字扩展为支持在构造时设置原型,支持foo简写:foo赋值,定义方法,进行超级调用以及使用表达式计算属性名称.它们在一起也使对象文件和类声明更加紧密的结合在一起,并使基于对象的设计受益于某些便利.
```js
var obj= {
    // __proto__
    __proto__:theProtoObj,
    // handler:handler的简写
    handler,
    // 方法
    toString() {
        //super call
        return "d"+super.toString();
    },
    // 被计算的动态属性名
    ['prop_'+(()=>42)()]:42
};
```
### Template Strings模板字符串
模板字符串为构造字符串提供了语法糖.可以选择是否添加标签以允许对字符串结构进行自定义,从而避免注入攻击或从字符串内容中构建更高级别的数据结构.
```js
// 字面字符串构建
`In JavaScript '\n' is a line-feed.`
// 多行字符串
`In JavaScript this is
 not legal.`
 //字符串插入
 var name = "Bob", time = "today";
`Hello ${name}, how are you ${time}?`
// 构造一个HTTP请求前缀，用于解释替换和构造 
POST`http://foo.org/bar?a=${a}&b=${b}
     Content-Type: application/json
     X-Credentials: ${credentials}
     { "foo": ${foo},
       "bar": ${bar}}`(myOnReadyStateChangeHandler);
```
### Destructuring解构
解构允许使用模式匹配进行绑定,并支持匹配数组和对象.解构是软失败,类似于标准对象查找`foo["bar"]`,在找不到时产生未定义值.
```js
// 列表匹配
var [a,,b]=[1,2,3]

// 对象匹配
var {op:a,lhs:{op:b}, rhs:c}=getASTNode()

// 对象匹配简写,绑定'op','lhs','rhs'
var {op, lhs, rhs}=getASTNode()

// 软失败解构
var [a]=[];
a === undefined;

// 软失败解构,并设置默认值
var [a=1]=[];
a===1;
```
### Default+Rest+Spread
被调用方评估默认参数值.在函数调用中将数组转换为连续的参数.将尾随参数绑定到数组.rest取代了对参数的要求,并更直接的解决了常见的情况.
```js
function f(x,y=12) {
    return x+y;
}

f(3) ==15;

f(3,4) == 7;

function f1(x,...y) {
    // y是一个数组
    return x*y.length;
}

f1(3,"Hello", true)=6;

function f2(x,y,z) {
    return x+y+z;
}

f(...[1,2,3])==6;
```
### Let+Const

块范围绑定结构,`let`是新的`var`. `Const`是单赋值的.静态限制会阻止分配前使用.

```js
function f() {
    {
        let x;
        {
            // okay,当前块作用域的名称
            const x="sneaky";
            // 错误,是常量
            x="foo";
        }
        // 错误,已经在当前块声明
        let x="inner";
    }
}
```
### Iterators+For...Of
```js
let fibonacci = {
    [Symbol.iterator] () {
        let pr=0,cur=1;
        return {
            next () {
                [pre,cur]=[cur,pre+cur];
                return {done:false, value:cur}
            }
        }
    }
}

for(var n of fibonacci) {
    //截断到长度为1000的序列
    if(n>1000)
        break;
    console.log(n);
}

interface IteratorResult {
    done: boolean;
    value: any;
}
interface Iterator {
    next() :IteratorResult;
}
interface Iterable {
    [Symbol.iterator]():Iterator
}
```


### Generators

```js
ar fibonacci = {
  [Symbol.iterator]: function*() {
    var pre = 0, cur = 1;
    for (;;) {
      var temp = pre;
      pre = cur;
      cur += temp;
      yield cur;
    }
  }
}

for (var n of fibonacci) {
  // truncate the sequence at 1000
  if (n > 1000)
    break;
  console.log(n);
}

interface Generator extends Iterator {
    next(value?: any): IteratorResult;
    throw(exception: any);
}
```