---
publish: true
aliases: ""
created: 2026-02-12
modified: 2026-03-12
cssclasses: ""
---


 ```java
// 第一种
BusinessDataServiceHelper.newDynamicObject(entityName, true, option);

// 第二种
DynamicFormModelProxy modelProxy = new DynamicFormModelProxy(entityName, UUID.randomUUID().toString(), new HashMap());
modelProxy.createNewData();

// 第三种
DynamicObject dynamicObject = new DynamicObject(EntityMetadataCache.getDataEntityType(PxtcItemConst.DT));
