---
publish: true
aliases: ""
created: 2026-01-29
modified: 2026-01-29
cssclasses: ""
---


## 概述

当X轴数据点较多时，可以通过添加滚动条功能，让用户可以滑动查看所有数据。

## 配置方法

### 基本配置

```java
// 8. 数据缩放配置（实现滚动）
if (xAxisData.size() > 10) {
    List<Map<String, Object>> dataZoomList = new ArrayList<>();

    Map<String, Object> dataZoom = new HashMap<>();
    dataZoom.put("type", "slider");          // 类型：滑动条
    dataZoom.put("show", true);              // 显示滚动条
    dataZoom.put("xAxisIndex", 0);           // 作用在第一个X轴
    dataZoom.put("start", 0);                // 起始位置：0%
    dataZoom.put("end", Math.min(10 * 100 / xAxisData.size(), 100));  // 结束位置
    dataZoom.put("height", 20);              // 滚动条高度
    dataZoom.put("bottom", 10);              // 距离底部距离
    dataZoomList.add(dataZoom);

    echartsOption.put("dataZoom", dataZoomList);
}
```

### 参数说明

| 参数 | 说明 | 示例值 |
|------|------|--------|
| type | 缩放类型 | "slider"（滑动条）或 "inside"（鼠标滚轮） |
| show | 是否显示 | true |
| xAxisIndex | 作用的X轴索引 | 0（第一个X轴） |
| start | 窗口起始位置 | 0（0%） |
| end | 窗口结束位置 | 根据数据量动态计算 |
| height | 滚动条高度 | 20（像素） |
| bottom | 距底部距离 | 10（像素） |

### 动态计算显示范围

```java
// 初始显示前10个数据
int initialShowCount = 10;
int endPercent = Math.min(initialShowCount * 100 / xAxisData.size(), 100);
dataZoom.put("end", endPercent);
```

**示例**：
- 总共20个人员：end = 10 × 100 / 20 = 50（显示50%）
- 总共15个人员：end = 10 × 100 / 15 = 67（显示67%）
- 总共8个人员：end = min(80, 100) = 80（显示80%，但不会启用滚动条）

## 同时支持鼠标滚轮缩放

```java
if (xAxisData.size() > 10) {
    List<Map<String, Object>> dataZoomList = new ArrayList<>();

    // 1. 滑动条
    Map<String, Object> sliderZoom = new HashMap<>();
    sliderZoom.put("type", "slider");
    sliderZoom.put("show", true);
    sliderZoom.put("xAxisIndex", 0);
    sliderZoom.put("start", 0);
    sliderZoom.put("end", Math.min(10 * 100 / xAxisData.size(), 100));
    dataZoomList.add(sliderZoom);

    // 2. 鼠标滚轮缩放
    Map<String, Object> insideZoom = new HashMap<>();
    insideZoom.put("type", "inside");
    insideZoom.put("xAxisIndex", 0);
    insideZoom.put("zoomOnMouseWheel", true);
    dataZoomList.add(insideZoom);

    echartsOption.put("dataZoom", dataZoomList);
}
```

## 调整网格布局

添加滚动条后，需要调整grid配置，为滚动条留出空间：

```java
Map<String, Object> gridMap = new HashMap<>();
gridMap.put("left", "3%");
gridMap.put("right", "4%");
gridMap.put("top", "15%");
gridMap.put("bottom", "15%");  // 为X轴标签和滚动条留出空间
gridMap.put("containLabel", true);
echartsOption.put("grid", gridMap);
```

## 效果演示

```shell
没有滚动条（≤10个数据）：
┌─────────────────────────────┐
│  A  B  C  D  E  F  G  H  I  J  │
│  █  █  █  █  █  █  █  █  █  █  │
└─────────────────────────────┘

有滚动条（>10个数据）：
┌─────────────────────────────┐
│  A  B  C  D  E  F  G  H  I  J  │  ← 只显示前10个
│  █  █  █  █  █  █  █  █  █  █  │
├─────────────────────────────┤
│ ─────●───────◯───────────●─── │  ← 滚动条
└─────────────────────────────┘
    滚动可查看：K L M N O …
```

## 关键要点

✅ **设置触发条件** - 数据点超过10个时才启用

✅ **合理设置初始范围** - 默认显示前10个数据

✅ **调整bottom距离** - 为滚动条预留空间

✅ **可选支持滚轮** - 提升用户体验

## 扩展功能

### 自定义滚动条样式

```java
dataZoom.put("backgroundColor", "#eee");        // 背景色
dataZoom.put("fillerColor", "rgba(80, 135, 236, 0.2)");  // 选中区域颜色
dataZoom.put("handleStyle", createMap(
    "color", "#5087EC",                         // 滑块颜色
    "borderColor", "#5087EC"                   // 边框颜色
));
```

### 锁定缩放

```java
dataZoom.put("zoomLock", true);  // 只能平移，不能缩放
```

## 常见问题

**Q：滚动条不显示？**
A：检查数据量是否超过阈值，确保`dataZoom`被添加到配置中。

**Q：滚动条位置不对？**
A：调整`bottom`参数，确保grid的bottom值足够大。

**Q：初始显示范围不对？**
A：检查`start`和`end`的计算逻辑，确保end值不超过100。
