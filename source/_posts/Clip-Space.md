---
title: Clip Space
date: 2022-01-17 10:27:10
tags: Math
categories: Unity Shader
math: true
---

## 裁剪空间 ##
顶点从观察空间转换到**裁剪空间(clip space，也被称为齐次裁剪空间)**所用的矩阵叫做**裁剪矩阵(clip matrix)**，也被称为投影矩阵(projection matrix)。

裁剪空间的目标是能够方便地对渲染图元进行裁剪：完全位于这块空间内部的图元将会被保留，完全位于这块空间外部的图元将会被剔除，而与这块空间边界相交的图元就会被裁剪。那么，这块空间是如何决定的呢？答案是由**视锥体(view frustum)**来决定。

视锥体指的是空间中的一块区域，这块区域决定了摄像机可以看到的空间。视锥体由六个平面包围而成，这些平面也被称为**裁剪平面(clip planes)**。视锥体有两种类型，这涉及两种投影类型：一种是**正交投影(orthographic projection)**，一种是**透视投影(perspective projection)**。下图显示了从同一位置、同一角度渲染同一个场景的两种摄像机的渲染结果。

![透视投影和正交投影](/posts_image/Clip_Space/Clip_Space_1.png "透视投影和正交投影")

透视投影模拟了人眼看世界的方式，而正交投影则完全保留了物体的距离和角度。因此，在追求真实感的3D游戏中我们往往会使用透视投影，而在一些2D游戏或渲染小地图等其他HUD元素时，我们会使用正交投影。在透视空间里使用的并不是笛卡尔坐标系，为了描述透视空间，科学家提出了“齐次坐标”的概念：

即，用 N+1 个数来表示 N 维空间中的点或向量，对于三维空间中的点，通常是使用 (x, y, z, w) 来表示三维空间中的点在齐次坐标空间中的位置。三维空间（笛卡尔坐标系）和齐次坐标系之间可以通过齐次除法进行相互转换，科学家定义的规则是：

![齐次除法规则](/posts_image/Clip_Space/Clip_Space_2.jpeg "齐次除法规则")

比如平行线在透视空间中相交于一个点，这个点的齐次坐标可以表示为 (x, y, z, 0)，这里 xyz 为多少并不重要，重要的是 w = 0，因为这使得这个齐次坐标点转换为三维坐标是（x/0, y/0, z/0）= (无穷， 无穷， 无穷) ，因此我们成功描述了这个点。

在视锥体的6块裁剪平面中，有两块裁剪平面比较特殊，它们分别被称为**近裁剪平面(near clip plane)**和**远裁剪平面(far clip plane)**。它们决定了摄像机可以看到的深度范围。正交投影和透视投影的视锥体如下图所示。

![透视投影和正交投影的视锥体](/posts_image/Clip_Space/Clip_Space_3.png "透视投影和正交投影的视锥体")

前面讲到，我们希望根据视锥体围成的区域对图元进行裁剪，但是，如果直接使用视锥体定义的空间来进行裁剪，那么不同的视锥体就需要不同的处理过程，而且对于透视投影的视锥体来说，想要判断一个顶点是否处于一个金字塔内部是比较麻烦的。因此，我们想用一种更加通用、方便和整洁的方式来进行裁剪的工作，这种方式就是通过一个投影矩阵把顶点转换到一个裁剪空间中。在裁剪空间之前，虽然我们使用了齐次坐标来表示点和矢量，但它们的第四个分量都是固定的：点的w分量是1，方向矢量的w分量是0。经过投影矩阵的变换后，我们就会赋予齐次坐标的第四个坐标更加丰富的含义。

投影矩阵有两个目的。

* 首先是为投影做准备。虽然投影矩阵的名称包含了投影二字，但是它并没有进行真正的投影工作，而是在为投影做准备。真正的投影发生在后面的**齐次除法(homogeneous division)**过程中。而经过投影矩阵的变换后，顶点的w分量将会具有特殊的意义。

投影可以理解为一个空间的降维，例如从四维空间投影到三维空间中。而投影矩阵实际上并不会真的进行这个步骤，它会为真正的投影做准备工作。真正的投影会在屏幕映射时发生，通过齐次除法来得到二维坐标。

* 其次是对x、y、z分量进行缩放。我们上面讲过直接使用视锥体的6个裁剪平面来进行裁剪会比较麻烦。而经过投影矩阵的缩放后，我们可以直接使用w分量作为一个范围值，如果x、y、z分量都位于这个范围内，就说明该顶点位于裁剪空间内。

裁剪空间中的顶点是用齐次坐标表示的，在屏幕映射阶段里要对裁剪空间的顶点进行一个统一的齐次除法操作，来把顶点从齐次坐标系转换到笛卡尔坐标系的归一化设备坐标（Normalized Device Coordinates, NDC）空间里，经过这一步之后，裁剪空间将会变换到一个立方体内。

OpenGL 和 DirectX 对 NDC 空间的定义有所不同，前者定义 NDC 空间的 xyz 取值范围是 [-1, 1] ，而后者定义 NDC 空间的 xy 取值范围为 [-1, 1]，z 的取值范围为 [0, 1]。而 Unity 选择了 OpenGL 的规范：

![齐次除法](/posts_image/Clip_Space/Clip_Space_4.jpeg "齐次除法")

转换之后如果 x,y,z 满足条件：

![裁剪条件](/posts_image/Clip_Space/Clip_Space_5.jpeg "裁剪条件")

符合此条件的顶点参与渲染，不符合的则裁剪掉不参与渲染。对于正交摄像机的话，他的裁剪空间是一个长方体，拍摄出来的画面是可以用三维坐标来描述的，不需要齐次坐标来描述，所以正交摄像机的投影矩阵对 w 分量没有进行操作（或者理解为正交摄像机的裁剪空间是 w 恒等于 1 的齐次坐标空间）。
  
下面，我们来看一下两种投影类型使用的投影矩阵具体是什么。

#### 1. 正交投影 ####

视锥体的意义在于定义了场景中的一块三维空间。所有位于这块空间内的物体都将会被渲染，否则就会被剔除或裁剪。我们已经知道视锥体由6个裁剪平面定义，那么这6个裁剪平面又是怎么决定的呢？在Unity中，它们由Camera组件中的参数和Game视图的纵横比共同决定。如下图所示。

![正交摄像机的参数对正交投影视锥体的影响](/posts_image/Clip_Space/Clip_Space_1_1.png "正交摄像机的参数对正交投影视锥体的影响")

由图我们可以看出，我们可以通过Camera组件的Size属性来改变视锥体竖直方向上高度的一半，而Clipping Planes中的Near和Far参数可以控制视锥体的近裁剪平面和远裁剪平面距离摄像机的远近。这样我们就可以求出视锥体近裁剪平面和远裁剪平面的高度，也就是：
$$
nearClipPlaneHeight = 2 \cdot Size
$$
$$
farClipPlaneHeight = NearClipPlaneHeight
$$
我们可以通过摄像机的纵横比得倒横向的信息。在Unity中，一个摄像机的纵横比由Game视图的纵横比和Viewport Rect中的W和H属性共同决定(实际上。Unity允许我们在脚本里通过Camera.aspect进行修改，但这里不做讨论)。假设，当前摄像机的纵横比为Aspect，我们定义：
$$
nearClipPlaneWidth = Aspect \cdot nearClipPlaneHeight
$$
$$
farClipPlaneWidth = nearClipPlaneWidth
$$

现在，我们可以根据已知的Near、Far、Size和Aspect的值来确定正交投影的裁剪矩阵。如下：

$$
M_{ortho} = 
\begin{bmatrix}
\cfrac{1}{Aspect \cdot Size}&0&0&0\\ 
0&\cfrac{1}{Size}&0&0\\ 
0&0&-\cfrac{2}{Far - Near}&-\cfrac{Far + Near}{Far - Near}\\
0&0&0&1\\
\end{bmatrix}
$$

需要注意的是,这里的投影矩阵是建立在Unity对坐标系的假定上面的，也就是说，我们针对的是观察空间为右手坐标系，使用列矩阵在矩阵右侧进行相乘，且变换后z分量范围将在 [-w, w]之间的情况。而在类似DirectX这样的图形接口中，它们希望变换后z分量范围将在 [0, w]之间，因此就需要对上面的透视矩阵进行一些更改。

**推导过程如下**

正交摄像机的视锥体空间定义为：L(left)、R(right)、T(top)、B(bottom)、N(near)、F(far)

转换到裁剪空间的顶点应该满足： $L \leq x \leq R$、$B \leq y \leq T$、$N \leq z \leq F$

先单独研究x分量，x分量最后是要映射到-1～1之间的，所以：

$L \leq x \leq R$

$0 \leq (x - L) \leq (R - L)$

$0 \leq \cfrac{x - L}{R - L} \leq 1$

$0 \leq 2 \times \cfrac{x - L}{R - L} \leq 2$

$-1 \leq \cfrac{2 \times (x - L)}{R - L} - 1\leq 1$

最后化简后得到，转换后的x‘分量与原本的x分量之间的表达式是

$x' = \cfrac{2 \times x - L - R}{R - L} = \cfrac{2}{R - L} \times x - \cfrac{R + L}{R - L}$

由于x、y、z都是转换到取值范围为 [-1, 1]的，所以我们可以同理得到其他两个变量：

$y' = \cfrac{2 \times y - B - T}{T - B} = \cfrac{2}{T - B} \times y - \cfrac{T + B}{T - B}$

$z' = \cfrac{2 \times z - N - F}{F - N} = \cfrac{2}{F - N} \times z - \cfrac{F + N}{F - N}$

结合正交相机提供的参数，我们可以得到：

L = -Width / 2、 R = Width / 2、 B = -Height / 2、 T = Height / 2

而且正交摄像机的参数提供的有： Aspect = Width / Height、 Height = 2 $\times$ Size

代入后可以得到：

x' = (2 / Width) $\times$ x = 1 / (Aspect $\times$ Size)

y' = (2 / Height) $\times$ y = 1 / Size

由x'、y‘、z’即可得到正交矩阵的P矩阵：

$$
\begin{bmatrix}
\cfrac{1}{Aspect \cdot Size}&0&0&0\\ 
0&\cfrac{1}{Size}&0&0\\ 
0&0&\cfrac{2}{Far - Near}&-\cfrac{Far + Near}{Far - Near}\\
0&0&0&1\\
\end{bmatrix}
$$

由于P矩阵是把顶点从观察空间(右手坐标系)转换到裁剪空间(左手坐标系)，因此要反转处理一下z轴：

$$
\begin{bmatrix}
\cfrac{1}{Aspect \cdot Size}&0&0&0\\ 
0&\cfrac{1}{Size}&0&0\\ 
0&0&-\cfrac{2}{Far - Near}&-\cfrac{Far + Near}{Far - Near}\\
0&0&0&1\\
\end{bmatrix}
$$

#### 2. 透视投影 ####

我们还是看一下透视投影中的6个裁剪平面是如何定义的。和正交投影类似，在Unity中，它们也是由Camera组件中的参数和Game视图的纵横比共同决定。如下图所示。

![透视摄像机的参数对透视投影视锥体的影响](/posts_image/Clip_Space/Clip_Space_2_1.png "透视摄像机的参数对透视投影视锥体的影响")

我们可以通过Camera组件的Field of View(简称FOV)属性来改变视锥体竖直方向的张开角度，而Clipping Planes中的Near和Far参数可以控制视锥体的近裁剪平面和远裁剪平面距离摄像机的远近。这样我们可以求出视锥体近裁剪平面和远裁剪平面的高度，也就是：
$$
nearClipPlaneHeight = 2 \cdot Near \cdot \tan\cfrac{FOV}{2}
$$
$$
farClipPlaneHeight = 2 \cdot Far \cdot \tan\cfrac{FOV}{2}
$$

现在我们还缺乏横向的信息。这可以通过摄像机的纵横比得到。假设，当前摄像机的纵横比为Aspect，那么：
$$
Aspect = \cfrac{nearClipPlaneWidth}{nearClipPlaneHeight}
$$
$$
Aspect = \cfrac{farClipPlaneWidth}{farClipPlaneHeight}
$$
现在我们可以根据已知的Near、Far、FOV和Aspect的值来确定透视投影的投影矩阵。
$$
M_{frustum} = 
\begin{bmatrix}
\cfrac{\cot\cfrac{FOV}{2}}{Aspect}&0&0&0\\ 
0&\cot\cfrac{FOV}{2}&0&0\\ 
0&0&-\cfrac{Far + Near}{Far - Near}&-\cfrac{2 \cdot Near \cdot Far}{Far - Near}\\
0&0&-1&0\\
\end{bmatrix}
$$
同样，这里的投影矩阵是建立在Unity对坐标系的假定上面的。

**推导过程如下**

正交矩阵的P矩阵已经推导出来，那么如果摄像机是透视模式的话，视锥体是一个金字塔形状的锥体而不是立方体，显而易见的问题是：随着z轴的变化，xy轴分量的取值范围会变大：

![随着z变大，xy取值范围变大](/posts_image/Clip_Space/Clip_Space_2_2.jpeg "随着z变大，xy取值范围变大")

因此我们首先要解决的问题是求出xy分量随着z分量的变化而变化的公式，即它们之间的关系：

![xy分量随着z分量的变化而变化](/posts_image/Clip_Space/Clip_Space_2_3.png "xy分量随着z分量的变化而变化")

如图，在视锥体中，存在一个顶点P（x, y, z），我们先把视锥体的原点O（0，0，0）向视锥体中心发出一道射线穿过近裁剪平面和后面的远裁剪平面（图中未画出），把点P向射线做一条辅助线 BP、连接 OP 相交近截面于点D，点 D 向射线做一条垂直线 AD 得到上图。可以看出图中存在两个相似三角形 OAD 和 OBP，则可以推论：

$BP = \sqrt{x^2 + y^2}$

$AD = \sqrt{dx^2 + dy^2}$

$OA = n$

$OB = z$

由于相似三角形的特性，存在：

$\cfrac{BP}{AD} = \cfrac{OB}{OA} = \cfrac{z}{n}$

即：$AD = \cfrac{n \cdot BP}{z} = \cfrac{n}{z} \cdot \sqrt{x^2 + y^2}$

简化之后可以得到：

$\sqrt{dx^2 + dy^2} = \sqrt{(\cfrac{nx}{z})^2 + (\cfrac{ny}{z})^2}$

则：

$dx = \cfrac{nx}{z}$

$dy = \cfrac{ny}{z}$

因此可以得出：对于锥体里的任何一个点 (x, y, z) ，其中的 x，y 分量都可以通过上述的公式映射到近裁剪平面上，而 z 轴分量则是原封不动地代表了顶点距离摄像机的位置，原本透视摄像机的视锥体是近裁剪平面小，远裁剪平面大的，现在通过上述公式就可以把所有顶点都映射到同一个近裁剪平面的取值范围了，即：left ≤ x ≤ right 、 bottom ≤ y ≤ top ，那么我们完全可以把之前正交摄像机得出的公式套用进去：

$dx = \cfrac{Near \cdot x}{z}$

$dy = \cfrac{Near \cdot y}{z}$

$x' = \cfrac{2}{Width} \cdot dx = \cfrac{2Near \cdot x}{Width \cdot z}$

$y' = \cfrac{2}{Height} \cdot dy = \cfrac{2Near \cdot y}{Height \cdot z}$

$z' = \cfrac{2z - Near - Far}{Far - Near} = \cfrac{2}{Far - Near} \cdot z - \cfrac{Far + Near}{Far - Near}$

此时我们发现，我们无法把上面的式子变成x' = ax + by + cz + d的形式，无法把公式写入矩阵中，我们先把等式两端乘上z分量，那么可以得到：

$x'z = \cfrac{2Near \cdot x}{Width}$

$y'z = \cfrac{2Near \cdot y}{Height}$

如果我们也能写出一个 z'z = ax + by + cz + d 的式子，那么我们就能把（x, y, z, w） 映射到 （x'z, y'z, z'z, w'z）点了，而且这个转换对后续的投影操作是没有任何影响的，最后的结果会是正确的（待会解释），我们先来求一下 z'z 的公式：

由于z分量的公式并不依赖于x、y分量，所以z‘z的公式应该是：z'z = qz + p，我们需要求出q和p，我们要把z分量从[Near, Far]映射到[-1, 1]：

当z = Near时，z' = -1，得到：$-Near = q \cdot Near + p$

当z = Far时，z’ = 1，得到：$Far = q \cdot Far + p$

结合上述两个公式可以求出：

$q = \cfrac{Far + Near}{Far - Near}$

$p = \cfrac{-2Near \cdot Far}{Far - Near}$

那么可以得到
$z'z = \cfrac{Far + Near}{Far - Near} \cdot z - \cfrac{2Near \cdot Far}{Far - Near}$

还有一个w分量，之前的w分量的公式都是w = 1，现在则为：w'z = z

那么x'z、y'z、z'z、w'z都知道了，把他们写入矩阵

$$
\begin{bmatrix}
\cfrac{2Near}{Width}&0&0&0\\ 
0&\cfrac{2Near}{Height}&0&0\\ 
0&0&\cfrac{Far + Near}{Far - Near}&\cfrac{-2Near \cdot Far}{Far - Near}\\
0&0&1&0\\
\end{bmatrix}
$$

不要忘了翻转z轴：
$$
\begin{bmatrix}
\cfrac{2Near}{Width}&0&0&0\\ 
0&\cfrac{2Near}{Height}&0&0\\ 
0&0&-\cfrac{Far + Near}{Far - Near}&\cfrac{-2Near \cdot Far}{Far - Near}\\
0&0&-1&0\\
\end{bmatrix}
$$

我们可以通过FOV和三角函数来求出Near的另一种表达方式

![透视摄像机](/posts_image/Clip_Space/Clip_Space_2_4.jpeg "透视摄像机")

上图中，角a是FOV的一半，即 $\cfrac{FOV}{2}$,则：

$\cot{a} = \cot{\cfrac{FOV}{2}} = \cfrac{Near}{Height / 2} = \cfrac{2 \cdot Near}{Height}$

因此 $Near = \cfrac{Height + \cot{\cfrac{FOV}{2}}}{2}$，再结合 $Aspect = \cfrac{Width}{Height}$ 可以得到

$$
\begin{bmatrix}
\cfrac{\cot\cfrac{FOV}{2}}{Aspect}&0&0&0\\ 
0&\cot\cfrac{FOV}{2}&0&0\\ 
0&0&-\cfrac{Far + Near}{Far - Near}&-\cfrac{2 \cdot Near \cdot Far}{Far - Near}\\
0&0&-1&0\\
\end{bmatrix}
$$

## 屏幕映射阶段 ##

顶点转换到裁剪空间上，经裁剪步骤去掉多余的顶点后，接着就到屏幕映射阶段了，这一阶段要进行投影操作：把顶点映射到屏幕上的像素坐标上去，这个过程分成两步：先齐次除法后进行坐标缩放映射。

正交摄像机里一个顶点和上述正交投影矩阵相乘后的结果：

$$
\begin{bmatrix}
\cfrac{1}{Aspect \cdot Size}&0&0&0\\ 
0&\cfrac{1}{Size}&0&0\\ 
0&0&-\cfrac{2}{Far - Near}&-\cfrac{Far + Near}{Far - Near}\\
0&0&0&1\\
\end{bmatrix}
\begin{bmatrix}
x\\
y\\
z\\
1\\
\end{bmatrix}
=
\begin{bmatrix}
\cfrac{x}{Aspect \cdot Size}\\
\cfrac{y}{Size}\\
-\cfrac{2z}{Far - Near} - \cfrac{Far + Near}{Far - Near}\\
1\\
\end{bmatrix}
$$
下图显示了经过上述投影矩阵后，正交投影的视锥体的变化：

![正交投影的视锥体的变化](/posts_image/Clip_Space/Clip_Space_3_1.png "正交投影的视锥体的变化")


我们刚才说，透视摄像机里求的P矩阵是观察空间的点 (x, y, z, w) 转 (x'z, y'z, z'z, w'z) 的转换矩阵，对最后的结果并没有影响，这是因为投影矩阵只是为屏幕映射阶段中的投影操作做准备而已，依据上述的透视模式下的投影矩阵，假设存在一个观察空间中的顶点（x, y, z, 1），那么它与投影矩阵参与运算后，w 值将会变为-z：

$$
\begin{bmatrix}
\cfrac{\cot\cfrac{FOV}{2}}{Aspect}&0&0&0\\ 
0&\cot\cfrac{FOV}{2}&0&0\\ 
0&0&-\cfrac{Far + Near}{Far - Near}&-\cfrac{2 \cdot Near \cdot Far}{Far - Near}\\
0&0&-1&0\\
\end{bmatrix}
\begin{bmatrix}
x\\
y\\
z\\
1\\
\end{bmatrix}
=
\begin{bmatrix}
x \cfrac{\cot{\cfrac{FOV}{2}}}{Aspect}\\
y \cot{\cfrac{FOV}{2}}\\
-z \cfrac{Far + Near}{Far - Near} - \cfrac{2 \cdot Near \cdot Far}{Far - Near}\\
-z\\
\end{bmatrix}
$$

可以看见转换后 w 分量的结果通常不会是 1，我们前面说过，只有 w = 1 的时候，齐次坐标才能和三维坐标等价，因此要把 x, y, z, w 分量都除以 w 分量，这样的话 w 分量就会变成 1，即：

$(x, y, z, w) \Rightarrow (\cfrac{x}{w}, \cfrac{y}{w}, \cfrac{z}{w}, 1)$ 

这个齐次除法操作对于(x'z, y'z, z'z, w'z)来说，结果是一样的

$(x'z, y'z, z'z, w'z) \Rightarrow (\cfrac{x'}{w'}, \cfrac{y'}{w'}, \cfrac{z'}{w'}, 1)$ 

所以说我们上面的投影矩阵对屏幕映射的过程是没有影响的，这个投影矩阵是正确的可用的。

对于正交摄像机来说，他的视锥体本身就是立方体，裁剪空间顶点的 w 分量也都是 1，所以这个步骤对它来说没什么变化。

以上就是齐次除法步骤，接下来，我们已经得到顶点的 x、y、z 在 NDC 空间中的值了，z 值将会在之后参与到深度测试中并有可能写入到深度缓冲里。xy 值则需要和屏幕的横纵像素进行映射计算得出顶点在屏幕上的像素坐标：

假设屏幕的横向像素数量pixelWidth = 400，纵向像素数量pixelHeight = 500

那么对于x坐标来说，它要从[-1, 1]映射到[0, 400],整个过程是：
$$
-1 \leq x \leq 1
$$
$$
-400 \leq 400x \leq 400
$$
$$
0 \leq 400x + 400 \leq 800
$$
$$
0 \leq (400 / 2)x + (400 / 2) \leq 400 
$$
因此可以得出公式，x映射到屏幕上：
$$
X = \cfrac{x \times pixelWidth}{2} + \cfrac{pixelWidth}{2}
$$
注意这里的x是经过齐次除法的！像素坐标Y同理

上面的 pixelWidth 和 pixelHeight 可能不是整个屏幕的大小，而可能是游戏窗口的大小，比较有的游戏要窗口化运行，那么 pixelWidth 和 pixelHeight 指的就是游戏窗口的大小，当然窗口可以随意拖动，顶点也一直会跟随窗口映射到不同的屏幕像素上，不过这些都是Unity帮我们自动处理的了。

## 参考 ##
《Unity Shader入门精要》

[齐次坐标系与Unity投影矩阵的推导](https://zhuanlan.zhihu.com/p/440717663)
