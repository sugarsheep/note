# 对代码行进行折行

> 可以在文本字符串中使用反斜杠对代码行进行换行,并不影响最终输出结果，还是输出到一行上
>
> ```js
> document.write("你好 \
> 世界!");
> ```

# 重新声明 JavaScript 变量

> 如果重新声明 JavaScript 变量，该变量的值不会丢失：
>
> 在以下两条语句执行后，变量 carname 的值依然是 "Volvo"：
>
> ```js
> var carname="Volvo"; 
> var carname;
> ```

# JavaScript 数据类型

> **值类型(基本类型)**：字符串（String）、数字(Number)、布尔(Boolean)、对空（Null）、未定义（Undefined）、Symbol。
>
> **引用数据类型**：对象(Object)、数组(Array)、函数(Function)。
>
> **注：**Symbol 是 ES6 引入了一种新的原始数据类型，表示独一无二的值。

## JavaScript 拥有动态类型

> JavaScript 拥有动态类型。这意味着相同的变量可用作不同的类型
>
> ```js
> var x;               // x 为 undefined
> var x = 5;           // 现在 x 为数字
> var x = "John";      // 现在 x 为字符串
> ```

## JavaScript 数组

> 下面的代码创建名为 cars 的数组：
>
> ```js
> var cars=new Array();
> cars[0]="Saab";
> cars[1]="Volvo";
> cars[2]="BMW";
> ```
>
> 或者 (condensed array):
>
> ```js
> var cars=new Array("Saab","Volvo","BMW");
> ```
>
> 或者 (literal array):
>
> ```js
> var cars=["Saab","Volvo","BMW"];
> ```

## JavaScript 对象

> 对象属性有两种寻址方式：
>
> ```js
> name=person.lastname;
> name=person["lastname"];
> ```

## Undefined 和 Null

> Undefined 这个值表示变量不含有值。
>
> 可以通过将变量的值设置为 null 来清空变量。
>
> ```js
> var cars = "hello";
> var cars = null;
> console.log(cars); //结果为null，不设置为null，结果为hello
> ```

## 声明变量类型

> 当您声明新变量时，可以使用关键词 "new" 来声明其类型：
>
> ```js
> var carname=new String;
> var x=      new Number;
> var y=      new Boolean;
> var cars=   new Array;
> var person= new Object;
> ```
>
> JavaScript 变量均为对象。当您声明一个变量时，就创建了一个新的对象。

## JavaScript 变量的生存期

> JavaScript 变量的生命期从它们被声明的时间开始。
>
> 局部变量会在函数运行以后被删除。
>
> 全局变量会在页面关闭后被删除。

## 向未声明的 JavaScript 变量分配值

> 如果您把值赋给尚未声明的变量，该变量将被自动作为 window 的一个属性。
>
> 这条语句：
>
> ```js
> carname="Volvo";
> ```
>
> 将声明 window 的一个属性 carname。
>
> 非严格模式下给未声明变量赋值创建的全局变量，是全局对象的可配置属性，可以删除。
>
> ```js
> var var1 = 1; // 不可配置全局属性
> var2 = 2; // 没有使用 var 声明，可配置全局属性
> 
> console.log(this.var1); // 1
> console.log(window.var1); // 1
> 
> delete var1; // false 无法删除
> console.log(var1); //1
> 
> delete var2; 
> console.log(delete var2); // true
> console.log(var2); // 已经删除 报错变量未定义
> ```
> 如果变量在函数内没有声明（没有使用 var 关键字），该变量为全局变量。
>
> 以下实例中 carName 在函数内，但是为全局变量。
>
> ```js
> // 此处可调用 carName 变量
>  
> function myFunction() {
>     carName = "Volvo";
>     // 此处可调用 carName 变量
> }
> ```

## js绝对相等

> === 为绝对相等，即数据类型与值都必须相等

# 循环

## For/In 循环

> JavaScript for/in 语句循环遍历对象的属性：
>
> ```js
> var person={fname:"John",lname:"Doe",age:25}; 
>  
> for (x in person)  // x 为属性名
> {
>     txt=txt + person[x];
> }
> ```
>
> 定义了数组后对数组进行赋值，中间如有某些下标未被使用（即未被赋值），在遍历的时候，采用一般的 for 循环和 for...in 循环得到的结果不同。
>
> for...in 循环会自动跳过那些没被赋值的元素，而 for 循环则不会，它会显示出 undefined。
>
> 点击下面的按钮，循环遍历
>
> ```js
> <button onclick="myFunction()">点击这里</button>
> <p id="demo"></p>
> <script>
> function myFunction(){
>     var array = new Array();
>     var x;
>     var txt=""
>     array[0] = 1;
>     array[3] = 2;
>     array[4] = 3;
>     array[10] = 4;
>     for( x in array ){
>         alert(array[x]);     // 依次显示出 1 2 3 4
>     } 
>     alert(array.length);    // 结果是11
>     for( var i=0 ; i<4 ; i++){
>         alert(array[i]);     // 依次显示出 1 undefined undefined 2 
>     }
>     document.getElementById("demo").innerHTML = txt;
> }
> </script>
> ```

## JavaScript 标签

> 正如您在 switch 语句那一章中看到的，可以对 JavaScript 语句进行标记。
>
> 如需标记 JavaScript 语句，请在语句之前加上冒号：
>
> ```js
> label:
> statements
> ```
>
> break 和 continue 语句仅仅是能够跳出代码块的语句。
>
> 语法:
>
> ```js
> break labelname; 
>  
> continue labelname;
> ```
>
> continue 语句（带有或不带标签引用）只能用在循环中。
>
> break 语句（不带标签引用），只能用在循环或 switch 中。
>
> 通过标签引用，break 语句可用于跳出任何 JavaScript 代码块：
>
> ```js
> cars=["BMW","Volvo","Saab","Ford"];
> list: 
> {
>     document.write(cars[0] + "<br>"); 
>     document.write(cars[1] + "<br>"); 
>     document.write(cars[2] + "<br>"); 
>     break list;
>     document.write(cars[3] + "<br>"); 
>     document.write(cars[4] + "<br>"); 
>     document.write(cars[5] + "<br>"); 
> }
> ```

# JavaScript typeof, null, 和 undefined

## typeof 操作符

> 你可以使用 typeof 操作符来检测变量的数据类型。
>
> ```js
> typeof "John"                // 返回 string 
> typeof 3.14                  // 返回 number
> typeof false                 // 返回 boolean
> typeof [1,2,3,4]             // 返回 object
> typeof {name:'John', age:34} // 返回 object
> ```

## null

> 在 JavaScript 中 null 表示 "什么都没有"。
>
> null是一个只有一个值的特殊类型。表示一个空对象引用。
>
> 用 typeof 检测 null 返回是object。
>
> 你可以设置为 null 来清空对象:
>
> ```js
> var person = null;           // 值为 null(空), 但类型为对象
> ```
>
> 你可以设置为 undefined 来清空对象:
>
> ```js
> var person = undefined;     // 值为 undefined, 类型为 undefined
> ```

## undefined 和 null 的区别

> null 和 undefined 的值相等，但类型不等：
>
> ```js
> typeof undefined             // undefined
> typeof null                  // object
> null === undefined           // false
> null == undefined            // true
> ```

# JavaScript 类型转换

> Number() 转换为数字， String() 转换为字符串， Boolean() 转化为布尔值。
>
> 请注意：
>
> - NaN 的数据类型是 number
> - 数组(Array)的数据类型是 object
> - 日期(Date)的数据类型为 object
> - null 的数据类型是 object
> - 未定义变量的数据类型为 undefined
>
> 如果对象是 JavaScript Array 或 JavaScript Date ，我们就无法通过 **typeof** 来判断他们的类型，因为都是 返回 object。

## 一元运算符 +

> **Operator +** 可用于将变量转换为数字：
>
> ```js
> var y = "5";      // y 是一个字符串
> var x = + y;      // x 是一个数字
> ```
>
> 如果变量不能转换，它仍然会是一个数字，但值为 NaN (不是一个数字):
>
> ```js
> var y = "John";   // y 是一个字符串
> var x = + y;      // x 是一个数字 (NaN)
> ```
>
> 

## 自动转换类型

> 当 JavaScript 尝试操作一个 "错误" 的数据类型时，会自动转换为 "正确" 的数据类型。
>
> 以下输出结果不是你所期望的：
>
> ```js
> 5 + null    // 返回 5         null 转换为 0
> "5" + null  // 返回"5null"   null 转换为 "null"
> "5" + 1     // 返回 "51"      1 转换为 "1"  
> "5" - 1     // 返回 4         "5" 转换为 5
> ```
>
> 

# JavaScript 变量提升

> JavaScript 中，函数及变量的声明都将被提升到函数的最顶部。
>
> JavaScript 中，变量可以在使用后声明，也就是变量可以先使用再声明。
>
> 变量提升：函数声明和变量声明总是会被解释器悄悄地被"提升"到方法体的最顶部。

## JavaScript 初始化不会提升

> JavaScript 只有声明的变量会提升，初始化的不会。
>
> 以下两个实例结果结果不相同：
>
> ```js
> var x = 5; // 初始化 x
> var y = 7; // 初始化 y
> 
> elem = document.getElementById("demo"); // 查找元素 
> 7elem.innerHTML = x + " " + y;           // 显示 x 和 y,5 7
> ```
>
> ```js
> var x = 5; // 初始化 x
> 
> elem = document.getElementById("demo"); // 查找元素 
> elem.innerHTML = x + " " + y;           // 显示 x 和 y,5 undefined
> 
> var y = 7; // 初始化 y
> ```
>
> JavaScript 严格模式(strict mode)不允许使用未声明的变量。

## 基础知识

### 使用立即执行函数封装模块

> Javascript不是一种模块化编程语言，在es6以前，它是不支持”类”（class），所以也就没有”模块”（module）了，为了解决这个问题，则使用立即执行函数封装模块

例如：

```js
        let myModule = (function(){
            let count = 0;
            let increment = ()=>{
                count = count+1;
                console.log(count);
            } 
            let decrement = ()=>{
                count = count-1;
                console.log(count);
            }
            return{
                increment,
                decrement
            }
        })();
        myModule.increment();
        myModule.decrement();
```

