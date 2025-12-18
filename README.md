# 遗传算法解决方块布局优化问题

## 项目简介

本项目使用遗传算法（Genetic Algorithm）解决方块布局优化问题（Square Packing Problem）。目标是将一组不同大小的方块放置在二维平面上，使它们不重叠，并最小化包围所有方块的矩形面积。

项目包含四个实验（Lab1-Lab4），展示了从控制台应用到 Web API 的渐进式开发过程。

## 项目结构

```
441_2_xaodze/
├── Lab1/                    # 控制台应用
│   ├── App/                 # 控制台主程序
│   └── GeneticAlg/          # 遗传算法核心库
├── Lab2/                    # WPF 桌面应用
│   ├── AppLab2/             # WPF 应用程序
│   └── GeneticAlg/          # 遗传算法核心库
├── Lab3/                    # 增强版 WPF 应用
│   ├── AppLab3/             # WPF 应用程序（含实验管理）
│   └── GeneticAlg/          # 遗传算法核心库（含实验状态）
└── Lab4/                    # Web API 应用
    ├── WebApp/              # ASP.NET Core Web API
    ├── GeneticAlg/          # 遗传算法核心库
    └── TestProject1/        # 单元测试项目
```

## 各实验室说明

### Lab1 - 控制台应用

**功能特点：**
- 基本的遗传算法实现
- 控制台输出每代的最优解
- 按任意键停止算法
- 显示最终方块位置

**运行方式：**
```bash
cd Lab1/App
dotnet run
```

**核心参数：**
- 种群大小：20
- 方块规格：可在 Program.cs 中自定义
- 每次迭代间隔：20ms

### Lab2 - WPF 桌面应用

**功能特点：**
- 图形化界面实时显示方块布局
- 不同大小方块用不同颜色表示
  - 1x1 方块：淡蓝色 (AliceBlue)
  - 2x2 方块：淡橙色 (LightSalmon)
  - 3x3 方块：淡粉色 (LightPink)
- 网格背景便于观察
- 支持启动、停止、重置功能
- 实时显示当前代数和损失值

**运行方式：**
```bash
cd Lab2/AppLab2
dotnet run
```

**操作说明：**
1. 输入不同尺寸方块的数量
2. 点击"Start"开始算法
3. 点击"Stop"停止算法
4. 点击"Reset"重置所有参数

### Lab3 - 增强版 WPF 应用

**功能特点：**
- 继承 Lab2 的所有功能
- 更大的种群规模（500）
- 支持暂停和继续功能
- 实验管理窗口
  - 保存当前实验状态
  - 加载历史实验
  - 查看实验详情

**运行方式：**
```bash
cd Lab3/AppLab3
dotnet run
```

**新增功能：**
- **Continue 按钮：** 继续已暂停的算法
- **Manage Experiment 按钮：** 打开实验管理窗口
  - 保存当前实验（含代数、种群、最优个体）
  - 加载历史实验状态
  - 删除实验记录

### Lab4 - Web API 应用

**功能特点：**
- RESTful API 接口
- 支持多实验并发管理
- 完整的单元测试覆盖
- 基于内存的实验状态管理

**API 端点：**

| 方法 | 端点 | 说明 |
|------|------|------|
| PUT | `/experiments` | 创建新实验 |
| POST | `/experiments/{id}` | 运行一步（100代） |
| GET | `/experiments` | 获取所有实验列表 |
| DELETE | `/experiments/{id}` | 删除指定实验 |

**运行方式：**
```bash
cd Lab4/WebApp
dotnet run
```

**API 使用示例：**

创建实验：
```bash
curl -X PUT http://localhost:5000/experiments \
  -H "Content-Type: application/json" \
  -d '{
    "experimentName": "test1",
    "populationSize": 1000,
    "squareSizes": [1, 1, 2, 2, 3, 3]
  }'
```

运行实验：
```bash
curl -X POST http://localhost:5000/experiments/test1
```

获取所有实验：
```bash
curl http://localhost:5000/experiments
```

删除实验：
```bash
curl -X DELETE http://localhost:5000/experiments/test1
```

**运行测试：**
```bash
cd Lab4/TestProject1
dotnet test
```

## 核心算法说明

### 遗传算法组件

**1. Individual（个体）**
- 表示一个可能的解决方案
- 包含一组方块的位置信息
- 计算损失函数（包围矩形面积）
- 支持交叉和变异操作

**2. Population（种群）**
- 管理一组个体
- 实现选择、交叉、变异操作
- 维护种群进化

**3. Square（方块）**
- 表示单个方块
- 属性：边长、X坐标、Y坐标
- 检测方块重叠

**4. GeneticAlgorithm（遗传算法）**
- 主控制类
- 管理种群和代数
- 执行算法迭代

### 算法流程

```
1. 初始化
   - 创建初始种群
   - 随机放置方块（确保不重叠）
   - 计算每个个体的损失值

2. 选择（Selection）
   - 按损失值排序
   - 保留前50%的优秀个体

3. 交叉（Crossover）
   - 从选中的个体中随机选择两个父代
   - 生成子代（随机继承父代方块位置）
   - 确保子代无重叠
   - 填充种群至原大小

4. 变异（Mutation）
   - 以10%概率对个体进行变异
   - 随机选择一个方块
   - 在X、Y方向上随机移动（-1, 0, 1）
   - 确保不重叠且在边界内

5. 重复步骤2-4直到满足条件
```

### 损失函数

损失值计算为包围所有方块的最小矩形面积：

```csharp
Loss = (MaxX - MinX) * (MaxY - MinY)
```

其中：
- MaxX：所有方块右边界的最大值
- MinX：所有方块左边界的最小值
- MaxY：所有方块上边界的最大值
- MinY：所有方块下边界的最小值

目标是最小化此损失值。

## 技术栈

- **语言：** C# (.NET 6.0+)
- **桌面应用：** WPF (Windows Presentation Foundation)
- **Web框架：** ASP.NET Core
- **测试框架：** xUnit + FluentAssertions
- **并发管理：** ConcurrentDictionary

## 系统要求

- .NET 6.0 SDK 或更高版本
- Windows 操作系统（Lab2、Lab3 需要 WPF 支持）
- Visual Studio 2022 或 Visual Studio Code（推荐）

## 安装与运行

### 克隆项目
```bash
git clone https://github.com/GuoHaojiee/441_2_xaodze.git
cd 441_2_xaodze
```

### 运行特定实验
```bash
# Lab1 控制台
cd Lab1/App
dotnet run

# Lab2 WPF
cd Lab2/AppLab2
dotnet run

# Lab3 增强版 WPF
cd Lab3/AppLab3
dotnet run

# Lab4 Web API
cd Lab4/WebApp
dotnet run
```

## 性能优化建议

1. **种群大小：** 增加种群大小可以提高解的质量，但会增加计算时间
2. **变异概率：** 当前设置为10%，可根据问题调整
3. **选择策略：** 当前使用精英选择（前50%），可以尝试锦标赛选择等其他策略
4. **并行计算：** 可以使用 Parallel.For 加速适应度计算

## 已知限制

1. Lab2 和 Lab3 仅支持 Windows 平台（WPF 限制）
2. Lab4 实验状态存储在内存中，重启服务会丢失
3. 方块重叠检测采用穷举法，大规模问题性能可能下降
4. 随机数生成器在高频调用时可能产生相同序列

## 扩展建议

- [ ] 实现数据库持久化（Lab4）
- [ ] 添加更多选择策略（锦标赛选择、轮盘赌选择）
- [ ] 支持更多形状（矩形、圆形等）
- [ ] 实现自适应变异率
- [ ] 添加算法性能指标可视化
- [ ] 支持导出布局结果为图片
- [ ] 实现跨平台桌面应用（使用 Avalonia UI）

## 贡献指南

欢迎提交 Issue 和 Pull Request！

## 许可证

本项目代码仅供学习和研究使用。

## 作者

GuoHaojiee

## 参考资料

- [遗传算法基础](https://en.wikipedia.org/wiki/Genetic_algorithm)
- [方块装箱问题](https://en.wikipedia.org/wiki/Packing_problems)
- [ASP.NET Core 文档](https://docs.microsoft.com/aspnet/core)
- [WPF 教程](https://docs.microsoft.com/dotnet/desktop/wpf/)
