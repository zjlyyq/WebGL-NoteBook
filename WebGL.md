<div style="width:100%;text-align:center;">
  <h2 style="padding:12px 24px;">
    WebGL入门指南笔记
  </h2>
  <p style="text-align:right;padding:0 24px;">
    张嘉璐
  </p>
  <img style="" src="https://img3.doubanio.com/view/subject/l/public/s26723153.jpg">
</div>



### WebGL简介

#### WebGL —— 一个技术定义

#### 3D图形学入门

##### 3D坐标系

z轴的方向是从屏幕里面指向外面。

<div style="width:100%;text-align:center;">
  <img style="" src="https://upload.wikimedia.org/wikipedia/commons/2/2c/3D_coordinate_system.svg">
  <p>
    图1-2 一个三维坐标系统
  </p>
</div>

##### 网格、多边形和顶点

最常用的一种绘制3D图形的方法就是使用网格(Mesh)。网格中各个顶点的x，y，z的坐标分量只是用于定义形状，其他特性由其余属性来定义。

<div style="width:100%;text-align:center;">
  <img style="" src="https://upload.wikimedia.org/wikipedia/commons/8/88/Blender3D_UVTexTut1.png">
  <p>
    图1-3 一个3D网格
  </p>
</div>

##### 材质、纹理和光源

网格表面由一个或多个位图来决定，这就是我们常说的纹理映射（texture map），或者简称为纹理。

在大多数图像系统中，网格表面的纹理被统称为材质。

材质依赖于一个或多个光源来呈现出外观效果。

例如，图1-3的头部模型拥有一层紫色的材质，光源则是从左侧照入（右侧脸颊有阴影）。

##### 变换与矩阵

3D网格的形状取决于顶点位置，如果每次一定模型都必须重新设置所有顶点的位置，将会是可怕和沉闷的。

大部分3D系统支持变换（transform），这是一种不需要遍历每个顶点就可以移动模型的操作，不需要明确地逐个改变顶点的位置。

变换通常是由矩阵来操作的。

##### 相机、透视、视口和投影



<div style="width:100%;text-align:center;">
  <img style="" src="http://www.ituring.com.cn/figures/2016/HTML5WebGL/05.d01z.007.png">
  <p>
    图1-4 相机、透视、视口和投影
  </p>
</div>

相机是一个强大的工具，它决定了观察者和3D场景之间的关系，造成一种仿真感。

##### 着色器

着色器通常用C语言编写，包含了将模型投射到屏幕上的算法，编译并和运行在GPU上。着色器给与了图形开发者完全的能力去操控每一个顶点和像素。

#### WebGL原生API

##### WebGL应用结构剖析

本质上来讲，WebGL只是一个绘制库，使用WebGL可以绘制出惊人的图形，并可以在大部分机器上充分利用强大的GPU硬件能力。

也可以把WebGL理解为另一种画布，类似于HTML5浏览器的2DCanvas。实际上，WebGL也正是使用了HTML5里的`<canvas>`元素来在浏览器页面中显示3D图形。

使用WebGL将图形渲染到页面中，至少需要执行如下步骤。

1. 创建一个画布元素。
2. 获取画布的上下文。
3. 初始化视口
4. 创建一个或多个包含渲染数据的数组（通常是顶点数组）。
5. 创建一个或多个矩阵，将顶点数据变换到屏幕空间去。
6. 创建一个或多个着色器来实现绘制算法。
7. 使用参数初始化着色器。
8. 绘制。

##### 画布元素与绘制上下文

所有的WebGL渲染都发生在一个上下文（context）中，这是一个JavaScript对象，可以提供完整的WebGL API。

<div>
  <p style="font-weight:blod">
    示例1-1 从Canvas中获取WebGL上下文
  </p>
</div>

```javascript
function initWebGL(canvas) {
    var gl;
    try 
    {
      gl = canvas.getContext("experimental-webgl");
    } 
    catch (e)
    {
      var msg = "Error creating WebGL Context!: " + e.toString();
      alert(msg);
      throw Error(msg);
    }
    return gl;        
  }
```

> trycatch不可缺少，不是所有设备都支持WebGL。

##### 视口

<div>
  <p style="font-weight:blod">
    示例1-2 设置WebGL视口
  </p>
</div>

```javascript
function initViewport(gl, canvas)
{
	gl.viewport(0, 0, canvas.width, canvas.height);
}
```

##### Buffer、ArrayBuffer和类型化数组

WebGL的绘制由图元（primitive）组成的，图元的种类包括三角形（三角形数组）、三角形带、点和线。图元使用的数据数组被称为Buffer，它定义了顶点的位置。



示例1-3展示了如何创建一个大小1x1的正方形的顶点数组。返回的JavaScript对象存储了顶点数组信息、数组中每个顶点信息、数组中每个顶点所占的尺寸（在这个示例中，包含三个浮点数来存储x，y，z的值）、需要绘制的顶点的数量、以及用于绘制的正方形的图元类型。

<div>
  <p style="font-weight:blod">
    示例1-3 创建顶点数组
  </p>
</div>

```javascript
// Create the vertex data for a square to be drawn
function createSquare(gl) {
  var vertexBuffer;
  vertexBuffer = gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER, vertexBuffer);
  var verts = [
    .5,  .5,  0.5,
    -.5,  .5,  0.0,
    .5, -.5,  0.5,
    -.5, -.5,  0.0
  ];
  gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(verts), gl.STATIC_DRAW);
  var square = {buffer:vertexBuffer, vertSize:3, nVerts:4,primtype:gl.TRIANGLE_STRIP};
  return square;
}
```

> 这里使用了Float32Array的数据类型。这是一种专门为了WebGL专门引入浏览器的新数据类型。Float32Array是ArrayBuffer的一种，也被称为类型化数组（type array），用于在JavaScript中存储二进制数据。在JavaScript中访问类型化数组可以使用传统数组相同的语法，但是速度更快并且占用更小的内存。在对性能要求很高的场合下，使用它来处理二进制数据是非常理想的选择。类型化数组还可以用于很多其他的场合，但是它最开始是因为WebGL才被引入的。

##### 矩阵

模型视图矩阵：综合了模型变换和相机之间的关系。

投影矩阵：这个矩阵被用在着色器中将3D左边转化为绘制的视口的2D坐标。

<div>
  <p style="font-weight:blod">
    示例1-4 设置模型矩阵和投影矩阵
  </p>
</div>

```javascript
function initMatrices()
{
  // The transform matrix for the square - translate back in Z for the camera
  modelViewMatrix = new Float32Array(
  [1, 0, 0, 0,
  0, 1, 0, 0, 
  0, 0, 1, 0, 
  0, 0, -3.333, 1]);

  // The projection matrix (for a 45 degree field of view)
  projectionMatrix = new Float32Array(
  [2.41421, 0, 0, 0,
  0, 2.41421, 0, 0,
  0, 0, -1.002002, -1, 
  0, 0, -0.2002002, 0]);
}
```

##### 着色器

顶点着色器：负责将物体的坐标转换到2D空间。

片元着色器：负责生成最后的颜色输出到每个转换后的顶点像素，而颜色输入可以是纯色、纹理、光源或者材质。

<div>
  <p style="font-weight:blod">
    示例1-5 顶点着色器和片元着色器
  </p>
</div>

```javascript
var vertexShaderSource =
		
		"    attribute vec3 vertexPos;\n" +
		"    uniform mat4 modelViewMatrix;\n" +
		"    uniform mat4 projectionMatrix;\n" +
		"    void main(void) {\n" +
		"		// Return the transformed and projected vertex value\n" +
		"        gl_Position = projectionMatrix * modelViewMatrix * \n" +
		"            vec4(vertexPos, 1.0);\n" +
		"    }\n";

	var fragmentShaderSource = 
		"    void main(void) {\n" +
		"    // Return the pixel color: always output white\n" +
    "    gl_FragColor = vec4(1.0, 1.0, 1.0, 1.0);\n" +
    "}\n";
```

##### 绘制图元

```javascript
function draw(gl, obj) {

  // clear the background (with black)
  gl.clearColor(0.0, 0.0, 0.0, 1.0);
  gl.clear(gl.COLOR_BUFFER_BIT);

  // set the vertex buffer to be drawn
  gl.bindBuffer(gl.ARRAY_BUFFER, obj.buffer);

  // set the shader to use
  gl.useProgram(shaderProgram);

 			// connect up the shader parameters: vertex position and projection/model matrices
  gl.vertexAttribPointer(shaderVertexPositionAttribute, obj.vertSize, gl.FLOAT, false, 0, 0);
  gl.uniformMatrix4fv(shaderProjectionMatrixUniform, false, projectionMatrix);
  gl.uniformMatrix4fv(shaderModelViewMatrixUniform, false, modelViewMatrix);

  // draw the object
  gl.drawArrays(obj.primtype, 0, obj.nVerts);
}
```

### 你的第一个WebGL程序

#### Three.js——一个JavaScript3D引擎

功能概述：

1. 掩盖了3D渲染的细节
2. 面向对象的
3. 功能非常丰富
4. 速度非常快
5. 支持交互
6. 包含数学库
7. 内置文件格式支持
8. 拓展性很强
9. 同时支持HTML5 2D canvas

#### 建立Three.js运行环境

