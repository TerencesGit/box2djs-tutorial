## 鼠标操作刚体
### 获取所在位置刚体
首先，我们需要通过一个位置坐标(x, y)来获取该坐标下的刚体，若无则返回null。

前面我们也提到过，我们可以通过画一个较小的矩形，然后判断是否有相交来获取刚体：

``` javascript
function GetBodyAtPosition(x, y) {
    // 首先创建一个近似于点的小区域
    var mousePVec = new b2Vec2(x, y);
    // 利用b2Vec2定义一个矢量，用来保存鼠标点击的点
    var aabb = new b2AABB();
    // 利用b2AABB创建一个环境
    aabb.minVertex.Set(mousePVec.x - 0.001, mousePVec.y - 0.001);
    aabb.maxVertex.Set(mousePVec.x + 0.001, mousePVec.y + 0.001);
    // 设置aabb的左上角及右下角坐标，这里是以鼠标点击位置为中心创建了一个长、宽均为0.002的矩形区域

    // 然后查询与指定区域有重叠的刚体
    var k_maxCount = 10;     // 设定所要查找形状的数量，注意合理设置其大小，过大会影响运行速度
    var shapes = new Array();  // 保存查找到的与已知边界盒相交的形状
    var count = world.Query(aabb, shapes, k_maxCount);    // 在世界中查找与边界盒相交的maxCount个形状，并返回边界盒区域内实际包含的形状的个数

    var findBody = null;  // 首先设定没有找到物体
    for (var i = 0; i < count; ++i) {
        if (shapes[i].GetBody().IsStatic() == false)
        // 条件假定查找到的形状不是静态刚体，比如墙
        {
            var tShape = shapes[i];    // 将查找到的形状赋给tShape变量
            var inside = tShape.GetBody();   // 将tShape对应的刚体赋给inside
            if (inside)        // 如果inside这个刚体存在
            {
                // 那么返回这个刚体，并跳出遍历
                findBody = tShape.GetBody();
                break;
            }
        }
    }
    return findBody;
}
```

### 添加鼠标点击事件
这里我们使用鼠标点击(mousedown)事件，去触发刚体的状态改变。

- 添加事件监听

``` javascript
document.addEventListener('mousedown', handleMousedown, false);
```

- 获取鼠标位置

``` javascript
// 获取鼠标坐标
function getMousePos(event) {
    var e = event || window.event;
    var scrollX = document.documentElement.scrollLeft || document.body.scrollLeft;
    var scrollY = document.documentElement.scrollTop || document.body.scrollTop;
    var x = e.pageX || e.clientX + scrollX;
    var y = e.pageY || e.clientY + scrollY;
    return { 'x': x, 'y': y };
}
```

- 处理鼠标移动

``` javascript
// 处理鼠标移动
function handleMousedown(e) {
    var e = e || window.event;
    // 获取鼠标x坐标
    var newMouse = getMousePos(e);
    var selectBody = GetBodyAtPosition(newMouse.x - canvas.offsetLeft, newMouse.y - canvas.offsetTop); // 选择刚体
    if (selectBody) {
        // 处理选中的刚体
    } else {
        // 未选中时处理
    }
}
```

### 操作刚体
- 给选中刚体添加速度向量

``` javascript
// 若有选中刚体，则处理
var LinearVelocity = new b2Vec2(500, -200); // 定义一个向量
selectBody.WakeUp(); // 激活休眠状态
selectBody.SetLinearVelocity(LinearVelocity); //给定一个速度向量
```

- 未选中刚体，则新建添加一个矩形刚体

``` javascript
 // 若无，则随机生成一个矩形添加
var width = parseInt(Math.random() * 50);
var height = parseInt(Math.random() * 50);
createBox(world, newMouse.x, newMouse.y, width, height)
```

### 完整代码
``` javascript
var canvas;
var ctx;
var canvasWidth;
var canvasHeight;

var world;

// 我们将创建世界封装至createWorld函数内
function createWorld() {
    // 世界的大小
    var worldAABB = new b2AABB();
    worldAABB.minVertex.Set(-4000, -4000);
    worldAABB.maxVertex.Set(4000, 4000);

    //定义重力
    var gravity = new b2Vec2(0, 300);

    // 是否休眠
    var doSleep = false;

    // 最终创建世界
    var world = new b2World(worldAABB, gravity, doSleep);

    return world;
}

//绘画功能
function drawWorld(world, context) {
    for (var j = world.m_jointList; j; j = j.m_next) {
        // 绘制关节
        // drawJoint(j, context);
    }
    for (var b = world.m_bodyList; b != null; b = b.m_next) {
        for (var s = b.GetShapeList(); s != null; s = s.GetNext()) {
            if (s.GetUserData() != undefined) {
                // 使用数据包括图片
                var img = s.GetUserData();

                // 图片的长和宽
                var x = s.GetPosition().x;
                var y = s.GetPosition().y;
                var topleftX = -img.clientWidth / 2;
                var topleftY = -img.clientHeight / 2;

                context.save();
                context.translate(x, y);
                context.rotate(s.GetBody().GetRotation());
                context.drawImage(img, topleftX, topleftY);
                context.restore();
            }
            drawShape(s, context);
        }
    }
}

// 创建圆形刚体
function createBall(world, x, y, r, custom) {
    // 创建圆形定义
    var ballSd = new b2CircleDef();
    ballSd.density = 1.0; // 设置密度
    if (custom === 'fixed') ballSd.density = 0.0; // 若传入'fixed'，则需固定，此时设置密度为0
    else ballSd.userData = custom; // 若传入其他，则视为图片数据
    ballSd.radius = 20; // 设置半径
    ballSd.restitution = 1.0; // 设置弹性
    ballSd.friction = 0; // 设置摩擦因子
    var ballBd = new b2BodyDef(); // 创建刚体定义
    ballBd.AddShape(ballSd); // 添加形状
    ballBd.position.Set(x || 0, y || 0); // 设置位置
    return world.CreateBody(ballBd); // 创建并返回刚体
}

// 创建矩形刚体
function createBox(world, x, y, width, height, custom) {
    var boxSd = new b2BoxDef(); // 创建一个形状Shape，然后设置有关Shape的属性
    boxSd.extents.Set(width || 1200, height || 5); // 设置矩形高、宽
    boxSd.density = 1.0; // 设置矩形的密度 
    if (custom === 'fixed') boxSd.density = 0.0; // 若传入'fixed'，则需固定，此时设置密度为0
    else boxSd.userData = custom; // 若传入其他，则视为图片数据
    boxSd.restitution = .3; // 设置矩形的弹性
    boxSd.friction = 1; // 设置矩形的摩擦因子，可以设置为0-1之间任意一个数，0表示光滑，1表示强摩擦

    var boxBd = new b2BodyDef(); // 创建刚体定义
    boxBd.AddShape(boxSd); // 添加形状
    boxBd.position.Set(x || 10, y || 10); // 设置位置
    return world.CreateBody(boxBd) // 创建并返回刚体
}

// 定义step函数，用于游戏的迭代运行
function step() {
    // 模拟世界
    world.Step(1.0 / 60, 1);
    // 清除画布
    ctx.clearRect(0, 0, canvasWidth, canvasHeight);
    // 重新绘制
    drawWorld(world, ctx);

    // 再次刷新
    setTimeout(step, 10);
}

// 从draw_world.js里面引用的绘画功能
function drawShape(shape, context) {
    context.strokeStyle = '#003300';
    context.beginPath();
    switch (shape.m_type) {
        // 绘制圆
        case b2Shape.e_circleShape:
            var circle = shape;
            var pos = circle.m_position;
            var r = circle.m_radius;
            var segments = 16.0;
            var theta = 0.0;
            var dtheta = 2.0 * Math.PI / segments;
            // 画圆圈
            context.moveTo(pos.x + r, pos.y);
            for (var i = 0; i < segments; i++) {
                var d = new b2Vec2(r * Math.cos(theta), r * Math.sin(theta));
                var v = b2Math.AddVV(pos, d);
                context.lineTo(v.x, v.y);
                theta += dtheta;
            }
            context.lineTo(pos.x + r, pos.y);

            // 画半径
            context.moveTo(pos.x, pos.y);
            var ax = circle.m_R.col1;
            var pos2 = new b2Vec2(pos.x + r * ax.x, pos.y + r * ax.y);
            context.lineTo(pos2.x, pos2.y);
            break;
        // 绘制多边形
        case b2Shape.e_polyShape:
            var poly = shape;
            var tV = b2Math.AddVV(poly.m_position, b2Math.b2MulMV(poly.m_R, poly.m_vertices[0]));
            context.moveTo(tV.x, tV.y);
            for (var i = 0; i < poly.m_vertexCount; i++) {
                var v = b2Math.AddVV(poly.m_position, b2Math.b2MulMV(poly.m_R, poly.m_vertices[i]));
                context.lineTo(v.x, v.y);
            }
            context.lineTo(tV.x, tV.y);
            break;
    }
    context.stroke();
}

function GetBodyAtPosition(x, y) {
    // 首先创建一个近似于点的小区域
    var mousePVec = new b2Vec2(x, y);
    // 利用b2Vec2定义一个矢量，用来保存鼠标点击的点
    var aabb = new b2AABB();
    // 利用b2AABB创建一个环境
    aabb.minVertex.Set(mousePVec.x - 0.001, mousePVec.y - 0.001);
    aabb.maxVertex.Set(mousePVec.x + 0.001, mousePVec.y + 0.001);
    // 设置aabb的左上角及右下角坐标，这里是以鼠标点击位置为中心创建了一个长、宽均为0.002的矩形区域

    // 然后查询与指定区域有重叠的刚体
    var k_maxCount = 10;     // 设定所要查找形状的数量，注意合理设置其大小，过大会影响运行速度
    var shapes = new Array();  // 保存查找到的与已知边界盒相交的形状
    var count = world.Query(aabb, shapes, k_maxCount);    // 在世界中查找与边界盒相交的maxCount个形状，并返回边界盒区域内实际包含的形状的个数

    var findBody = null;  // 首先设定没有找到物体
    for (var i = 0; i < count; ++i) {
        if (shapes[i].GetBody().IsStatic() == false)
        // 条件假定查找到的形状不是静态刚体，比如墙
        {
            var tShape = shapes[i];    // 将查找到的形状赋给tShape变量
            var inside = tShape.GetBody();   // 将tShape对应的刚体赋给inside
            if (inside)        // 如果inside这个刚体存在
            {
                // 那么返回这个刚体，并跳出遍历
                findBody = tShape.GetBody();
                break;
            }
        }
    }
    return findBody;
}

// 获取鼠标坐标
function getMousePos(event) {
    var e = event || window.event;
    var scrollX = document.documentElement.scrollLeft || document.body.scrollLeft;
    var scrollY = document.documentElement.scrollTop || document.body.scrollTop;
    var x = e.pageX || e.clientX + scrollX;
    var y = e.pageY || e.clientY + scrollY;
    return { 'x': x, 'y': y };
}

// 处理鼠标移动
function handleMousedown(e) {
    var e = e || window.event;
    // 获取鼠标x坐标
    var newMouse = getMousePos(e);
    var selectBody = GetBodyAtPosition(newMouse.x - canvas.offsetLeft, newMouse.y - canvas.offsetTop); // 选择刚体
    if (selectBody) {
        // 若有选中刚体，则处理
        var LinearVelocity = new b2Vec2(500, -200); // 定义一个向量
        selectBody.WakeUp(); // 激活休眠状态
        selectBody.SetLinearVelocity(LinearVelocity);      //给定一个速度向量
    } else {
        // 若无，则随机生成一个矩形添加
        var width = parseInt(Math.random() * 50);
        var height = parseInt(Math.random() * 50);
        createBox(world, newMouse.x, newMouse.y, width, height);
    }
}

document.addEventListener('mousedown', handleMousedown, false);

window.onload = function () {
    canvas = document.getElementById('canvas');
    ctx = canvas.getContext('2d');
    canvasWidth = parseInt(canvas.width);
    canvasHeight = parseInt(canvas.height);
    // 启动游戏
    world = createWorld();
    var ball1 = createBall(world, 100, 20, 20);
    var ball2 = createBall(world, 300, 60, 10);
    var box1 = createBox(world, 100, 200, 25, 30, 'fixed');
    var box2 = createBox(world, 200, 50, 20, 20);
    var box3 = createBox(world, 400, 80, 20, 20, document.getElementById('box'));
    var wallLeft = createBox(world, 0, 0, 10, 600, 'fixed');
    var wallRight = createBox(world, 1290, 0, 10, 400, 'fixed');
    var ground = createBox(world, 30, 595, 1200, 5, 'fixed');
    step();
};
```

[此处查看项目代码（仅包含src部分）](https://github.com/godbasin/box2djs-tutorial/tree/master/6-practice/6-4-mouse-operate-body/6-4-mouse-operate-body)  
[此处查看页面效果](http://old7pzwup.bkt.clouddn.com/6-4-mouse-operate-body/index.html)  


### [返回总目录](https://github.com/godbasin/box2djs-tutorial)  