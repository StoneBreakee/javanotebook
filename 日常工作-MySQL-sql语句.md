``` sql
alter table dps.dps_story
add column assigned_iteration_id int(11) null defalut null after iteration_id;
```

``` sql
ALTER TABLE `siway`.`dps_story` 
DROP COLUMN `assigned_iteration_id`;
```

mysql 默认事务隔离级别 
select @@tx_isolation 当前会话  
select @@global.tx_isolation; 系统隔离级别  

同一套应用系统部署多套，每一套对应一个独立的节点，节点之间共用同一套数据库，如何让多个节点中的一个且只有一个获得某任务的执行权限？  

[Spring Service中的private方法与protected方法的区别？](https://www.iteye.com/topic/1125911#2369539)  
[BigDecimal divide 除法触发异常](https://www.cnblogs.com/LeoBoy/p/5897754.html)
[spring bean scope章节译文](https://blog.csdn.net/dhaiuda/article/details/81900815)  
[stackoverflow request vs session 区别](https://stackoverflow.com/questions/3106452/how-do-servlets-work-instantiation-sessions-shared-variables-and-multithreadi/3106909#3106909)  
[Servlet Filter 详解](https://www.cnblogs.com/zlbx/p/4888312.html)
[Servlet FilterChain 责任链模式](https://www.cnblogs.com/lizo/p/7503862.html)  
[Git add 忽略某个文件的变更](https://blog.csdn.net/mp624183768/article/details/81127635)  
[Git 取消文件跟踪](https://www.cnblogs.com/zhuchenglin/p/7128383.html)

Vue

- 数据与方法  
当一个Vue实例被创建时，它向Vue的**响应式系统**中加入了其`data`对象中所有的属性。当这些属性值发生变化时，响应式系统会通知视图，视图将会产生响应，将视图中的值更新为最新的。注意：只有实例被创建时其data对象中的属性才具备响应式的特点，其属性才是响应式的。如果在创建之后再向data添加新属性，则该新属性不具有响应式的特点。  
访问Vue实例data对象中的属性，即`vue.property`，另Vue实例也自带系统为其定义的属性，通过`$`访问，即`vue.$property`。  
生命周期钩子函数 <-----> 回调  
`v-`指令的职责是，当表达式的值改变时，将产生连带影响，响应式地作用于DOM  

|v-指令|动作|
|-----|----|
|`v-bind`|属性绑定|
|`v-model`|双向绑定|
[vue的生命周期图](https://cn.vuejs.org/v2/guide/instance.html)  
[vue的生命周期回调函数说明](https://cn.vuejs.org/v2/api/#created)
![生命周期图](./vue-lifecycle.png)

- 计算属性  
<pre><code>
    new Vue({ 
        el:"#...",
        data:{
        message:"..."
        },
        computed:{
            reverseMessage:function(){
                return this.message.split('').reverse().join('')
            }
        }
    })
</code></pre>
计算属性 vs 方法  
计算属性是 **基于它们的依赖进行缓存的**。只有在相关依赖发生改变时它们才会重新求值。这就意味着只要message没有  
发生改变，多次访问reverseMessage计算属性会立即返回之前的计算结果。

- 侦听器
- 绑定class列表与内联样式  
内联样式：单独定义在元素内部的样式，类似于`<div style="...">`  
class列表：类似于`<div class="a b c ...">`

- 条件渲染 `v-if`
- 事件修饰符  
访问原生DOM事件 $event

### 2019/4/9
添加数据列: 
``` sql
alter table `student` add column `weight` int(11) unsigned default null comment '权重，用于拖拽排序';
```
复制数据列的值到另一列
``` sql
update `student` set weight = id;
```
