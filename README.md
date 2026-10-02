# Volvo_Battery_fault_analyze

# 第四组电池数据分析

`Group4_data_analyze.ipynb` 用于绘制第四组电池 **K8608** 的电压、SOC、容量，以及容量随里程的变化。只分析第四组，不处理第一、二、三组，也不处理 CSV 数据。

当前数据包含 **96 个电芯、5 次读数**。第 34 号电芯是预先指定的重点观察对象，代码用虚线、红色刻度或红色曲线突出显示；这一标记并非自动故障识别结果。

本说明对应 2026-10-02 提供的 Notebook 版本。

## 1. 文件与数据

当前代码使用以下文件：

```text
D:\Desktop\BatteryAnalysis\
├── Group4_data_analyze.ipynb
├── UU_dataset_25w37.xlsx
└── Group4_KM_Capacity.xlsx
```

### 原始数据：UU_dataset_25w37.xlsx

- 工作表名称：`UU_dataset_24w33`。注意它与文件名中的 `25w37` 不同。
- 筛选条件：`Dataset == 4`。
- 筛选后的电池 ID：`K8608`。
- 电压、SOC 和容量对比图均按 `Readout` 升序排列。

| 字段 | 含义 | 代码中的用途 |
| --- | --- | --- |
| `Dataset` | 数据组编号 | 仅保留第 4 组 |
| `Batt_ID` | 电池 ID | 当前第 4 组对应 K8608 |
| `Readout` | 读取日期 | 日期排序和曲线图例 |
| `KM` | 累计里程，km | 容量—里程分析 |
| `V_01`–`V_96` | 各电芯电压，V | 原始电压与电压偏差 |
| `S_01`–`S_96` | 各电芯 SOC，% | SOC 对比 |
| `C_01`–`C_96` | 各电芯容量 | 容量对比 |

容量直接使用原始 C 值，代码没有将容量除以额定容量，也没有归一化到 100%。因此容量轴使用 `capacity`、`Capacity` 或 `source units`，不自动将原始值解释为百分比。

### 里程数据：Group4_KM_Capacity.xlsx

- 工作表名称：`Sheet1`。
- 共 5 行数据、97 列：`KM` 加上 96 个 `C_` 容量列。
- 当前文件的里程和容量与原始文件第四组对应记录一致。
- Notebook 直接使用这个文件，不会自动从原始文件生成或更新它。
- 读取后按 `KM` 升序排序，不再执行 `Dataset` 筛选；该文件必须只包含第四组。

## 2. 运行环境

Notebook 保存的内核信息为 **Battery Analysis (.venv)，Python 3.14.7**。所需 Python 包：

| 包 | 用途 |
| --- | --- |
| `pandas` | 读取、筛选、排序和数值计算 |
| `openpyxl` | 支持读取 `.xlsx` 文件 |
| `matplotlib` | 绘图与保存 PNG |
| `ipykernel` | 在 VS Code 中运行 Notebook 单元格 |

`pathlib` 是 Python 标准库，无需单独安装。代码使用 `pd.to_datetime(..., format="mixed")`，需要支持该参数的 pandas 版本（2.0 或更新版本）。Notebook 未附带依赖版本锁定文件。

### 使用已有环境

1. 在 VS Code 中打开 `D:\Desktop\BatteryAnalysis` 文件夹。
2. 打开 `Group4_data_analyze.ipynb`。
3. 在右上角选择已配置的 **Battery Analysis (.venv)** 内核。
4. 如需确认当前解释器，可在临时单元格执行：

```python
import sys
print(sys.executable)
```

此前配置的虚拟环境位于 `D:\Desktop\BatteryAnalysis\电池异常分析\.venv`。若该环境仍可用，直接使用即可。

### 仅在新机器或尚未配置环境时

先安装 VS Code 的 Microsoft Python、Jupyter 扩展，并确保终端可使用 `uv`。以下命令在 **PowerShell 终端**执行，不放进 Python 单元格：

```powershell
Set-Location -LiteralPath "D:\Desktop\BatteryAnalysis"
uv venv --python 3.14 .venv
uv pip install --python ".\.venv\Scripts\python.exe" pandas openpyxl matplotlib ipykernel
& ".\.venv\Scripts\python.exe" -m ipykernel install --user --name group4-analysis --display-name "Group4 Analysis (.venv)"
```

这套新环境位于项目根目录的 `.venv`，与此前子文件夹中的环境位置不同。安装后在 Notebook 右上角选择 **Group4 Analysis (.venv)**。若列表未刷新，可保存文件后执行 `Developer: Reload Window` 再选择内核。

环境创建及安装方式参考 [uv 环境说明](https://docs.astral.sh/uv/pip/environments/)；命名内核的注册方式参考 [IPython 内核安装说明](https://ipython.readthedocs.io/en/stable/install/kernel_install.html)。

## 3. 单元格与运行顺序

下表编号指 Notebook 中从上往下的**实际单元格位置**，包含第一个 Markdown 单元格，不是运行时的 `In[n]` 编号。

| 单元格 | 内容 | 图形表现 | 输出文件 |
| --- | --- | --- | --- |
| 1 | 内核连接说明 | Markdown 文本，不执行 | 无 |
| 2 | 容量与容量偏差组合图 | 上下两图；第一条蓝线在两张子图中均为虚线 | `group4_capacity.png` |
| 3 | SOC 原始值 | 独立图；第一条蓝线为虚线，其余为实线 | `group4_soc_raw.png` |
| 4 | 容量原始值 | 独立图；第一条蓝线为虚线，其余为实线 | `group4_capacity_raw.png` |
| 5 | 电压原始值 | 独立图；全部实线，带日期图例 | `group4_voltage_raw.png` |
| 6 | 电压相对中位数的偏差 | 独立图；全部实线，没有单独日期图例，颜色顺序与单元格 5 一致 | `group4_voltage_deviation.png` |
| 7 | 容量—里程，黑色对照版 | 第 34 号红线，其他电芯黑线 | `Group4_KM_Capacity.png` |
| 8 | 容量—里程，灰色对照版 | 第 34 号红线，其他电芯灰线，透明度 0.5 | `Group4_KM_Capacity.png` |

推荐按顺序运行 **2、3、4、5、6、8**；需要黑色对照版时再运行 7。每个代码单元均包含自己的导入和数据读取，可独立运行。

**单元格 7 和 8 保存到同一个文件。** 后运行的版本会覆盖之前保存的 PNG；从上到下“全部运行”后，磁盘中保留灰色对照版。若需同时保留两版，应分别修改输出文件名。

当前 Notebook **没有单独的 SOC 偏差图单元格，也没有单独的容量偏差图单元格**；容量偏差目前在单元格 2 的下半图中。此说明描述现有代码，不代表新增了这些功能。

在 VS Code 中点击目标单元格左侧的运行按钮即可执行。修改代码后需要重新运行单元格，已有图像不会随代码编辑自动更新。操作入口参考 [VS Code Notebook 文档](https://code.visualstudio.com/docs/datascience/jupyter-notebooks)。

## 4. 计算方法

### 数据处理

1. 原始表筛选 `Dataset == 4`。
2. `Readout` 用 `format="mixed"` 转换为日期，并按日期排序；里程表按 `KM` 排序。
3. 按 `V_`、`S_`、`C_` 前缀分别选择指标列，并按后缀数字排序，保证电芯编号为 1–96。
4. 使用 `pd.to_numeric(..., errors="raise")` 转换数值。
5. 按原始记录绘制曲线；没有去重、平滑、插值生成新记录或归一化操作。折线只是连接已有观测点。

### 容量偏差

对同一次读数的 96 个容量取中位数，再逐个相减：

```python
capacity_difference = c - c.median()
```

中位数包含第 34 号本身。96 个值从小到大排列后，中位数为第 48、49 个值的平均数；这代表电芯容量的中间水平，不是整包总容量。

例如，最后一次读数中，第 34 号容量为 `82.68`，整包容量中位数为 `87.77`：

```text
82.68 − 87.77 = −5.09
```

图上 `−5.09` 表示该次读数中第 34 号低于中位数 5.09 个原始容量单位。它不是容量随时间下降的量，也不是除以中位数后的相对百分比。该电芯首次至末次的容量下降为 `86.06 − 82.68 = 3.38`。

### 电压偏差

```python
voltage_difference_mV = (v - v.median()) * 1000
```

先计算电压与当次整包电压中位数的差，再由 V 转换为 mV。正值表示高于中位数，负值表示低于中位数。

### 容量—里程图

- X 轴：`KM`，标签 `Mileage(KM)`，从左到右里程增加。
- Y 轴：原始 `C_` 值，标签 `Capacity`，固定范围 80–100，刻度间隔 5。
- 每条线代表同一电芯在五次读数中的容量变化。
- 第 34 号为红线和圆点；灰色对照版的其他 95 个电芯为灰线。
- 坐标轴不倒置，也没有将初始容量设为 100。80–100 只是显示范围。

## 5. 图形阅读与重叠说明

电芯编号图中，横轴是 **Cell number**，不同颜色代表不同日期；第 34 号处的峰或谷是电芯之间的差异，不是电压、SOC 或容量随时间突然跳变。

第 34 号的竖虚线及红色刻度用于定位。只有容量—里程图中，红线才固定代表第 34 号；其他按日期绘制的图中，红色对应一个读取日期。

### 为什么看不到第一条蓝色容量曲线？

当前数据中，2021-05-29 与 2021-05-31 的 **96 个电芯容量全部相同**，最大绝对差为 0。两条容量曲线及对应的容量偏差曲线完全重合，后画的橙线会遮住蓝线。

容量图使用 `linestyle="--"` 和较高 `zorder` 将蓝色虚线放在上层，让橙色实线能从间隙中显示出来。SOC 图的蓝色虚线是绘图样式，不代表其前两次 SOC 数据也完全相同。

### 为什么第 34 号只有四个可见红点？

实际保留了五条记录：

| 日期 | KM | C_34 |
| --- | ---: | ---: |
| 2021-05-29 | 27,669 | 86.06 |
| 2021-05-31 | 27,687 | 86.06 |
| 2021-08-20 | 29,567 | 84.89 |
| 2021-08-23 | 29,589 | 84.69 |
| 2022-08-31 | 39,278 | 82.68 |

前两次容量相同，里程只差 18 km。与整张图超过 11,000 km 的跨度相比，这个间距很小，两个标记几乎重合。没有删除第五个点；可放大左侧区域检查。

## 6. 图片保存在哪里

所有图片均以 **300 dpi** 保存。对当前硬编码路径：

| 输出文件 | 保存位置 |
| --- | --- |
| `group4_capacity.png` | `D:\Desktop\BatteryAnalysis` |
| `group4_soc_raw.png` | `D:\Desktop\BatteryAnalysis` |
| `group4_capacity_raw.png` | `D:\Desktop\BatteryAnalysis` |
| `group4_voltage_raw.png` | Notebook 内核的当前工作目录 |
| `group4_voltage_deviation.png` | Notebook 内核的当前工作目录 |
| `Group4_KM_Capacity.png` | `D:\Desktop\BatteryAnalysis` |

两个电压单元格使用相对路径保存，实际位置可在临时单元格中查看：

```python
from pathlib import Path
print(Path.cwd())
```

如果希望电压图也始终保存在 Excel 旁边，可将对应单元格的保存语句改成：

```python
from pathlib import Path
output = Path(path).parent / "group4_voltage_raw.png"
plt.savefig(output, dpi=300)
```

偏差图使用 `group4_voltage_deviation.png`。`Path(path)` 兼容字符串路径；字符串本身没有 `.parent` 属性。重复运行会覆盖同名图片，Notebook 不会修改输入 Excel。

## 7. 常用修改

| 需求 | 修改位置 |
| --- | --- |
| 更换文件夹 | 修改各目标单元格中的 `path`；路径在多个单元格中分别定义 |
| 调整容量—里程显示范围 | 修改 `ax.set_ylim(80, 100)` 和 `ax.set_yticks(...)` |
| 调整其他电芯颜色 | 修改 `color="gray"` 或 `color="black"` |
| 调整灰线深浅 | 修改 `alpha`；值越小越透明 |
| 保存不同版本图片 | 修改 `output` 或 `savefig` 的文件名 |
| 更换重点电芯 | 同时修改 `C_34`、其他电芯排除条件、图例、竖线位置及红色刻度条件 |

第 8 单元格中“其他电芯：黑色”的注释仍为旧文字，实际代码是 `color="gray"`、`alpha=0.5`，因此画出灰色。

## 8. 常见问题

| 提示或现象 | 处理方法 |
| --- | --- |
| `ModuleNotFoundError` | 确认 Notebook 内核与安装依赖的虚拟环境一致；使用该环境的 Python 路径安装包 |
| `externally managed` | uv 的基础 Python 不允许直接安装依赖；在项目虚拟环境中安装 |
| 找不到 `.venv` 内核 | 安装 `ipykernel` 并按环境配置部分注册命名内核，然后重新选择 |
| `FileNotFoundError` | 核对真实路径；当前文件位于 `D:\Desktop\BatteryAnalysis`，路径包含 `Desktop` |
| 工作表不存在 | 原始文件使用 `UU_dataset_24w33`；里程表使用 `Sheet1` |
| `'str' object has no attribute 'parent'` | 使用 `Path(path).parent`，或一开始将 `path` 定义为 `Path(...)` |
| PowerShell 显示 `>>` | 当前命令尚未结束，常见原因是引号不配对；按 Ctrl+C 后重新输入完整命令 |
| 改了代码但图片没变 | 重新运行相应单元格，并确认查看的是新输出文件 |
| 全部运行后里程图变成灰色版 | 第 8 单元格覆盖了第 7 单元格保存的同名文件，这是当前代码的正常行为 |

文件路径写在 Python 字符串中，例如 `r"D:\Desktop\BatteryAnalysis\UU_dataset_25w37.xlsx"`，不要混入聊天排版中的 `**`、`&#x73;` 或给下划线加额外反斜杠。PowerShell 命令不要粘进 Python 单元格。

## 9. 分析范围

本项目按所提供数值为可信数据进行可视化。图形展示电芯间的一致性、相对偏差及观测区间内的容量变化。

Notebook 没有自动异常阈值、故障分类模型或电芯均衡电量计算，也没有利用电流、温度及均衡日志来分离原因。第 34 号的突出显示是分析者预先指定的观察重点；这些图本身不将异常唯一归因为内短路、自放电或均衡失效。

本 README 已按现有 Notebook 源码核对单元格、文件名、保存路径和绘图样式，并检查两份输入表的第四组里程与容量一致性；编写说明时未重新执行整个 Notebook。
