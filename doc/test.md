# Sentinel源码解析（一）: Entry的构建

## 一、Entry介绍
- Entry是sentinel中用来表示是资源是否可以继续执行一个凭证，可以理解为token。
- 使用SphU.entry()时，会返回一个entry，如果抛出了一个异常BlockException，说明资源被保护了。
- 使用 SphO.entry() 时，资源被保护了会返回false，反之true。


## 二、组成
[1.Sentinel之Entry构建源码解析](../sentinel-core/src/main/java/com/alibaba/csp/sentinel/CtEntry.java)

