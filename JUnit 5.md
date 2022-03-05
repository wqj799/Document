# JUnit 5

## 什么是JUnit 5

JUnit 5 是由来自三个不同的子项目的几个不同模块组成。

**JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage**

- JUnit Platform 是在JVM上启动测试框架的基础。他还定义了TestEngine API用来开发在此平台上运行的测试框架。此外，这个平台提供了启动控制器（Console Launcher），用来从命令行启动平台，并且提供JUnit平台套件引擎（JUnit Platform Suite Engine）用于在此平台上运行一个或多个自定义测试套件。对各种IDE（如IntelliJ IDEA，Eclipse等）和构建工具（如Gradle，Maven等）提供很好的支持。
- JUnit Jupiter 是在JUnit 5 中编写测试和扩展时使用的新编程模型和扩展模型的组合。Jupiter子项目提供了一个TestEngine用于在此平台上运行基于Jupiter的测试。
- JUnit Vintage 提供了一个TestEngine，用于在此平台上运行基于JUnit 3和Junit 4的测试。他要求在class path或module path中出现JUnit 4.12或更高的版本。

## 支持的版本

JUnit 5 在运行时需要Java 8或更高的版本。但是，在编译阶段可以使用先前的jdk版本。

## 测试类和方法

- 测试类（Test Class）：包含至少一个测试方法的任何顶级类，静态成员类或是@Nested注解的类。**这些类不能是抽象的，并且必须包含一个构造函数。**

- 测试方法（Test Method）：任何直接使用`@Test`, `@RepeatedTest`, `@ParameterizedTest`, `@TestFactory`, 或 `@TestTemplate`注解的实例方法。

- 生命周期方法（Lifecycle Method）：任何直接使用`@BeforeAll`, `@AfterAll`, `@BeforeEach`, 或`@AfterEach`注解的方法。

下面是一个标准的测试类：

```java
import static org.junit.jupiter.api.Assertions.fail;
import static org.junit.jupiter.api.Assumptions.assumeTrue;

import org.junit.jupiter.api.AfterAll;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.BeforeAll;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Disabled;
import org.junit.jupiter.api.Test;

class StandardTests {

    @BeforeAll
    static void initAll() {
    }

    @BeforeEach
    void init() {
    }

    @Test
    void succeedingTest() {
    }

    @Test
    void failingTest() {
        fail("a failing test");
    }

    @Test
    @Disabled("for demonstration purposes")
    void skippedTest() {
        // not executed
    }

    @Test
    void abortedTest() {
        assumeTrue("abc".contains("Z"));
        fail("test should have been aborted");
    }

    @AfterEach
    void tearDown() {
    }

    @AfterAll
    static void tearDownAll() {
    }

}
```

## 显示名称和显示名称生成器

- @DisplayName
  
  测试类和测试方法可以通过此注解声明自定义的显示名称。名称中可以使用空格，特殊字符，甚至是emojis。在测试报告和IDE中将会显示这些名称。
  
  ```java
  import org.junit.jupiter.api.DisplayName;
  import org.junit.jupiter.api.Test;
  
  @DisplayName("A special test case")
  class DisplayNameDemo {
  
      @Test
      @DisplayName("Custom test name containing spaces")
      void testWithDisplayNameContainingSpaces() {
      }
  
      @Test
      @DisplayName("╯°□°）╯")
      void testWithDisplayNameContainingSpecialCharacters() {
      }
  
      @Test
      @DisplayName("😱")
      void testWithDisplayNameContainingEmoji() {
      }
  }
  ```

- @DisplayNameGeneration
  
  JUnit Jupiter 支持自定义显示名称生成器。
  
  **注意：通过@DisplayName注解提供的值优先于通过@DisplayNameGeneration注解生成的显示名称。**
  
  | DisplayNameGenerator | 行为                          |
  | -------------------- | --------------------------- |
  | Standard             | 匹配JUnit 5 发布以来的标准显示名称生成器行为。 |
  | Simple               | 删除没有参数的方法的末尾括号。             |
  | ReplaceUnderscores   | 用空格替换下划线。                   |
  | IndicativeSentences  | 通过连接符将测试名称和测试类组合成完整的句子。     |
  
  ```java
  import org.junit.jupiter.api.DisplayName;
  import org.junit.jupiter.api.DisplayNameGeneration;
  import org.junit.jupiter.api.DisplayNameGenerator;
  import org.junit.jupiter.api.IndicativeSentencesGeneration;
  import org.junit.jupiter.api.Nested;
  import org.junit.jupiter.api.Test;
  import org.junit.jupiter.params.ParameterizedTest;
  import org.junit.jupiter.params.provider.ValueSource;
  
  public class DisplayNameGeneratorDemo {
      @Nested
      @DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
      class A_year_is_not_supported {
  
          @Test
          void if_it_is_zero() {
          }
  
          @DisplayName("A negative value for year is not supported by the leap year computation.")
          @ParameterizedTest(name = "For example, year {0} is not supported.")
          @ValueSource(ints = { -1, -4 })
          void if_it_is_negative(int year) {
          }
      }
  
      @Nested
      @IndicativeSentencesGeneration(separator = " -> ", generator = DisplayNameGenerator.ReplaceUnderscores.class)
      class A_year_is_a_leap_year {
  
          @Test
          void if_it_is_divisible_by_4_but_not_by_100() {
          }
  
          @ParameterizedTest(name = "Year {0} is a leap year.")
          @ValueSource(ints = { 2016, 2020, 2048 })
          void if_it_is_one_of_the_following_years(int year) {
          }
      }
  }
  ```
  
  ```html
  +-- DisplayNameGeneratorDemo [OK]
    +-- A year is not supported [OK]
    | +-- A negative value for year is not supported by the leap year computation. [OK]
    | | +-- For example, year -1 is not supported. [OK]
    | | '-- For example, year -4 is not supported. [OK]
    | '-- if it is zero() [OK]
    '-- A year is a leap year [OK]
      +-- A year is a leap year -> if it is divisible by 4 but not by 100. [OK]
      '-- A year is a leap year -> if it is one of the following years. [OK]
        +-- Year 2016 is a leap year. [OK]
        +-- Year 2020 is a leap year. [OK]
        '-- Year 2048 is a leap year. [OK]
  ```

- 设置默认的显示名称生成器
  
  可以设置默认的Display Name Generator，通过在配置文件中（如`src/test/resources/junit-platform.properties`）设置`junit.jupiter.displayname.generator.default`来配置，它的值为要使用的生成器的完全类限定名。如：
  
  ```properties
  junit.jupiter.displayname.generator.default = org.junit.jupiter.api.DisplayNameGenerator$ReplaceUnderscores
  ```
  
  总之，将根据以下的顺序显示测试类或方法的显示名称：
  
  1. @DisplayName注解设置的值
  
  2. 通过调用@DisplayNameGeneration注解中指定的DisplayNameGenerator
  
  3. 通过设置默认的DisplayNameGenerator
  
  4. 通过调用`org.junit.jupiter.api.DisplayNameGenerator.Standard`

## 断言

JUnit Jupiter提供了许多的断言方法。所有的断言都是`static`方法，位于[org.junit.jupiter.api.Assertions](https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/Assertions.html) class中。

```java
import static java.time.Duration.ofMillis;
import static java.time.Duration.ofMinutes;
import static org.junit.jupiter.api.Assertions.assertAll;
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertNotNull;
import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.junit.jupiter.api.Assertions.assertTimeout;
import static org.junit.jupiter.api.Assertions.assertTimeoutPreemptively;
import static org.junit.jupiter.api.Assertions.assertTrue;

import java.util.concurrent.CountDownLatch;

import example.domain.Person;
import example.util.Calculator;

import org.junit.jupiter.api.Test;

class AssertionsDemo {

    private final Calculator calculator = new Calculator();

    private final Person person = new Person("Jane", "Doe");

    @Test
    void standardAssertions() {
        assertEquals(2, calculator.add(1, 1));
        assertEquals(4, calculator.multiply(2, 2),
                "The optional failure message is now the last parameter");
        assertTrue('a' < 'b', () -> "Assertion messages can be lazily evaluated -- "
                + "to avoid constructing complex messages unnecessarily.");
    }

    @Test
    void groupedAssertions() {
        // assertAll将断言所有的可执行方法都不会抛出异常。
        // 如果可执行方法发生异常，则所有的异常都会一起抛出。
        assertAll("person",
            () -> assertEquals("Jane", person.getFirstName()),
            () -> assertEquals("Doe", person.getLastName())
        );
    }

    @Test
    void dependentAssertions() {
        // 在代码块中，如果断言失败，则将跳过同一块中的后续代码。
        assertAll("properties",
            () -> {
                String firstName = person.getFirstName();
                assertNotNull(firstName);

                // 尽当前一个断言成功后才执行。
                assertAll("first name",
                    () -> assertTrue(firstName.startsWith("J")),
                    () -> assertTrue(firstName.endsWith("e"))
                );
            },
            () -> {
                // 即使上面的代码块即使断言失败，这里的代码块也会运行，代码块直接互相独立处理。
                String lastName = person.getLastName();
                assertNotNull(lastName);

                assertAll("last name",
                    () -> assertTrue(lastName.startsWith("D")),
                    () -> assertTrue(lastName.endsWith("e"))
                );
            }
        );
    }

    @Test
    void exceptionTesting() {
        Exception exception = assertThrows(ArithmeticException.class, () ->
            calculator.divide(1, 0));
        assertEquals("/ by zero", exception.getMessage());
    }

    @Test
    void timeoutNotExceeded() {
        // 断言代码块的执行时间在超时之前完成。
        assertTimeout(ofMinutes(2), () -> {
            // 执行耗时少于两分钟的任务。
        });
    }

    @Test
    void timeoutNotExceededWithResult() {
        // 断言成功后可以得到代码块中的返回值。
        String actualResult = assertTimeout(ofMinutes(2), () -> {
            return "a result";
        });
        assertEquals("a result", actualResult);
    }

    @Test
    void timeoutNotExceededWithMethod() {
        // 断言可以调用方法引用并返回一个对象。
        String actualGreeting = assertTimeout(ofMinutes(2), AssertionsDemo::greeting);
        assertEquals("Hello, World!", actualGreeting);
    }

    @Test
    void timeoutExceeded() {
        // 以下断言失败，错误消息类似于：
        // execution exceeded timeout of 10 ms by 91 ms
        assertTimeout(ofMillis(10), () -> {
            // 模拟耗时超过 10 毫秒的任务。
            Thread.sleep(100);
        });
    }

    @Test
    void timeoutExceededWithPreemptiveTermination() {
        // 以下断言失败，并显示类似于以下内容的错误消息：
        // execution timed out after 10 ms
        assertTimeoutPreemptively(ofMillis(10), () -> {
            // 模拟耗时超过 10 毫秒的任务。
            new CountDownLatch(1).await();
        });
    }

    private static String greeting() {
        return "Hello, World!";
    }

}
```

注意：使用assertTimeoutPreemptively()抢占式超时和声明式超时相反，Assertions类中的各种assertTimeoutPreemptively()方法在与调用代码不同的线程中执行。所以，如果在执行代码时使用`java.lang.ThreadLocal`存储，则可能会出现副作用。

一个常见的例子是测试Spring Framework中事务的支持。具体来说，Spring的测试支持在调用测试方法之前将事务状态绑定到当前的线程（使用ThreadLocal的方式）。因此，如果使用assertTimeoutPreemptively()时调用了Spring的事务管理组件，则这些组件采取的任何操作都不会随着测试管理的事务回滚。

## 假设

JUnit jupiter中可以使用JUnit 4 中的假设方法，并添加了一些支持lambda表达式和方法引用的方法。

```java
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assumptions.assumeTrue;
import static org.junit.jupiter.api.Assumptions.assumingThat;

import example.util.Calculator;

import org.junit.jupiter.api.Test;

class AssumptionsDemo {

    private final Calculator calculator = new Calculator();

    @Test
    void testOnlyOnCiServer() {
        // 此时下面的假设是正确的，则会继续执行其余的测试。
        assumeTrue(true);
        // 此时下面的假设是不正确，则不会执行其余的测试。
        assumeTrue(false);
        // 其余的测试
    }

    @Test
    void testOnlyOnDeveloperWorkstation() {
        // 可以使用lambda表达式
        assumeTrue("DEV".equals(System.getenv("ENV")),
            () -> "Aborting test: not on developer workstation");
        // 其余的测试
    }
}
```

## 禁用测试

- @Disabled

可以通过此注释可以禁用整个测试或是单独的测试方法。

```java
import org.junit.jupiter.api.Disabled;
import org.junit.jupiter.api.Test;

// 禁用整个测试类
@Disabled("Disabled until bug #99 has been fixed")
class DisabledClassDemo {

    @Test
    void testWillBeSkipped() {
    }

}
```

```java
import org.junit.jupiter.api.Disabled;
import org.junit.jupiter.api.Test;

class DisabledTestsDemo
 {
    // 禁用单个测试方法
    @Disabled("Disabled until bug #42 has been resolved")
    @Test
    void testWillBeSkipped() {
    }

    @Test
    void testWillBeExecuted() {
    }

}
```

## 执行条件测试

JUnit Jupiter 中的 ExecutionCondition 扩展 API 允许开发人员以编程方式基于特定条件启用或禁用容器或测试。

- 操作系统条件
  
  可以通过@EnabledOnOs和@DisabledOnOs注释在特定操作系统上启用或禁用容器或测试。
  
  ```java
  @Test
  @EnabledOnOs(MAC)
  void onlyOnMacOs() {
      // ...
  }
  
  @TestOnMac
  void testOnMac() {
      // ...
  }
  
  @Test
  @EnabledOnOs({ LINUX, MAC })
  void onLinuxOrMac() {
      // ...
  }
  
  @Test
  @DisabledOnOs(WINDOWS)
  void notOnWindows() {
      // ...
  }
  
  @Target(ElementType.METHOD)
  @Retention(RetentionPolicy.RUNTIME)
  @Test
  @EnabledOnOs(MAC)
  @interface TestOnMac {
  }
  ```

- Java运行时环境条件
  
  可以通过 @EnabledOnJre 和 @DisabledOnJre 注释在特定版本的 Java 运行时环境 (JRE) 上启用或禁用容器或测试，或者通过 @EnabledForJreRange 和 @DisabledForJreRange 注释在特定范围的 JRE 版本上启用或禁用容器或测试。 范围默认为 JRE.JAVA_8 作为下边界（最小）和 JRE.OTHER 作为上边界（最大），这允许使用半开放范围。
  
  ```java
  @Test
  @EnabledOnJre(JAVA_8)
  void onlyOnJava8() {
      // ...
  }
  
  @Test
  @EnabledOnJre({ JAVA_9, JAVA_10 })
  void onJava9Or10() {
      // ...
  }
  
  @Test
  @EnabledForJreRange(min = JAVA_9, max = JAVA_11)
  void fromJava9to11() {
      // ...
  }
  
  @Test
  @EnabledForJreRange(min = JAVA_9)
  void fromJava9toCurrentJavaFeatureNumber() {
      // ...
  }
  
  @Test
  @EnabledForJreRange(max = JAVA_11)
  void fromJava8To11() {
      // ...
  }
  
  @Test
  @DisabledOnJre(JAVA_9)
  void notOnJava9() {
      // ...
  }
  
  @Test
  @DisabledForJreRange(min = JAVA_9, max = JAVA_11)
  void notFromJava9to11() {
      // ...
  }
  
  @Test
  @DisabledForJreRange(min = JAVA_9)
  void notFromJava9toCurrentJavaFeatureNumber() {
      // ...
  }
  
  @Test
  @DisabledForJreRange(max = JAVA_11)
  void notFromJava8to11() {
      // ...
  }
  ```

- 系统属性条件
  
  容器或测试可以通过 @EnabledIfSystemProperty 和 @DisabledIfSystemProperty 注释根据命名的 JVM 系统属性的值启用或禁用。 通过 matches 属性提供的值将被解释为正则表达式。
  
  ```java
  @Test
  @EnabledIfSystemProperty(named = "os.arch", matches = ".*64.*")
  void onlyOn64BitArchitectures() {
      // ...
  }
  
  @Test
  @DisabledIfSystemProperty(named = "ci-server", matches = "true")
  void notOnCiServer() {
      // ...
  }
  ```
  
  **从 JUnit Jupiter 5.6 开始，@EnabledIfSystemProperty 和@DisabledIfSystemProperty 是可重复的注解。**

- 自定义条件
  
  可以通过给@EnabledIf和@DisabledIf注解传入方法名，根据方法的返回布尔值来启用或禁用测试。
  
  ```java
  @Test
  @EnabledIf("customCondition")
  void enabled() {
      // ...
  }
  
  @Test
  @DisabledIf("customCondition")
  void disabled() {
      // ...
  }
  
  // 将customCondition方法名传给@EnabledIf或@DisabledIf注解
  boolean customCondition() {
      return true;
  }
  ```
  
  另外，条件方法可以放在测试类之外，此时，需要通过全限定类名来调用。
  
  ```java
  package example;
  
  import org.junit.jupiter.api.Test;
  import org.junit.jupiter.api.condition.EnabledIf;
  
  class ExternalCustomConditionDemo {
  
      @Test
      @EnabledIf("example.ExternalCondition#customCondition")
      void enabled() {
          // ...
      }
  }
  
  class ExternalCondition {
  
      static boolean customCondition() {
          return true;
      }
  }
  ```
  
  **在在类级别使用@EnabledIf 或@DisabledIf 时，条件方法必须始终是静态的。位于外部类中的条件方法也必须是静态的。**

## 标记和过滤

测试类和方法可以使用@Tag注解进行标记，这些标记可以被过滤。

```java
import org.junit.jupiter.api.Tag;
import org.junit.jupiter.api.Test;

@Tag("fast")
@Tag("model")
class TaggingDemo {

    @Test
    @Tag("taxes")
    void testingTaxCalculation() {
    }

}
```

## 测试执行顺序

- 方法顺序
  
  要控制测试方法的执行顺序，使用@TestMethodOrder注解测试类或测试接口，并指定所需的MethodOrderer实现。可以使用自定义的MethodOrderer实现或是使用以下内置的MethodOrderer实现：
  
  1. MethodOrderer.DisplayName：根据显示名称按字母数字对测试方法进行排序。
  
  2. MethodOrderer.MethodName：根据名称和形式参数列表按字母数字对测试方法进行排序。
  
  3. MethodOrderer.OrderAnnotation：根据@Order 注解指定的值对测试方法进行数字排序。
  
  4. MethodOrderer.Random：对测试方法进行伪随机排序并支持自定义种子的配置。
  
  5. MethodOrderer.Alphanumeric：根据测试方法的名称和形式参数列表按字母数字对测试方法进行排序； 不推荐使用 MethodOrderer.MethodName，将在 6.0 中删除。
  
  以下展示了如何使用@Order注解指定测试顺序：
  
  ```java
  import org.junit.jupiter.api.MethodOrderer.OrderAnnotation;
  import org.junit.jupiter.api.Order;
  import org.junit.jupiter.api.Test;
  import org.junit.jupiter.api.TestMethodOrder;
  
  @TestMethodOrder(OrderAnnotation.class)
  class OrderedTestsDemo {
  
      @Test
      @Order(1)
      void nullValues() {
          // perform assertions against null values
      }
  
      @Test
      @Order(2)
      void emptyValues() {
          // perform assertions against empty values
      }
  
      @Test
      @Order(3)
      void validValues() {
          // perform assertions against valid values
      }
  
  }
  ```

- 设置默认的测试方法顺序
  
  在配置文件中（如src/test/resources/junit-platform.properties）使用`junit.jupiter.testmethod.order.default`属性设置默认顺序：
  
  ```properties
  junit.jupiter.testmethod.order.default = org.junit.jupiter.api.MethodOrderer$OrderAnnotation
  ```

- 类顺序
  
  您可以实现自己的自定义 ClassOrderer 或使用以下内置 ClassOrderer 实现之一：
  
  1. ClassOrderer.ClassName：根据完全限定的类名按字母数字对测试类进行排序。
  
  2. ClassOrderer.DisplayName：根据显示名称按字母数字对测试类进行排序。
  
  3. ClassOrderer.OrderAnnotation：根据通过@Order 注解指定的值对测试类进行数字排序。
  
  4. ClassOrderer.Random：对测试类进行伪随机排序并支持自定义种子的配置。

- 设置默认的测试类顺序
  
  在配置文件中使用`junit.jupiter.testclass.order.default`属性设置默认类顺序：
  
  ```properties
  junit.jupiter.testclass.order.default = org.junit.jupiter.api.ClassOrderer$OrderAnnotation
  ```
  
  要在本地为 @Nested 测试类配置测试类执行顺序，请在要排序的 @Nested 测试类的封闭类上声明 @TestClassOrder 注释，并提供对要直接使用的 ClassOrderer 实现的类引用 @TestClassOrder 注释。 配置的 ClassOrderer 将递归地应用于@Nested 测试类及其@Nested 测试类。
  
  ```java
  import org.junit.jupiter.api.ClassOrderer;
  import org.junit.jupiter.api.Nested;
  import org.junit.jupiter.api.Order;
  import org.junit.jupiter.api.Test;
  import org.junit.jupiter.api.TestClassOrder;
  
  @TestClassOrder(ClassOrderer.OrderAnnotation.class)
  class OrderedNestedTestClassesDemo {
  
      @Nested
      @Order(1)
      class PrimaryTests {
  
          @Test
          void test1() {
          }
      }
  
      @Nested
      @Order(2)
      class SecondaryTests {
  
          @Test
          void test2() {
          }
      }
  }
  ```

## 测试实例生命周期

为了允许单独执行各个测试方法并避免由于可变测试实例状态而导致的副作用，JUnit在执行每个测试方法之前为每个测试类创建一个新的实例。这是JUnit Jupiter的默认行为。

**注意：如果通过条件（例如，@Disabled、@DisabledOnOs 等）禁用了给定的测试方法，即使“per-method”测试实例生命周期模式处于活动状态，测试类仍将被实例化。**

如果您希望 JUnit Jupiter 在同一个测试实例上执行所有测试方法，请使用 @TestInstance(Lifecycle.PER_CLASS) 注释您的测试类。 使用此模式时，运行每个测试类之前只会创建一个测试实例。因此，如果您的测试方法依赖于存储在实例变量中的状态，您可能需要在 @BeforeEach 或 @AfterEach 方法中重置该状态。

“per-class”模式比默认的“per-method”模式有一些额外的好处。 具体来说，使用“per-class”模式，可以在非静态方法以及接口默认方法上声明@BeforeAll 和@AfterAll。 因此，“per-class”模式还可以在@Nested 测试类中使用@BeforeAll 和@AfterAll 方法。

如果测试类或测试接口没有使用@TestInstance 注释，JUnit Jupiter 将使用默认的生命周期模式。 标准默认模式是 PER_METHOD； 但是，可以更改执行整个测试计划的默认值。 要更改默认测试实例生命周期模式，请将 junit.jupiter.testinstance.lifecycle.default 配置参数设置为 TestInstance.Lifecycle 中定义的枚举常量的名称，忽略大小写。 这可以作为 JVM 系统属性、作为传递给 Launcher 的 LauncherDiscoveryRequest 中的配置参数或通过 JUnit 平台配置文件提供。

```properties
-Djunit.jupiter.testinstance.lifecycle.default=per_class
```

**如果构建将“per-class”语义配置为默认值，但 IDE 中的测试使用“per-method”语义执行，则可能难以调试构建服务器上发生的错误**。所以通常会使用配置文件的方式更改默认生命周期模式，如在配置文件（src/test/resources/junit-platform.properties）中设置值：

```properties
junit.jupiter.testinstance.lifecycle.default = per_class
```

## 嵌套测试

@Nested 测试为测试编写者提供了更多的能力来表达几组测试之间的关系。 这种嵌套测试利用了 Java 的嵌套类。

```java
import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assertions.assertFalse;
import static org.junit.jupiter.api.Assertions.assertThrows;
import static org.junit.jupiter.api.Assertions.assertTrue;

import java.util.EmptyStackException;
import java.util.Stack;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Nested;
import org.junit.jupiter.api.Test;

@DisplayName("A stack")
class TestingAStackDemo {

    Stack<Object> stack;

    @Test
    @DisplayName("is instantiated with new Stack()")
    void isInstantiatedWithNew() {
        new Stack<>();
    }

    @Nested
    @DisplayName("when new")
    class WhenNew {

        @BeforeEach
        void createNewStack() {
            stack = new Stack<>();
        }

        @Test
        @DisplayName("is empty")
        void isEmpty() {
            assertTrue(stack.isEmpty());
        }

        @Test
        @DisplayName("throws EmptyStackException when popped")
        void throwsExceptionWhenPopped() {
            assertThrows(EmptyStackException.class, stack::pop);
        }

        @Test
        @DisplayName("throws EmptyStackException when peeked")
        void throwsExceptionWhenPeeked() {
            assertThrows(EmptyStackException.class, stack::peek);
        }

        @Nested
        @DisplayName("after pushing an element")
        class AfterPushing {

            String anElement = "an element";

            @BeforeEach
            void pushAnElement() {
                stack.push(anElement);
            }

            @Test
            @DisplayName("it is no longer empty")
            void isNotEmpty() {
                assertFalse(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when popped and is empty")
            void returnElementWhenPopped() {
                assertEquals(anElement, stack.pop());
                assertTrue(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when peeked but remains not empty")
            void returnElementWhenPeeked() {
                assertEquals(anElement, stack.peek());
                assertFalse(stack.isEmpty());
            }
        }
    }
}
```

## 构造函数和方法的依赖注入

在所有以前的JUnit版本中，测试构造函数或方法都不允许有参数（至少在标准Runner实现中不允许）。现在JUnit Jupiter允许测试构造函数和方法具有参数。这允许更大的灵活性并为构造函数和方法启用依赖注入。

ParameterResolver 为希望在运行时动态解析参数的测试扩展定义 API。 如果测试类构造函数、测试方法或生命周期方法接受参数，则该参数必须在运行时由已注册的ParameterResolver 解析。

目前有三个自动注册的内置解析器：

1. TestInfoParameterResolver：如果构造函数或方法参数的类型为 TestInfo，则 TestInfoParameterResolver 将提供与当前容器或测试对应的 TestInfo 实例作为参数的值。 然后可以使用 TestInfo 检索有关当前容器或测试的信息，例如显示名称、测试类、测试方法和相关标记。 显示名称可以是技术名称，例如测试类或测试方法的名称，也可以是通过 @DisplayName 配置的自定义名称。
   
   ```java
   import static org.junit.jupiter.api.Assertions.assertEquals;
   import static org.junit.jupiter.api.Assertions.assertTrue;
   
   import org.junit.jupiter.api.BeforeEach;
   import org.junit.jupiter.api.DisplayName;
   import org.junit.jupiter.api.Tag;
   import org.junit.jupiter.api.Test;
   import org.junit.jupiter.api.TestInfo;
   
   @DisplayName("TestInfo Demo")
   class TestInfoDemo {
   
       TestInfoDemo(TestInfo testInfo) {
           assertEquals("TestInfo Demo", testInfo.getDisplayName());
       }
   
       @BeforeEach
       void init(TestInfo testInfo) {
           String displayName = testInfo.getDisplayName();
           assertTrue(displayName.equals("TEST 1") || displayName.equals("test2()"));
       }
   
       @Test
       @DisplayName("TEST 1")
       @Tag("my-tag")
       void test1(TestInfo testInfo) {
           assertEquals("TEST 1", testInfo.getDisplayName());
           assertTrue(testInfo.getTags().contains("my-tag"));
       }
   
       @Test
       void test2() {
       }
   
   }
   ```

2. RepetitionInfoParameterResolver：如果@RepeatedTest、@BeforeEach 或@AfterEach 方法中的方法参数是RepetitionInfo 类型，则RepetitionInfoParameterResolver 将提供RepetitionInfo 的实例。 RepetitionInfo 然后可用于检索有关当前重复的信息以及相应@RepeatedTest 的重复总数。 但是请注意，RepetitionInfoParameterResolver 没有在@RepeatedTest 的上下文之外注册。

3. TestReporterParameterResolver：如果构造函数或方法参数的类型为 TestReporter，则 TestReporterParameterResolver 将提供 TestReporter 的实例。 TestReporter 可用于发布有关当前测试运行的附加数据。 数据可以通过 TestExecutionListener 中的 reportingEntryPublished() 方法使用，允许在 IDE 中查看或包含在报告中。
   
   在 JUnit Jupiter 中，您应该使用 TestReporter 来将信息打印到 JUnit 4 中的标准输出或标准错误。使用 @RunWith(JUnitPlatform.class) 会将所有报告的条目输出到标准输出。 此外，一些 IDE 将报告条目打印到标准输出或在用户界面中显示它们以获取测试结果。
   
   ```java
   class TestReporterDemo {
   
       @Test
       void reportSingleValue(TestReporter testReporter) {
           testReporter.publishEntry("a status message");
       }
   
       @Test
       void reportKeyValuePair(TestReporter testReporter) {
           testReporter.publishEntry("a key", "a value");
       }
   
       @Test
       void reportMultipleKeyValuePairs(TestReporter testReporter) {
           Map<String, String> values = new HashMap<>();
           values.put("user name", "dk38");
           values.put("award year", "1974");
   
           testReporter.publishEntry(values);
       }
   
   }
   ```
   
   ```html
   TestIdentifier [reportSingleValue(TestReporter)]
   ReportEntry [timestamp = 2022-02-28T22:07:43.578297100, value = 'a status message']
   TestIdentifier [reportMultipleKeyValuePairs(TestReporter)]
   ReportEntry [timestamp = 2022-02-28T22:07:43.587273600, user name = 'dk38', award year = '1974']
   TestIdentifier [reportKeyValuePair(TestReporter)]
   ReportEntry [timestamp = 2022-02-28T22:07:43.589268300, a key = 'a value']
   ```
   
   其他参数解析器必须通过@ExtendWith 注册适当的扩展来显式启用。
   
   查看 RandomParametersExtension 以获取自定义 ParameterResolver 的示例。MyRandomParametersTest 演示了如何将随机值注入 @Test 方法。
   
   ```java
   @ExtendWith(RandomParametersExtension.class)
   class MyRandomParametersTest {
   
       @Test
       void injectsInteger(@Random int i, @Random int j) {
           assertNotEquals(i, j);
       }
   
       @Test
       void injectsDouble(@Random double d) {
           assertEquals(0.0, d, 1.0);
       }
   
   }
   ```

## 测试接口和默认方法

JUnit Jupiter 允许在接口默认方法上声明 @Test、@RepeatedTest、@ParameterizedTest、@TestFactory、@TestTemplate、@BeforeEach 和 @AfterEach。 @BeforeAll 和 @AfterAll 可以在测试接口中的静态方法上声明，如果测试接口或测试类使用 @TestInstance(Lifecycle.PER_CLASS) 注释，则可以在接口默认方法上声明。

```java
@TestInstance(Lifecycle.PER_CLASS)
interface TestLifecycleLogger {

    static final Logger logger = Logger.getLogger(TestLifecycleLogger.class.getName());

    @BeforeAll
    default void beforeAllTests() {
        logger.info("Before all tests");
    }

    @AfterAll
    default void afterAllTests() {
        logger.info("After all tests");
    }

    @BeforeEach
    default void beforeEachTest(TestInfo testInfo) {
        logger.info(() -> String.format("About to execute [%s]",
            testInfo.getDisplayName()));
    }

    @AfterEach
    default void afterEachTest(TestInfo testInfo) {
        logger.info(() -> String.format("Finished executing [%s]",
            testInfo.getDisplayName()));
    }

}
```

## 重复测试

JUnit Jupiter 通过使用 @RepeatedTest 注释方法并指定所需的重复总数，提供了将测试重复指定次数的能力。 重复测试的每次调用都像执行常规 @Test 方法一样，完全支持相同的生命周期回调和扩展。

```java
@RepeatedTest(10)
void repeatedTest() {
    // ...
}
```

除了指定重复次数之外，还可以通过 @RepeatedTest 注释的 name 属性为每个重复配置自定义显示名称。 此外，显示名称可以是由静态文本和动态占位符组合而成的模式。 当前支持以下占位符：

- {displayName}：@RepeatedTest 方法的显示名称

- {currentRepetition}：当前重复次数

- {totalRepetitions}：总重复次数

为了以编程方式检索有关当前重复和重复总数的信息，开发人员可以选择将 RepetitionInfo 的实例注入到 @RepeatedTest、@BeforeEach 或 @AfterEach 方法中。

RepeatedTestsDemo 类演示了重复测试的几个示例：

```java
import static org.junit.jupiter.api.Assertions.assertEquals;

import java.util.logging.Logger;

import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.RepeatedTest;
import org.junit.jupiter.api.RepetitionInfo;
import org.junit.jupiter.api.TestInfo;

class RepeatedTestsDemo {

    private Logger logger = // ...

    @BeforeEach
    void beforeEach(TestInfo testInfo, RepetitionInfo repetitionInfo) {
        int currentRepetition = repetitionInfo.getCurrentRepetition();
        int totalRepetitions = repetitionInfo.getTotalRepetitions();
        String methodName = testInfo.getTestMethod().get().getName();
        logger.info(String.format("About to execute repetition %d of %d for %s", //
            currentRepetition, totalRepetitions, methodName));
    }

    @RepeatedTest(10)
    void repeatedTest() {
        // ...
    }

    @RepeatedTest(5)
    void repeatedTestWithRepetitionInfo(RepetitionInfo repetitionInfo) {
        assertEquals(5, repetitionInfo.getTotalRepetitions());
    }

    @RepeatedTest(value = 1, name = "{displayName} {currentRepetition}/{totalRepetitions}")
    @DisplayName("Repeat!")
    void customDisplayName(TestInfo testInfo) {
        assertEquals("Repeat! 1/1", testInfo.getDisplayName());
    }

    @RepeatedTest(value = 1, name = RepeatedTest.LONG_DISPLAY_NAME)
    @DisplayName("Details...")
    void customDisplayNameWithLongPattern(TestInfo testInfo) {
        assertEquals("Details... :: repetition 1 of 1", testInfo.getDisplayName());
    }

    @RepeatedTest(value = 5, name = "Wiederholung {currentRepetition} von {totalRepetitions}")
    void repeatedTestInGerman() {
        // ...
    }

}
```

```html
├─ RepeatedTestsDemo ✔
│  ├─ repeatedTest() ✔
│  │  ├─ repetition 1 of 10 ✔
│  │  ├─ repetition 2 of 10 ✔
│  │  ├─ repetition 3 of 10 ✔
│  │  ├─ repetition 4 of 10 ✔
│  │  ├─ repetition 5 of 10 ✔
│  │  ├─ repetition 6 of 10 ✔
│  │  ├─ repetition 7 of 10 ✔
│  │  ├─ repetition 8 of 10 ✔
│  │  ├─ repetition 9 of 10 ✔
│  │  └─ repetition 10 of 10 ✔
│  ├─ repeatedTestWithRepetitionInfo(RepetitionInfo) ✔
│  │  ├─ repetition 1 of 5 ✔
│  │  ├─ repetition 2 of 5 ✔
│  │  ├─ repetition 3 of 5 ✔
│  │  ├─ repetition 4 of 5 ✔
│  │  └─ repetition 5 of 5 ✔
│  ├─ Repeat! ✔
│  │  └─ Repeat! 1/1 ✔
│  ├─ Details... ✔
│  │  └─ Details... :: repetition 1 of 1 ✔
│  └─ repeatedTestInGerman() ✔
│     ├─ Wiederholung 1 von 5 ✔
│     ├─ Wiederholung 2 von 5 ✔
│     ├─ Wiederholung 3 von 5 ✔
│     ├─ Wiederholung 4 von 5 ✔
│     └─ Wiederholung 5 von 5 ✔
```

## 参数化测试

参数化测试可以使用不同的参数多次运行测试。 它们的声明方式与常规的 @Test 方法一样，但使用 @ParameterizedTest 注释。 此外，您必须声明至少一个将为每次调用提供参数的源，然后在测试方法中使用这些参数。

以下示例演示了一个参数化测试，该测试使用 @ValueSource 批注将字符串数组指定为参数源。

```java
@ParameterizedTest
@ValueSource(strings = { "racecar", "radar", "able was I ere I saw elba" })
void palindromes(String candidate) {
    assertTrue(StringUtils.isPalindrome(candidate));
}
```

**为了使用参数化测试，您需要添加对 junit-jupiter-params 的依赖项。**

开箱即用的 JUnit Jupiter 提供了相当多的源注释。

1. @ValueSource
   
   @ValueSource 是最简单的来源之一。 它允许您指定单个文字值数组，并且只能用于为每个参数化测试调用提供单个参数。
   
   - `short`
   
   - `byte`
   
   - `int`
   
   - `long`
   
   - `float`
   
   - `double`
   
   - `char`
   
   - `boolean`
   
   - `java.lang.String`
   
   - `java.lang.Class`

为了检查极端情况并验证我们的软件在提供错误输入时的正确行为，将null和空值提供给我们的参数化测试可能很有用。

- @NullSource：为带 @ParameterizedTest 注解的方法提供单个 null 参数。不能用于具有原始类型的参数。

- @EmptySource：为带@ParameterizedTest 注解的方法为以下类型的参数提供单个空参数：java.lang.String、java.util.List、java.util.Set、java.util.Map、原始数组（例如， int[]、char[][] 等）、对象数组（例如，String[]、Integer[][] 等）。不支持受支持类型的子类型。

- @NullAndEmptySource：组合注释，结合了@NullSource 和@EmptySource 的功能。

如果您需要为参数化测试提供多种不同类型的空白字符串，您可以使用@ValueSource - 例如@ValueSource(strings = {" ", " ", "\t", "\n"}) .

您还可以结合@NullSource、@EmptySource 和@ValueSource 来测试更广泛的null、empty和blank input。 以下示例演示了如何为字符串实现此目的。

```java
@ParameterizedTest
@NullSource
@EmptySource
@ValueSource(strings = { " ", "   ", "\t", "\n" })
void nullEmptyAndBlankStrings(String text) {
    assertTrue(text == null || text.trim().isEmpty());
}
```

使用组合的 @NullAndEmptySource 注释将上述内容简化如下。

```java
@ParameterizedTest
@NullAndEmptySource
@ValueSource(strings = { " ", "   ", "\t", "\n" })
void nullEmptyAndBlankStrings(String text) {
    assertTrue(text == null || text.trim().isEmpty());
}
```

2. @EnumSource
   
   @EnumSource 提供了一种使用 Enum 常量的便捷方式。
   
   ```java
   @ParameterizedTest
   @EnumSource
   void testWithEnumSourceWithAutoDetection(ChronoUnit unit) {
      assertNotNull(unit);
   }
   ```
   
   指定枚举中的部分：
   
   ```java
   @ParameterizedTest
   @EnumSource(names = { "DAYS", "HOURS" })
   void testWithEnumSourceInclude(ChronoUnit unit) {
      assertTrue(EnumSet.of(ChronoUnit.DAYS, ChronoUnit.HOURS).contains(unit));
   }
   ```
   
   从枚举中排除或指定正则表达式：
   
   ```java
   @ParameterizedTest
   @EnumSource(mode = EXCLUDE, names = { "ERAS", "FOREVER" })
   void testWithEnumSourceExclude(ChronoUnit unit) {
      assertFalse(EnumSet.of(ChronoUnit.ERAS, ChronoUnit.FOREVER).contains(unit));
   }
   ```
   
   ```java
   @ParameterizedTest
   @EnumSource(mode = MATCH_ALL, names = "^.*DAYS$")
   void testWithEnumSourceRegex(ChronoUnit unit) {
      assertTrue(unit.name().endsWith("DAYS"));
   }
   ```

3. @MethodSource
   
   @MethodSource 允许您引用测试类或外部类的一个或多个工厂方法。
   
    测试类中的工厂方法必须是静态的，除非测试类使用 @TestInstance(Lifecycle.PER_CLASS)注释。而外部类中的工厂方法必须始终是静态的。此外，此类工厂方法不得接受任何参数。
   
   每个工厂方法都必须生成一个参数流，并且流中的每组参数都将作为物理参数提供给带注释的@ParameterizedTest 方法的单独调用。一般来说，这会转化为参数流（即 Stream<Arguments>）；但是，实际的具体返回类型可以采用多种形式。在此上下文中，“流”是 JUnit 可以可靠地转换为 Stream 的任何内容，例如 Stream、DoubleStream、LongStream、IntStream、Collection、Iterator、Iterable、对象数组或基元数组。如果参数化测试方法接受单个参数，则流中的“参数”可以作为参数的实例、对象数组（例如 Object[]）或单个值提供。
   
   ```java
   @ParameterizedTest
   @MethodSource("stringProvider")
   void testWithExplicitLocalMethodSource(String argument) {
       assertNotNull(argument);
   }
   
   static Stream<String> stringProvider() {
       return Stream.of("apple", "banana");
   }
   ```
   
   If you do not explicitly provide a factory method name via `@MethodSource`, JUnit Jupiter will search for a *factory* method that has the same name as the current `@ParameterizedTest` method by convention. This is demonstrated in the following example.
   
   ```java
   @ParameterizedTest
   @MethodSource
   void testWithDefaultLocalMethodSource(String argument) {
       assertNotNull(argument);
   }
   
   static Stream<String> testWithDefaultLocalMethodSource() {
       return Stream.of("apple", "banana");
   }
   ```
   
   如果参数化测试方法声明了多个参数，则需要返回 Arguments 实例或对象数组的集合、流或数组，如下所示
   
   ```java
   @ParameterizedTest
   @MethodSource("stringIntAndListProvider")
   void testWithMultiArgMethodSource(String str, int num, List<String> list) {
       assertEquals(5, str.length());
       assertTrue(num >=1 && num <=2);
       assertEquals(2, list.size());
   }
   
   static Stream<Arguments> stringIntAndListProvider() {
       return Stream.of(
           arguments("apple", 1, Arrays.asList("a", "b")),
           arguments("lemon", 2, Arrays.asList("x", "y"))
       );
   }
   ```
   
   可以通过提供其完全限定的方法名称来引用外部静态工厂方法，如下例所示。
   
   ```java
   package example;
   
   import java.util.stream.Stream;
   
   import org.junit.jupiter.params.ParameterizedTest;
   import org.junit.jupiter.params.provider.MethodSource;
   
   class ExternalMethodSourceDemo {
   
       @ParameterizedTest
       @MethodSource("example.StringsProviders#tinyStrings")
       void testWithExternalMethodSource(String tinyString) {
           // test with tiny string
       }
   }
   
   class StringsProviders {
   
       static Stream<String> tinyStrings() {
           return Stream.of(".", "oo", "OOO");
       }
   }
   ```

4. @CsvSource
   
   @CsvSource 允许您将参数列表表示为逗号分隔值（即 CSV 字符串文字）。 通过 @CsvSource 中的 value 属性提供的每个字符串都代表一个 CSV 记录，并导致一次参数化测试的调用。 第一条记录可以选择用于提供 CSV 标头。
   
   ```java
   @ParameterizedTest
   @CsvSource({
       "apple,         1",
       "banana,        2",
       "'lemon, lime', 0xF1",
       "strawberry,    700_000"
   })
   void testWithCsvSource(String fruit, int rank) {
       assertNotNull(fruit);
       assertNotEquals(0, rank);
   }
   ```
   
   默认分隔符是逗号 (,)，但您可以通过设置 delimiter 属性来使用其他字符。 或者，delimiterString 属性允许您使用字符串分隔符而不是单个字符。 但是，不能同时设置两个分隔符属性。
   
   默认情况下，@CsvSource 使用单引号 (') 作为其引号字符，但这可以通过 quoteCharacter 属性进行更改。
   
   | 示例输入                                                                                    | 结果                            |
   |:--------------------------------------------------------------------------------------- | ----------------------------- |
   | `@CsvSource({ "apple, banana" })`                                                       | `"apple"`, `"banana"`         |
   | `@CsvSource({ "apple, 'lemon, lime'" })`                                                | `"apple"`, `"lemon, lime"`    |
   | `@CsvSource({ "apple, ''" })`                                                           | `"apple"`, `""`               |
   | `@CsvSource({ "apple, " })`                                                             | `"apple"`, `null`             |
   | `@CsvSource(value = { "apple, banana, NIL" }, nullValues = "NIL")`                      | `"apple"`, `"banana"`, `null` |
   | `@CsvSource(value = { " apple , banana" }, ignoreLeadingAndTrailingWhitespace = false)` | `" apple "`, `" banana"`      |

5. @CsvFileSource
   
   @CsvFileSource 允许您使用来自类路径或本地文件系统的逗号分隔值 (CSV) 文件。 CSV 文件中的每条记录都会导致一次参数化测试的调用。 第一条记录可以选择用于提供 CSV 标头。 您可以通过 numLinesToSkip 属性指示 JUnit 忽略标题。 如果您希望在显示名称中使用标题，可以将 useHeadersInDisplayName 属性设置为 true。 下面的示例演示了 numLinesToSkip 和 useHeadersInDisplayName 的使用。
   
   默认分隔符是逗号 (,)，但您可以通过设置 delimiter 属性来使用其他字符。 或者，delimiterString 属性允许您使用字符串分隔符而不是单个字符。 但是，不能同时设置两个分隔符属性。
   
   ```java
   @ParameterizedTest
   @CsvFileSource(resources = "/two-column.csv", numLinesToSkip = 1)
   void testWithCsvFileSourceFromClasspath(String country, int reference) {
       assertNotNull(country);
       assertNotEquals(0, reference);
   }
   
   @ParameterizedTest
   @CsvFileSource(files = "src/test/resources/two-column.csv", numLinesToSkip = 1)
   void testWithCsvFileSourceFromFile(String country, int reference) {
       assertNotNull(country);
       assertNotEquals(0, reference);
   }
   
   @ParameterizedTest(name = "[{index}] {arguments}")
   @CsvFileSource(resources = "/two-column.csv", useHeadersInDisplayName = true)
   void testWithCsvFileSourceAndHeaders(String country, int reference) {
       assertNotNull(country);
       assertNotEquals(0, reference);
   }
   ```
   
   与@CsvSource 中使用的默认语法相比，@CsvFileSource 默认使用双引号 (") 作为引号字符，但这可以通过 quoteCharacter 属性进行更改。

6. @ArgumentsSource
   
   @ArgumentsSource 可用于指定自定义的、可重用的 ArgumentsProvider。 请注意，必须将 ArgumentsProvider 的实现声明为顶级类或静态嵌套类。
   
   ```java
   @ParameterizedTest
   @ArgumentsSource(MyArgumentsProvider.class)
   void testWithArgumentsSource(String argument) {
       assertNotNull(argument);
   }
   ```
   
   ```java
   public class MyArgumentsProvider implements ArgumentsProvider {
   
       @Override
       public Stream<? extends Arguments> provideArguments(ExtensionContext context) {
           return Stream.of("apple", "banana").map(Arguments::of);
       }
   }
   ```

### 参数转换

- 扩大转换 
  
  JUnit Jupiter 支持对提供给 @ParameterizedTest 的参数进行扩展原始转换。 例如，使用 @ValueSource(ints = { 1, 2, 3 }) 注释的参数化测试不仅可以声明为接受 int 类型的参数，还可以接受 long、float 或 double 类型的参数。

- 隐式转换
  
  为了支持像@CsvSource 这样的用例，JUnit Jupiter 提供了许多内置的隐式类型转换器。 转换过程取决于每个方法参数的声明类型。
  
  字符串实例被隐式转换为以下目标类型。
  
  | Target Type                | Example                                                                                                           |
  | -------------------------- | ----------------------------------------------------------------------------------------------------------------- |
  | `boolean`/`Boolean`        | `"true"` → `true`                                                                                                 |
  | `byte`/`Byte`              | `"15"`, `"0xF"`, or `"017"` → `(byte) 15`                                                                         |
  | `char`/`Character`         | `"o"` → `'o'`                                                                                                     |
  | `short`/`Short`            | `"15"`, `"0xF"`, or `"017"` → `(short) 15`                                                                        |
  | `int`/`Integer`            | `"15"`, `"0xF"`, or `"017"` → `15`                                                                                |
  | `long`/`Long`              | `"15"`, `"0xF"`, or `"017"` → `15L`                                                                               |
  | `float`/`Float`            | `"1.0"` → `1.0f`                                                                                                  |
  | `double`/`Double`          | `"1.0"` → `1.0d`                                                                                                  |
  | `Enum` subclass            | `"SECONDS"` → `TimeUnit.SECONDS`                                                                                  |
  | `java.io.File`             | `"/path/to/file"` → `new File("/path/to/file")`                                                                   |
  | `java.lang.Class`          | `"java.lang.Integer"` → `java.lang.Integer.class` *(use `$` for nested classes, e.g. `"java.lang.Thread$State"`)* |
  | `java.lang.Class`          | `"byte"` → `byte.class` *(primitive types are supported)*                                                         |
  | `java.lang.Class`          | `"char[]"` → `char[].class` *(array types are supported)*                                                         |
  | `java.math.BigDecimal`     | `"123.456e789"` → `new BigDecimal("123.456e789")`                                                                 |
  | `java.math.BigInteger`     | `"1234567890123456789"` → `new BigInteger("1234567890123456789")`                                                 |
  | `java.net.URI`             | `"https://junit.org/"` → `URI.create("https://junit.org/")`                                                       |
  | `java.net.URL`             | `"https://junit.org/"` → `new URL("https://junit.org/")`                                                          |
  | `java.nio.charset.Charset` | `"UTF-8"` → `Charset.forName("UTF-8")`                                                                            |
  | `java.nio.file.Path`       | `"/path/to/file"` → `Paths.get("/path/to/file")`                                                                  |
  | `java.time.Duration`       | `"PT3S"` → `Duration.ofSeconds(3)`                                                                                |
  | `java.time.Instant`        | `"1970-01-01T00:00:00Z"` → `Instant.ofEpochMilli(0)`                                                              |
  | `java.time.LocalDateTime`  | `"2017-03-14T12:34:56.789"` → `LocalDateTime.of(2017, 3, 14, 12, 34, 56, 789_000_000)`                            |
  | `java.time.LocalDate`      | `"2017-03-14"` → `LocalDate.of(2017, 3, 14)`                                                                      |
  | `java.time.LocalTime`      | `"12:34:56.789"` → `LocalTime.of(12, 34, 56, 789_000_000)`                                                        |
  | `java.time.MonthDay`       | `"--03-14"` → `MonthDay.of(3, 14)`                                                                                |
  | `java.time.OffsetDateTime` | `"2017-03-14T12:34:56.789Z"` → `OffsetDateTime.of(2017, 3, 14, 12, 34, 56, 789_000_000, ZoneOffset.UTC)`          |
  | `java.time.OffsetTime`     | `"12:34:56.789Z"` → `OffsetTime.of(12, 34, 56, 789_000_000, ZoneOffset.UTC)`                                      |
  | `java.time.Period`         | `"P2M6D"` → `Period.of(0, 2, 6)`                                                                                  |
  | `java.time.YearMonth`      | `"2017-03"` → `YearMonth.of(2017, 3)`                                                                             |
  | `java.time.Year`           | `"2017"` → `Year.of(2017)`                                                                                        |
  | `java.time.ZonedDateTime`  | `"2017-03-14T12:34:56.789Z"` → `ZonedDateTime.of(2017, 3, 14, 12, 34, 56, 789_000_000, ZoneOffset.UTC)`           |
  | `java.time.ZoneId`         | `"Europe/Berlin"` → `ZoneId.of("Europe/Berlin")`                                                                  |
  | `java.time.ZoneOffset`     | `"+02:30"` → `ZoneOffset.ofHoursMinutes(2, 30)`                                                                   |
  | `java.util.Currency`       | `"JPY"` → `Currency.getInstance("JPY")`                                                                           |
  | `java.util.Locale`         | `"en"` → `new Locale("en")`                                                                                       |
  | `java.util.UUID`           | `"d043e930-7b3b-48e3-bdbe-5a3ccfb833db"` → `UUID.fromString("d043e930-7b3b-48e3-bdbe-5a3ccfb833db")`              |
  
  除了从字符串到上表中列出的目标类型的隐式转换之外，如果目标类型恰好声明了一个合适的工厂方法或工厂构造函数，JUnit Jupiter 还提供了一种从 String 到给定目标类型的自动转换的回退机制 定义如下。
  
   工厂方法：在目标类型中声明的非私有静态方法，它接受单个字符串参数并返回目标类型的实例。 方法的名称可以是任意的，不需要遵循任何特定的约定。
  
   工厂构造函数：目标类型中接受单个字符串参数的非私有构造函数。 请注意，目标类型必须声明为顶级类或静态嵌套类。
  
  ```java
  @ParameterizedTest
  @ValueSource(strings = "42 Cats")
  void testWithImplicitFallbackArgumentConversion(Book book) {
      assertEquals("42 Cats", book.getTitle());
  }
  ```
  
  ```java
  public class Book {
  
      private final String title;
  
      private Book(String title) {
          this.title = title;
      }
  
      public static Book fromTitle(String title) {
          return new Book(title);
      }
  
      public String getTitle() {
          return this.title;
      }
  }
  ```

- 显示转换
  
  您可以使用 @ConvertWith 注释显式指定 ArgumentConverter 用于某个参数，而不是依赖于隐式参数转换，如下例所示。 请注意，必须将 ArgumentConverter 的实现声明为顶级类或静态嵌套类。
  
  ```java
  @ParameterizedTest
  @EnumSource(ChronoUnit.class)
  void testWithExplicitArgumentConversion(
          @ConvertWith(ToStringArgumentConverter.class) String argument) {
  
      assertNotNull(ChronoUnit.valueOf(argument));
  }
  ```
  
  ```java
  public class ToStringArgumentConverter extends SimpleArgumentConverter {
  
      @Override
      protected Object convert(Object source, Class<?> targetType) {
          assertEquals(String.class, targetType, "Can only convert to String");
          if (source instanceof Enum<?>) {
              return ((Enum<?>) source).name();
          }
          return String.valueOf(source);
      }
  }
  ```
  
  如果转换器仅用于将一种类型转换为另一种类型，则可以扩展 TypedArgumentConverter 以避免类型检查
  
  ```java
  public class ToLengthArgumentConverter extends TypedArgumentConverter<String, Integer> {
  
      protected ToLengthArgumentConverter() {
          super(String.class, Integer.class);
      }
  
      @Override
      protected Integer convert(String source) {
          return (source != null ? source.length() : 0);
      }
  
  }
  ```

- 参数聚合
  
  默认情况下，提供给 @ParameterizedTest 方法的每个参数都对应一个方法参数。 因此，预期会提供大量参数的参数源可能会导致较大的方法签名。
  
   在这种情况下，可以使用 ArgumentsAccessor 代替多个参数。 使用此 API，您可以通过传递给测试方法的单个参数访问提供的参数。 此外，支持类型转换，如隐式转换中所述。
  
  ```java
  @ParameterizedTest
  @CsvSource({
      "Jane, Doe, F, 1990-05-20",
      "John, Doe, M, 1990-10-22"
  })
  void testWithArgumentsAccessor(ArgumentsAccessor arguments) {
      Person person = new Person(arguments.getString(0),
                                 arguments.getString(1),
                                 arguments.get(2, Gender.class),
                                 arguments.get(3, LocalDate.class));
  
      if (person.getFirstName().equals("Jane")) {
          assertEquals(Gender.F, person.getGender());
      }
      else {
          assertEquals(Gender.M, person.getGender());
      }
      assertEquals("Doe", person.getLastName());
      assertEquals(1990, person.getDateOfBirth().getYear());
  }
  ```
  
  - 自定义聚合器
    
    除了使用 ArgumentsAccessor 直接访问 @ParameterizedTest 方法的参数外，JUnit Jupiter 还支持使用自定义的、可重用的聚合器。
    
    要使用自定义聚合器，请实现 ArgumentsAggregator 接口并通过 @ParameterizedTest 方法中兼容参数上的 @AggregateWith 注释注册它。 当调用参数化测试时，聚合的结果将作为相应参数的参数提供。 请注意，必须将 ArgumentsAggregator 的实现声明为顶级类或静态嵌套类。
    
    ```java
    @ParameterizedTest
    @CsvSource({
        "Jane, Doe, F, 1990-05-20",
        "John, Doe, M, 1990-10-22"
    })
    void testWithArgumentsAggregator(@AggregateWith(PersonAggregator.class) Person person) {
        // perform assertions against person
    }
    ```
    
    ```java
    public class PersonAggregator implements ArgumentsAggregator {
        @Override
        public Person aggregateArguments(ArgumentsAccessor arguments, ParameterContext context) {
            return new Person(arguments.getString(0),
                              arguments.getString(1),
                              arguments.get(2, Gender.class),
                              arguments.get(3, LocalDate.class));
        }
    }
    ```
    
    如果您发现自己在代码库中为多个参数化测试方法反复声明@AggregateWith(MyTypeAggregator.class)，您可能希望创建一个自定义组合注释，例如使用@AggregateWith(MyTypeAggregator.class) 进行元注释的@CsvToMyType。 以下示例使用自定义 @CsvToPerson 注释演示了这一点。
    
    ```java
    @ParameterizedTest
    @CsvSource({
        "Jane, Doe, F, 1990-05-20",
        "John, Doe, M, 1990-10-22"
    })
    void testWithCustomAggregatorAnnotation(@CsvToPerson Person person) {
        // perform assertions against person
    }
    ```
    
    ```java
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.PARAMETER)
    @AggregateWith(PersonAggregator.class)
    public @interface CsvToPerson {
    }
    ```

- 自定义显示名称
  
  默认情况下，参数化测试调用的显示名称包含调用索引和该特定调用的所有参数的字符串表示形式。 如果存在于字节码中（对于 Java，必须使用 -parameters 编译器标志编译测试代码），它们中的每一个前面都有参数名称（除非该参数只能通过 ArgumentsAccessor 或 ArgumentAggregator 获得）。 但是，您可以通过 @ParameterizedTest 注释的 name 属性自定义调用显示名称，如下例所示：
  
  ```java
  @DisplayName("Display name of container")
  @ParameterizedTest(name = "{index} ==> the rank of ''{0}'' is {1}")
  @CsvSource({ "apple, 1", "banana, 2", "'lemon, lime', 3" })
  void testWithCustomDisplayNames(String fruit, int rank) {
  }
  ```
  
  自定义显示名称中支持以下占位符：
  
  | Placeholder            | Description                                                       |
  | ---------------------- | ----------------------------------------------------------------- |
  | `{displayName}`        | the display name of the method                                    |
  | `{index}`              | the current invocation index (1-based)                            |
  | `{arguments}`          | the complete, comma-separated arguments list                      |
  | `{argumentsWithNames}` | the complete, comma-separated arguments list with parameter names |
  | `{0}`, `{1}`, …​       | an individual argument                                            |

- 生命周期和互操作性
  
  参数化测试的每次调用都具有与常规 @Test 方法相同的生命周期。 例如，@BeforeEach 方法将在每次调用之前执行。 与动态测试类似，调用将一一出现在 IDE 的测试树中。 您可以在同一个测试类中随意混合常规的@Test 方法和@ParameterizedTest 方法。

## 测试模板

@TestTemplate 方法不是常规的测试用例，而是测试用例的模板。 因此，它被设计为根据注册提供者返回的调用上下文的数量多次调用。 因此，它必须与已注册的 TestTemplateInvocationContextProvider 扩展一起使用。 测试模板方法的每次调用都像执行常规 @Test 方法一样，完全支持相同的生命周期回调和扩展。

- 动态测试
  
  Annotations 中描述的 JUnit Jupiter 中的标准 @Test 注解与 JUnit 4 中的 @Test 注解非常相似。两者都描述了实现测试用例的方法。这些测试用例是静态的，因为它们是在编译时完全指定的，并且它们的行为不能被运行时发生的任何事情改变。假设提供了动态行为的基本形式，但有意限制了它们的表达能力。
  除了这些标准测试之外，JUnit Jupiter 中还引入了一种全新的测试编程模型。这种新型测试是一种动态测试，它在运行时由带有@TestFactory 注释的工厂方法生成。
  与@Test 方法相比，@TestFactory 方法本身不是测试用例，而是测试用例的工厂。因此，动态测试是工厂的产品。从技术上讲，@TestFactory 方法必须返回单个 DynamicNode 或 Stream、Collection、Iterable、Iterator 或 DynamicNode 实例数组。 DynamicNode 的可实例化子类是 DynamicContainer 和 DynamicTest。 DynamicContainer 实例由显示名称和动态子节点列表组成，可以创建任意嵌套的动态节点层次结构。 DynamicTest 实例将延迟执行，从而实现动态甚至非确定性的测试用例生成。
  @TestFactory 返回的任何 Stream 都将通过调用 stream.close() 正确关闭，从而可以安全地使用 Files.lines() 等资源。
  与@Test 方法一样，@TestFactory 方法不能是私有的或静态的，并且可以选择声明要由 ParameterResolvers 解析的参数。
  DynamicTest 是在运行时生成的测试用例。它由显示名称和可执行文件组成。 Executable 是一个 @FunctionalInterface，这意味着动态测试的实现可以作为 lambda 表达式或方法引用提供。

- 动态测试生命周期
  
  动态测试的执行生命周期与标准 @Test 用例的执行生命周期完全不同。 具体来说，单个动态测试没有生命周期回调。 这意味着 @BeforeEach 和 @AfterEach 方法及其对应的扩展回调是为 @TestFactory 方法执行的，但不是为每个动态测试执行的。 换句话说，如果您从 lambda 表达式中的测试实例访问动态测试的字段，这些字段将不会被回调方法或由同一 @TestFactory 方法生成的各个动态测试的执行之间的扩展重置。
  
  以下 DynamicTestsDemo 类演示了测试工厂和动态测试的几个示例。
  第一种方法返回无效的返回类型。由于在编译时无法检测到无效的返回类型，因此在运行时检测到它时会抛出 JUnitException。
  接下来的六个方法是非常简单的示例，演示了 DynamicTest 实例的 Collection、Iterable、Iterator、数组或 Stream 的生成。这些示例中的大多数并没有真正展示动态行为，而只是展示了原则上支持的返回类型。但是，dynamicTestsFromStream() 和 dynamicTestsFromIntStream() 展示了为给定的字符串集或输入数字范围生成动态测试是多么容易。
  下一种方法本质上是真正动态的。 generateRandomNumberOfTests() 实现了一个生成随机数的迭代器、一个显示名称生成器和一个测试执行器，然后将所有三者提供给 DynamicTest.stream()。尽管 generateRandomNumberOfTests() 的非确定性行为当然与测试可重复性相冲突，因此应谨慎使用，但它有助于展示动态测试的表现力和力量。
  next 方法在灵活性方面类似于 generateRandomNumberOfTests()；但是，dynamicTestsFromStreamFactoryMethod() 通过 DynamicTest.stream() 工厂方法从现有 Stream 生成动态测试流。
  出于演示目的，dynamicNodeSingleTest() 方法生成单个 DynamicTest 而不是流，dynamicNodeSingleContainer() 方法使用 DynamicContainer 生成动态测试的嵌套层次结构。
  
  ```java
  import static example.util.StringUtils.isPalindrome;
  import static org.junit.jupiter.api.Assertions.assertEquals;
  import static org.junit.jupiter.api.Assertions.assertFalse;
  import static org.junit.jupiter.api.Assertions.assertNotNull;
  import static org.junit.jupiter.api.Assertions.assertTrue;
  import static org.junit.jupiter.api.DynamicContainer.dynamicContainer;
  import static org.junit.jupiter.api.DynamicTest.dynamicTest;
  import static org.junit.jupiter.api.Named.named;
  
  import java.util.Arrays;
  import java.util.Collection;
  import java.util.Iterator;
  import java.util.List;
  import java.util.Random;
  import java.util.function.Function;
  import java.util.stream.IntStream;
  import java.util.stream.Stream;
  
  import example.util.Calculator;
  
  import org.junit.jupiter.api.DynamicNode;
  import org.junit.jupiter.api.DynamicTest;
  import org.junit.jupiter.api.Named;
  import org.junit.jupiter.api.Tag;
  import org.junit.jupiter.api.TestFactory;
  import org.junit.jupiter.api.function.ThrowingConsumer;
  
  class DynamicTestsDemo {
  
      private final Calculator calculator = new Calculator();
  
      // This will result in a JUnitException!
      @TestFactory
      List<String> dynamicTestsWithInvalidReturnType() {
          return Arrays.asList("Hello");
      }
  
      @TestFactory
      Collection<DynamicTest> dynamicTestsFromCollection() {
          return Arrays.asList(
              dynamicTest("1st dynamic test", () -> assertTrue(isPalindrome("madam"))),
              dynamicTest("2nd dynamic test", () -> assertEquals(4, calculator.multiply(2, 2)))
          );
      }
  
      @TestFactory
      Iterable<DynamicTest> dynamicTestsFromIterable() {
          return Arrays.asList(
              dynamicTest("3rd dynamic test", () -> assertTrue(isPalindrome("madam"))),
              dynamicTest("4th dynamic test", () -> assertEquals(4, calculator.multiply(2, 2)))
          );
      }
  
      @TestFactory
      Iterator<DynamicTest> dynamicTestsFromIterator() {
          return Arrays.asList(
              dynamicTest("5th dynamic test", () -> assertTrue(isPalindrome("madam"))),
              dynamicTest("6th dynamic test", () -> assertEquals(4, calculator.multiply(2, 2)))
          ).iterator();
      }
  
      @TestFactory
      DynamicTest[] dynamicTestsFromArray() {
          return new DynamicTest[] {
              dynamicTest("7th dynamic test", () -> assertTrue(isPalindrome("madam"))),
              dynamicTest("8th dynamic test", () -> assertEquals(4, calculator.multiply(2, 2)))
          };
      }
  
      @TestFactory
      Stream<DynamicTest> dynamicTestsFromStream() {
          return Stream.of("racecar", "radar", "mom", "dad")
              .map(text -> dynamicTest(text, () -> assertTrue(isPalindrome(text))));
      }
  
      @TestFactory
      Stream<DynamicTest> dynamicTestsFromIntStream() {
          // Generates tests for the first 10 even integers.
          return IntStream.iterate(0, n -> n + 2).limit(10)
              .mapToObj(n -> dynamicTest("test" + n, () -> assertTrue(n % 2 == 0)));
      }
  
      @TestFactory
      Stream<DynamicTest> generateRandomNumberOfTestsFromIterator() {
  
          // Generates random positive integers between 0 and 100 until
          // a number evenly divisible by 7 is encountered.
          Iterator<Integer> inputGenerator = new Iterator<Integer>() {
  
              Random random = new Random();
              int current;
  
              @Override
              public boolean hasNext() {
                  current = random.nextInt(100);
                  return current % 7 != 0;
              }
  
              @Override
              public Integer next() {
                  return current;
              }
          };
  
          // Generates display names like: input:5, input:37, input:85, etc.
          Function<Integer, String> displayNameGenerator = (input) -> "input:" + input;
  
          // Executes tests based on the current input value.
          ThrowingConsumer<Integer> testExecutor = (input) -> assertTrue(input % 7 != 0);
  
          // Returns a stream of dynamic tests.
          return DynamicTest.stream(inputGenerator, displayNameGenerator, testExecutor);
      }
  
      @TestFactory
      Stream<DynamicTest> dynamicTestsFromStreamFactoryMethod() {
          // Stream of palindromes to check
          Stream<String> inputStream = Stream.of("racecar", "radar", "mom", "dad");
  
          // Generates display names like: racecar is a palindrome
          Function<String, String> displayNameGenerator = text -> text + " is a palindrome";
  
          // Executes tests based on the current input value.
          ThrowingConsumer<String> testExecutor = text -> assertTrue(isPalindrome(text));
  
          // Returns a stream of dynamic tests.
          return DynamicTest.stream(inputStream, displayNameGenerator, testExecutor);
      }
  
      @TestFactory
      Stream<DynamicTest> dynamicTestsFromStreamFactoryMethodWithNames() {
          // Stream of palindromes to check
          Stream<Named<String>> inputStream = Stream.of(
                  named("racecar is a palindrome", "racecar"),
                  named("radar is also a palindrome", "radar"),
                  named("mom also seems to be a palindrome", "mom"),
                  named("dad is yet another palindrome", "dad")
              );
  
          // Returns a stream of dynamic tests.
          return DynamicTest.stream(inputStream,
              text -> assertTrue(isPalindrome(text)));
      }
  
      @TestFactory
      Stream<DynamicNode> dynamicTestsWithContainers() {
          return Stream.of("A", "B", "C")
              .map(input -> dynamicContainer("Container " + input, Stream.of(
                  dynamicTest("not null", () -> assertNotNull(input)),
                  dynamicContainer("properties", Stream.of(
                      dynamicTest("length > 0", () -> assertTrue(input.length() > 0)),
                      dynamicTest("not empty", () -> assertFalse(input.isEmpty()))
                  ))
              )));
      }
  
      @TestFactory
      DynamicNode dynamicNodeSingleTest() {
          return dynamicTest("'pop' is a palindrome", () -> assertTrue(isPalindrome("pop")));
      }
  
      @TestFactory
      DynamicNode dynamicNodeSingleContainer() {
          return dynamicContainer("palindromes",
              Stream.of("racecar", "radar", "mom", "dad")
                  .map(text -> dynamicTest(text, () -> assertTrue(isPalindrome(text)))
          ));
      }
  
  }
  ```

## 超时

@Timeout 注解允许人们声明如果测试、测试工厂、测试模板或生命周期方法的执行时间超过给定的持续时间，它应该失败。 持续时间的时间单位默认为秒，但可配置。

```java
class TimeoutDemo {

    @BeforeEach
    @Timeout(5)
    void setUp() {
        // fails if execution time exceeds 5 seconds
    }

    @Test
    @Timeout(value = 100, unit = TimeUnit.MILLISECONDS)
    void failsIfExecutionTimeExceeds100Milliseconds() {
        // fails if execution time exceeds 100 milliseconds
    }

}
```

## 并行执行

并行测试执行是一项实验性功能

## 内置扩展

- @TempDir
  
  内置的 TempDirectory 扩展用于为单个测试或测试类中的所有测试创建和清理临时目录。 默认情况下已注册。 要使用它，请使用 @TempDir 注释 java.nio.file.Path 或 java.io.File 类型的字段，或将使用 @TempDir 注释的 java.nio.file.Path 或 java.io.File 类型的参数添加到 生命周期方法或测试方法。
