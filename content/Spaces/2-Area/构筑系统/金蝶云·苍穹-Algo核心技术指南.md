---
publish: true
aliases: ""
created: 2026-01-23
modified: 2026-01-23
cssclasses: ""
---


### 一、FilterInfo 标准化处理框架

#### 1.1 过滤条件命名规范

```Java
/**
 * 统一的过滤条件处理机制
 * - 快速查询：字段名 + "_fast" （模糊搜索，支持多字段OR查询）
 * - 通用查询：字段名 + "_coft" （精确查询，支持各种比较操作）
 * - 基础资料查询：字段名 + "_coft.id" （引用对象查询，使用compareType）
 */
private List<QFilter> processFilterInfoToQFilters(FilterInfo filterInfo) {
    List<QFilter> filters = new ArrayList<>();
    
    if (filterInfo == null) return filters;
    
    // 处理快速过滤条件（模糊搜索）
    processFastFilters(filters, filterInfo.getFastFilter());
    
    // 处理自定义过滤条件（精确查询）
    processCustomFilters(filters, filterInfo.getFilterItems());
    
    return filters;
}
```

#### 1.2 快速查询实现模式

```Java
    private void processFastFilters(List<QFilter> filters, FastFilter fastFilter) {
        if (fastFilter == null || fastFilter.getQFilters() == null || fastFilter.getQFilters().isEmpty()) {
        return;
        }

        // 遍历处理所有的快速查询QFilter
        List<QFilter> fastQFilters = fastFilter.getQFilters();
        for (int i = 0; i < fastQFilters.size(); i++) {
        QFilter fastQFilter = fastQFilters.get(i);
        Object filterValue = fastQFilter.getValue();

        if (filterValue == null) {
        LOG.debug("快速查询QFilter[{}]的值为空，跳过处理", i);
        continue;
        }

        String filterValueStr = String.valueOf(filterValue);
        LOG.debug("快速查询QFilter[{}]原始值：{}", i, filterValueStr);

        // 解析快速查询值：格式为 "字段名列表#搜索值"
        if (!filterValueStr.contains("#")) {
        LOG.warn("快速查询QFilter[{}]值格式不正确，期望格式为'字段名列表#搜索值'，实际值：{}", i, filterValueStr);
        continue;
        }

        String[] parts = filterValueStr.split("#", 2);
        if (parts.length != 2) {
        LOG.warn("快速查询QFilter[{}]值分割失败，期望2个部分，实际：{}个", i, parts.length);
        continue;
        }

        String fieldNamesStr = parts[0];  // 字段名列表
        String searchValue = parts[1];    // 搜索值

        if (fieldNamesStr.isEmpty() || searchValue.isEmpty()) {
        LOG.warn("快速查询QFilter[{}]字段名或搜索值为空，字段名：{}，搜索值：{}", i, fieldNamesStr, searchValue);
        continue;
        }

        // 解析字段名列表并替换为数据库字段路径
        String[] fieldNames = fieldNamesStr.split(",");
        List<String> mappedFields = new ArrayList<>();

        for (String fieldName : fieldNames) {
        fieldName = fieldName.trim();
        if (!fieldName.isEmpty()) {
        // 映射字段名到数据库字段路径
        String mappedField = mapFastFilterField(fieldName);
        if (!mappedField.equals(fieldName)) { // 只有映射成功的字段才添加
        mappedFields.add(mappedField);
        LOG.debug("QFilter[{}]字段映射：{} → {}", i, fieldName, mappedField);
        }
        }
        }

        if (!mappedFields.isEmpty()) {
        // 重新构建字段名列表字符串
        String mappedFieldsStr = String.join(",", mappedFields);
        String newFilterValue = mappedFieldsStr + "#" + searchValue;

        // 创建新的QFilter，使用替换后的字段名
        QFilter newQFilter = new QFilter(fastQFilter.getProperty(), fastQFilter.getCP(), newFilterValue);
        filters.add(newQFilter);

        LOG.info("快速查询QFilter[{}]字段替换完成：{} → {}，搜索值：{}", i, fieldNamesStr, mappedFieldsStr, searchValue);
        } else {
        LOG.warn("快速查询QFilter[{}]没有有效的字段，原始字段列表：{}", i, fieldNamesStr);
        }
        }
        }
```

#### 1.3 基础资料查询映射

```Java
private QFilter createMappedFilter(FilterItemInfo filterItem) {
    String propName = filterItem.getPropName();
    Object value = filterItem.getValue();
    String compareType = filterItem.getCompareType(); // 重要：使用前端传递的比较方式
    
    switch (propName) {
        // 基础资料查询（支持 equals、in、like 等多种比较方式）
        case "project_info_coft.id":
            return new QFilter("pxtc_product.pxtc_project", compareType, value);
        case "supplier_info_coft.id":
            return new QFilter("pxtc_supplier", compareType, value);
        
        // 日期查询（支持 >=、<=、between 等比较方式）
        case "order_date_coft":
            return new QFilter("pxtc_work_order.createtime", compareType, value);
            
        default:
            LOG.warn("未知的过滤属性：{}", propName);
            return null;
    }
}
```

### 二、DataSet 高级操作技术

#### 2.1 智能JOIN策略

```Java
/**
 * 动态JOIN策略：根据过滤条件智能选择JOIN类型
 * - 有过滤条件：INNER JOIN（提高查询效率）
 * - 无过滤条件：LEFT JOIN（保证数据完整性）
 */
private DataSet performOptimizedJoin(DataSet leftDataSet, DataSet rightDataSet,
                                   String[] selectFields, String operationName, boolean useInnerJoin) {
    long startTime = System.currentTimeMillis();
    
    JoinType joinType = useInnerJoin ? JoinType.INNER : JoinType.LEFT;
    DataSet result = leftDataSet.join(rightDataSet, joinType)
            .on("partId", "partId")  // 支持多字段关联：.on("field1", "field1").on("field2", "field2")
            .select(selectFields)
            .finish();
    
    long duration = System.currentTimeMillis() - startTime;
    String joinTypeName = useInnerJoin ? "INNER JOIN" : "LEFT JOIN";
    LOG.debug("{} ({}) 完成，关联记录数：{}，耗时：{}ms", 
              operationName, joinTypeName, result.size(), duration);
    
    return result;
}
```

#### 2.2 复杂聚合计算

```Java
/**
 * 自定义聚合函数：支持分组拼接、去重、统计等复杂场景
 */
// 1. 分组字符串拼接（带分隔符）
.agg(new CustomAggFunction<Object>("groupConcat", DataType.StringType) {
    public String newAggValue() {
        return "";
    }
    
    public String addValue(Object current, Object newItem) {
        if (newItem == null) return String.valueOf(current);
        
        String currentStr = String.valueOf(current);
        String newStr = String.valueOf(newItem);
        return StringUtils.isEmpty(currentStr) ? newStr : 
               StringUtils.join(new Object[]{currentStr, newStr}, ";");
    }
    
    public String combineAggValue(Object value1, Object value2) {
        return addValue(value1, value2);
    }
    
    public String getResult(Object value) {
        return String.valueOf(value);
    }
}, "sourceField", "targetField")

// 2. 去重分组拼接
.agg(new GroupConcatWithDistinctFunction("distinctConcat", DataType.StringType), 
     "sourceField", "distinctTargetField")

// 3. 条件聚合统计
.agg("sum", "CASE WHEN status = '已完成' THEN amount ELSE 0 END", "completedAmount")
.agg("count", "CASE WHEN priority = '高' THEN 1 ELSE NULL END", "highPriorityCount")
```

#### 2.3 动态字段计算

```Java
/**
 * 复杂业务逻辑的SQL公式计算
 */
// 按时/延迟状态计算（多条件嵌套CASE）
String currentDate = KDDateFormatUtils.getDateTimeFormat().format(KDDateUtils.now());
String onTimeDelayFormula = "CASE " +
    "WHEN billstatus != '" + BusinessConstants.COMPLETED_STATUS + "' THEN " +
    "  CASE WHEN TO_DATE('" + currentDate + "') > pxtc_finish_time THEN '0' ELSE '1' END " +
    "ELSE " +
    "  CASE WHEN pxtc_act_end_time > pxtc_finish_time THEN '0' ELSE '1' END " +
    "END";

// 含税价格计算（空值处理 + 数学运算）
String taxPriceFormula = "ROUND(" +
    "(CASE WHEN pxtc_material_price IS NULL THEN 0 ELSE pxtc_material_price END) * " +
    BusinessConstants.TAX_RATE.toString() + ", 2)";

// 距离交期天数计算
String daysToDeadlineFormula = "CEIL((" +
    "CASE WHEN pxtc_finish_time IS NULL THEN 0 " +
    "ELSE (pxtc_finish_time - TO_DATE('" + currentDate + "')) / 86400 END" +
    ") * 1.0)";

fields.put(onTimeDelayFormula, "onTimeDelayFlag");
fields.put(taxPriceFormula, "materialPriceWithTax");
fields.put(daysToDeadlineFormula, "daysToDeadline");
```

### 三、高级查询与数据处理

#### 3.1 标准查询模式

```Java
/**
 * 标准的报表查询模式：过滤 → 查询 → 映射
 */
private DataSet queryBusinessData(List<QFilter> filters, Map<String, String> fieldMapping) {
    // 1. 构建过滤条件（添加默认过滤）
    List<QFilter> allFilters = new ArrayList<>(filters);
    allFilters.add(new QFilter("billstatus", QCP.equals, BusinessConstants.APPROVED_STATUS));
    
    // 2. 构建查询字段（通过字段映射）
    String selectFields = buildSelectSql(fieldMapping);
    
    // 3. 执行查询
    return QueryServiceHelper.queryDataSet(
        this.getClass().getName(),
        BUSINESS_ENTITY_TYPE,
        selectFields,
        allFilters.toArray(new QFilter[]{}),
        "createtime desc"  // 默认排序
    );
}

/**
 * 字段映射构建器
 */
private String buildSelectSql(Map<String, String> fieldMapping) {
    return fieldMapping.entrySet().stream()
        .map(entry -> entry.getKey() + " AS " + entry.getValue())
        .collect(Collectors.joining(", "));
}
```

#### 3.2 分层数据获取架构

```Java
/**
 * 企业级报表的分层查询架构模式
 */
@Override
public DataSet query(ReportQueryParam reportQueryParam, Object context) throws Throwable {
    long startTime = System.currentTimeMillis();
    LOG.info("开始执行{}报表查询", getReportName());
    
    try {
        FilterInfo filterInfo = reportQueryParam.getFilter();
        
        // 1. 处理过滤条件
        List<QFilter> baseFilters = processFilterInfoToQFilters(filterInfo);
        List<QFilter> detailFilters = processDetailFilters(filterInfo);
        
        // 2. 分层数据查询
        DataSet baseData = queryBaseData(baseFilters);           // 主表数据
        DataSet detailData = queryDetailData(detailFilters);     // 明细数据
        DataSet summaryData = querySummaryData(baseFilters);     // 汇总数据
        DataSet referenceData = queryReferenceData();           // 参考数据
        
        // 3. 数据关联与计算
        DataSet result = buildCompleteDataSet(baseData, detailData, summaryData, referenceData);
        
        // 4. 最终处理
        result = applyFinalCalculations(result);
        result = applySorting(result, reportQueryParam.getSortInfo());
        
        long duration = System.currentTimeMillis() - startTime;
        LOG.info("{}报表查询完成，返回记录数：{}，耗时：{}ms", 
                getReportName(), result.size(), duration);
        
        return result;
        
    } catch (Exception e) {
        LOG.error("{}报表查询失败", getReportName(), e);
        throw new ReportException("报表查询异常：" + e.getMessage(), e);
    }
}
```

### 四、DataSet 数据操作高级技巧

#### 4.1 DataSet 合并与连接

```Java
/**
 * DataSet 高级合并操作
 */
// 1. 同结构数据合并（垂直合并）
DataSet merged = dataSet1.union(dataSet2).union(dataSet3);

// 2. 不同结构数据关联（水平合并）
String[] selectFields = Stream.concat(
    Arrays.stream(leftFields),
    Arrays.stream(rightFields)
).toArray(String[]::new);

DataSet joined = leftDataSet.leftJoin(rightDataSet)
    .on("primaryKey", "foreignKey")
    .on("secondaryKey", "secondaryKey")  // 多字段关联
    .select(selectFields)
    .finish();

// 3. 复杂关联条件
DataSet complexJoined = leftDataSet.leftJoin(rightDataSet)
    .on("orgId", "orgId")
    .where("left.createDate >= right.effectiveDate")  // 自定义关联条件
    .select(selectFields)
    .finish();
```

#### 4.2 DataSet 动态字段计算与重建

```Java
/**
 * DataSet 字段值动态计算和重建
 */
private DataSet rebuildDataSetWithCalculations(DataSet sourceDataSet) {
    DataSet copyDataSet = sourceDataSet.copy();
    DataSetBuilder builder = Algo.create(this.getClass().getName())
        .createDataSetBuilder(copyDataSet.getRowMeta());
    
    String[] fieldNames = copyDataSet.getRowMeta().getFieldNames();
    
    // 预计算汇总值
    Map<String, BigDecimal> summaryValues = calculateSummaryValues(copyDataSet.copy());
    
    while (copyDataSet.hasNext()) {
        Row row = copyDataSet.next();
        List<Object> valueList = new ArrayList<>(fieldNames.length);
        
        for (int i = 0; i < fieldNames.length; i++) {
            String fieldName = fieldNames[i];
            Object originalValue = row.get(i);
            
            switch (fieldName) {
                case "calculatedAmount":
                    // 复杂计算逻辑
                    BigDecimal unitPrice = row.getBigDecimal("unitPrice");
                    Integer quantity = row.getInteger("quantity");
                    BigDecimal taxRate = row.getBigDecimal("taxRate");
                    
                    BigDecimal amount = (unitPrice != null && quantity != null) ?
                        unitPrice.multiply(new BigDecimal(quantity)) : BigDecimal.ZERO;
                    
                    if (taxRate != null) {
                        amount = amount.multiply(BigDecimal.ONE.add(taxRate));
                    }
                    
                    valueList.add(amount);
                    break;
                    
                case "percentageOfTotal":
                    // 占比计算
                    BigDecimal currentAmount = row.getBigDecimal("amount");
                    BigDecimal totalAmount = summaryValues.get("totalAmount");
                    
                    if (currentAmount != null && totalAmount != null && totalAmount.compareTo(BigDecimal.ZERO) > 0) {
                        BigDecimal percentage = currentAmount.divide(totalAmount, 4, RoundingMode.HALF_UP)
                            .multiply(new BigDecimal(100));
                        valueList.add(percentage);
                    } else {
                        valueList.add(BigDecimal.ZERO);
                    }
                    break;
                    
                case "statusDescription":
                    // 状态转换
                    String statusCode = row.getString("statusCode");
                    String description = getStatusDescription(statusCode);
                    valueList.add(description);
                    break;
                    
                default:
                    // 保持原值
                    valueList.add(originalValue);
                    break;
            }
        }
        
        builder.append(valueList.toArray());
    }
    
    return builder.build();
}

/**
 * 预计算汇总值以提高性能
 */
private Map<String, BigDecimal> calculateSummaryValues(DataSet dataSet) {
    Map<String, BigDecimal> summaryValues = new HashMap<>();
    BigDecimal totalAmount = BigDecimal.ZERO;
    
    while (dataSet.hasNext()) {
        Row row = dataSet.next();
        BigDecimal amount = row.getBigDecimal("amount");
        if (amount != null) {
            totalAmount = totalAmount.add(amount);
        }
    }
    
    summaryValues.put("totalAmount", totalAmount);
    return summaryValues;
}
```

### 五、性能优化与监控

#### 5.1 查询性能优化策略

```Java
/**
 * 多层次性能优化策略
 */
public class PerformanceOptimizedReportPlugin extends AbstractReportListDataPlugin {
    
    // 1. 查询字段最小化
    private Map<String, String> getOptimizedFieldMapping(Set<String> requiredFields) {
        return fieldMapping.entrySet().stream()
            .filter(entry -> requiredFields.contains(entry.getValue()))
            .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue));
    }
    
    // 2. 分批查询大数据集
    private DataSet queryLargeDataSet(List<QFilter> filters, int batchSize) {
        List<DataSet> batches = new ArrayList<>();
        
        // 分页查询
        int offset = 0;
        DataSet batch;
        do {
            String limitSql = String.format("SELECT * FROM (%s) WHERE ROWNUM BETWEEN %d AND %d",
                baseSql, offset + 1, offset + batchSize);
                
            batch = QueryServiceHelper.queryDataSet(
                this.getClass().getName(),
                entityType,
                limitSql,
                filters.toArray(new QFilter[]{}),
                null
            );
            
            if (batch.size() > 0) {
                batches.add(batch);
            }
            
            offset += batchSize;
            
        } while (batch.size() == batchSize);
        
        // 合并批次数据
        return batches.stream().reduce(DataSet::union).orElse(null);
    }
    
    // 3. 查询结果缓存
    private final Map<String, CachedResult> queryCache = new ConcurrentHashMap<>();
    
    private DataSet getCachedQuery(String cacheKey, Supplier<DataSet> querySupplier) {
        CachedResult cached = queryCache.get(cacheKey);
        
        if (cached != null && !cached.isExpired()) {
            LOG.debug("命中查询缓存：{}", cacheKey);
            return cached.getResult();
        }
        
        DataSet result = querySupplier.get();
        queryCache.put(cacheKey, new CachedResult(result, Duration.ofMinutes(10)));
        
        return result;
    }
    
    // 4. 执行时间监控
    private <T> T monitorExecution(String operationName, Supplier<T> operation) {
        long startTime = System.currentTimeMillis();
        
        try {
            T result = operation.get();
            long duration = System.currentTimeMillis() - startTime;
            
            LOG.info("{} 执行完成，耗时：{}ms", operationName, duration);
            
            // 性能告警
            if (duration > PERFORMANCE_THRESHOLD) {
                LOG.warn("{} 执行耗时过长：{}ms，建议优化", operationName, duration);
            }
            
            return result;
            
        } catch (Exception e) {
            long duration = System.currentTimeMillis() - startTime;
            LOG.error("{} 执行失败，耗时：{}ms", operationName, duration, e);
            throw e;
        }
    }
}

/**
 * 缓存结果包装类
 */
private static class CachedResult {
    private final DataSet result;
    private final long expireTime;
    
    public CachedResult(DataSet result, Duration duration) {
        this.result = result.copy(); // 防止外部修改
        this.expireTime = System.currentTimeMillis() + duration.toMillis();
    }
    
    public boolean isExpired() {
        return System.currentTimeMillis() > expireTime;
    }
    
    public DataSet getResult() {
        return result.copy(); // 返回副本
    }
}
```

#### 5.2 内存使用优化

```Java
/**
 * 内存使用优化策略
 */
// 1. 及时释放大型DataSet
private void processLargeDataSet(DataSet largeDataSet) {
    try {
        // 处理逻辑
        while (largeDataSet.hasNext()) {
            Row row = largeDataSet.next();
            // 处理单行数据
            processRow(row);
        }
    } finally {
        // 显式释放资源
        if (largeDataSet != null) {
            largeDataSet.close();
        }
    }
}

// 2. 流式处理避免内存溢出
private DataSet streamProcessLargeResult(DataSet sourceData) {
    DataSetBuilder builder = Algo.create(this.getClass().getName())
        .createDataSetBuilder(createResultRowMeta());
    
    // 分块处理
    List<Row> batch = new ArrayList<>(BATCH_SIZE);
    
    while (sourceData.hasNext()) {
        batch.add(sourceData.next());
        
        if (batch.size() >= BATCH_SIZE) {
            processBatch(batch, builder);
            batch.clear(); // 及时清理
        }
    }
    
    // 处理剩余数据
    if (!batch.isEmpty()) {
        processBatch(batch, builder);
    }
    
    return builder.build();
}
```

### 六、错误处理与日志

#### 6.1 完善的异常处理体系

```Java
/**
 * 分层异常处理策略
 */
public class ReportException extends Exception {
    private final String errorCode;
    private final Object[] parameters;
    
    public ReportException(String errorCode, String message, Object… parameters) {
        super(message);
        this.errorCode = errorCode;
        this.parameters = parameters;
    }
}

// 统一异常处理
private DataSet executeQueryWithErrorHandling(String operationName, Supplier<DataSet> querySupplier) {
    try {
        return querySupplier.get();
        
    } catch (SQLException e) {
        LOG.error("{}数据库查询失败：{}", operationName, e.getMessage(), e);
        throw new ReportException("DB_QUERY_ERROR", 
            "数据库查询失败：" + e.getMessage(), operationName);
            
    } catch (OutOfMemoryError e) {
        LOG.error("{}内存不足：{}", operationName, e.getMessage(), e);
        throw new ReportException("MEMORY_ERROR", 
            "查询数据量过大，请缩小查询范围", operationName);
            
    } catch (TimeoutException e) {
        LOG.error("{}查询超时：{}", operationName, e.getMessage(), e);
        throw new ReportException("TIMEOUT_ERROR", 
            "查询超时，请优化查询条件", operationName);
            
    } catch (Exception e) {
        LOG.error("{}未知异常：{}", operationName, e.getMessage(), e);
        throw new ReportException("UNKNOWN_ERROR", 
            "系统异常：" + e.getMessage(), operationName);
    }
}
```

#### 6.2 结构化日志体系

```Java
/**
 * 结构化日志记录
 */
private static final Log LOG = LogFactory.getLog(ReportPlugin.class);

// 1. 业务日志
LOG.info("报表查询开始 - 报表名称：{}，用户：{}，查询条件：{}", 
         getReportName(), getCurrentUser(), getFilterSummary(filterInfo));

// 2. 性能日志
LOG.info("查询性能统计 - 操作：{}，记录数：{}，耗时：{}ms，内存使用：{}MB",
         operationName, recordCount, duration, getMemoryUsage());

// 3. 错误日志
LOG.error("报表查询异常 - 报表：{}，用户：{}，错误：{}，堆栈：{}", 
          getReportName(), getCurrentUser(), e.getMessage(), getStackTrace(e));

// 4. 调试日志
LOG.debug("SQL执行详情 - 语句：{}，参数：{}，执行计划：{}", 
          sql, Arrays.toString(parameters), getExecutionPlan());
```

## 🏆 企业级报表开发最佳实践

### 实战案例：复杂制造业报表系统

基于实际的**模具工单零件明细报表**开发经验，总结企业级报表开发的核心模式：

#### 1. 架构设计模式

```Java
/**
 * 企业级报表标准架构：分层查询 + 数据关联 + 业务计算
 */
@Override
public DataSet query(ReportQueryParam reportQueryParam, Object context) throws Throwable {
    // 🎯 第一层：过滤条件预处理
    FilterInfo filterInfo = reportQueryParam.getFilter();
    List<QFilter> baseFilters = processFilterInfoToQFilters(filterInfo);
    List<QFilter> detailFilters = processDetailFilters(filterInfo);
    
    // 🎯 第二层：分模块数据获取
    DataSet partBaseData = queryPartBaseInfo(baseFilters);          // 零件基础信息
    DataSet operationData = queryOperationBatchInfo(detailFilters);  // 工件分配信息
    DataSet costData = queryManufacturingCostInfo();               // 制造成本
    DataSet outsourceRequestData = queryOutsourceRequestInfo();    // 委外申请
    DataSet outsourcePurchaseData = queryOutsourcePurchaseInfo();  // 委外采购
    
    // 🎯 第三层：智能数据关联
    DataSet result = buildIntegratedDataSet(
        partBaseData, operationData, costData, 
        outsourceRequestData, outsourcePurchaseData, detailFilters
    );
    
    // 🎯 第四层：业务计算与优化
    return applyBusinessCalculations(result);
}
```

#### 2. 核心技术要点总结

|   |   |   |
|---|---|---|
|技术领域|关键技术|应用场景|
|**过滤处理**|FilterInfo标准化、动态映射|支持快速查询、精确查询、基础资料查询|
|**数据关联**|智能JOIN、多表关联|根据过滤条件选择INNER/LEFT JOIN|
|**聚合计算**|CustomAggFunction、分组统计|复杂字符串拼接、去重汇总、条件统计|
|**性能优化**|分层查询、缓存策略、监控告警|大数据量处理、查询优化、资源管控|
|**业务计算**|动态SQL公式、多条件逻辑|财务计算、状态判断、KPI指标|
|**错误处理**|分层异常、结构化日志|故障诊断、性能分析、运维支持|

#### 3. 开发流程与规范

**📋 开发检查清单**

- 需求分析：明确业务场景、数据源、性能要求
- 架构设计：设计分层查询、字段映射、关联策略
- 过滤实现：实现FilterInfo标准处理机制
- 查询优化：字段最小化、索引优化、执行监控
- 数据关联：智能JOIN选择、多表关联优化
- 业务计算：复杂逻辑实现、性能验证
- 异常处理：完善错误处理、日志记录
- 性能测试：大数据量验证、并发测试
- 文档完善：技术文档、使用说明

**🎯 质量标准**

- **性能标准**：单次查询 < 5秒，并发支持 > 100用户
- **可维护性**：代码复杂度控制、模块解耦、注释完整
- **扩展性**：支持新字段、新计算逻辑、新数据源
- **稳定性**：异常覆盖率 > 95%，错误恢复机制完善

## 💡 常见问题与解决方案

### Q1: FilterInfo 处理中的常见陷阱

**问题**：过滤条件映射不正确，导致查询结果异常 **解决方案**：

```Java
// ✅ 正确做法：统一映射管理
private String mapFilterField(String fieldName) {
    return FILTER_FIELD_MAPPING.getOrDefault(fieldName, fieldName);
}

// ❌ 错误做法：硬编码映射
if ("project_name_fast".equals(fieldName)) {
    return "pxtc_product.pxtc_project.name";
}
```

### Q2: DataSet 内存泄漏问题

**问题**：大量DataSet操作导致内存溢出 **解决方案**：

```Java
// ✅ 正确做法：及时释放资源
try (DataSet dataSet = queryLargeData()) {
    return processData(dataSet);
} // 自动关闭资源

// ❌ 错误做法：资源未释放
DataSet dataSet = queryLargeData();
return processData(dataSet); // 可能导致内存泄漏
```

### Q3: JOIN操作性能问题

**问题**：多表JOIN查询过慢 **解决方案**：

```Java
// ✅ 优化策略：智能JOIN + 字段最小化
boolean hasDetailFilter = !detailFilters.isEmpty();
String[] minimalFields = getRequiredFields(reportConfig);

DataSet result = performOptimizedJoin(
    baseData, detailData, minimalFields, 
    "main_join", hasDetailFilter  // 根据过滤条件选择JOIN类型
);
```

## 🔧 BOS-Algo 7.0 核心技术框架

### 七、DataSet 缓存与构建机制

#### 7.1 DataSet 缓存策略

```Java
/**
 * 1. 基础缓存操作
 */
// 创建缓存DataSet
CacheHint hint = new CacheHint();
hint.setTimeout(60 * 60 * 1000); // 设置超时时间：60分钟（默认30分钟）
CachedDataSet cachedDataSet = dataSet.cache(hint);

// 获取缓存ID
String cachedId = cachedDataSet.getCachedId();

// 通过ID获取缓存DataSet
CachedDataSet retrievedDataSet = Algo.getCachedDataSet(cachedId);

// 按范围遍历缓存数据
Iterator<Row> iterator = cachedDataSet.iterator(int begin, int length);
// 或者
List<Row> list = cachedDataSet.getList(int begin, int length);

// 关闭缓存
cachedDataSet.close();
```

#### 7.2 Cache Builder 模式

```Java
/**
 * 2. 可追加的缓存构建器
 * 适用于需要动态添加数据行的场景
 */
CacheHint hint = new CacheHint();
hint.setTimeout(60 * 60 * 1000);

CachedDataSet.Builder cacheBuilder = DataSet.createCacheBuilder(hint);

// 动态添加数据行
cacheBuilder.append(row1);
cacheBuilder.append(row2);
cacheBuilder.appendAll(rowList);

// 构建最终的缓存DataSet
CachedDataSet finalCachedDataSet = cacheBuilder.build();

// 继续添加数据（构建后仍可追加）
cacheBuilder.append(newRow);
```

#### 7.3 内存DataSet构建器

```Java
/**
 * 3. DataSetBuilder：内存中构造DataSet
 */
// 定义行结构
RowMeta rowMeta = new RowMeta(new FieldMeta[]{
    new FieldMeta("id", DataType.StringType),
    new FieldMeta("name", DataType.StringType),
    new FieldMeta("amount", DataType.BigDecimalType)
});

// 创建构建器
DataSetBuilder builder = Algo.createDataSetBuilder(rowMeta);

// 添加数据行
builder.append(new Object[]{"001", "产品A", new BigDecimal("100.50")});
builder.append(new Object[]{"002", "产品B", new BigDecimal("200.75")});

// 构建DataSet
DataSet dataset = builder.build();
```

### 八、Input 数据源接口体系

#### 8.1 并行数据获取

```Java
/**
 * 4. Input接口：多数据源并行查询
 */
// 创建多个Input实现并行查询（默认4个并行）
int parallelCount = 8;
OrmInput[] inputs = new OrmInput[parallelCount];

for (int i = 0; i < parallelCount; i++) {
    // 将过滤条件分片，支持并行执行
    QFilter[] partitionFilters = partitionFilters(allFilters, i, parallelCount);
    
    inputs[i] = new OrmInput(
        algoKey,           // 算法键
        entityName,        // 实体名称
        selectFields,      // 查询字段
        partitionFilters   // 分片过滤条件
    );
}

// 并行执行查询
DataSet parallelResult = Algo.createDataSet(inputs);
```

#### 8.2 Input 类型详解

```Java
/**
 * Input的4个主要实现类
 */
// 1. OrmInput：ORM查询
OrmInput ormInput = new OrmInput(algoKey, entityName, selectFields, filters);

// 2. DbInput：直接数据库查询
DbInput dbInput = new DbInput(algoKey, sql, parameters);

// 3. OqlInput：OQL查询语言
OqlInput oqlInput = new OqlInput(algoKey, oqlStatement, parameters);

// 4. CollectInput：集合数据输入
CollectInput collectInput = new CollectInput(algoKey, collection);

// 5. 自定义Input
public class CustomReportInput implements CustomizedInput {
    @Override
    public DataSet createDataSet() {
        // 自定义数据获取逻辑
        return buildCustomDataSet();
    }
}
```

### 九、表达式计算引擎

#### 9.1 表达式语法支持

```Java
/**
 * 5. 计算表达式特性总览
 */
// 基础四则运算
"famount * 2 as amount"
"famount + fqty as total"  
"famount / fqty as unit_price"
"(famount + qty) * 2 as calculated_amount"

// 逻辑表达式
"famount > fqty as is_greater"
"famount > fqty and fqty > 100 as complex_condition"

// 宏和常量
"null as empty_value"
"true as is_active" 
"false as is_disabled"
"100 as constant_amount"
"'ABC' as constant_name"

// 字符串操作
"fname + 'ABC' as concatenated_name"

// Case When语句（两种形式）
"CASE status WHEN 'A' THEN '激活' WHEN 'I' THEN '停用' ELSE '未知' END as status_desc"
"CASE WHEN amount > 1000 THEN '大额' ELSE '小额' END as amount_level"

// In操作符
"status in ('A', 'P', 'C') as is_valid_status"

// Null判断
"amount is null as is_amount_null"
"name is not null as has_name"

// 参数化查询
"amount > ? and status = ?" // 通过Map传入参数值
```

#### 9.2 日期函数库

```Java
/**
 * 6. 日期函数完整支持
 */
// 当前日期时间
"NOW() as current_datetime"
"DATE() as current_date"

// 日期部分提取
"YEAR(create_date) as create_year"
"MONTH(create_date) as create_month" 
"DAY(create_date) as create_day"

// 日期构造
"DATE(2024, 12, 25) as christmas_date"

// 日期格式转换
"TO_DATE('2024-01-15', 'yyyy-MM-dd') as parsed_date"
"TO_CHAR(create_date, 'yyyy年MM月dd日') as formatted_date"

// 日期计算
"DATEADD(YEAR, 1, create_date) as next_year_date"    // 加一年
"DATEADD(MONTH, -3, create_date) as three_months_ago" // 减三个月
"DATEADD(DAY, 30, create_date) as thirty_days_later"  // 加30天

// 日期差值计算
"DATEDIF(start_date, end_date, 'Y') as years_diff"    // 年差
"DATEDIF(start_date, end_date, 'M') as months_diff"   // 月差
"DATEDIF(start_date, end_date, 'D') as days_diff"     // 日差
```

#### 9.3 字符串函数库

```Java
/**
 * 7. 字符串处理函数
 */
// 字符串连接
"CONCAT(first_name, ' ', last_name) as full_name"
"first_name + ' ' + last_name as full_name_alt"

// 字符串包含判断
"CONTAINS(description, '关键词') as contains_keyword"

// 前缀后缀判断
"STARTSWITH(product_code, 'PRD') as is_product_code"
"ENDSWITH(file_name, '.pdf') as is_pdf_file"

// 大小写转换
"UPPER(name) as upper_name"
"LOWER(email) as lower_email"

// 字符串长度
"LEN(description) as desc_length"

// 实战应用示例
"CASE WHEN LEN(phone) = 11 AND STARTSWITH(phone, '1') THEN '手机号' " +
"     WHEN LEN(phone) = 8 THEN '座机号' " +
"     ELSE '未知类型' END as phone_type"
```

### 十、聚合函数高级应用

#### 10.1 标准聚合函数

```Java
/**
 * 8. GroupBy中的聚合函数应用
 */
DataSet groupedResult = sourceDataSet
    .groupBy("department", "category")
    .agg()
    // 基础聚合
    .sum("amount")                                    // 求和
    .sum("quantity * unit_price", "total_value")      // 表达式求和
    .avg("score")                                     // 平均值
    .avg("amount / quantity", "avg_unit_price")       // 表达式平均值
    .max("create_date")                               // 最大值
    .max("amount + tax", "max_total_amount")          // 表达式最大值
    .min("update_date")                               // 最小值
    .count("record_count")                            // 计数
    .count()                                          // 等价于COUNT(*)
    // 特殊聚合：最值对应的属性
    .maxP("amount", "customer_name", "top_customer")   // 金额最大的客户
    .minP("create_date", "creator", "first_creator")   // 最早创建的人
    .finish();
```

#### 10.2 自定义聚合函数

```Java
/**
 * 9. 自定义聚合函数：复杂业务逻辑
 */
// 分组字符串拼接（带分隔符）
.agg(new CustomAggFunction<String>("group_concat", DataType.StringType) {
    @Override
    public String newAggValue() {
        return ""; // 初始值
    }
    
    @Override
    public String addValue(String current, Object newItem) {
        if (newItem == null) return current;
        
        String newStr = String.valueOf(newItem);
        return StringUtils.isEmpty(current) ? newStr : current + ";" + newStr;
    }
    
    @Override
    public String combineAggValue(String value1, String value2) {
        return addValue(value1, value2);
    }
    
    @Override
    public String getResult(String value) {
        return value;
    }
}, "description", "descriptions")

// 去重分组拼接
.agg(new GroupConcatWithDistinctFunction("distinct_concat", DataType.StringType), 
     "tag_name", "distinct_tags")

// 条件统计聚合
.agg("sum", "CASE WHEN status = '已完成' THEN amount ELSE 0 END", "completed_amount")
.agg("count", "CASE WHEN priority = '高' THEN 1 ELSE NULL END", "high_priority_count")
.agg("avg", "CASE WHEN rating > 0 THEN rating ELSE NULL END", "avg_rating")
```

### 十一、特殊函数与高级技巧

#### 11.1 行间计算函数

```Java
/**
 * 10. 特殊函数：行间数据引用
 */
// 获取上一行的当前列值
"preRowValue() as prev_value"

// 获取上一行指定字段的值
"preRowValue('amount') as prev_amount"

// 业务应用：计算环比增长率
"CASE WHEN preRowValue('amount') > 0 THEN " +
"    ROUND((amount - preRowValue('amount')) * 100.0 / preRowValue('amount'), 2) " +
"ELSE NULL END as growth_rate"

// 业务应用：累计计算
"amount + CASE WHEN preRowValue('cumulative_amount') IS NULL THEN 0 " +
"              ELSE preRowValue('cumulative_amount') END as cumulative_amount"
```

#### 11.2 综合实战示例

```Java
/**
 * 11. 复杂报表计算的完整示例
 */
DataSet complexCalculatedResult = baseDataSet
    .select(
        // 基础字段
        "order_id", "customer_name", "product_name", 
        "quantity", "unit_price", "order_date",
        
        // 金额计算
        "quantity * unit_price as subtotal",
        "quantity * unit_price * 1.13 as total_with_tax",
        
        // 日期计算  
        "YEAR(order_date) as order_year",
        "CONCAT(YEAR(order_date), 'Q', CEILING(MONTH(order_date) / 3.0)) as quarter",
        "DATEDIF(order_date, NOW(), 'D') as days_since_order",
        
        // 条件逻辑
        "CASE WHEN quantity > 100 THEN '大订单' " +
        "     WHEN quantity > 50 THEN '中订单' " +
        "     ELSE '小订单' END as order_size",
        
        // 字符串处理
        "UPPER(LEFT(customer_name, 1)) + LOWER(SUBSTRING(customer_name, 2)) as formatted_name",
        
        // 状态判断
        "CASE WHEN DATEDIF(order_date, NOW(), 'D') > 30 THEN '延期' " +
        "     WHEN status = '已发货' THEN '正常' " +
        "     ELSE '处理中' END as delivery_status",
        
        // 环比计算（需要按时间排序）
        "CASE WHEN preRowValue('total_amount') > 0 THEN " +
        "    ROUND((total_amount - preRowValue('total_amount')) * 100.0 / preRowValue('total_amount'), 2) " +
        "ELSE 0 END as month_over_month_growth"
    )
    .groupBy("customer_name", "quarter")
    .agg()
    .sum("subtotal", "total_subtotal")
    .sum("total_with_tax", "total_with_tax_sum")
    .count("order_count") 
    .avg("quantity", "avg_quantity")
    .maxP("subtotal", "product_name", "best_selling_product")
    .agg(new CustomAggFunction<String>("product_list", DataType.StringType) {
        public String newAggValue() { return ""; }
        public String addValue(String current, Object newItem) {
            String item = String.valueOf(newItem);
            return StringUtils.isEmpty(current) ? item : current + "," + item;
        }
        public String combineAggValue(String v1, String v2) { return addValue(v1, v2); }
        public String getResult(String value) { return value; }
    }, "product_name", "all_products")
    .finish();
```

---
