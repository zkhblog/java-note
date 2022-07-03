# 插件机制
1 MyBatis通过插件(Interceptor)可以做到拦截四大对象相关方法的执行，根据需要来完成相关数据的动态改变  
2 四大对象在各自创建时，都会执行interceptorChain.pluginAll()，会经过每个插件对象的plugin()方法，目的是为当前的四大对象创建代理  

