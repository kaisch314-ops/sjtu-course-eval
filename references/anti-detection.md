# SJTU 教学评价系统反脚本检测机制分析

## 检测源码

反检测逻辑位于页面加载的 JS 文件中：

```
https://i.sjtu.edu.cn/js/comp/jwglxt/jxpjgl/xspj/xspj_display.js
```

### 保存按钮的 mouseenter 监听

```javascript
$("#btn_xspj_bc,#btn_xspj_tj").off("mouseenter").on("mouseenter", function(e) {
    if (!$(this).data("enter")) {
        $(this).data("enter", "1")
    }
})
```

当鼠标真实悬停在保存/提交按钮上时，jQuery 会在按钮元素上设置 `data("enter", "1")` 标记。

### 保存时的检测逻辑

```javascript
$("#btn_xspj_bc")
    .off("touchend click")
    .on("touchend click", function(e) {
        if (isPc && !$(this).data("enter") && $("#xspjjbzrkz").val() == '1') {
            $.alert($.i18n.get("qwsyjbzr"));
            return;
        }
        // ... 正常保存流程
    });
```

### 触发警告的条件

三个条件**同时满足**时弹出警告 **"请勿使用类似脚本注入方式自动评价!"**：

1. `isPc` — 当前设备被识别为 PC（非移动端）
2. `!$(this).data("enter")` — 保存按钮从未触发过 `mouseenter` 事件
3. `$("#xspjjbzrkz").val() == '1'` — 学校开启了脚本检测开关

## 为什么常见绕过方法失败

| 方法 | 结果 | 原因 |
|------|------|------|
| 直接修改 `textarea.value` | 触发警告 | 保存按钮缺少 `enter` 标记 |
| 使用 WebBridge `click` 点击 radio | 触发警告 | 保存按钮缺少 `enter` 标记 |
| 添加操作延迟 | 触发警告 | 检测与速度无关 |
| 模拟 `mousedown`/`mouseup` | 触发警告 | 只检测 `mouseenter` |

## 正确绕过方式

直接通过 `evaluate` 给保存按钮注入标记数据：

```javascript
const btn = document.getElementById("btn_xspj_bc");
$(btn).data("enter", "1");
```

注入后，后续无论是 `evaluate` 直接 `btn.click()` 还是 WebBridge `click` 工具，都不会再触发警告。

## 对提交按钮同样适用

提交按钮 `#btn_xspj_tj` 共享同一段 `mouseenter` 监听代码，同样需要注入标记：

```javascript
$("#btn_xspj_tj").data("enter", "1");
```
