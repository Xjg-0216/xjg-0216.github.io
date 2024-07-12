<!--
 * @Descripttion: 
 * @Author: xujg
 * @version: 
 * @Date: 2024-07-12 10:50:17
 * @LastEditTime: 2024-07-12 11:23:54
-->
# 飞行姿态

!!! note
    **滚转角 (roll)**：飞行器绕前后轴的旋转角度
    **俯仰角 (pitch)**：飞行器绕左右轴的旋转角度
    **偏航角 (yaw)**：飞行器绕垂直轴的旋转角度


python可视化演示：
代码如下：
```python
import matplotlib.pyplot as plt
from matplotlib.animation import FuncAnimation
import numpy as np

# Initialize the figure and axis
fig = plt.figure(figsize=(10, 7))
ax = fig.add_subplot(111, projection='3d')

# Define the aircraft's orientation (simple representation)
aircraft = np.array([[-1, -1, 0],
                     [1, -1, 0],
                     [1, 1, 0],
                     [-1, 1, 0],
                     [0, 0, 1]])

# Function to create rotation matrices
def rotate_aircraft(roll, pitch, yaw):
    R_roll = np.array([[1, 0, 0],
                       [0, np.cos(roll), -np.sin(roll)],
                       [0, np.sin(roll), np.cos(roll)]])
    
    R_pitch = np.array([[np.cos(pitch), 0, np.sin(pitch)],
                        [0, 1, 0],
                        [-np.sin(pitch), 0, np.cos(pitch)]])
    
    R_yaw = np.array([[np.cos(yaw), -np.sin(yaw), 0],
                      [np.sin(yaw), np.cos(yaw), 0],
                      [0, 0, 1]])
    
    return R_yaw @ R_pitch @ R_roll @ aircraft.T

# Animation function for Roll
def animate_roll(i):
    ax.clear()
    roll = np.radians(i % 360)
    rotated_aircraft = rotate_aircraft(roll, 0, 0)
    
    ax.plot3D(rotated_aircraft[0], rotated_aircraft[1], rotated_aircraft[2], 'o-')
    ax.set_xlim([-2, 2])
    ax.set_ylim([-2, 2])
    ax.set_zlim([-2, 2])
    ax.set_xlabel('X-axis (Roll)')
    ax.set_ylabel('Y-axis')
    ax.set_zlabel('Z-axis')
    ax.set_title('Aircraft Roll (Rotation around X-axis)')

# Animation function for Pitch
def animate_pitch(i):
    ax.clear()
    pitch = np.radians(i % 360)
    rotated_aircraft = rotate_aircraft(0, pitch, 0)
    
    ax.plot3D(rotated_aircraft[0], rotated_aircraft[1], rotated_aircraft[2], 'o-')
    ax.set_xlim([-2, 2])
    ax.set_ylim([-2, 2])
    ax.set_zlim([-2, 2])
    ax.set_xlabel('X-axis')
    ax.set_ylabel('Y-axis (Pitch)')
    ax.set_zlabel('Z-axis')
    ax.set_title('Aircraft Pitch (Rotation around Y-axis)')

# Animation function for Yaw
def animate_yaw(i):
    ax.clear()
    yaw = np.radians(i % 360)
    rotated_aircraft = rotate_aircraft(0, 0, yaw)
    
    ax.plot3D(rotated_aircraft[0], rotated_aircraft[1], rotated_aircraft[2], 'o-')
    ax.set_xlim([-2, 2])
    ax.set_ylim([-2, 2])
    ax.set_zlim([-2, 2])
    ax.set_xlabel('X-axis')
    ax.set_ylabel('Y-axis')
    ax.set_zlabel('Z-axis (Yaw)')
    ax.set_title('Aircraft Yaw (Rotation around Z-axis)')

# Create the animations
anim_roll = FuncAnimation(fig, animate_roll, frames=360, interval=20)
anim_roll.save('aircraft_roll.gif', writer='imagemagick')

anim_pitch = FuncAnimation(fig, animate_pitch, frames=360, interval=20)
anim_pitch.save('aircraft_pitch.gif', writer='imagemagick')

anim_yaw = FuncAnimation(fig, animate_yaw, frames=360, interval=20)
anim_yaw.save('aircraft_yaw.gif', writer='imagemagick')
```

结果如下：

### 滚转角 
(roll)：飞行器绕前后轴（X轴）的旋转角度
* **范围**：从 `-180°` 到 `180°`。
* **正方向**：如果右机翼向下，左机翼向上，则为正滚转；反之则为负滚转。
![alt text](./img/aircraft_roll.gif)

### 俯仰角 
(pitch)：飞行器向上或向下的角度
* **范围**：从 `-90°` 到 `90°`。
* **正方向**：如果机头向上，机尾向下，则为正俯仰；反之则为负俯仰。
![alt text](./img/aircraft_pitch.gif)

### 偏航角
(yaw)：飞行器向左或向右的角度
* **范围**：从 `-180°` 到 `180°`。
* **正方向**：如果机头向右，机尾向左，则为正偏航；反之则为负偏航。
![alt text](./img/aircraft_yaw.gif)