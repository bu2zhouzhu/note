## [注解](https://github.com/junit-team/junit4/wiki/Test-fixtures)
| 注解           | 描述           |
| ------------ | ------------ |
| @Test        | 定义一个方法为测试方法  |
| @Before      | 在每一个测试方法前执行  |
| @After       | 在每一个测试方法后执行  |
| @BeforeClass | 在所有测试方法前执行一次 |
| @AfterClass  | 在所有测试方法后执行一次 |
| @Ignore      | 忽略一个测试       |

## [断言](https://github.com/junit-team/junit4/wiki/Assertions)
| 段言`                                      | 描述          |
| ---------------------------------------- | ----------- |
| assertTrue(boolean x)                    | 验证参数是 true  |
| assertFalse(boolean x)                   | 验证参数是 false |
| assertEquals(Object expected, Object actual) | 验证两个对象是相等的  |

## [忽略测试](https://github.com/junit-team/junit4/wiki/Ignoring-tests)

要忽略一个测试，在 `@Test` 注解之前或之后添加 `@Ignore` 注解。`@Ignore` 注解有一个可选参数（一个字符串）如果你想记录方法被忽略的原因。

```java
@Ignore("Test is ignored as a demonstration")
@Test
public void testSame() {
    assertThat(1, is(1));
}
```



## [测试执行顺序](https://github.com/junit-team/junit4/wiki/Test-execution-order)

JUnit 没有指定测试方法的调用顺序。测试方法是按照反射接口返回的顺序调用的。实际上 JDK 7 返回一个随机的顺序。

你可以使用注解指定测试方法按照方法名称的顺序执行。为了激活这个特性，使用 `@FixMethodOrder(MethodSorters.NAME_ASCENDING)`注解注解你的测试类。

### 例子

```java
import org.junit.FixMethodOrder;
import org.junit.Test;
import org.junit.runners.MethodSorters;

@FixMethodOrder(MethodSorters.NAME_ASCENDING)
public class TestMethodOrder {

    @Test
    public void testA() {
        System.out.println("first");
    }
    @Test
    public void testB() {
        System.out.println("second");
    }
    @Test
    public void testC() {
        System.out.println("third");
    }
}
```



## [异常测试](https://github.com/junit-team/junit4/wiki/Exception-testing)

### Try/Catch 方式

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

### expected 参数 

`@Test` 注解有一个可选的参数 `expected` , 它的值为 `Throwable` 的子类

```java
@Test(expected = IndexOutOfBoundsException.class)
public void testException2() {
    List<String> list = new ArrayList<>();
    list.get(0);
}
```

### ExpectedException Rule

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

## [Runner](https://github.com/junit-team/junit4/wiki/Test-runners)

一个 JUnit Runner 是继承自 Junit 的 Runner 抽象类的类。Runner 用来运行测试类。用来运行测试的 Runner 用 `@RunWith` 注解来设置。

```java
@RunWith(MyTestRunner.class)
public class MyTestClass {
 
  @Test
  public void myTest() {
    ..
  }
}
```

![Runner 类图](https://www.mscharhag.com/files/2014/junit-runner.png)

### [Suite](https://github.com/junit-team/junit4/wiki/Aggregating-tests-in-suites)

如果你有多个测试类，你可以把他们组合到一个测试套件中。运行一个测试套件会运行套件中所有的测试类，按照注解的先后顺序。一个测试套件可以包含其它的测试套件。

```java
@RunWith(Suite.class)
@SuiteClasses({
        MyClassTest.class,
        MySecondClassTest.class })
public class AllTests {
}
```

### [Categories](https://github.com/junit-team/junit4/wiki/Categories)

`Categories` runner 只运行拥有 `@IncludeCategory` 标注的类和方法。类或接口都可以用作种类。子类工作，所以如果有 `@IncludeCategory(SuperClass.class)` ，一个拥有 `@Category({SubClass.class})` 注解的方法会被执行。

你也可以用 `@ExcludeCategory` 注解排除种类。

例子：

```java
public interface FastTests { /* category marker */ }
public interface SlowTests { /* category marker */ }

public class A {
  @Test
  public void a() {
    fail();
  }

  @Category(SlowTests.class)
  @Test
  public void b() {
  }
}

@Category({SlowTests.class, FastTests.class})
public class B {
  @Test
  public void c() {

  }
}

@RunWith(Categories.class)
@IncludeCategory(SlowTests.class)
@SuiteClasses( { A.class, B.class }) // Note that Categories is a kind of Suite
public class SlowTestSuite {
  // Will run A.b and B.c, but not A.a
}

@RunWith(Categories.class)
@IncludeCategory(SlowTests.class)
@ExcludeCategory(FastTests.class)
@SuiteClasses( { A.class, B.class }) // Note that Categories is a kind of Suite
public class SlowTestSuite {
  // Will run A.b, but not A.a or B.c
}
```

### [Parameterized](https://github.com/junit-team/junit4/wiki/Parameterized-tests)

```java
import static org.junit.Assert.assertEquals;

import java.util.Arrays;
import java.util.Collection;

import org.junit.Test;
import org.junit.runner.RunWith;
import org.junit.runners.Parameterized;
import org.junit.runners.Parameterized.Parameters;

@RunWith(Parameterized.class)
public class FibonacciTest {
    @Parameters
    public static Collection<Object[]> data() {
        return Arrays.asList(new Object[][] {     
                 { 0, 0 }, { 1, 1 }, { 2, 1 }, { 3, 2 }, { 4, 3 }, { 5, 5 }, { 6, 8 }  
           });
    }

    private int fInput;

    private int fExpected;

    public FibonacciTest(int input, int expected) {
        fInput= input;
        fExpected= expected;
    }

    @Test
    public void test() {
        assertEquals(fExpected, Fibonacci.compute(fInput));
    }
}
```

```java
public class Fibonacci {
    public static int compute(int n) {
    	int result = 0;
    	
        if (n <= 1) { 
        	result = n; 
        } else { 
        	result = compute(n - 1) + compute(n - 2); 
        }
        
        return result;
    }
}
```

### 第三方

- [MockitoJUnitRunner](http://static.javadoc.io/org.mockito/mockito-core/2.15.0/org/mockito/junit/MockitoJUnitRunner.html)

## [Rules](https://github.com/junit-team/junit4/wiki/Rules)

### TemporaryFolder 规则

临时文件夹规则允许创建文件和文件夹，它们将在测试方法结束后被删除（无论测试成功还是失败）。

```java
public static class HasTempFolder {
  @Rule
  public final TemporaryFolder folder = new TemporaryFolder();

  @Test
  public void testUsingTempFolder() throws IOException {
    File createdFile = folder.newFile("myfile.txt");
    File createdFolder = folder.newFolder("subfolder");
    assertTrue(createdFile.exists());
  }
} 
```

### ExpectedException 规则

期待异常规则允许指定测试方法内期待的异常类型和消息内容

```java
public static class HasExpectedException {
  @Rule
  public final ExpectedException thrown = ExpectedException.none();

  @Test
  public void throwsNothing() {

  }

  @Test
  public void throwsNullPointerException() {
    thrown.expect(NullPointerException.class);
    throw new NullPointerException();
  }

  @Test
  public void throwsNullPointerExceptionWithMessage() {
    thrown.expect(NullPointerException.class);
    thrown.expectMessage("What happened?");
    throw new NullPointerException("What happened?");
  }
}
```

## 参考

[JUnit4 官方文档](https://github.com/junit-team/junit4/wiki)

[Vogella JUnit 教程](http://www.vogella.com/tutorials/JUnit/article.html)

[Understanding JUnit's Runner architecture](https://www.mscharhag.com/java/understanding-junits-runner-architecture)

