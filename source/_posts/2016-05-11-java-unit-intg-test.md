---
title: Java单元和集成测试配置
date: 2016-05-11 21:47:36
category: Frameworks
tags: [Java, Spring Boot, Gradle, IntelliJ, Unit Test, Integration Test]
description: 基于Spring Boot搭建一个Java工程，通过Gradle进行构建，使用IntelliJ IDE开发，对于在build.gradle中配置Integration Test和Unit Test有多种方式。
---

基于Spring Boot搭建一个Java工程，通过Gradle进行构建，使用IntelliJ IDE开发，对于在`build.gradle`中配置Integration Test和Unit Test有多种方式。接下来分别介绍两种方式:

假设在IntelliJ中创建好如下Tree结构:
![](/assets/java-unit-intg-test/java_project_tree.png)

### 方法一: use another source set

Add into the source sets

```
sourceSets {
    main {
        java.srcDirs = ['src/main/java']
        resources.srcDirs = ['src/main/resources']
    }
    test {
        java.srcDirs = ['src/test/unit/java']
        resources.srcDirs = ['src/test/unit/resources', 'src/test/intg/resources']
    }
    integrationTest {
        java.srcDirs = ['src/test/intg/java']
    }
}
```
Add into the idea intelliJ

```
idea {
    module {
        testSourceDirs += sourceSets.integrationTest.java.srcDirs
    }
}
```

Apply the dependencies for integration test

```
dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    compile("org.springframework.boot:spring-boot-starter-data-jpa")
    testCompile("com.jayway.jsonpath:json-path")
    testCompile("org.springframework.boot:spring-boot-starter-test")
    integrationTestCompile sourceSets.main.output
    integrationTestCompile sourceSets.test.output
    integrationTestCompile configurations.testCompile
    integrationTestRuntime configurations.testRuntime
}
```

Create the unit and integrationTest tasks

```
task unitTest(dependsOn: test)

task integrationTest(type: Test) {
    testClassesDir = sourceSets.integrationTest.output.classesDir
    classpath = sourceSets.integrationTest.runtimeClasspath
}

build.dependsOn integrationTest
```

### 方法二: use the same source set as test

Add into the source sets

```
sourceSets {
    main {
        java.srcDirs = ['src/main/java']
        resources.srcDirs = ['src/main/resources']
    }
    test {
        java.srcDirs = ['src/test/unit/java', 'src/test/intg/java']
        resources.srcDirs = ['src/test/unit/resources', 'src/test/intg/resources']
    }
}
```

Exclude the test classes `*IntegrationTest.class`

```
test {
    exclude '**/*IntegrationTest.class'
}
```

Add the dependencies for test

```
dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    compile("org.springframework.boot:spring-boot-starter-data-jpa")
    testCompile("com.jayway.jsonpath:json-path")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}
```

Create the unit and integrationTest tasks

```
task unitTest(dependsOn: test)

task integrationTest(type: Test) {
    include '**/*IntegrationTest.class'
}

build.dependsOn integrationTest
```

### 集成测试实现DEMO

``` ApplicationIntegrationTest.java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = Application.class)
@WebAppConfiguration
@Transactional
public abstract class ApplicationIntegrationTest {
    protected MockMvc mockMvc;
}
```

``` XXXControllerIntegrationTest.java
public class XXXControllerIntegrationTest extends ApplicationIntegrationTest {
    @Autowired
    private XXXController xxxController;

    @Autowired
    private XXXRepository xxxRepository;

    @Before
    public void setUp() {
        mockMvc = MockMvcBuilders.standaloneSetup(xxxController).build();
    }

    @Test
    public void should_get_xxxs_by_book_xxx() throws Exception {
        xxxRepository.save(new XXX("123456", "This is a integ test"));

        mockMvc.perform(get(format("/xxx/%s/yyys", vvv)))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$", hasSize(1)))
                .andExpect(jsonPath("$[0].uuid").value("123456"))
                .andExpect(jsonPath("$[0].content").value("This is a integ test"));
    }
}
```

### 单元测试实现DEMO

单元测试采用JUnit和Mockito测试框架实现.

``` DefaultXxxServiceTest.java
public class DefaultXxxServiceTest {
    @InjectMocks
    private DefaultXxxService xxxService;

    @Mock
    private XxxRepository xxxRepository;

    @Before
    public void setUp() {
        MockitoAnnotations.initMocks(this);
    }

    @Test
    public void should_retrieve_xxxs_by_yyy_uuid() {
        String yyyUuid = "book123456";
        List<Xxx> expectedXxxs = asList(new Xxx(), new Xxx());
        when(xxxRepository.findByYyy(yyyUuid)).thenReturn(expectedXxxs);

        Iterable<Xxx> xxxs = xxxService.retrieveXxxs(yyyUuid);

        assertThat(xxxs, is(expectedXxxs));
    }
}
```

### Run unit test
`./gradlew test`, it depends on `build` task.

### Run integration test
`./gradlew integrationTest` or `./gradlew iT`, it depends on `build` task.

### Run build exclude integration test
`./gradlew build -x integrationTest` or `./gradlew build -x iT`
