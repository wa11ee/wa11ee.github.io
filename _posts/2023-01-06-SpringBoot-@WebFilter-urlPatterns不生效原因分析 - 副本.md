---
layout: post
title: SpringBoot @WebFilter urlPatterns不生效原因分析                             # Title of the page
hide_title: false                                  # Hide the title when displaying the post, but shown in lists of posts
excerpt_separator: <!--more-->
#feature-img: "assets/img/sample.png"              # Add a feature-image to the post
#thumbnail: "assets/img/avatar.png"  # Add a thumbnail image on blog view
#color: rgb(80,140,22)                             # Add the specified color as feature image, and change link colors in post
bootstrap: true                                   # Add bootstrap to the page
tags: [java]
---
todo 
<!--more-->
#### _利用枚举构建不同优先级线程池_
```
public enum ThreadPoolInstances implements Consumer {
    HIGH_LEVEL(10, new ThreadPoolExecutor.CallerRunsPolicy()),
    DEFAULT(5, new ThreadPoolExecutor.AbortPolicy());

    private final ThreadPoolExecutor executor;

    ThreadPoolInstances(int coreSize, RejectedExecutionHandler handler) {
        this.executor = new ThreadPoolExecutor(Math.max(coreSize, 1), 50, 300L, TimeUnit.SECONDS,
                new LinkedBlockingQueue<>(300), handler);
    }

    @Override
    public void consumer(Runnable runnable) {
        this.executor.submit(runnable);
    }
}
```
