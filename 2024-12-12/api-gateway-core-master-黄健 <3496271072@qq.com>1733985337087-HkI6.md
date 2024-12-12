# 评审项目： API 网关核心代码评审.

### 😀代码评分：85

#### 😀代码逻辑与目的：
代码片段展示了 `MapperMethod` 类的构造和执行方法。`MapperMethod` 用于将 HTTP 请求映射到具体的处理方法，并执行相应的操作。在执行阶段，根据不同的 HTTP 命令类型，从 `GatewaySession` 获取资源或执行操作。

#### 🎯代码优点：
- 使用反射来映射 HTTP 请求到具体的方法，增加了代码的灵活性。
- 使用枚举 `HttpCommandType` 来处理不同的 HTTP 命令，代码清晰。

#### 🤔问题点：
- `uri` 字段在修改后被删除，这可能导致 `session.get(uri, args)` 的调用错误，因为 `uri` 已经不再使用。
- `execute` 方法中直接使用 `session.get(methodName, args)` 可能不正确，因为 `args` 应该包含具体的参数，而不是整个请求对象。
- 在 `MapperProxy` 的 `intercept` 方法中，只取了 `args` 的第一个元素，这可能导致其他参数丢失。

#### 🎯修改建议：
- 将 `uri` 字段重命名为 `methodName`，以保持与 `execute` 方法中使用的变量一致。
- 修改 `execute` 方法，确保正确处理 `args` 参数。
- 在 `MapperProxy` 的 `intercept` 方法中，应该传递完整的 `args` 参数给 `MapperMethod`。

#### 💻修改后的代码：
```java
diff --git a/api-gateway-core-04/src/main/java/com/hj/gateway/binding/MapperMethod.java b/api-gateway-core-04/src/main/java/com/hj/gateway/binding/MapperMethod.java
index 8f974b6..9a3c468 100644
--- a/api-gateway-core-04/src/main/java/com/hj/gateway/binding/MapperMethod.java
+++ b/api-gateway-core-04/src/main/java/com/hj/gateway/binding/MapperMethod.java
@@ -17,7 +17,7 @@ import java.lang.reflect.Method;
  */
 public class MapperMethod {
 
-    private String uri;
+    private String methodName;
     private final HttpCommandType command;
 
     public MapperMethod(String uri, Method method, Configuration configuration) {
@@ -25,7 +25,7 @@ public class MapperMethod {
-        this.uri = uri;
+        this.methodName = configuration.getHttpStatement(uri).getMethodName();
         this.command = configuration.getHttpStatement(uri).getHttpCommandType();
     }
 
     public Object execute(GatewaySession session, Object[] args) {
@@ -33,7 +33,7 @@ public class MapperMethod {
         switch (command) {
             case GET:
-                result = session.get(uri, args);
+                result = session.get(methodName, args);
                 break;
             case POST:
                 break;
diff --git a/api-gateway-core-04/src/main/java/com/hj/gateway/binding/MapperProxy.java b/api-gateway-core-04/src/main/java/com/hj/gateway/binding/MapperProxy.java
index ca479e5..b0b9c39 100644
--- a/api-gateway-core-04/src/main/java/com/hj/gateway/binding/MapperProxy.java
+++ b/api-gateway-core-04/src/main/java/com/hj/gateway/binding/MapperProxy.java
@@ -27,7 +27,7 @@ public class MapperProxy implements MethodInterceptor {
     @Override
     public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
         MapperMethod linkMethod = new MapperMethod(uri, method, gatewaySession.getConfiguration());
-        return linkMethod.execute(gatewaySession, args[0]);
+        return linkMethod.execute(gatewaySession, args);
     }
 
 }
```