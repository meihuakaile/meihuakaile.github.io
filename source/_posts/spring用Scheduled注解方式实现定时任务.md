---
title: spring用Scheduled注解方式实现定时任务
tags: ['spring注解', 'spring定时任务', 'Scheduled']
categories: [java, spring]
copyright: true
---
#####  1.spring配置文件中写：

    
    
    <!-spring扫描注解包的配置-->
    <context:component-scan base-package="XXX" />
    <!—开启这个配置，spring才能识别@Scheduled注解 -->
    <task:annotation-driven/>

#####  2.定时任务的类

    
    
    import lombok.extern.slf4j.Slf4j;
    /**
    * TestQuartz
    *
    * @author chenliclchen
    * @date 17-11-2 下午5:19
    */
    @Slf4j
    @Component
    public class TestQuartz {
     
        @Scheduled(cron = "*/1 0 0 * * ?")
        public void everyMinute(){
            log.info("everyMinute");
        }
        @Scheduled(cron = "0 0 19 * * ?")
        public void hours(){
            log.info("hours");
        }
    }

这样项目启动之后，每分钟会输出“everyMinute”，每天19点输出“hours”。  

