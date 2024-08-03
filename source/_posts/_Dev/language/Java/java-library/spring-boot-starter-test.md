# Spring整合Junit测试

依赖包：spring-test、Junit4



## 断言

`assertEquals(期望值，实际值)`

`assertEquals(期望对象，实际对象)` 利用Equals

`assertSame(期望对象，实际对象)` 利用内存地址

`assertNotSame(期望对象，实际对象)`

`assertNull(对象1，对象2)` 检查一个对象是否为空

`assertNotNull(对象1，对象2)`

`assertTrue(布尔条件)`

`assertFalse(布尔条件)`



## Junit注解

| 注解                                                   |      | ‘释义                                                        | 成员变量           |
| ------------------------------------------------------ | ---- | ------------------------------------------------------------ | ------------------ |
| @RunWith(SpringJUnit4ClassRunner.class)                |      | 对类使用，让测试运行于Spring测试环境<br />指定Runner（运行器） | SpringRunner.class |
| @ContextConfiguration("classpath:spring-service1.xml") |      | 对类使用，指定Spring上下文配置，即依赖注入所需的xml文件位置  |                    |
| @Test                                                  |      | 对方法使用，测试方法                                         |                    |
|                                                        |      |                                                              |                    |

  

## 例子

```java
@Autowire Integer a;
@Test(expected=ArithmeticException.class)
public void xx(){
	//断言除数是否为0
	assertTrue(a==0);
	int n=2/a;	
}
```

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:spring-servicel.xml")
public class OrderTest {
    @Autowired
	private OrderService orderService;
    @Test
    public void testAddOrder{
        Order order new Order("100006", "100002", 2, 1799, "");
		orderService.addorder(order);
    }
}
```

