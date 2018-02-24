## 注解
| 注解           | 描述           |
| ------------ | ------------ |
| @Test        | 定义一个方法为测试方法  |
| @Before      | 在每一个测试方法前执行  |
| @After       | 在每一个测试方法后执行  |
| @BeforeClass | 在所有测试方法前执行一次 |
| @AfterClass  | 在所有测试方法后执行一次 |
| @Ignore      | 忽略一个测试       |

## 断言
| 段言                                       | 描述          |
| ---------------------------------------- | ----------- |
| assertTrue(boolean x)                    | 验证参数是 true  |
| assertFalse(boolean x)                   | 验证参数是 false |
| assertEquals(Object expected, Object actual) | 验证两个对象是相等的  |

## 异常测试

方法1

```java
@Test
public void testException1() {
    try {
        List<String> list = new ArrayList<>();
        list.get(0);
        fail();
    } catch (Exception e) {
        String actual = e.getMessage();
        String expected = "Index: 0, Size: 0";
        assertEquals(expected, actual);
    }
}
```

方法2

```java
@Test(expected = IndexOutOfBoundsException.class)
public void testException2() {
    List<String> list = new ArrayList<>();
    list.get(0);
}
```

方法3

```java
@Rule
public ExpectedException exception = ExpectedException.none();

@Test
public void testException3() {
    exception.expect(IndexOutOfBoundsException.class);
    exception.expectMessage("Index: 0, Size: 0");
    List<String> list = new ArrayList<>();
    list.get(0);
}
```


## Test Suite

如果你有多个测试类，你可以把他们组合到一个测试套件中。运行一个测试套件会运行套件中所有的测试类，按照注解的先后顺序。一个测试套件可以包含其它的测试套件。

```java
@RunWith(Suite.class)
@SuiteClasses({
        MyClassTest.class,
        MySecondClassTest.class })

public class AllTests {
}
```

