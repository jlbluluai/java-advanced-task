# Table of Contents

* [Week5-作业-2](#week5-作业-2)
  * [题目](#题目)
  * [解题](#解题)
    * [方案1：通过xml配置bean](#方案1：通过xml配置bean)
    * [方案2：通过注解自动装配](#方案2：通过注解自动装配)
    * [方案3：通过Java代码配置](#方案3：通过java代码配置)

# Week5-作业-2

## 题目

> 写代码实现 Spring Bean 的装配，方式越多越好（XML、Annotation 都可以）

## 解题

[代码仓库](https://github.com/jlbluluai/xyz-study/tree/master/xyz-study-common/src/main/java/com/xyz/study/common/jike/week5/task2)

### 方案1：通过xml配置bean

核心配置
```
    <bean id="user"
          class="com.xyz.study.jike.week5.task2.User">
        <property name="id" value="10001"/>
        <property name="name" value="xyz"/>
    </bean>
```

### 方案2：通过注解自动装配

核心配置
```
@Service
```

```
<context:component-scan base-package="com.xyz.study.jike.week5.task2" />
```

### 方案3：通过Java代码配置

核心代码
```
@Configuration
public class BeanConfig {

    @Bean
    public Student student() {
        Student student = new Student();
        student.setId(1000001);
        student.setName("小叶子");
        return student;
    }
}
```
