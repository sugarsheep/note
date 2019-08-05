## 说明

## 目录

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

### 纯函数

> - 函数的返回结果只依赖于它的参数：不依赖于类似于外部变量这种变量
> - 函数执行过程里面没有副作用：如参数是对象，不会修改对象的值，除了修改外部的变量，一个函数在执行过程中还有很多方式产生外部可观察的变化，比如说调用 DOM API 修改页面，或者你发送了 Ajax 请求，还有调用 window.reload 刷新浏览器，甚至是 console.log 往控制台打印数据也是副作用

### Object.assign()

> 使用后面的参数扩展第一个参数

```js
Object.assign({}, state, {
        visibilityFilter: action.filter
      })
```