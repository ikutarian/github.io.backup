---
title: Spring的事务传播性笔记
date: 2020-01-24 23:17:52
tags:
  - Spring
  - 事务
  - 传播性
categories:
  - Spring
---

在 Spring 中，对于事务的传播性，有 7 种：

- REQUIRED
- SUPPORTS
- MANDATORY
- REQUIRES_NEW
- NOT_SUPPORTED
- NEVER
- NESTED

<!-- more -->

# 例子

为了更好的理解，举一个对学生表（`student`）插入 3 个员工的例子

员工表的定义如下

```sql
CREATE TABLE `student` (
	`id`   INT(11) NOT NULL AUTO_INCREMENT,
	`name` VARCHAR(50) NOT NULL,
	`age`  INT(11) NOT NULL,
	PRIMARY KEY (`id`)
)
```

测试代码如下

```java

import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.ikutarian.mapper.StudentMapper;
import com.ikutarian.pojo.Student;
import com.ikutarian.service.StudentService;
import org.springframework.stereotype.Service;

@Service
public class StudentServiceImpl extends ServiceImpl<StudentMapper, Student> implements StudentService {

    /**
     * 保存 3 个学生的信息
     */
    @Override
    public void saveStudents() {
        saveStudentOne();
        saveOtherStudent();
    }

    /**
     * 保存编号 1 的学生
     */
    public void saveStudentOne() {
        Student student = new Student();
        student.setName("no-1");
        student.setAge(1);
        this.save(student);
    }

    /**
     * 保存其他学生
     */
    public void saveOtherStudent() {
        saveStudentTwo();
        int a = 1 / 0;
        saveStudentThree();
    }

    /**
     * 保存编号 2 的学生
     */
    public void saveStudentTwo() {
        Student student = new Student();
        student.setName("no-other-2");
        student.setAge(2);
        this.save(student);
    }

    /**
     * 保存编号 3 的学生
     */
    public void saveStudentThree() {
        Student student = new Student();
        student.setName("no-other-3");
        student.setAge(3);
        this.save(student);
    }
}
```

插入 3 个员工信息调用的是 `saveStudents()` 方法，这个方法会调用 `saveStudentOne()` 和 `saveOtherStudent()` 方法保存编号 1 和编号 2、3 的学生信息。`saveOtherStudent()` 方法再分别调用 2 个方法去保存编号 2 和编号 3 的学生信息。并且 `saveOtherStudent()` 方法会抛出除零的运行时异常（`java.lang.ArithmeticException: / by zero`)

# 不开启事务

不开启事务，直接运行 `saveStudents()` 方法，因为在 `saveOtherStudent()` 中会抛出异常，所以只可以插入编号 1 和 2 两条数据

{% asset_img Snipaste_2020-01-25_11-53-52.png %}

# 开启事务

在 Spring 中开启事务，只需要给方法加上 `@Transactional` 注解即可。`@Transactional` 中关于传播类型的定义如下，可以看到默认的事务传播类型是 `Propagation.REQUIRED`

```java
@Target({ElementType.METHOD, ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Inherited
@Documented
public @interface Transactional {

  // 省略...

	/**
	 * The transaction propagation type.
	 * <p>Defaults to {@link Propagation#REQUIRED}.
	 * @see org.springframework.transaction.interceptor.TransactionAttribute#getPropagationBehavior()
	 */
	Propagation propagation() default Propagation.REQUIRED;

  // 省略...
}
```

# REQUIRED

`REQUIRED` 是默认的事务传播类型，定义如下

```java
/**
  * Support a current transaction, create a new one if none exists.
  * Analogous to EJB transaction attribute of the same name.
  * <p>This is the default setting of a transaction annotation.
  */
REQUIRED(TransactionDefinition.PROPAGATION_REQUIRED),
```

意思是说：

> 如果没有事务，就开启事务，有事务就使用当前的事务

## 只给 `saveStudents()` 方法加上 `@Transactional` 注解

```java
/**
  * 保存 3 个学生的信息
  */
@Transactional(propagation = Propagation.REQUIRED)
@Override
public void saveStudents() {
    saveStudentOne();
    saveOtherStudent();
}

// 省略...
```

因为 `saveOtherStudent()` 方法会抛出异常，所以数据是无法插入到数据库的

{% asset_img Snipaste_2020-01-25_12-03-39.png %}

## 只给 `saveOtherStudent()` 方法加上 `@Transactional` 注解

```java
// 省略...

/**
 * 保存其他学生
 */
@Transactional(propagation = Propagation.REQUIRED)
public void saveOtherStudent() {
    saveStudentTwo();
    int a = 1 / 0;
    saveStudentThree();
}

// 省略...
```

只有 `saveOtherStudent()` 方法开启了事务，而 `saveOtherStudent()` 因为开启了事务并且抛出了异常，造成事务回滚。这时候应该是只会插入编号 1 的数据




什么情况下@Transactional注解会不起作用

https://blog.csdn.net/m0_37779570/article/details/81352587
https://www.ibm.com/developerworks/cn/java/j-master-spring-transactional-use/index.html
https://segmentfault.com/a/1190000014617571