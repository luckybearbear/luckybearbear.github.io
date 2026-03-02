---
publish: true
aliases: ""
created: 2026-01-29T14:17:33.334+08:00
modified: 2026-03-02T13:42:38.459+08:00
cssclasses: ""
---


## 概述

本文档介绍如何在ECharts中配置双Y轴图表，使左右Y轴的分割线完美对齐。

## 核心问题

**现象**：双Y轴的分割线不对齐，看起来很混乱。

**原因**：
- 左右Y轴的分段数（splitNumber）不同
- 最大值不是分段数的整数倍
- 间隔计算不一致

## 解决方案

### 1. 统一分段数

```java
int splitNumber = 5;  // 左右Y轴使用相同的分段数

// 左Y轴
Map<String, Object> yAxisLeft = new HashMap<>();
yAxisLeft.put("splitNumber", splitNumber);

// 右Y轴
Map<String, Object> yAxisRight = new HashMap<>();
yAxisRight.put("splitNumber", splitNumber);
```

### 2. 调整最大值为分段数的倍数

```java
/**
 * 调整最大值为能被指定数字整除的数值
 */
private BigDecimal adjustToDivisible(BigDecimal value, int divisor) {
    if (value.compareTo(BigDecimal.ZERO) == 0) {
        return BigDecimal.ZERO;
    }

    // 向上取整
    value = value.setScale(0, RoundingMode.UP);

    // 调整为divisor的倍数
    BigDecimal bdDivisor = new BigDecimal(divisor);
    BigDecimal remainder = value.remainder(bdDivisor);

    if (remainder.compareTo(BigDecimal.ZERO) != 0) {
        value = value.subtract(remainder).add(bdDivisor);
    }

    return value;
}
```

### 3. 完整配置示例

```java
// 计算最大值
BigDecimal maxActualHours = barData.stream()
    .max(BigDecimal::compareTo)
    .orElse(BigDecimal.ZERO);
BigDecimal maxEfficiency = lineData.stream()
    .max(BigDecimal::compareTo)
    .orElse(BigDecimal.ZERO);

int splitNumber = 5;

// 左Y轴：实际工时
Map<String, Object> yAxisLeft = new HashMap<>();
yAxisLeft.put("type", "value");
yAxisLeft.put("name", "实际工时");
yAxisLeft.put("position", "left");
yAxisLeft.put("min", 0);
yAxisLeft.put("splitNumber", splitNumber);
yAxisLeft.put("splitLine", createMap("show", true, "lineStyle", createMap("color", "#eee")));

if (maxActualHours.compareTo(BigDecimal.ZERO) > 0) {
    BigDecimal maxLeft = maxActualHours.multiply(new BigDecimal("1.1"));
    maxLeft = adjustToDivisible(maxLeft, splitNumber);
    yAxisLeft.put("max", maxLeft);
    BigDecimal intervalLeft = maxLeft.divide(new BigDecimal(splitNumber), 2, RoundingMode.HALF_UP);
    yAxisLeft.put("interval", intervalLeft);
}

// 右Y轴：效率百分比
Map<String, Object> yAxisRight = new HashMap<>();
yAxisRight.put("type", "value");
yAxisRight.put("name", "效率(%)");
yAxisRight.put("position", "right");
yAxisRight.put("min", 0);
yAxisRight.put("splitNumber", splitNumber);
yAxisRight.put("splitLine", createMap("show", false));  // 不显示分割线，避免重复

if (maxEfficiency.compareTo(BigDecimal.ZERO) > 0) {
    BigDecimal maxRight = maxEfficiency.multiply(new BigDecimal("1.2"));
    maxRight = adjustToDivisible(maxRight, splitNumber);
    yAxisRight.put("max", maxRight);
    BigDecimal intervalRight = maxRight.divide(new BigDecimal(splitNumber), 2, RoundingMode.HALF_UP);
    yAxisRight.put("interval", intervalRight);
}
```

## 工作原理

### 示例

假设数据：

- 左Y轴最大工时：523
- 右Y轴最大效率：168

**调整过程**：

1. **左Y轴**：
   - 523 × 1.1 = 575.3
   - 向上取整 = 576
   - 调整为5的倍数 = 580
   - 间隔 = 580 / 5 = 116
   - 刻度：0, 116, 232, 348, 464, 580

2. **右Y轴**：
   - 168 × 1.2 = 201.6
   - 向上取整 = 202
   - 调整为5的倍数 = 205
   - 间隔 = 205 / 5 = 41
   - 刻度：0, 41, 82, 123, 164, 205

**结果**：两个Y轴都有6个刻度点，分割线完全对齐。

## 关键要点

✅ **统一splitNumber** - 左右Y轴使用相同的分段数

✅ **调整最大值** - 确保能被splitNumber整除

✅ **显式设置间隔** - 避免ECharts自动调整

✅ **右轴隐藏分割线** - 只显示左轴的分割线

## 常见问题

**Q：为什么右轴要隐藏分割线？**
A：左右轴的分割线位置相同，显示右轴会造成重复，视觉混乱。

**Q：splitNumber设置多少合适？**
A：一般设置为5或6，过多会导致刻度密集，过少会导致精度不足。

**Q：如果最大值已经是5的倍数怎么办？**
A：`adjustToDivisible`方法会检查余数，如果是0则不调整。
