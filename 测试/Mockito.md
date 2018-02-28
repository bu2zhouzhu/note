## 创建模拟对象

#### 使用静态方法 mock() 创建

```java
Flower flowerMock = Mockito.mock(Flower.class);
```

#### 使用注解 @Mock 创建

`@Mock` 注解生效需要使用 `MockitoJUnitRunner` 或 `MockitoRule`

使用  `MockitoJUnitRunner` 

```
@RunWith(MockitoJUnitRunner.class)
```

使用  `MockitoRule`

```
@Rule public MockitoRule mockitoRule = MockitoJUnit.rule();
```

使用 `@Mock` 创建模拟对象

```java
@Mock
private Flower flowerMock;
```



## 验证行为

```java
//Let's import Mockito statically so that the code looks clearer
import static org.mockito.Mockito.*;

//mock creation
List mockedList = mock(List.class);

//using mock object
mockedList.add("one");
mockedList.clear();

//verification
verify(mockedList).add("one");
verify(mockedList).clear();
```





## 参考

[Mockito 文档](javadoc.io/page/org.mockito/mockito-core/latest/org/mockito/Mockito.html)

[Mockito Wiki](https://github.com/mockito/mockito/wiki)

[Vogella Mockito 教程](http://www.vogella.com/tutorials/Mockito/article.html)

