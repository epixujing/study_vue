关于vue
===
  >v-for + v-bind命令连用 
===
```
 <template v-for="(item,index) in items">
  <li :class='isActive[index]?"isActive":"noActive"' data-id='index' @click='toggle(index)'> {{item}}   </li>
```

v-for渲染出来多个具体的li标签  通过状态控制具体的标签class  通过点击事件修改状态，修改了的状态再改变具体的class   实现整个过程  

    指令影响状态   状态影响视图   我们只需要关注状态和指令   视图的修改默认交给框架  
其中发现的问题  
**1** 模板渲染的规则究竟是什么   就是vue如何处理具体的bind命令  bind命令里面都可以输入什么？
知道可以使用[],那么里面可以是用函数么  尝试了系统默认的console.log() 是不可以的  
**2** click命令的传参问题   在v-for中 是可以是用index当做参数传入的  通过传入的参数的不同  来识别点击的具体元素是什么 相比于操作DOM更加简单


关于CSS
===
>css的基本功还是不扎实