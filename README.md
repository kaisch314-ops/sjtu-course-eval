# SJTU Course Evaluation Automation

上海交通大学 i.sjtu.edu.cn 教学评价系统的自动化填写工具。

## 背景

SJTU 教学评价系统有前端反脚本检测机制，直接通过自动化工具点击保存会触发警告：

> **"请勿使用类似脚本注入方式自动评价!"**

本工具分析并绕过了该检测，实现了一键批量评价所有课程。

## 核心原理

反检测逻辑位于 `xspj_display.js`：

```javascript
// 保存按钮要求必须有 mouseenter 事件标记
$("#btn_xspj_bc,#btn_xspj_tj").off("mouseenter").on("mouseenter", function(e) {
    if (!$(this).data("enter")) {
        $(this).data("enter", "1")
    }
})
```

系统**并非检测**填写方式或事件 `isTrusted`，而是检测保存按钮是否经历过 `mouseenter` 事件。

**绕过方法**：直接注入标记数据：
```javascript
$("#btn_xspj_bc").data("enter", "1");
```

详见 [references/anti-detection.md](references/anti-detection.md)。

## 使用方法

### 前置条件

- [Kimi WebBridge](https://kimi.com/features/webbridge) 已运行
- 浏览器已登录 SJTU i.sjtu.edu.cn

### 一键评价所有课程

```bash
python scripts/eval_batch.py --all
```

### 只评价指定课程

```bash
python scripts/eval_batch.py --course 3
```

### 只保存当前页面

```bash
python scripts/eval_batch.py --save-only
```

### 命令行参数

| 参数 | 说明 |
|------|------|
| `--session` | WebBridge session 名称（默认 `sjtu-eval`） |
| `--course` | 只处理指定课程行 ID |
| `--all` | 处理所有未评课程 |
| `--save-only` | 只保存当前页面 |
| `--wait` | 切换课程后等待秒数（默认 3） |

## 工作流程

1. **导航** → 打开 SJTU 教学评价页面
2. **切换课程** → 点击左侧课程列表
3. **填写单选题** → 每组选第一个选项（非常认同/基本都在听课等）
4. **填写主观题** → 两个文本框均填"很有趣"
5. **绕过检测** → 注入 `mouseenter` 标记
6. **保存** → 点击保存按钮
7. **循环** → 处理下一门未评课程

## 文件结构

```
.
├── README.md                   # 本文档
├── SKILL.md                    # Kimi Skill 核心指南
├── scripts/
│   └── eval_batch.py           # 批量自动化脚本
└── references/
    └── anti-detection.md       # 反检测源码分析
```

## 注意事项

- 默认只**保存**不**提交**
- 主观题默认填写"很有趣"，可在脚本中修改
- 如需提交，脚本中需对 `#btn_xspj_tj` 同样注入 `enter` 标记

## License

MIT
