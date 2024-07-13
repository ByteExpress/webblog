## 简介
HikariCP是快速、简单、可靠且可用于生产的JDBC连接池。

在Spring Boot 2.0版本中，默认的数据库连接池技术已经从Tomcat池切换到HikariCP。HikariCP默认的连接为10，当我们需要自定义连接数时，往往需要通过监控连接池信息来判断配置的连接信息是否适合，从而调整到对于当前应用的最合理配置。

### 代码示例

``` java
import com.zaxxer.hikari.HikariDataSource;
import com.zaxxer.hikari.HikariPoolMXBean;
import lombok.extern.slf4j.Slf4j;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;
import java.util.Arrays;
import java.util.Set;
import java.util.stream.Collectors;

/**
 * 系统监控任务
 */
@Slf4j
@Component
public class SysMonitorJobHandler {
    @Resource
    private HikariDataSource dataSource;

    @PostConstruct
    public void monitorDbPool() {
        ScheduledExecutorService executor = Executors.newSingleThreadScheduledExecutor();
        executor.scheduleAtFixedRate(() -> {
            HikariPoolMXBean hikariPoolMXBean = dataSource.getHikariPoolMXBean();
            if (null != hikariPoolMXBean) {
                log.info("total:{}, active:{}, idle:{}, awaiting:{}", 
                        hikariPoolMXBean.getTotalConnections(),
                        hikariPoolMXBean.getActiveConnections(),
                        hikariPoolMXBean.getIdleConnections(),
                        hikariPoolMXBean.getThreadsAwaitingConnection());
            }
        }, 0, 1000, TimeUnit.MILLISECONDS);
    }
}
```

### 配置信息

### 
1. connectionTimeout
客户端从连接池等待连接的最大毫秒数:
```
spring.datasource.hikari.connection-timeout=20000 
```
2. minimumIdle
连接池中维护的最小空闲连接数:
```
spring.datasource.hikari.minimum-idle=5
```
3. maximumPoolSize
最大连接池数大小（服务启动后就会建立maximumPoolSize个连接）:
```
spring.datasource.hikari.maximum-pool-size=15
```
4. idleTimeout
客户端从连接池等待连接的最大毫秒数:
```
spring.datasource.hikari.connection-timeout=20000 
```
5. maxLifetime
连接池中连接关闭后的最长生存时间（以毫秒为单位），使用中的连接不会失效:
```
spring.datasource.hikari.max-lifetime=1200000 
```
6. autoCommit
从池中返回的连接的默认自动提交行为，默认为true:
```
spring.datasource.hikari.auto-commit=true 
```

