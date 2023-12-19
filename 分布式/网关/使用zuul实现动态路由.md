```
https://mp.weixin.qq.com/s?__biz=MjM5OTI2NDMwMg==&mid=2247483694&idx=1&sn=4bd9c5865f18f25dd3f38e1f0a9e8701&chksm=a73f686f9048e17903787b98dda5d5c74bb165c27b44a44887e23b6f534d3a54c3b6a13f1d3b&token=1502955171&lang=zh_CN#rd
```

# 实现要点
① 网关启动时，动态路由的数据怎样加载进来  
② 静态路由与动态路由义哪个为准（静态路由指配置文件里写死的路由配置）  
③ 监听动态路由的数据源变化  
④ 数据有变化时怎样通知zuul刷新路由

# 具体实现
### 1、实现动态路由的数据加载
```java
public abstract class AbstractDynRouteLocator extends SimpleRouteLocator implements RefreshableRouteLocator {
    private ZuulProperties properties;
    
    public AbstractDynRouteLocator(String servletPath, ZuulProperties zuulProperties) {
        super(servletPath, zuulProperties);
        this.properties = zuulProperties;
    }
    
    @Override
    public void refresh() {
        doRefresh();
    }
    
    @Override
    protected Map<String, ZuulRoute> locateRoutes() {
        LinkedHashMap<String, ZuulRoute> routesMap = new LinkedHashMap();
        // 从application.properties中加载静态路由信息
        routesMap.putAll(super.locateRoutes());
        // 从数据源中加载动态路由信息
        routesMap.putAll(loadDynamicRoute());
        
        // 优化一下配置
        LinkedHashMap<String, ZuulRoute> values = new LinkedHashMap();
        for (Map.Entry<String, ZuulRoute> entry : routesMap.entrySet()) {
            String path = entry.getKey();
            if (!path.startsWith("/")) {
                path = "/" + path;
            }
            if (StringUtils.hasText(this.properties.getPrefix())) {
                path = this.properties.getPrefix() + path;
                if (!path.startsWith("/")) {
                    path = "/" + path;
                }
            }
            values.put(path, entry.getValue());
        }
        return values;
    }
    
    // 加载路由配置，由子类去实现
    public abstract Map<String, ZuulRoute> loadDynamicRoute();
}
```

### 2、实现```loadDynamicRoute```方法获取动态数据
```
@Override
public Map<String, ZuulProperties.ZuulRoute> loadDynamicRoute() {
    Map<String, ZuulRoute> routes = new LinkedHashMap<>();
    if (zuulRouteEntities == null) {
        zuulRouteEntities = getNacosConfig();
    }
    
    for (ZuulRouteEntity result : zuulRouteEntities) {
        if (StrUtl.isBlank(result.getPaht()) || !result.isEnabled()) {
            continue;
        }
        
        ZuulRoute zuulRoute = new ZuulRoute();
        BeanUtil.copyProperties(result, zuulRoute);
        routes.put(zuulRoute.getPath(), zuulRoute);
    }
    return routes;
}

private List<ZuulRouteEntity> getNacosConfig() {
    try {
        String content = nacosConfigProperties.configServiceInstance().getConfig(ZUUL_DATA_ID, ZUUL_GROUP_ID,5000);
        return getListByStr(content);
    } catch (NacosException e) {
        log.error("listenerNacos-error", e);
    }
    return new ArrayList<>(0);
}
```

### 3、增加```NacosListener```监听路由数据变化
```
private void addListener() {
    try {
        nacosConfigProperties.configServiceInstance().addListener(ZUUL_DATA_ID, ZUUL_GROUP_ID, new Listener() {
            @Override
            public Executor getExecutor() {
                return null;
            }

            @Override
            public void receiveConfigInfo(String configInfo) {
                //赋值路由信息
                locator.setZuulRouteEntities(getListByStr(configInfo));
                RoutesRefreshedEvent routesRefreshedEvent = new RoutesRefreshedEvent(locator);
                publisher.publishEvent(routesRefreshedEvent);
            }
        });
    } catch(NacosException e) {
        log.error("nacos-addListener-error", e);
    }
}
```