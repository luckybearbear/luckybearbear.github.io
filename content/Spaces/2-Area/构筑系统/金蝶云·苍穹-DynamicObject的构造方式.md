---
publish: true
aliases: ""
created: 2026-02-12T10:14:07.145+08:00
modified: 2026-03-02T13:42:50.164+08:00
cssclasses: ""
---


 ```java
// 第一种
BusinessDataServiceHelper.newDynamicObject(entityName, true, option);

// 第二种
DynamicFormModelProxy modelProxy = new DynamicFormModelProxy(entityName, UUID.randomUUID().toString(), new HashMap());
modelProxy.createNewData();

// 第三种
DynamicObjectType dynamicObjectType = EntityMetadataCache.getDataEntityType(PxtcOperationConst.DT);
new DynamicObject(DynamicObjectType dt) 
