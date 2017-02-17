## 关节(joint)
###  关于
关节的作用是把物体约束到世界，或约束到其它物体上。在游戏中的典型例子是木偶，跷跷板和滑轮。关节可以用许多种不同的方法结合起来，创造出有趣的运动。

有些关节提供了限制(limit)，以便你控制运动范围。有些关节还提供了马达(motor)，它可以以指定的速度驱动关节，直到你指定了更大的力或扭矩。

关节马达有许多不同的用途。你可以使用关节来控制位置，只要提供一个与目标之距离成正比例的关节速度即可。你还可以模拟关节摩擦：将关节速度置零，并且提供一个小的、但有效的最大力或扭矩； 那么马达就会努力保持关节不动，直到负载变得过大。

### 关节定义
各种关节类型都派生自b2JointDef。所有关节都连接两个不同的物体，可能其中一个是静态物体。如果你想浪费内存的话，那就创建一个连接两个静态物体的关节。

你可以为任何一种关节指定用户数据。你还可以提供一个标记，用于预防相连的物体发生碰撞。实际上，这是默认行为，你可以设置collideConnected布尔值来允许相连的物体碰撞。

很多关节定义需要你提供一些几何数据。一个关节常常需要一个锚点(anchor point)来定义，这是固 定于相接物体中的点。在Box2D中这些点需要在局部坐标系中指定，这样，即便当前物体的变化违反了关节约束，关节还是可以被指定——在游戏存取进度时这经常会发生。另外，有些关节定义需要默认的物体之间的相对角度。这样才能通过关节限制或固定的相对角来正确地约束旋转。

初始化几何数据可能有些乏味。所以很多关节提供了初始化函数，消除了大部分工作。然而，这些初始化函数通常只应用于原型，在产品代码中应该直接地定义几何数据。这能使关节行为更加稳固。

其余的关节定义数据依赖于关节的类型。

### 总览
4. 关节(joint)  
  [4.1 距离关节(distance-joint)](https://github.com/godbasin/box2djs-tutorial/tree/master/4-joint/4-1-distance-joint.md)  
  [4.2 旋转关节(revolute-joint)](https://github.com/godbasin/box2djs-tutorial/tree/master/4-joint/4-2-revolute-joint.md)  
  [4.3 移动关节(prismatic-joint)](https://github.com/godbasin/box2djs-tutorial/tree/master/4-joint/4-3-prismatic-joint.md)  
  [4.4 滑轮关节(pulley-joint)](https://github.com/godbasin/box2djs-tutorial/tree/master/4-joint/4-4-pulley-joint.md)  
  [4.5 齿轮关节(gear-joint)](https://github.com/godbasin/box2djs-tutorial/tree/master/4-joint/4-5-gear-joint.md)  

