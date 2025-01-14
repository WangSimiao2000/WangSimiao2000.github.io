---
title: GAMES101 作业1
date: 2024-12-21 16:40:00 +0800
categories: [笔记, GAMES101]
tags: [GAMES101, 旋转矩阵, 透视投影矩阵]
math: true
---

这是GAMES101的正式作业的作业1, 要求实现两个矩阵计算, 包括模型变换(只有旋转)矩阵和透视投影矩阵

我的所有GAMES101作业的仓库地址: [GAMES101-Assignments](https://github.com/WangSimiao2000/GAMES101-Assignments)

## 模型变换矩阵

参数只有旋转角度`rotate_angle`, 要求实现绕Z轴旋转`rotate_angle`角度的矩阵

```cpp
Eigen::Matrix4f get_model_matrix(float rotation_angle)
{
    Eigen::Matrix4f model = Eigen::Matrix4f::Identity();

    // TODO: Implement this function
    // Create the model matrix for rotating the triangle around the Z axis.
    // Then return it.
    
    // 将角度从度转为弧度（如果传入的是度的话）
    float angle_in_radians = rotation_angle * MY_PI / 180.0f;

    // 创建绕Z轴的旋转矩阵
	Eigen::Matrix4f rotation_matrix = Eigen::Matrix4f::Identity(); // 初始化为单位矩阵
    rotation_matrix(0, 0) = cos(angle_in_radians);  // cos(θ)
    rotation_matrix(0, 1) = -sin(angle_in_radians); // -sin(θ)
    rotation_matrix(1, 0) = sin(angle_in_radians);  // sin(θ)
    rotation_matrix(1, 1) = cos(angle_in_radians);  // cos(θ)

	model = rotation_matrix * model;

    return model;
}
```

原理很简单, 就是绕Z轴旋转, 旋转矩阵如下:

$$
\begin{bmatrix}
\cos(\theta) & -\sin(\theta) & 0 & 0 \\
\sin(\theta) & \cos(\theta) & 0 & 0 \\
0 & 0 & 1 & 0 \\
0 & 0 & 0 & 1
\end{bmatrix}
$$

我们只需要先将角度转为弧度, 然后使用`Eigen::Matrix4f::Identity()`初始化一个单位矩阵, 然后根据上述公式填入对应的值即可

## 透视投影矩阵

参数分别是`eye_fov`, `aspect_ratio`, `zNear`, `zFar`, 分别是视野角, 长宽比, 近裁剪面, 远裁剪面, 要求实现透视投影矩阵

这里的透视投影矩阵不是直接用以下公式计算的:

$$
\begin{bmatrix}
\frac{1}{\tan(\frac{\text{fov}}{2})} & 0 & 0 & 0 \\
0 & \frac{1}{\tan(\frac{\text{fov}}{2})} & 0 & 0 \\
0 & 0 & \frac{z_{\text{far}} + z_{\text{near}}}{z_{\text{near}} - z_{\text{far}}} & \frac{2 \cdot z_{\text{far}} \cdot z_{\text{near}}}{z_{\text{near}} - z_{\text{far}}} \\
0 & 0 & -1 & 0
\end{bmatrix}
$$

而是根据GAME101课程中给出的方法, 先计算正交投影矩阵, 然后通过一个将透视空间(锥体)变换成正交空间(长方体)的矩阵, 将这个矩阵与正交投影矩阵相乘得到透视投影矩阵, 原理见[这里](https://wangsimiao2000.github.io/posts/GAMES101-Lecture04-02)

```cpp
Eigen::Matrix4f get_projection_matrix(float eye_fov, float aspect_ratio,
                                      float zNear, float zFar)
{
    // Students will implement this function

    Eigen::Matrix4f projection = Eigen::Matrix4f::Identity();

    // TODO: Implement this function
    // Create the projection matrix for the given parameters.
    // Then return it.

    float top, bottom, left, right;
	top = zNear * tan(eye_fov / 2.0f);
	bottom = -top;
	right = top * aspect_ratio;
	left = -right;

	// 透视投影矩阵
	// 正交投影矩阵=缩放矩阵dot平移矩阵
	// 透视投影矩阵=正交投影矩阵dot挤压操作矩阵
	Eigen::Matrix4f S_ortho = Eigen::Matrix4f::Identity(); // 缩放矩阵
	S_ortho(0, 0) = 2.0f / (right - left);
	S_ortho(1, 1) = 2.0f / (top - bottom);
	S_ortho(2, 2) = 2.0f / (zNear - zFar);

	Eigen::Matrix4f T_ortho = Eigen::Matrix4f::Identity(); // 平移矩阵
	T_ortho(0, 3) = -(right + left) / 2.0f;
	T_ortho(1, 3) = -(top + bottom) / 2.0f;
	T_ortho(2, 3) = -(zNear + zFar) / 2.0f;

	Eigen::Matrix4f orthoMatrix = Eigen::Matrix4f::Identity(); // 正交投影矩阵
	orthoMatrix = S_ortho * T_ortho;

	Eigen::Matrix4f persp2ortho = Eigen::Matrix4f::Zero(); // 将透视空间转换为正交空间的矩阵
	persp2ortho(0, 0) = zNear;
	persp2ortho(1, 1) = zNear;
	persp2ortho(2, 2) = zNear + zFar;
	persp2ortho(2, 3) = -zNear * zFar;
	persp2ortho(3, 2) = -1;

	// 透视投影矩阵
	projection = orthoMatrix * persp2ortho;

    return projection;
}
```

最后, 运行测试代码, 生成的图片如下:

![GAMES101-Assignment1](/assets/posts/GAMES101-Assignment1/01.png){:width="400px"}

**注意**: 这里需要把opencv文件夹的`\opencv\build\x64\vc16\bin`目录里的`opencv_world4100d.dll`文件复制到项目build文件夹的Debug文件夹下, 否则会报错`"由于找不到 opencv world4100d.dll，无法继续执行代码。重新安装程序可能会解决此问题。"`

![Error](/assets/posts/GAMES101-Assignment1/02.png){:width="700px"}

## 绕任意轴旋转的矩阵(额外题目)

实现一个构建绕任意轴旋转的旋转矩阵的函数，可以使用 Rodrigues 旋转公式（罗德里格旋转公式）。它是基于轴-角表示法的一种方法。

### 罗德里格旋转公式：
假设旋转轴是单位向量 $\mathbf{u} = (x, y, z)$，旋转角度是 $\theta$，则旋转矩阵为：

$$
R = I + \sin(\theta)K + (1 - \cos(\theta))K^2
$$

其中：
- $I$ 是 3x3 单位矩阵。
- $K$ 是旋转轴 $\mathbf{u}$ 的反对称矩阵：

$$
K = \begin{bmatrix}
0 & -z & y \\
z & 0 & -x \\
-y & x & 0
\end{bmatrix}
$$

```cpp
Eigen::Matrix4f get_rotation(Vector3f axis, float angle) {
	// 构建一个函数，返回绕任意轴旋转的旋转矩阵
	// axis: 旋转轴，angle: 旋转角度

	// 归一化旋转轴
	axis.normalize();
	float x = axis[0], y = axis[1], z = axis[2]; // 旋转轴的三个分量
	float radian = angle * MY_PI / 180.0f; // 角度转弧度

	Eigen::Matrix3f K = Eigen::Matrix3f::Zero(); // 构建反对称矩阵
	K(0, 1) = -z; 
    K(0, 2) = y;
	K(1, 0) = z;
	K(1, 2) = -x;
	K(2, 0) = -y;
	K(2, 1) = x;

	// 计算旋转矩阵 R = I + sin(θ)K + (1 - cos(θ))K^2
    Eigen::Matrix3f I = Eigen::Matrix3f::Identity();
	Eigen::Matrix3f R = I + sin(radian) * K + (1 - cos(radian)) * K * K;

	// 将3x3的旋转矩阵转换为4x4的旋转矩阵
	Eigen::Matrix4f rotation_matrix = Eigen::Matrix4f::Identity();
	rotation_matrix.block<3, 3>(0, 0) = R;

	return rotation_matrix;

}
```



### 步骤：
1. **归一化旋转轴**：确保旋转轴是单位向量。
2. **计算旋转矩阵**：使用 Rodrigues 公式生成 3x3 的旋转矩阵。
3. **扩展为 4x4 矩阵**：将其扩展为 4x4 矩阵以适配齐次坐标。
