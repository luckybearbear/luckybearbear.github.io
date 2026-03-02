---
publish: true
aliases: ""
created: 2026-01-29T14:15:32.044+08:00
modified: 2026-03-02T13:42:52.646+08:00
cssclasses: ""
---


## 概述

本文档介绍在金蝶云苍穹平台上使用ECharts的基础知识，包括如何通过`IClientViewProxy`设置图表数据。

## 核心技术方案

### 为什么使用IClientViewProxy？

**问题**：在`afterBindData`中设置图表，平台后续会调用`bindData`方法覆盖配置。

**解决方案**：在`afterQuery`中使用`IClientViewProxy`设置图表属性。

```java
@Override
public void afterQuery(ReportQueryParam queryParam) {
    super.afterQuery(queryParam);

    // 构建ECharts配置
    Map<String, Object> echartsOption = new HashMap<>();
    // … 配置内容 …

    // 关键：使用IClientViewProxy设置，不会被覆盖
    IClientViewProxy clientViewProxy = this.getView().getService(IClientViewProxy.class);
    clientViewProxy.setFieldProperty("pxtc_customchartap", "data", echartsOption);
}
```

### 基础配置结构

```java
Map<String, Object> echartsOption = new HashMap<>();

// 1. 提示框
Map<String, Object> tooltipMap = new HashMap<>();
tooltipMap.put("trigger", "axis");
echartsOption.put("tooltip", tooltipMap);

// 2. 图例
Map<String, Object> legendMap = new HashMap<>();
legendMap.put("data", Arrays.asList("系列1", "系列2"));
echartsOption.put("legend", legendMap);

// 3. X轴
List<Map<String, Object>> xAxisList = new ArrayList<>();
Map<String, Object> xAxis = new HashMap<>();
xAxis.put("type", "category");
xAxis.put("data", Arrays.asList("A", "B", "C"));
xAxisList.add(xAxis);
echartsOption.put("xAxis", xAxisList);

// 4. Y轴
List<Map<String, Object>> yAxisList = new ArrayList<>();
Map<String, Object> yAxis = new HashMap<>();
yAxis.put("type", "value");
yAxisList.add(yAxis);
echartsOption.put("yAxis", yAxisList);

// 5. 系列
List<Map<String, Object>> seriesList = new ArrayList<>();
Map<String, Object> series = new HashMap<>();
series.put("type", "bar");
series.put("data", Arrays.asList(10, 20, 30));
seriesList.add(series);
echartsOption.put("series", seriesList);
```

## 辅助方法

```java
/**
 * 创建Map的辅助方法
 */
private Map<String, Object> createMap(Object… keyValuePairs) {
    Map<String, Object> map = new HashMap<>();
    for (int i = 0; i < keyValuePairs.length; i += 2) {
        if (i + 1 < keyValuePairs.length) {
            map.put(String.valueOf(keyValuePairs[i]), keyValuePairs[i + 1]);
        }
    }
    return map;
}
```

## 关键要点

✅ **在afterQuery中设置** - 避免被覆盖

✅ **使用Map对象** - 平台自动序列化为JSON

✅ **遵循ECharts配置规范** - 与原生ECharts配置一致

✅ **设置容器样式** - 确保图表铺满

## 参考资源

- [ECharts官方文档](https://echarts.apache.org/zh/option.html)
- [如何基于苍穹实现不同类型的echart图形？](https://vip.kingdee.com/knowledge/413278405967467008?productLineId=29&isKnowledge=2&lang=zh-CN)
