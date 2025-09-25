# 卡尔曼滤波
卡尔曼滤波（Kalman Filter, KF）是一种**递归的最优估计方法**，常用于在有噪声的情况下，对动态系统的状态（位置、速度、姿态等）进行估计和预测。它广泛应用在**导航、信号处理、目标跟踪、机器人定位**等领域。

## 1. 基本思想

系统的真实状态（例如目标位置和速度）无法直接精确观测，只能通过含噪声的观测值来获取。
卡尔曼滤波通过：

* **预测 (Predict)**：根据运动模型，利用上一次状态估计预测当前状态；
* **更新 (Update)**：结合当前观测值修正预测结果。

这样，就能在“模型预测”和“测量修正”之间取得平衡，从而得到比单独依赖模型或观测更准确的估计。

---

## 2. 数学模型

卡尔曼滤波适用于 **线性、高斯噪声** 的情况。

* **状态转移方程**（系统模型）：

$$
x_k = A x_{k-1} + B u_k + w_k
$$

其中：

* $x_k$：系统在时刻 $k$ 的状态向量（如位置、速度）

* $A$：状态转移矩阵（描述系统的运动模型）

* $B$：控制输入矩阵

* $u_k$：控制量（如加速度、转向）

* $w_k$：过程噪声，服从高斯分布 $N(0,Q)$

* **观测方程**（测量模型）：

$$
z_k = H x_k + v_k
$$

其中：

* $z_k$：观测量（雷达测到的位置、传感器读数）
* $H$：观测矩阵（状态与观测的关系）
* $v_k$：观测噪声，服从高斯分布 $N(0,R)$

---

## 3. 两个主要步骤

### (1) 预测步骤

$$
\hat{x}_k^- = A \hat{x}_{k-1} + B u_k
$$

$$
P_k^- = A P_{k-1} A^T + Q
$$

* $\hat{x}_k^-$：预测状态
* $P_k^-$：预测协方差（不确定性）

### (2) 更新步骤

计算卡尔曼增益：

$$
K_k = P_k^- H^T (H P_k^- H^T + R)^{-1}
$$

更新状态：

$$
\hat{x}_k = \hat{x}_k^- + K_k (z_k - H \hat{x}_k^-)
$$

更新协方差：

$$
P_k = (I - K_k H) P_k^-
$$

这里的 **卡尔曼增益 $K_k$** 决定了预测和观测的权重分配：

* 若观测噪声大（R 大），则更多依赖预测；
* 若模型不确定（Q 大），则更多依赖观测。

---

## 4. 特点

* **优点**：

  * 计算效率高，递归形式适合实时应用；
  * 在高斯噪声和线性系统下是最优估计器。
* **缺点**：

  * 假设系统是线性的、噪声是高斯分布；
  * 对非线性系统需使用 **扩展卡尔曼滤波 (EKF)** 或 **无迹卡尔曼滤波 (UKF)**。


## 5. C++ 中常见的卡尔曼滤波实现

### 5.1 常见状态/观测定义

常用的状态向量（8 维）：

```
x = [x_c, y_c, a, h, vx, vy, va, vh]^T
```

* `x_c,y_c`：bbox 中心坐标
* `a`：宽高比 (w/h) 或宽高比的某种表征（各实现细节略有差异）
* `h`：bbox 高度（或 h）
* `vx,vy,va,vh`：对应的速度分量（状态向量后半段）

观测向量 `z` 通常是 4 维：

```
z = [x_c, y_c, a, h]^T
```

维度说明：

* 状态 `x` 是 8×1 向量
* 状态协方差 `P` 是 8×8 矩阵
* 观测矩阵 `H` 是 4×8（把状态投影到观测空间）
* 观测协方差 `R` 是 4×4
* 过程噪声协方差 `Q` 是 8×8


### 5.2 核心矩阵：状态转移矩阵 A（匀速模型）

典型的 `A`（8×8）用于“位置 += 速度”模型（离散时间步长假定为 1）：

```
A = [ I4  I4
      0   I4 ]
```

更明确地（块矩阵）：

* 左上 4×4：单位（位置由位置保留）
* 左下 4×4：全 0（速度不影响之前的速度）
* 右上 4×4：单位（位置 += 速度）
* 右下 4×4：单位（速度保持不变）

数学上，预测步骤等价于：

```
x^- = A x
P^- = A P A^T + Q
```

实现注意：

* 有些实现把位置和速度顺序颠倒，注意和代码里的索引对齐。
* 如果帧间时间 dt ≠ 1，需要把速度项乘以 dt。



### 5.3 观测矩阵 H（把状态投影到测量）

`H` 是 4×8，常为：

```
H = [ I4  0 ]
```

也就是取状态的前 4 个分量作为观测预测：`z^- = H x^- = [x_c, y_c, a, h]`

---

### 5.4. 初始化（init / initiate）

当检测器在新目标上生成第一次检测时，会调用 `init`：

实现步骤：

1. 用检测框（`z`）构造初始状态 `x`：把位置部分设为 `z`，速度部分设为 0（或者小值）。
2. 初始化 `P`：通常对位置维度赋较小方差（例如 `10` 或 `1e-2`），对速度维度赋较大方差（例如 `1e3`）表示对速度不确定。

代码示例（伪）：

```cpp
Vector8 x;
x.head(4) = z;         // z is 4x1
x.tail(4).setZero();   // initial velocity unknown

Matrix8 P;
P.setZero();
P.block<4,4>(0,0) = pos_var * I4;    // position variance
P.block<4,4>(4,4) = vel_var * I4;    // velocity variance
```


### 5.5 预测（predict）

典型实现（伪）：

```cpp
x = A * x;
P = A * P * A.transpose() + Q;
```

说明：

* `Q`（过程噪声）常选为对角矩阵。其数值决定模型对“加速度/模型误差”的容忍度：`Q` 大 → 预测更“发散”、更信任观测。
* 有些实现把 `Q` 设计为与目标尺度（h）有关，使得大目标的过程噪声比例不同。


### 5.6 投影与更新（project + update）

先将预测的状态投影到观测空间：

```cpp
z_pred = H * x;            // 4x1
S = H * P * H.transpose() + R;  // 4x4, 观测协方差
K = P * H.transpose() * S.inverse(); // 8x4 卡尔曼增益
```

随后用观测 `z` 更新：

```cpp
x = x + K * (z - z_pred);
P = (I - K * H) * P; // 标准形式
```

实现细节：

* 为数值稳定，有时采用 Joseph form 更新协方差：

  ```
  P = (I - K H) P (I - K H)^T + K R K^T
  ```
* `S` 的逆或 Cholesky 分解需要数值稳定性处理（避免非正定）。实际中用 `LDLT` 或 `LLT` 更稳定。

---

### 5.7 门控（Gating）与马氏距离（Mahalanobis distance）

更新前常需要判断检测与预测的“距离”是否合理（用于关联、滤掉异常匹配）。常用两种距离：

* IoU（基于 bbox 的重叠）
* 马氏距离（考虑预测协方差）：

马氏距离计算（预测投影到观测空间）：

```
d^2 = (z - z_pred)^T * S^{-1} * (z - z_pred)
```

如果 `d^2` 超过 `chi2` 阈值（自由度 = 4，对应置信度比如 0.995），则认为该检测与该轨迹不相容（用于匈牙利前做 gating）。

实现要点：

* `S` 是 4×4，不同目标/尺度 `S` 会不同，马氏距离自适应尺度，很适合多尺度目标。
* 常在 `KalmanFilter::gating_distance()` 或 `KalmanFilter::distance()` 等函数中实现。

---

### 5.8 批量预测（multi_predict）

为性能考虑，ByteTrack 往往会对所有活跃轨迹做一次批量预测（一次矩阵乘法循环），然后在匹配之前把所有 `x_pred` 投影出来以供计算 cost matrix（IoU 或 Mahalanobis）。伪代码：

```cpp
for each track:
  x_pred = A * x_old
  P_pred = A * P_old * A^T + Q
  z_pred = H * x_pred
  S = H * P_pred * H^T + R
```


### 5.9 在 STrack / ByteTrack 的使用位置（结合跟踪流程）

* **创建轨迹**：检测器的某个未匹配检测会 `initiate()` 一个新的轨迹（调用 Kalman `init`）。
* **预测**：每帧开始会对所有活跃轨迹 `predict()`（得到 `x^-` 与 `P^-`）。
* **计算 cost matrix**：用预测值和检测框计算 IoU/马氏距离矩阵（用于匈牙利匹配）。常用混合策略：优先用「检测置信度高的」与预测匹配（ByteTrack 的 trick），并把低置信检测当作补充。
* **更新**：对匹配到检测的轨迹 `update()`。
* **管理生命周期**：连续 `n` 帧丢失后删除轨迹或标记为 lost。


## 6 典型 C++ 代码片段


* `DETECTBOX`：4 元素的检测向量，格式通常是 `[x_c, y_c, a, h]`（中心 x, 中心 y, 宽高比, 高度），形状可视为 `1×4`（或 `4×1`，见下面讨论）。
* `KAL_MEAN`：状态向量（8 维），对应 `[x_c,y_c,a,h,vx,vy,va,vh]`，长度 8。
* `KAL_COVA`：状态协方差矩阵，`8×8`。
* `KAL_HMEAN`/`KAL_HCOVA`：观测空间的 mean（4）和协方差 `4×4`。
* `DETECTBOXSS`：`N×4` 的矩阵（多条检测差值矩阵）。

> 注意：源码里你有大量 `transpose()` 与 `block<1,4>` 的混用，表明实现上状态可能是以**行向量（1×8）**为主（而不是更常见的列向量 8×1）。下面解释时我既会按“数学上更标准”的列向量写法给出公式，也会指出源码中对应的维度/转置位置，帮助你一一对应。

---

### 6.1 `chi2inv95`

```cpp
const double KalmanFilter::chi2inv95[10] = { 0, 3.8415, 5.9915, 7.8147, 9.4877, 11.070, 12.592, 14.067, 15.507, 16.919 };
```

* 这是卡方分布在 95% 置信度下不同自由度（degree-of-freedom）的临界值表：`chi2inv95[d]` 给出 `d` 自由度的临界值（例如 `d=4` 时约 `9.4877`）。
* 在 gating（门控）时常用来判断马氏距离是否在接受范围内（如果 `maha^2 > chi2inv95[d]` 则拒绝匹配）。


### 6.2 构造函数 `KalmanFilter::KalmanFilter()`

```cpp
int ndim = 4;
double dt = 1.;

_motion_mat = Eigen::MatrixXf::Identity(8, 8);
for (int i = 0; i < ndim; i++) {
    _motion_mat(i, ndim + i) = dt;
}
_update_mat = Eigen::MatrixXf::Identity(4, 8);

this->_std_weight_position = 1. / 20;
this->_std_weight_velocity = 1. / 160;
```

* `ndim = 4`：状态中“可观测的”部分长度（`x_c,y_c,a,h`），总状态维度 8（含速度分量）。
* `_motion_mat`：这是状态转移矩阵 `A`（8×8），按匀速模型构造（離散時間 `dt` = 1）。数学上对应：

  $$
  A =
  \begin{bmatrix}
  I_4 & I_4\\
  0   & I_4
  \end{bmatrix}
  $$

  即位置部分 `pos' = pos + vel * dt`，速度保持不变。
* `_update_mat`：观测矩阵 `H`，`4×8`，通常是 `[I4  0]` —— 把状态的前 4 个量映射为观测空间。
* `_std_weight_position` / `_std_weight_velocity`：权重常量，用来把尺度（通常用 `h`）转成噪声标准差（std）。采用相对尺度而非绝对像素值，能让噪声随目标大小调整。


### 6.3 `initiate(const DETECTBOX &measurement)` —— 初始化新轨迹

关键代码：

```cpp
DETECTBOX mean_pos = measurement;
DETECTBOX mean_vel; for (i...) mean_vel(i) = 0;

KAL_MEAN mean;
for (i=0;i<8;i++) {
  if (i<4) mean(i) = mean_pos(i);
  else mean(i) = mean_vel(i-4);
}

KAL_MEAN std;
std(0) = 2 * _std_weight_position * measurement[3];
std(1) = 2 * _std_weight_position * measurement[3];
std(2) = 1e-2;
std(3) = 2 * _std_weight_position * measurement[3];
std(4) = 10 * _std_weight_velocity * measurement[3];
std(5) = 10 * _std_weight_velocity * measurement[3];
std(6) = 1e-5;
std(7) = 10 * _std_weight_velocity * measurement[3];

KAL_MEAN tmp = std.array().square();
KAL_COVA var = tmp.asDiagonal();
return std::make_pair(mean, var);
```

* **mean**（初始状态）：把检测量 `measurement` 的前 4 个分量放到 `mean` 的前 4 位（位置相关），速度分量初始化为 0（或者可以用两帧差分估计）。
* **协方差 `P` 的初始化**：

  * 对位置：`std = 2 * std_weight_position * h` → 标准差与目标高度 `h` 成比例；示例：若 `std_weight_position=1/20` 且 `h=50`，则 `std = 2 * (1/20) * 50 = 5` → 方差 = 25。
  * 对速度：`std = 10 * std_weight_velocity * h` → 速度的不确定性通常比位置信息大/小要看常数（此实现用了 `10 * 1/160 * h` → ≈ `h/16`）。
  * 第 3 个分量（aspect ratio）和第 6 个速度分量被硬编码为小常数（`1e-2`, `1e-5`），因为 aspect 比较稳定或尺度变化较小。
* 最终返回 `(mean, var)`：`mean` 是 8 维向量，`var` 是 `8×8` 对角方差矩阵（只初始化对角）。

---

### 6.4 `predict(KAL_MEAN &mean, KAL_COVA &covariance)` —— 预测步骤

代码：

```cpp
DETECTBOX std_pos;
std_pos << _std_weight_position * mean(3), _std_weight_position * mean(3), 1e-2, _std_weight_position * mean(3);
DETECTBOX std_vel;
std_vel << _std_weight_velocity * mean(3), _std_weight_velocity * mean(3), 1e-5, _std_weight_velocity * mean(3);
KAL_MEAN tmp;
tmp.block<1, 4>(0, 0) = std_pos;
tmp.block<1, 4>(0, 4) = std_vel;
tmp = tmp.array().square();
KAL_COVA motion_cov = tmp.asDiagonal();

KAL_MEAN mean1 = this->_motion_mat * mean.transpose();
KAL_COVA covariance1 = this->_motion_mat * covariance *(_motion_mat.transpose());
covariance1 += motion_cov;

mean = mean1;
covariance = covariance1;
```

数学对应（标准 KF）：

* 状态预测： $ \hat{x}^- = A \hat{x} $。
* 协方差预测： $P^- = A P A^T + Q$，其中 `Q` 是**过程噪声协方差**（这里叫 `motion_cov`）。
  实现细节说明：
* `motion_cov`（Q）是根据当前 `mean(3)`（也就是 `h` —— 目标高度）动态计算的：`std_pos = _std_weight_position * h`（与 `init` 不同，预测时没有乘 `2`，这是实现上的选择）；速度项同理。

  * 这样做的目的是：目标越大（`h` 大），预测时的绝对误差也可能越大，所以将 `Q` 随 `h` 放大。
* `mean1 = _motion_mat * mean.transpose()`：源码里用到了多次 `transpose()` —— 这是因为代码中状态可能以**行向量**存储（`1×8`），而矩阵 `A` 以 `8×8` 形式存储，乘法需要列向量。
* 最后把 `mean` 与 `covariance` 替换为预测值。

注意/建议：

* `motion_cov` 应保证数值稳定（不要太小导致 `P` 丧失正定）。
* 如果帧间时间 `dt` 不是 1，应把 `A` 和 `Q` 按 `dt` 调整（例如位置项乘以 `dt`，过程噪声按 `dt` 的多项式扩展）。


### 6.5 `project(const KAL_MEAN &mean, const KAL_COVA &covariance)` —— 将状态投影到观测空间

代码：

```cpp
DETECTBOX std;
std << _std_weight_position * mean(3), _std_weight_position * mean(3), 1e-1, _std_weight_position * mean(3);
KAL_HMEAN mean1 = _update_mat * mean.transpose();
KAL_HCOVA covariance1 = _update_mat * covariance * (_update_mat.transpose());
Eigen::Matrix<float, 4, 4> diag = std.asDiagonal();
diag = diag.array().square().matrix();
covariance1 += diag;
return std::make_pair(mean1, covariance1);
```

数学对应：

* `z_pred = H * x^-`（预测的观测）
* `S = H P^- H^T + R`，R 是观测噪声协方差（代码中用 `diag`，由 `std` 得到）
* 返回 `(z_pred, S)`
  实现/注意点：
* `std` 中第 3 个元素（aspect）在 `project` 中使用 `1e-1`（比 `init` 使用的 `1e-2` 稍大），这种不一致可能是作者刻意在观测噪声上更保守，也可能是笔误，建议统一或注释说明。
* `covariance1` 在这里就是 `S`（投影协方差），用于后续计算卡尔曼增益与 gating。


### 6.6 `update(...)` —— 用观测修正预测

代码（重要段）：

```cpp
KAL_HDATA pa = project(mean, covariance);
KAL_HMEAN projected_mean = pa.first;
KAL_HCOVA projected_cov = pa.second;

Eigen::Matrix<float, 4, 8> B = (covariance * (_update_mat.transpose())).transpose();
Eigen::Matrix<float, 8, 4> kalman_gain = (projected_cov.llt().solve(B)).transpose(); // eg.8x4
Eigen::Matrix<float, 1, 4> innovation = measurement - projected_mean; //eg.1x4
auto tmp = innovation * (kalman_gain.transpose());
KAL_MEAN new_mean = (mean.array() + tmp.array()).matrix();
KAL_COVA new_covariance = covariance - kalman_gain * projected_cov * (kalman_gain.transpose());
return std::make_pair(new_mean, new_covariance);
```

数学对应（经典 KF）：

* 预测观测： `z_pred = H x^-`， `S = H P^- H^T + R`（已由 `project` 得到）
* 卡尔曼增益： $K = P^- H^T S^{-1}$。源码通过线性代数变换与 LLT 求解避免直接求逆：

  * 取 `B = (P * H^T)^T`，然后 `projected_cov.llt().solve(B)` 计算 `S^{-1} * B`，再转置得到 `K`（数学上等于 `P H^T S^{-1}`，前面解释里可验证）。
* 创新（innovation）： `y = z - z_pred` （观测减预测观测）
* 状态更新： `x = x^- + K y`（源码实现为行向量形式的 `mean + (innovation * K^T)`）
* 协方差更新（你用的是标准形式）：

  $$
  P = P^- - K S K^T
  $$

* 建议：使用 **Joseph 形式** 更新可以更数值稳定并更能保证 `P` 正定：

    $$
    P = (I - K H) P^- (I - K H)^T + K R K^T.
    $$
* 你用 `projected_cov.llt().solve` 来解 `S X = B`，比直接 `S.inverse()` 更高效且更稳定（这是好的实践）。

潜在问题：

* 如果 `projected_cov` 不是严格正定，`LLT` 会失败。建议在失败时退用 `LDLT`，或者在对角线上加小量 `eps`（jitter）后再做分解。
* `innovation` 与 `mean` 的形状（行向量 vs 列向量）要统一；源码中通过 `transpose()` 多次切换方向，易出错；建议统一成列向量形式，简化表达。

---

### 6.7 `gating_distance(...)` —— 计算（批量）马氏距离，用于关联 gating

代码：

```cpp
KAL_HDATA pa = this->project(mean, covariance);
... // pa.first = mean1 (1x4), pa.second = covariance1 (4x4)

DETECTBOXSS d(measurements.size(), 4);
for (box in measurements) {
  d.row(pos++) = box - mean1;
}
Eigen::Matrix<float, -1, -1, RowMajor> factor = covariance1.llt().matrixL();
Eigen::Matrix<float, -1, -1> z = factor.triangularView<Eigen::Lower>().solve<Eigen::OnTheRight>(d).transpose();
auto zz = ((z.array())*(z.array())).matrix();
auto square_maha = zz.colwise().sum();
return square_maha;
```

数学意义（逐步）：

* `d_i = z_i - z_pred`，对 `N` 个检测得到 `N×4` 的差值矩阵 `d`。
* 对于每个 `d_i`，马氏距离平方定义为 $d_i^T S^{-1} d_i$（`S` 为投影协方差）。
* 计算方法：对 `S` 做 Cholesky 分解 `S = L L^T`，解 `L y = d_i^T` 得 `y = L^{-1} d_i^T`，则 `d_i^T S^{-1} d_i = ||y||^2`。源码就是用这个稳定做法（避免直接求 `S^{-1}`）。
* 返回结果 `square_maha` 为每个 `measurement` 的 **平方马氏距离**（可以与 `chi2inv95[4]` 比较做门控）。

实现细节/注意点：

* `only_position` 分支未实现（源码 `printf("not implement!"); exit(0);`）。`only_position` 若为真，通常意图是只基于位置（x,y）计算门控距离（自由度 = 2），用于某些只在 xy 上 gating 的策略。可以实现一个子 `H_pos` 和相应的 `S_pos`。
* 同样要对 `LLT` 失败加保护（数值稳定性）。

---

### 6.8 参数含义与常数解释（为什么这么设）

* `_std_weight_position = 1/20` 与 `_std_weight_velocity = 1/160`：把目标高度 `h` 映射为噪声尺度。

  * 例子： `h = 50px`：

    * `init` 时 `pos_std = 2 * (1/20) * 50 = 5` → `pos_var = 25`（初始位置不确定度 ~5 px）
    * `init` 时 `vel_std = 10 * (1/160) * 50 = 3.125` → `vel_var ≈ 9.77`
  * 这些值是经验常数，能按具体场景（帧率、相机分辨率、目标运动速度）来调。
* 小常数 `1e-2`, `1e-5`：表示对某些维度（如 aspect 或其速度）的强先验（非常确定或非常稳定）——但要小心，不要把某些分量设得太小，导致数值问题。

---

### 6.9 常见改进建议（代码/数值稳定/性能）

1. **统一向量方向**：把状态 `x` 用固定的列向量（`Eigen::Matrix<float,8,1>`）表示，减少 `transpose()` 调用 —— 更直观也更不易出错。
2. **协方差更新采用 Joseph 形式**，保证数值稳定性：

   ```cpp
   Matrix8f I = Matrix8f::Identity();
   P = (I - K*H) * P * (I - K*H).transpose() + K * R * K.transpose();
   ```
3. **LLT 失败保护**：在 `project`/`update`/`gating` 中，如果 `LLT` 失败，先尝试 `LDLT`，或给 `cov += eps * I` 后重试（`eps` 如 `1e-6`~`1e-3` 视量级）。
4. **避免频繁动态内存**：用 `Eigen::Matrix<float,8,8>` 等固定尺寸类型提高性能（避免 `MatrixXf` 的动态分配）。
5. **批量预测**：如果有大量轨迹，按批（batch）做 `A * X` 可以更快（线性代数优化）。
6. **自适应噪声**：可以把 `R` 或 `Q` 与检测器置信度 `score` 关联（高置信度 → 小 `R`，信任观测更多）。
7. **帧间时间 `dt`**：若视频帧率不稳定，`A` 中 `dt` 不应固定为 `1`，应传入真实 `dt` 并把 `Q` 按 `dt` 调整（例如 `Q` 会含 `dt^2`、`dt^3` 项）。
8. **实现 `only_position` gating**：当只以 `x,y` 做 gating 时，投影 `H_pos` = pick x,y dims，`S_pos` 为对应子矩阵，degree-of-freedom = 2。

---

### 6.10. 小示例：数值化走一遍（便于理解）

假设进入 `initiate` 的检测 `measurement = [100, 200, 1.0, 50]`（`h=50`）：

* 初始化 mean = `[100,200,1.0,50, 0,0,0,0]`
* 初始化 std：

  * `std(0)=2*(1/20)*50 = 5` → var = 25
  * `std(4)=10*(1/160)*50 = 3.125` → var ≈ 9.77
  * 其他按源码（`1e-2`, `1e-5`）
* 得到 `P = diag([25,25,1e-4,25, 9.77,9.77,1e-10,9.77])`（约）
  下一帧 predict（假设 mean 的速度现在仍 0）：
* `mean^- = A * mean` → 位置没变（速度 = 0），`P^- = A P A^T + Q`，`Q` 根据 `h` 再生成并加入到对角上，导致 `P^-` 的位置与速度方差略增加。
  如果检测器返回 `z = [103, 199, 0.98, 49]`：
* `project` 得到 `z_pred` 与 `S`
* `kalman_gain K = P^- H^T S^{-1}`
* `innovation y = z - z_pred ≈ [3, -1, -0.02, -1]`
* `x = x^- + K y` → 得到新的位置/速度估计（融合了预测与检测）

