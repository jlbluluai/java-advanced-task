# Table of Contents

* [Week5-作业-8](#week5-作业-8)
  * [题目](#题目)
  * [解题](#解题)
    * [实现自动配置](#实现自动配置)
    * [实现starter](#实现starter)

# Week5-作业-8

## 题目

> 给前面课程提供的 Student/Klass/School 实现自动配置和 Starter。

## 解题

[starter仓库](https://github.com/jlbluluai/xyz-study/tree/master/xyz-study-jike-starter-demo/src/main)

[测试引用starter代码仓库](https://github.com/jlbluluai/xyz-study/tree/master/xyz-study-common/src/main/java/com/xyz/study/common/jike/week5/task8)

### 实现自动配置

核心配置类
```
@Configuration
public class BeanConfiguration {

    @Resource
    private ApplicationContext applicationContext;

    /**
     * spring.auto.bean.kclass.enable 为 true时才加载bean
     */
    @ConditionalOnProperty(value = "spring.auto.bean.kclass.enable", havingValue = "true")
    @Bean
    public Klass class1() {
        Klass class1 = new Klass();
        class1.setStudents(Collections.singletonList((Student) applicationContext.getBean("student100")));
        return class1;
    }

    /**
     * 根据spring.auto.bean.student的配置属性自动注入
     */
    @ConfigurationProperties(prefix = "spring.auto.bean.student")
    @Bean(name = "student100")
    public Student student() {
        return new Student();
    }

    /**
     * class1 student100 bean均存在时才加载该bean
     */
    @ConditionalOnBean(name = {"class1", "student100"})
    @Bean
    public ISchool school() {
        return new School();
    }
}
```

application.yml
```
spring:
  auto:
    bean:
      kclass:
        enable: true
      student:
        id: 1
        name: xyz
```

spring.factories
```
# Auto Configuration
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.xyz.study.jike.BeanAutoConfiguration
```

### 实现starter

starter项目install

在引入项目pom下加入
```
        <dependency>
            <groupId>com.xyz.study</groupId>
            <artifactId>xyz-study-jike-starter-demo</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
```

测试代码
```
@SpringBootApplication
public class Week5Task8Demo {

    public static void main(String[] args) {
        ApplicationContext context = SpringApplication.run(Week5Task8Demo.class, args);

        // 获取starter中的bean进行相关的操作
        Student student = (Student) context.getBean("student100");
        System.out.println(student);

        Klass class1 = (Klass) context.getBean("class1");
        System.out.println(class1);

        ISchool school = (ISchool) context.getBean(ISchool.class);
        System.out.println(school);
    }
}
```

结果
```
Student(id=1, name=xyz, beanName=student100, applicationContext=org.springframework.context.annotation.AnnotationConfigApplicationContext@dd0c991, started on Sun Jul 25 13:42:39 CST 2021)
Klass(students=[Student(id=1, name=xyz, beanName=student100, applicationContext=org.springframework.context.annotation.AnnotationConfigApplicationContext@dd0c991, started on Sun Jul 25 13:42:39 CST 2021)])
School(class1=Klass(students=[Student(id=1, name=xyz, beanName=student100, applicationContext=org.springframework.context.annotation.AnnotationConfigApplicationContext@dd0c991, started on Sun Jul 25 13:42:39 CST 2021)]), student100=Student(id=1, name=xyz, beanName=student100, applicationContext=org.springframework.context.annotation.AnnotationConfigApplicationContext@dd0c991, started on Sun Jul 25 13:42:39 CST 2021))
```
