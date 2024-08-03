# Spring Data JPA

JPA(Java Persistence API) 是规范，Hibernate 实现了该规范。Spring Data JPA 依赖于 Hibernate

[官方文档](https://spring.io/projects/spring-data-jpa)	|	[Spring Data JPA API 文档](https://docs.spring.io/spring-data/data-jpa/docs/current/api/)



## 配置

POM

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

application-{profile}.yml

```properties
spring:
  datasource:
    url: jdbc:mysql://localhost:3307/sleeve?characterEncoding=utf-8&serverTimezone=Asia/Shanghai
    username: root
    password: root
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:    
	hibernate:
      ddl-auto: update  # DDL模式，Data Definition Language
# DDL模式定义了Hibernate在启动时如何处理数据库和表的结构，可以自动创建、更新、验证
# 默认情况，使用嵌入式数据库且无schema manager则为create-drop，否则为None
# validate - 每次加载hibernate时，验证创建数据库表结构，只会和数据库中的表进行比较，不会创建新表，但是会插入新值。(推荐)
# update - 每次加载hibernate时，执行必要的修改操作使数据库表与实体类的定义保持一致，它不会删除数据库已存在但实体类中未定义的列或表(最常用的，实体与数据库同一字段类型不一致则可能导致数据丢失)
# none - 不执行任何操作
# create-drop：每次加载hibernate时，根据model类生成表，但是sessionFactory一关闭,表就自动删除。(没用)
# create - 每次加载hibernate时，删除现有的数据库表(如果存在)，然后重新创建它们。这将导致数据丢失。(没用)
	show_sql: true
	properties:
      hibernate:        
        format_sql: true
```



## 注解



### Model

领域模型(Domain Model)

| 注解                                         | 元注解@Target | 释义                                                         | 成员变量 |
| -------------------------------------------- | ------------- | ------------------------------------------------------------ | -------- |
| @Entity                                      | 修饰实体类    | 修饰class，用来标识了该类是一个持久化的实体，会把成员变量持久化到数据库<br />修饰方法，把该方法返回的对象注册到IOC容器中<br /><br />如果你要用到第三方类库里面某个方法的时候，你就只能用@Bean把这个方法的对象注册到spring容器，因为用@Component你需要配置组件扫描到这个第三方类路径而且还要在别人源代码加上这个注解，很明显是不现实的。<br /><br />除了关联属性外，只能使用基础类型和数据的字段进行映射<br /><br />@Entity 注解的类的成员变量需要与数据库字段相对应，当数据库中存储的是JSON数据时，类成员变量使用对象会报错，可以使用String，或采取别的方式解决 |          |
| @Table(name = "banner1")                     | 修饰实体类    | 修饰entity类，改表名。如Order是JPA关键字<br />所以要使用`@Table(name = "`Order`")` |          |
| @Id                                          | 修饰成员变量  | import javax.persistence.Id;<br />用来标识成员变量对应对应数据库表中的主键 |          |
| @Column(length = 16)                         | 修饰成员变量  | 修饰成员变量，改字段长度                                     |          |
| @Transient                                   | 修饰成员变量  | 暂时的，不持久化到数据库                                     |          |
| @GeneratedValue(strategy=GenerationType.xxx) | 修饰成员变量  | 生成值的策略：<br />**AUTO** (默认)：值由程序控制，会多创建一个表<br />**SEQUENCE**：根据底层数据库的序列来生成主键，条件是数据库支持序列。<br />**IDENTITY**：自增<br />**TABLE**：使用一个特定的数据库表格来保存主键。 |          |
| @JsonIgnore                                  | 修饰成员变量  | 不序列化该字段，即不会在请求中返回该字段                     |          |



#### 审查

| 注解                 | 元注解@Target | 释义                                                         | 成员变量 |
| -------------------- | ------------- | ------------------------------------------------------------ | -------- |
| `@EnableJpaAuditing` | Type          | 启动 JPA 审查，提供以下注解的支持                            |          |
| @CreatedDate         |               | 声明该字段为创建时间字段。在这个实体被insert的时候，会设置值 |          |
| @CreatedBy           |               | 声明该字段为创建人。在这个实体被insert的时候，会设置值       |          |
| @LastModifiedDate    |               | 最后修改时间 实体被update的时候会设置                        |          |
| @LastModifiedBy      |               | 最后修改人 实体被update的时候会设置                          |          |



#### 关系

| 注解                                                         | 元注解@Target | 释义                                                         | 成员变量 |
| ------------------------------------------------------------ | ------------- | ------------------------------------------------------------ | -------- |
| **Bean层**                                                   |               | **修饰表明表间关系的导航属性（成员变量）**                   |          |
| 单向一对多：正向 (一到多)需要导航关系                        |               | 双向一对多：正向反向都需要导航关系                           |          |
| @OneToMany(mappedBy = "banner",fetch = FetchType.LAZY)       |               | 修饰**导航属性**，不使用@JoinColumn指明外键会创建第三张表存储关系，该注解用于实现关联查询<br /><br />fetch属性：LAZY (**懒加载**默认)，在使用该导航属性时才执行关联查询，用于节约SQL性能<br />为配合LAZY，Controller返回的应是VO对象<br />EAGER (**急加载**)<br /><br />mappedBy属性：指向**关系的维护端** (多端)的导航属性<br />关联查询查导航属性name |          |
| @JoinColumn(foreignKey = @ForeignKey(value = ConstraintMode.NO_CONSTRAINT), insertable = false, updatable = false, name = "bannerId") |               | 修饰**导航属性**，name属性**指向外键**，会在数据库中创建外键字段<br />在**单向一对多关系**中，修饰一端 (关系的被维护端)，外键在多端的表上。需要在一端创建导航属性，在多端创建外键字段bannerId。<br />在**双向一对多关系**中，修饰多端 (关系的维护端)，外键在多端的表上。需要在一端创建导航属性，在多端创建导航属性，不需要在多端创建外键属性bannerId (如果想有外键属性可以加上insertalbe/updatable属性)（双向一对多也可以不使用该注解，也会自动创建外键） |          |
| @org.hibernate.annotations.ForeignKey(name = "null")         |               | 修饰导航属性，禁止物理外键生成，用逻辑外键，落伍的方式       |          |
| @ManyToOne                                                   |               | 修饰**导航属性**，配合@OneToMany使用，实现双向一对多关系     |          |
|                                                              |               |                                                              |          |
| @ManyToMany(mappedBy = "spuList")                            |               | 表示多对多关系，会创建第三张表存储关系<br /><br />mappedBy属性：指向**关系的维护端** (多端)的导航属性 |          |
| @JoinTable(name = "theme_spu", joinColumns = @JoinColumn(name = "theme_id"),        inverseJoinColumns = @JoinColumn(name = "spu_id")) |               | 设置第三张表（theme_spu）命名规范<br />外键字段、反转外键字段 |          |



### DAO

Repository 持久层

| 注解                                    | 元注解@Target | 释义                                                         | 成员变量 |
| --------------------------------------- | ------------- | ------------------------------------------------------------ | -------- |
| @Query(value = "", nativeQuery = false) |               | 创建查询，编写JPQL语句，并通过类似 “:name” 来映射@Param指定的参数，就像例子中的第三个findUser函数一样。<br />nativeQuery 参数表示使用原生native SQL还是JPQL(Java Persistence Query Language)<br />例子：@Query("select t from Theme t where t.name in (:names)")<br />这里 Theme t ，类似于 sql 里的 as ，把实体简写成一个字母，这里是把Theme简写成t |          |
| @Param                                  |               | 例子：User getUser(@Param(“userId”)String userId,@Param(“password”)String password);<br/>import org.springframework.data.repository.query.Param; spring的注解，参数要按照先后顺序传入<br/>import org.apache.ibatis.annotations.Param; mybatis的注解，只需要使用参数名即可，更常用 |          |
| @Modifying                              |               | 对方法使用，更新插入删除的方法                               |          |



# 未整理



### entity 实体层

必须使用 @Entity 和 @Id 注解

#### IDEA 实体导出

从数据库导出实体类

![](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_19_12_22.png)

#### 双向一对多

```java
package com.zjbti.missyou.model_discard;

import javax.persistence.*;
import java.util.List;

/**
 * @date 2021/2/4 @Created by cxc
 */
@Entity
@Table(name = "banner")
public class Banner {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    @Column(length = 16)
    private String name;
    @Transient
    private String description;
    private String img;
    private String title;

    @OneToMany(mappedBy = "banner", fetch = FetchType.EAGER)
//    @JoinColumn(name = "bannerId")
//    @org.hibernate.annotations.ForeignKey(name = "null")
    private List<BannerItem> items;
}

```

```java
package com.zjbti.missyou.model_discard;

import javax.persistence.*;

/**
 * @date 2021/2/5 @Created by cxc
 */
@Entity
public class BannerItem {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String img;
    private String keyword;
    private Short type;
    private String name;

    private Long bannerId;

    @ManyToOne
    @JoinColumn(foreignKey = @ForeignKey(value = ConstraintMode.NO_CONSTRAINT), insertable = false, updatable = false, name = "bannerId")
    private Banner banner;
}

```

#### 双向多对多

小心循环序列化，造成内存泄露

解决循环序列化可以采用：1、使用VO 2、@JsonIgnore

```java
package com.zjbti.missyou.model_discard;

import javax.persistence.*;
import java.util.List;

/**
 * @date 2021/2/6 @Created by cxc
 */
@Entity
public class Theme {

    @Id
    private Long id;
    private String title;
    private String name;

    @ManyToMany
    @JoinTable(name = "theme_spu", joinColumns = @JoinColumn(name = "theme_id"),
            inverseJoinColumns = @JoinColumn(name = "spu_id"))
    private List<Spu> spuList;
}

```

```java
package com.zjbti.missyou.model_discard;

import javax.persistence.Entity;
import javax.persistence.Id;
import javax.persistence.ManyToMany;
import java.util.List;

/**
 * @date 2021/2/6 @Created by cxc
 */
@Entity
public class Spu {

    @Id
    private Long id;
    private String title;
    private String subtitle;

    @ManyToMany(mappedBy = "spuList")
    private List<Theme> themeList;
}

```

#### VO 层

查询出来的对象需要经过一层VO对象的封装(拷贝给VO对象)，才能再传给前端。**Java项目每一层都是一次封装**

VO：value object 是从服务端返回到前端的数据对象

自己的项目一般直接返回model实体层的数据，因为model对应数据库中的数据不一定都是你需要的，避免多余数据的序列化，可提高查询性能 (懒加载)

例：

CategoryPureVO类：用于过滤导航属性、不要的字段、还可以避免 循环序列化

##### 简单封装：拷贝对象的几种方法

###### 1、一个个get再set

```
SpuSimplifyVO vo = new SpuSimplifyVO();
vo.setXX(spu.getXX());
```

###### 2、使用spring的BeanUtils 浅拷贝

基础类型的拷贝使用 BeanUtils 足够了

例：

```java
import org.springframework.beans.BeanUtils;
BeanUtils.copyProperties(spu, vo);return vo;
```

```java
	public CategoryPureVO(Category category) {
        BeanUtils.copyProperties(category, this);
    }
```

###### 3、使用 DozerBeanMapper 深拷贝（dozer: 推土机）

其优势是可以实现对象的深拷贝

```xml
        <dependency>
            <groupId>com.github.dozermapper</groupId>
            <artifactId>dozer-core</artifactId>
            <version>6.5.0</version>
        </dependency>
```

```java
		Mapper mapper = DozerBeanMapperBuilder.buildDefault();		
		SpuSimplifyVO vo = mapper.map(spu, SpuSimplifyVO.class);  // 第一个参数是需要被拷贝的对象，第二个参数是拷贝后的目标对象的类型
```

map(源对象，目标类)：深拷贝的核心方法

##### 泛型类的封装

```java
package com.zjbti.missyou.vo;

import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import org.springframework.data.domain.Page;

import java.util.List;

/**
 * 泛型封装，封装需要返回到前端的分页页面数据 Page<T>
 * @date 2021/4/29 @Created by cxc
 */
@Getter
@Setter
@NoArgsConstructor
public class Paging<T> {
    private Long total;
    private Integer count;
    private Integer page;
    private Integer totalPage;
    private List<T> items;

    public Paging(Page<T> pageT) {
        this.initPageParameter(pageT);
        this.items = pageT.getContent();  // 这里不放入initPageParameter方法中是因为，扩展Paging不需要复制这个属性
    }

//    参数复制
    void initPageParameter(Page<T> pageT) {
        this.total = pageT.getTotalElements();
        this.count = pageT.getSize();
        this.page = pageT.getNumber();
        this.totalPage = pageT.getTotalPages();
    }

}

```

##### 高级VO案例

**静态方法**

```java
package com.zjbti.missyou.vo;

import com.zjbti.missyou.model.Coupon;
import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import org.springframework.beans.BeanUtils;

import java.math.BigDecimal;
import java.util.ArrayList;
import java.util.Date;
import java.util.List;
import java.util.stream.Collectors;

/**
 * @date 2021/5/9 @Created by cxc
 */
@Getter
@Setter
@NoArgsConstructor
public class CouponPureVo {

    private Long id;
    private String title;
    private Date startTime;
    private Date endTime;
    private String description;
    private BigDecimal fullMoney;
    private BigDecimal minus;
    private BigDecimal rate;
    private Integer type;
    private String remark;
    private Boolean wholeStore;

    public CouponPureVo(Object[] objects) {
        Coupon coupon = (Coupon) objects[0];
        BeanUtils.copyProperties(coupon, this);
    }

    public CouponPureVo(Coupon coupon) {
        BeanUtils.copyProperties(coupon, this);
    }

    public static List<CouponPureVo> getList(List<Coupon> couponList) {
        List<CouponPureVo> vos = new ArrayList<>();
        return couponList.stream()
                .map(CouponPureVo::new)
                .collect(Collectors.toList());
    }
}

```

**继承净化**

```java
package com.zjbti.missyou.vo;

import com.zjbti.missyou.model.Activity;
import lombok.Getter;
import lombok.Setter;

import java.util.List;
import java.util.stream.Collectors;

/**
 * @date 2021/5/9 @Created by cxc
 */
@Getter
@Setter
public class ActivityCouponVO extends ActivityPureVO {

    private List<CouponPureVo> coupons;

    public ActivityCouponVO(Activity activity) {
        super(activity);
        coupons = activity.getCouponList().stream()
                .map(CouponPureVo::new)
                .collect(Collectors.toList());
    }
}

```



## Repository 持久层

Repository 持久层

[JpaRepository 的 API 文档](https://docs.spring.io/spring-data/data-jpa/docs/current/api/org/springframework/data/jpa/repository/JpaRepository.html)

```java
// 继承JpaRepository<Bean类, PrimarilyKey主键的类型>，**自动注入容器**
public interface UserRepository extends JpaRepository<User, Long> {

    User findByName(String name);

    // 通过解析方法名创建查询。
    User findByNameAndAge(String name, Integer age);

    @Query("from User u where u.name=:name")
    User findUser(@Param("name") String name);

}
```

### 命名方法 JPA持久化接口

JPA接口命名规则，Jpa有词法分析器，分析方式类似普通SQL解析

| JpaRepository API方法                                    | 释义                                                         |
| -------------------------------------------------------- | ------------------------------------------------------------ |
| 基础方法                                                 | **接口中不需要实现的方法**                                   |
| findAll()<br />findAll(Pageable page)                    | 重载：就是函数或者方法有相同的名称，但是参数列表不相同的情形，这样的同名不同参数的函数或者方法之间，互相称之为重载函数或者重载方法。<br />根据分页对象，查询数据库中的数据 |
| findOne()                                                |                                                              |
| save()                                                   | 存储数据                                                     |
|                                                          |                                                              |
| 自定义方法                                               | **接口中需要实现的方法**                                     |
| findOneBy成员变量OrderBy成员变量Desc(成员变量形参)       | IDE有只能感知，JPA会自动生成SQL：select * from 表 where 成员变量 = 参数 <br />例如：Page\<Spu> findByCategoryId(Long cid, Pageable pageable);<br />asc升序，desc倒序 |
| **常用JPA关键字**                                        | **SQL关键字：**or、and、like、OrderBy、>、<                  |
|                                                          |                                                              |
| **复杂SQL需要自己写 JPQL/SQL**                           | JPQL：Java Persistence Query Language：Java持久化查询语言，操作的是model模型，查出来的还是model模型<br />缺点：维护SQL、存储过程很麻烦 |
| @Query("select t from Theme t where t.name in (:names)") | 创建查询，编写JPQL语句，并通过类似 “:name” **用冒号变量来映射@Param指定的参数**，就像例子中的第三个findUser函数一样。<br />nativeQuery 参数表示使用原生native SQL还是JPQL(Java Persistence Query Language)<br />例子：@Query("select t from Theme t where t.name in (:names)")<br />这里 Theme t ，类似于 sql 里的 as ，把实体简写成一个字母，这里是把Theme简写成t |

### JPQL

JPQL操作的是model，一般连表查询采用JPQL，有导航关系的连表查询也可以用命名方法查询（如，`findByCouponListId`）

普通多对多Join连表查询：

```sql
// idea JPQL Fragment中
select * from coupon join coupon_category on coupon.id = coupon_category.coupon_id join category on coupon_category.category_id = category.id where category.id = 32
```

```sql
// idea JPQL Fragment中
select c from Coupon c 
join 
c.categoryList ca
join
Activity a on a.id = c.activityId
where ca.id = :cid
and a.startTime < :now
and a.endTime > :now
```

如果配置了导航属性，JPQL中不用 on 关键字

#### IDEA 小技巧

IDEA 可以使用Spring Data QL Fragment 分段，写 JPQL/SQL

![1620399586191](https://cdn.jsdelivr.net/gh/ChenXiangcheng1/image-hosting2/img/2023_09_19_12_21.png)

#### 返回 Optional

```java
// Theme findByName(String name);
Optional<Theme> findByName(String name);
```



### 分页

分页逻辑：在数据库查询时分页，而不是全查询出来再分页

传统分页参数：Page（页码）、每页的条数（size）

移动端分页参数：start（从第几条开始获取数据）、count（一页获取多少条）

#### 分页对象Pageable

```java
// 构建JPA分页对象，参数page、size、Sort
Pageable page = PageRequest.of(pageNum, size, Sort.by("createTime").descending());return this.spuRepository.findAll(page);  // 返回的对象是Page<T>
```

JPA操作的是Model对象，而不是我们直接操作数据库，所以参数用的不是数据库中的下划线字段，而是model中的驼峰属性

#### 带条件的SQL分页查询

Page\<Spu> findByCategoryId(Long cid, Pageable pageable);

### 自动配置

去看 SpringBoot 自动配置解析笔记

## 分页操作

### 分页参数转换

```java
package com.zjbti.missyou.bo;

import lombok.Builder;
import lombok.Getter;
import lombok.Setter;

/**
 * Bo: business object，业务对象，由service传送到controller
 * @date 2021/4/22 @Created by cxc
 */
@Getter
@Setter
@Builder
public class PageCounter {

    private Integer page;
    private Integer count;
}

```

```java
package com.zjbti.missyou.util;

import com.zjbti.missyou.bo.PageCounter;

/**
 * 分页参数转换  start、count ==> pageNum、size
 * @date 2021/4/22 @Created by cxc
 */
public class CommonUtil {

    // 两个参数、所以封装了一下 PageCounter
    public static PageCounter convertToPageParameter(Integer start, Integer count) {
        int pageNum = start / count;
        PageCounter pageCounter = PageCounter.builder()
                .page(pageNum)
                .count(count)
                .build();
        return pageCounter;
    }
}

```

### 分页结果的封装VO

#### 分页结果Page\<T>对象的方法

| page\<T>           | 泛型<泛型参数> |
| ------------------ | -------------- |
| getContent()       | 每条内容的List |
| getTotalElements() | 总共多少条     |
| getSize()          | 一页多少条     |
| getNumber()        | 当前页码       |
| getTotalPages()    | 总页数         |
|                    |                |

#### 封装分页查询后的数据Page\<T>为VO(Paging)

查询出来的对象，要经过一层VO的封装才能返回到前端

用泛型类提取通用的类，简化代码的编写

详见**泛型类的封装**的笔记

```java
package com.zjbti.missyou.vo;

import lombok.Getter;
import lombok.NoArgsConstructor;
import lombok.Setter;
import org.springframework.data.domain.Page;

import java.util.List;

/**
 * 泛型封装，封装需要返回到前端的分页页面数据 Page<T>
 * @date 2021/4/29 @Created by cxc
 */
@Getter
@Setter
@NoArgsConstructor
public class Paging<T> {
    private Long total;
    private Integer count;
    private Integer page;
    private Integer totalPage;
    private List<T> items;

    public Paging(Page<T> pageT) {
        this.initPageParameter(pageT);
        this.items = pageT.getContent();  // 这里不放入initPageParameter方法中是因为，扩展Paging不需要复制这个属性
    }

//    参数复制
    void initPageParameter(Page<T> pageT) {
        this.total = pageT.getTotalElements();
        this.count = pageT.getSize();
        this.page = pageT.getNumber();
        this.totalPage = pageT.getTotalPages();
    }

}
```

#### 配合Dozer实现扩展Page\<T>(Paging)对象（泛型类的继承）

将Page\<T>结果对象封装为可以深拷贝的VO对象

实现可以进行属性的复制的 Page\<T>对象

```java
package com.zjbti.missyou.vo;

import com.github.dozermapper.core.DozerBeanMapperBuilder;
import com.github.dozermapper.core.Mapper;
import org.springframework.data.domain.Page;

import java.util.ArrayList;
import java.util.List;

/**
 * 实现深拷贝的Page对象，T (model) ==> K VO对象
 * @date 2021/4/29 @Created by cxc
 */
// 需要两个泛型参数，T是源类型，K是目标类型。T是父类需要的泛型参数是，K是业务需要的泛型参数
// 构造方法参数和泛函参数是不一样的，这里的PagingDozer构造方法是需要page<T>对象，返回的泛函参数是T
public class PagingDozer<T, K> extends Paging {

    @SuppressWarnings("unchecked") // suppress：抑制  // 该注解用于抑制没有权限访问的域的警告
    public PagingDozer(Page<T> pageT, Class<K> classK) {
        this.initPageParameter(pageT);

        List<T> tList = pageT.getContent();
        Mapper mapper = DozerBeanMapperBuilder.buildDefault();
        List<K> voList = new ArrayList<>();
        tList.forEach(t->{
            K vo = mapper.map(t, classK);
            voList.add(vo);
        });
        this.setItems(voList);
    }
}

```

Java泛型扩展：是一种伪泛型，从K难以拿到源类classK。而.net和C++的模板可以实现真正的泛型。为什么Java不实现呢，因为Java比较保守，为了维护以前JDK的版本

### 重构过程（附Controller代码）

抽象来源于具体

重构之前：

```java
@GetMapping("/latest?start= & count=")
    public List<SpuSimplifyVO> getLatestSpuList(@RequestParam(defaultValue = "0") Integer start,
                                                @RequestParam(defaultValue = "10") Integer count) {
        PageCounter pageCounter = CommonUtil.convertToPageParameter(start, count);
        // 查询出来的对象，要经过一层VO的封装才能返回到前端
        Page<Spu> spuList = this.spuService.getLatestPagingSpu(pageCounter.getPage(), pageCounter.getCount());
        Mapper mapper = DozerBeanMapperBuilder.buildDefault();
        List<SpuSimplifyVO> vos = new ArrayList<>();
        spuList.forEach(s->{
            SpuSimplifyVO vo = mapper.map(s, SpuSimplifyVO.class);
            vos.add(vo);
        });
        return vos;
    }
```

重构之后：

使Controller中代码更整洁，阅读效率更高

```java
    @GetMapping("/latest")
    public PagingDozer<Spu, SpuSimplifyVO> getLatestSpuList(@RequestParam(defaultValue = "0") Integer start,
                                                @RequestParam(defaultValue = "10") Integer count) {
        PageCounter pageCounter = CommonUtil.convertToPageParameter(start, count);
        Page<Spu> page = this.spuService.getLatestPagingSpu(pageCounter.getPage(), pageCounter.getCount());
        return new PagingDozer<>(page, SpuSimplifyVO.class);  // 因为泛型参数可以从方法签名的返回中推断出来，所以这里不需要了 PagingDozer<Spu, SpuSimplifyVO>     
    }
```

## 详解SKU的规格设计

尽可能不要硬编码

JSR303校验的message模板配置



可以适当去看数据库的设计的笔记的关于电商项目数据库设计的部分



## 通用泛型Converter

### 场景

场景1：当Spring使用@Entity注解，若数据库存储JSON数据时，项目中对应model的改成员变量使用对象会报错，使用String对应Json不会。这时可使用序列化的方式
场景2：当调用第三方API接口时，返回JSON字符串，需要序列化为对象

Springboot的JPA默认序列化库是Jackson，也可以去使用阿里巴巴的FastJson

单体JSON对象的映射处理

```java
    private String specs;

    public List<Spec> getSpecs() {
        String specs = this.specs;
        // Jackson
        return List<Spec>;
    }

    public void setSpecs(List<Spec> data) {
        // Jackson
        this.specs = String
    }
```



做个懒人，可以自己封装一下，使只用写一次，在任何实体里面都可以把数据库的JSON字段装化成Map的形式

### mapper序列化器方法

| 序列化器方法                                         | 释义                             |
| ---------------------------------------------------- | -------------------------------- |
| \<T> T readValue(String content, Class<T> valueType) | 将JSONcontent序列化为T类型的对象 |
|                                                      |                                  |
|                                                      |                                  |



### 单体JSON到Map的映射AttributeConverter

使用Jackson进行序列化操作

配合AttributeConverter，泛型参数为EntityAttribute实体属性类型 和 DatabaseColumn数据库类型

```java
package com.zjbti.missyou.util;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.zjbti.missyou.exception.http.ServerErrorException;
import org.apache.commons.collections.map.HashedMap;
import org.springframework.beans.factory.annotation.Autowired;

import javax.persistence.AttributeConverter;
import javax.persistence.Converter;
import java.util.Map;

/**
 * @date 2021/5/3 @Created by cxc
 */
@Converter // 该注解表示交给容器管理，当需要时自动调用
public class MapAndJson implements AttributeConverter<Map<String, Object>, String> {

    @Autowired
    private ObjectMapper mapper;  // Jackson的mapper序列化器

    @Override
    public String convertToDatabaseColumn(Map<String, Object> stringObjectMap) {
        try {
            return mapper.writeValueAsString(stringObjectMap);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
            throw new ServerErrorException(9999);
        }
        // IO异常不用全局处理，只用返回500服务器异常错误就好，因为IO异常往往是硬件或网络问题，前端不需要了解。还可以记入日志
    }

    @Override
    @SuppressWarnings("unchecked")
    public Map<String, Object> convertToEntityAttribute(String s) {
        try {
            return mapper.readValue(s, HashedMap.class);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
            throw new ServerErrorException(9999);
        }
//        map只是一个接口，所以不能map.class，要具体的
    }
}

```

应用：

```java
    @Convert(converter = MapAndJson.class)
    private Map<String, Object> test;
```

之所以是Map的value不是强类型的而是Object类型，是因为需要工具类的通用性，但是弊端是使Map失去了类的业务能力

### 数组类型JSON与List的映射

```java
package com.zjbti.missyou.util;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.zjbti.missyou.exception.http.ServerErrorException;
import org.springframework.beans.factory.annotation.Autowired;

import javax.persistence.AttributeConverter;
import javax.persistence.Converter;
import java.util.List;

/**
 * @date 2021/5/3 @Created by cxc
 */
@Converter
public class ListAndJson implements AttributeConverter<List<Object>, String> {

    @Autowired
    private ObjectMapper mapper;

    @Override
    public String convertToDatabaseColumn(List<Object> objects) {
        try {
            return mapper.writeValueAsString(objects);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
            throw new ServerErrorException(9999);
        }
    }

    @Override
    @SuppressWarnings("unchecked")
    public List<Object> convertToEntityAttribute(String s) {
        try {
            if (s == null) {
                return null;
            }
            List<Object> t = mapper.readValue(s, List.class);
            return t;
        } catch (JsonProcessingException e) {
            e.printStackTrace();
            throw new ServerErrorException(9999);
        }
    }
}

```

应用：

```java
	@Convert(converter = ListAndJson.class)
    private List<Object> specs;
```

### 谈Java类的内聚性、方法外置问题

List、Map表达业务对象：缺点集合失去了业务的能力 (方法的能力)

如果要定义List、Map业务对象的方法，要使方法依附在对象上外置

### 通用泛型类对数据库Json的序列化与反序列化

单体泛型反序列化与List泛型反序列化（两种方法签名设计）

```java
package com.zjbti.missyou.util;

import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.core.type.TypeReference;
import com.fasterxml.jackson.databind.ObjectMapper;
import com.zjbti.missyou.exception.http.ServerErrorException;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

/**
 * Generic：通用的
 * @date 2021/5/4 @Created by cxc
 */
@Component
public class GenericAndJson {

    private static ObjectMapper mapper;

    @Autowired
    public void setMapper(ObjectMapper mapper) {
        GenericAndJson.mapper = mapper;
    }

    public static <T> String objectToJson(T o) {
        try {
            return GenericAndJson.mapper.writeValueAsString(o);
        } catch (JsonProcessingException e) {
            e.printStackTrace();
            throw new ServerErrorException(9999);
        }
    }
    
// 单体泛型反序列化
/*
// 这方法失败了，返回的是ArrayList<HashMap>而不是ArrayList<Spec>
    public static <T> T jsonToObject(String s, Class<T> classT) {
        if (s == null) {
            return null;
        }
        try {
            T o = GenericAndJson.mapper.readValue(s, classT);
            return o;
        } catch (JsonProcessingException e) {
            e.printStackTrace();
            throw new ServerErrorException(9999);
        }
    }
*/
    public static <T> T jsonToObject(String s, TypeReference<T> tr) {
        if (s == null) {
            return null;
        }
        try {
            T o = GenericAndJson.mapper.readValue(s, tr);
            return o;
        } catch (JsonProcessingException e) {
            e.printStackTrace();
            throw new ServerErrorException(9999);
        }
    }
    
//    List泛型反序列化
    
//    方法签名设计2，Spec == T
// 这方法失败了，返回的是ArrayList<HashMap>而不是ArrayList<Spec>
/*
        public static <T> List<T> jsonToList(String s) {
        if (null == s) {
            return null;
        }
        try {
            List<T> list = GenericAndJson.mapper.readValue(s, new TypeReference<List<T>>() {
            });
            return list;
        } catch (JsonProcessingException e) {
            e.printStackTrace();
            throw new ServerErrorException(9999);
        }
    }
*/
    
//    方法签名设计1，List<Spec> == T
//    public static <T> T jsonToList(String s, Class<T> classT)时，classT为List<Spec>.class不能传参
//    所以使用TypeReference<T>，调用时构造TypeReference<List<Spec>>
    public static <T> T jsonToList(String s, TypeReference<T> tr) {
        if (null == s) {
            return null;
        }
        try {
            T list = GenericAndJson.mapper.readValue(s, tr);
            return list;
        } catch (JsonProcessingException e) {
            e.printStackTrace();
            throw new ServerErrorException(9999);
        }

    }
}

```

```java
package com.zjbti.missyou.model;

import com.fasterxml.jackson.core.type.TypeReference;
import com.zjbti.missyou.util.GenericAndJson;
import lombok.Getter;
import lombok.Setter;

import javax.persistence.Entity;
import javax.persistence.Id;
import java.math.BigDecimal;
import java.util.Collections;
import java.util.List;

/**
 * @date 2021/4/22 @Created by cxc
 */
@Entity
@Getter
@Setter
public class Sku extends BaseEntity {
    @Id
    private Long id;
    private BigDecimal price;
    private BigDecimal discountPrice;
    private Boolean online;
    private String img;
    private String title;
    private Long spuId;
    private Long categoryId;
    private Long rootCategoryId;

    private String specs;
    private String code;
    private Long stock;

    public List<Spec> getSpecs() {
        if (null == this.specs) {
            return Collections.emptyList();
        }
        return GenericAndJson.jsonToObject(this.specs, new TypeReference<List<Spec>>() {
        });
    }

    public void setSpecs(List<Spec> specs) {
        if (specs.isEmpty()) {
            return;
        }
        this.specs = GenericAndJson.objectToJson(specs);
    }
}

```



