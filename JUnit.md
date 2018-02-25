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
| 段言`                                        | 描述                 |
| -------------------------------------------- | -------------------- |
| assertTrue(boolean x)                        | 验证参数是 true      |
| assertFalse(boolean x)                       | 验证参数是 false     |
| assertEquals(Object expected, Object actual) | 验证两个对象是相等的 |

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

## [Runner](https://github.com/junit-team/junit4/wiki/Test-runners)

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

### Parameterized

### 第三方

- [MockitoJUnitRunner](http://static.javadoc.io/org.mockito/mockito-core/2.15.0/org/mockito/junit/MockitoJUnitRunner.html)

## Rules

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