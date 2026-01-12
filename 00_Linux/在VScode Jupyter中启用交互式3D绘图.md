# 在VScode Jupyter中启用交互式3D绘图

```python
%matplotlib widget
#!pip install ipympl # 先安装这个库，重启Jupyter才能交互
import numpy as np
import matplotlib.pyplot as plt

fig = plt.figure()
ax = fig.add_subplot(111, projection='3d')

X = np.arange(-4, 4, 0.25)
Y = np.arange(-4, 4, 0.25)
X, Y = np.meshgrid(X, Y)
R = np.sqrt(X ** 2 + Y ** 2)
Z = np.sin(R)

# 绘制表面图
ax.plot_surface(X, Y, Z, cmap='rainbow')

# 绘制等高线
ax.contourf(X, Y, Z, zdir='z', offset=-2, cmap='rainbow')

# 设置 Z 轴范围
ax.set_zlim(-2, 2)

# 显示图形
plt.show()
```

