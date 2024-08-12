---
title: SpringBoot线程池和异步任务
date: 2024-04-20
tags: 
    - SprinBoot
categories: Summary
copyright: true
---

> 通用的SpringBoot项目下线程池的配置和使用，异步任务的创建和使用等。

<!--more-->

## :books: 1. 线程池配置

### 1.1. 参数配置

- resouces/appliccation.yml

    ``` yml
    async:
    executor:
        thread:
        keepAliveSeconds: 60
        namePrefix: async-service-
    ```

### 1.2. 开启线程池

- 从运行时拿Cpu核心数，提高可迁移性
- `baseServiceExecutor`用拒绝的策略，防止请求过载
- `downloadServiceExecutor`用等待的策略，保证所有下载线程都会被执行
- 若有多个线程池，最好定义一些Bean(name = "executorName")的线程池名字，方便后续使用`@Qualifier("executorName")`指定线程池

- config/ExecutorConfiguration.java

    ``` java
    package com.xxx.xxx.config;

    import lombok.Data;
    import org.springframework.boot.context.properties.ConfigurationProperties;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    import org.springframework.context.support.PropertySourcesPlaceholderConfigurer;
    import org.springframework.scheduling.annotation.EnableAsync;
    import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

    import java.util.concurrent.ThreadPoolExecutor;

    @Data
    @EnableAsync
    @Configuration
    @ConfigurationProperties(prefix = "async.executor.thread")
    public class ExecutorConfiguration {

        public static final int cpuNum = Runtime.getRuntime().availableProcessors();
        private int corePoolSize;
        private int maxPoolSize;
        private int queueCapacity;
        private int keepAliveSeconds;
        private String namePrefix;

        @Bean(name = "asyncBaseServiceExecutorConfigurer")
        public static PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer() {
            return new PropertySourcesPlaceholderConfigurer();
        }

        @Bean(name = "asyncBaseServiceExecutor")
        public ThreadPoolTaskExecutor baseServiceExecutor() {
            ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
            executor.setCorePoolSize(cpuNum + 1);
            executor.setMaxPoolSize((cpuNum + 1) * 2);
            executor.setQueueCapacity(cpuNum * 20);
            executor.setKeepAliveSeconds(keepAliveSeconds);
            executor.setThreadNamePrefix(namePrefix + "base");
            executor.setRejectedExecutionHandler(new ThreadPoolExecutor.AbortPolicy());
            executor.initialize();
            return executor;
        }

        @Bean(name = "asyncDownloadServiceExecutor")
        public ThreadPoolTaskExecutor downloadServiceExecutor() {
            ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
            executor.setCorePoolSize(cpuNum * 2);
            executor.setMaxPoolSize(cpuNum * 4);
            executor.setQueueCapacity(cpuNum * 20);
            executor.setKeepAliveSeconds(keepAliveSeconds);
            executor.setThreadNamePrefix(namePrefix + "download");
            executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
            executor.initialize();
            return executor;
        }
    }
    ```

- 启动类增加支持Async的注解

    ``` java
    @EnableAsync
    ```

- 用到线程池的服务接口增加注解，并标记使用哪个线程池

    ``` java
    @Async("asyncBaseServiceExecutor")
    ```

  - `Method annotated with @Async should return 'void' or "Future-like" type`，异步调用不能有返回值或者要用带`Future`的返回，可以用`CompletableFuture<T>`的结构封装原来的返回类型`T`

## :satellite: 2. 接口中的异步任务改写

### 2.1. 引入线程池和注入

- controller/BaseController.java

    ``` java
    private final ThreadPoolTaskExecutor baseServiceExecutor;

    @Autowired
    public BaseController(XXX xxx, @Qualifier("asyncBaseServiceExecutor") ThreadPoolTaskExecutor baseServiceExecutor) {
        this.xxx = xxx;
        this.baseServiceExecutor = baseServiceExecutor;
    }
    ```

### 2.2. 接口内多任务的异步编排

- 接口请求成功马上返回结果，可以是查询的结果也可以是空，根据需求；如果需要返回的是多线程服务的查询，return的Result也要用Future结构封装
- 执行异步的下载任务，防止阻塞接口的响应
- `CompletableFuture`的`runasync()`和`supplyasync()`的区别
  - `runasync()`方法接收一个`Runnable`类型的参数，不会返回任何结果
  - `supplyasync()`方法接收一个`Supplier`类型的参数，需要返回结果
- `CompletableFuture`的`thenCompose()`和`thenApply()`的区别
  - `thenCompose()`用来连接两个`CompletableFuture`，是生成一个新的`CompletableFuture`
  - `thenApply()`转换的是泛型中的类型，是同一个`CompletableFuture`

``` java
@RequestMapping("/******")
private Result<String> listByTime(@RequestParam(required = false) String startTime,
                                  @RequestParam(required = false) String endTime) {
    log.info("Select all log base by time");
    // If the endTime is null, set it to the current time
    endTime = Optional.ofNullable(endTime)
                .orElseGet(() -> LocalDateTime.now().format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")));

    String finalEndTime = endTime;
    AtomicLong DownloadNums = new AtomicLong(0);
    long startCostTime = System.currentTimeMillis();
    CompletableFuture.runAsync(() -> {
        try {
            List<xxx> logs = BaseService.listByTime(startTime, finalEndTime, pageNum, pageSize).get();
            DownloadNums.getAndAdd(progressService.progressLogsAndInitiateDownloa(logs, startTime, finalEndTime, domainIdMap).get().get());
        } catch (Exception e) {
            log.error("Error initiating log download", e);
        } finally {
            long endCostTime = System.currentTimeMillis();
            log.info("Download finished, total {} files,time cost: {} ms", DownloadNums, endCostTime - startCostTime);
        }
    }, baseServiceExecutor);

    return Result.success("Download is in progress in the background");
}
```

## :mag_right: 3. 并行下载请求

- 网络请求下载任务要关注一些异常的捕获，直接抛出将漏掉部分下载内容
- 为了统计下载任务总数，用了原子类来计数`AtomicLong`，和使用`CountDownLatch`统计预估下载总线程数量，再用`latch.countDown()`和`latch.wait()`等待所有线程结束统一返回

``` java
package com.***.***.utils;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;
import org.springframework.stereotype.Component;

import java.io.File;
import java.io.IOException;
import java.security.NoSuchAlgorithmException;
import java.util.AbstractMap;
import java.util.Map;
import java.util.Set;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.atomic.AtomicLong;

@Component
public class GetObject {

    private static ThreadPoolTaskExecutor downloadServiceExecutor;

    @Autowired
    private void setThreadPoolTaskExecutor(@Qualifier("asyncDownloadServiceExecutor") ThreadPoolTaskExecutor downloadServiceExecutor) {
        GetObject.downloadServiceExecutor = downloadServiceExecutor;
    }

    private static final Logger log = LoggerFactory.getLogger(GetObject.class);

    public static AtomicLong figDownload(Map<Integer, Set<AbstractMap.SimpleEntry<String, String>>> domainIdFileKeysResponse, String downloadSubName, Map<Integer, String> domainIdMap) {
        String pathName = "D:\\download\\" + downloadSubName + File.separator;
        File file = new File(pathName);
        if (!file.exists()) {
            boolean wasSuccessful = file.mkdirs();
            log.debug("Create directory {} {}", pathName, wasSuccessful ? "succeed" : "failed");
        }
        AtomicLong totalDownloadedFiles = new AtomicLong(0);
        int totalTasks = domainIdFileKeysResponse.values().stream().mapToInt(Set::size).sum();
        CountDownLatch latch = new CountDownLatch(totalTasks);
        try {
            for (Map.Entry<Integer, Set<AbstractMap.SimpleEntry<String, String>>> entry : domainIdFileKeysResponse.entrySet()) {
                Integer domainId = entry.getKey();
                String domainName = domainIdMap.get(domainId);
                File fileDomain = new File(pathName + domainId + domainName + File.separator);
                if (!fileDomain.exists()) {
                    boolean wasSuccessful = fileDomain.mkdirs();
                    log.debug("Create directory {}{}{} {}", pathName, domainId, domainName, wasSuccessful ? "succeed" : "failed");
                }
                Set<AbstractMap.SimpleEntry<String, String>> fileUrlResponses = entry.getValue();
                for (AbstractMap.SimpleEntry<String, String> fileUrlResponse : fileUrlResponses) {
                    downloadServiceExecutor.execute(() -> {
                        String responseData = fileUrlResponse.getValue();
                        String fileUrl = fileUrlResponse.getKey();
                        String downloadPathPre = pathName + domainId + domainName + File.separator;
                        try {
                            // download the picture
                            String sha256Name = ImageDownloader.downloadImgFromUrl(fileUrl, downloadPathPre);
                                if (sha256Name != null) {
                                    JsonDownloader.saveStringAsJson(responseData, downloadPathPre + sha256Name.split("\\.")[0] + ".json");
                                    totalDownloadedFiles.incrementAndGet();
                                }
                        }   catch (IOException e) {
                            log.error("Error Message:{}, URL:{}", e.getMessage(), fileUrl);
                        } catch (NoSuchAlgorithmException e) {
                            throw new RuntimeException(e);
                        } finally {
                            latch.countDown();
                        }
                    });
                }
            }
            latch.await();
        } catch (Exception e) {
            log.error("Error occurred: ", e);
        }
        return totalDownloadedFiles;
    }
}
```
