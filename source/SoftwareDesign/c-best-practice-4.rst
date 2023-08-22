【C语言最佳实践】状态机
===============================
| 作者：wallace-lai
| 时间：2022-08-04

一、状态机要解决的问题
-------------------------------

状态机所要解决的问题：

（1）计算机最擅长处理重复性的工作

（2）现实世界中大量事物的工作流程取决于其他事物的状态

（3）状态机是针对多种状态之间的迁移而建立的一个简单数学模型

（4）状态机可用来指导硬件模块或者软件模块的设计

状态机的概念：

（1）一个硬件或者一个软件模块

（2）我们给状态机一个初始状态

（3）然后给状态机一个事件

（4）状态机依据事件和当前状态完成一个动作或者转换到另一个状态

（5）终止执行或者等待下一个事件 如此，状态机就成了一个可重复工作的机器

二、状态机相关理论
-------------------------------

术语：

- 状态（state）：可分为当前状态（现态）和迁移后的新状态（次态）

- 事件（Event）：又称为条件

- 动作（Action） 

- 迁移、转换或者变换

分类：

- Moore状态机：次态和事件无关（极其简单的状态机）

- Mealy状态机：次态和现态及事件有关

三、正确理解状态机
-------------------------------

所有含有条件分支代码的程序或者函数都可以看成一个状态机？

- 简单的事件驱动代码，并不构成状态机 

- 状态机里必须存在状态迁移

四、状态机的软件实现
-------------------------------

**确定性状态机**

（1）有限的状态

（2）有限的输入事件

（3）有确定的状态迁移规则

**自定义状态机**

（1）自定义状态

（2）有限的输入事件

（3）某种确定的状态迁移规则

**确定性状态机实现步骤**

（1）使用状态迁移图/表清理迁移关系

（2）状态数量较少时，一个包含上下文信息的函数搞定，内部使用条件分支代码

（3）状态数量较多时，为每个状态定义回调函数，在回调函数中实现对这个状态的处理

**状态机举例：判断立即数**

（1）前缀\ ``0x``\ 表示十六进制、前缀\ ``0``\ 表示八进制，其他表示十进制

（2）状态：为止、零前缀、八进制、十六进制、十进制、错误

.. code:: c

   enum {
       SSM_UNKNOWN = 0,
       SSM_PREFIX_ZERO,
       SSM_RESULT_FIRST,

       SSM_OCT = SSM_RESULT_FIRST,
       SSM_HEX,
       SSM_DEC,
       SSM_ERR,
   };

   struct _scale_state_machine {
       int state;
   };

   static int init_scale_state_machine(struct _scale_state_machine *sm)
   {
       sm->state = SSM_UNKNOWN;
   }

   static int scale_sm_check_scale(struct _scale_state_machine *sm, char ch)
   {
       switch (sm->state) {
           case SSM_UNKNOWN:
               if (ch == '0')
                   sm->state = SSM_PREFIX_ZERO;
               else if (ch >= '1' && ch <= '9') {
                   sm->state = SSM_DEC;
               } else {
                   sm->state = SSM_ERR;
               }
               break;
           case SSM_PREFIX_ZERO:
               if (ch == 'x' || ch == 'X') {
                   sm->state = SSM_HEX;
               } else if (ch >= '0' || ch <= '7') {
                   sm->state = SSM_OCT;
               } else {
                   sm->state = SSM_ERR;
               }
               break;
       }

       return sm->state;
   }

   int check_scale(const char *literal)
   {
       struct _scale_state_machine sm;

       init_scale_state_machine(&sm);

       while (*literal) {
           int scale = scale_sm_check_scale(&sm, *literal);
           if (scale >= SSM_RESULT_FIRST)
               return scale;
           
           literal++;
       }

       return SSM_ERR;
   }

**复杂案例：Lexbor实现HTML规范解析器**

略

**自定义状态机**

（1）事件是确定的、有限的

（2）状态是一种抽象对象而不是一个简单的枚举变量；状态被组织为链表或者树形数据结构

（3）每个状态可根据输入事件构造一个抽象的迁移对象

（4）迁移对象实现动作及状态迁移；通常，状态迁移发生在相邻的状态节点之间

（5）状态机构造状态数据结构，并根据状态返回的迁移函数运行，直到停止

**应用场景**

（1）轨迹生成器

- 状态是代表不同轨迹的时间曲线方程（如线性、贝塞尔曲线等） 

- 事件是定时器

（2）动画控制器 

- 状态是不同的动画效果 

- 事件是定时器或者用户输入

总结
-------------------------------

如果学过形式语言与自动机课程的话，上面的内容应该没什么难度
