---
title: javascript数据结构-栈
date: 2019-01-21 09:41:54
tags: 数据结构
---

## 栈的定义

栈是一种特殊的线性表，仅能够在栈顶进行操作，有着后进先出的特点

## 实现栈

```js
function Stack(){
    var items = [] //使用数组存储数据
    
    this.push = function(item){
        items.push(item)
    }
    
    this.pop = function(){
        return items.pop()
    }
    
    this.top = function(){
        return items[items.length-1]
    }
    
    this.isEmpty = function(){
        return items.length == 0;
    }
    
    this.size = function(){
        return items.length
    }
    
    this.clear = function(){
        items = []
    }
}
```

## 应用

1. 找合法配对括号

   ```
    "sdf(ds(ew(we)rw)rwqq)qwewe"  合法
    "(sd(qwqw)sd(sd))"    合法
    "()()sd()(sd()fw))("  不合法
   ```

   思路：遍历数组，如果为左括号，将左括号压入栈中，如果为右括号，判断是否为空，不为空说明内部有左括号，使用pop方法弹出抵消掉，最后栈内还有左括号说明不合法，反之合法

   具体代码实现：

   ```js
   function is_leagl_brackets(string){
       var stack = new Stack();
       for(var i=0; i<string.length; i++ ){
           var item = string[i];
           if(item == "("){
               // 将左括号压入栈
               stack.push(item);
           }else if (item==")"){
               // 如果为空,就说明没有左括号与之抵消
               if(stack.isEmpty()){
                   return false;
               }else{
                   // 将栈顶的元素弹出
                   stack.pop();
               }
           }
   
       }
       return stack.size() == 0;
   };
   
   console.log(is_leagl_brackets("()()))"));
   console.log(is_leagl_brackets("sdf(ds(ew(we)rw)rwqq)qwewe"));
   console.log(is_leagl_brackets("()()sd()(sd()fw))("));
   ```

   

2. 计算逆波兰表达式（后缀表达式）

   ```
    * ["4", "13", "5", "/", "+"] 等价于(4 + (13 / 5)) = 6
    * ["10", "6", "9", "3", "+", "-11", "*", "/", "*", "17", "+", "5", "+"]
    * 等价于((10 * (6 / ((9 + 3) * -11))) + 17) + 5
   ```

   思路：遍历数组，非表达式压入栈中，从栈顶部弹出两个元素，对这两个元素进行计算，结果压入栈中

   具体代码实现：

   ```js
   function calc_exp(exp){
       var stack = new Stack();
       for(var i = 0; i < exp.length;i++){
           var item = exp[i];
   
           if(["+", "-", "*", "/"].indexOf(item) >= 0){
               // 从栈顶弹出两个元素
               var value_1 = stack.pop();
               var value_2 = stack.pop();
               // 拼成表达式
               var exp_str = value_2 + item + value_1;
               // 计算并取整
               var res = parseInt(eval(exp_str));
               // 将计算结果压如栈
               stack.push(res.toString());
           }else{
               stack.push(item);
           }
       }
       // 表达式如果是正确的,最终,栈里还有一个元素,且正是表达式的计算结果
       return stack.pop();
   };
   
   
   var exp_1 = ["4", "13", "5", "/", "+"];
   var exp_2 = ["10", "6", "9", "3", "+", "-11", "*", "/", "*", "17", "+", "5", "+"];
   var exp_3 = [ '1', '4', '5', '+', '3', '+', '+', '3', '-', '9', '8', '+', '+' ];
   console.log(calc_exp(exp_1));
   console.log(calc_exp(exp_2));
   console.log(calc_exp(exp_3));
   ```

3. 实现一个有min方法的栈，返回栈里最小的元素

   思路：用两个栈，一个放普通的元素stack1 一个放最小的元素stack2，判断stack2中的元素是否为空或者比当前push的元素小，则把当前元素压入stack2中，如果stack2中的元素大于当前push的元素，则把stack2顶部的元素取出再压入stack2中

   具体代码实现：

   ```js
   function MinStack(){
       var data_stack = new Stack();   // 存储数据
       var min_stack = new Stack();    // 存储最小值
   
       // push方法
       this.push = function(item){
           data_stack.push(item);
   
           // min_stack为空或者栈顶元素大于item
           if(min_stack.isEmpty() || item < min_stack.top()){
               min_stack.push(item);
           }else{
               min_stack.push(min_stack.top());
           }
   
       };
   
       // 弹出栈顶元素
       this.pop = function(){
           data_stack.pop();
           min_stack.pop();
       };
   
       // 返回栈的最小值
       this.min = function(){
           return min_stack.top();
       };
   };
   
   
   minstack = new MinStack();
   
   //minstack.push(3);
   //minstack.push(6);
   //minstack.push(8);
   //console.log(minstack.min());
   //minstack.push(2);
   //console.log(minstack.min());
   //minstack.pop();
   //console.log(minstack.min());
   ```

4. 中序表达式转后序表达式