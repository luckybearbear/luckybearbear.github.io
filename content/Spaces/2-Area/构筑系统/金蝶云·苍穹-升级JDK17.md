---
publish: true
aliases: ""
created: 2025-12-19T11:50:43.277+08:00
modified: 2026-03-02T13:42:30.795+08:00
cssclasses: ""
---


# 本地开发环境升级

如果使用gradle版本较低，需要同步升级，如果需要保留之前的版本，建议通过环境变量来控制对应的版本，解压后配置到对应的路径即可（非必须，也可以通过idea直接改变）  
[[Windows配置多版本JDK]]

# 项目配置

1. 替换JDK
2. 修改启动设置与追加jvm变量

	```shell
	
	#移除了
	
	-XX:+PrintGCApplicationStoppedTime
	
	-XX:+PrintGCTimeStamps
	
	-XX:+UseAdaptiveSizePolicy 或 -XX:-UseAdaptiveSizePolicy
	
	-XX:G1LogLevel=finest
	
	-XX:+UseParallelOldGC
	
	#追加了
	
	--add-modules=ALL-SYSTEM --add-opens java.base/java.lang=ALL-UNNAMED --add-opens java.base/jdk.internal.reflect=ALL-UNNAMED --add-opens java.base/java.lang.reflect=ALL-UNNAMED --add-opens java.base/java.lang.ref=ALL-UNNAMED --add-opens java.base/sun.reflect.annotation=ALL-UNNAMED --add-opens java.base/sun.net.util=ALL-UNNAMED --add-opens java.base/java.util=ALL-UNNAMED --add-opens java.sql/java.sql=ALL-UNNAMED --add-opens java.base/java.lang.invoke=ALL-UNNAMED --add-opens java.base/java.math=ALL-UNNAMED --add-opens java.base/java.nio=ALL-UNNAMED --add-opens java.base/sun.nio.ch=ALL-UNNAMED --add-opens java.xml.crypto/com.sun.org.apache.xml.internal.security.utils=ALL-UNNAMED --add-opens java.xml/com.sun.xml.internal.stream=ALL-UNNAMED --add-opens java.xml/com.sun.xml.internal.stream.writers=ALL-UNNAMED --add-opens java.desktop/java.beans=ALL-UNNAMED --add-opens java.base/jdk.internal.misc=ALL-UNNAMED --add-opens java.base/java.util.concurrent.atomic=ALL-UNNAMED --add-opens java.base/java.util.concurrent=ALL-UNNAMED -Djava.locale.providers=COMPAT
	
	```

3. 更新苍穹资源包
4. Gradle升级至8.14.2，
	-  [[Spaces/2-Area/构筑系统/IntelliJ IDEA 下载 Gradle 慢的终极解决方案\|下载缓慢的解决方案]]
	 - [[Spaces/2-Area/构筑系统/Gradle 缓存从 C 盘迁移到 D 盘\|C盘空间紧张解决方案]] 