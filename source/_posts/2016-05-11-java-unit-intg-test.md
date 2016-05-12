---
title: Java Unit & Integration Test in Spring Boot
date: 2016-05-11 21:47:36
category:
tags:
description:
published: false
---

基于Spring Boot搭建一个Java工程，通过Gradle进行构建，使用IntelliJ IDE开发，对于在`build.gradle`中配置Integration Test有多种方式，介绍两种:

1. Method One
```
sourceSets {
    main {
        java.srcDirs = ['src/main/java']
        resources.srcDirs = ['src/main/resources']
    }
    test {
        java.srcDirs = ['src/test/java']
        resources.srcDirs = ['src/test/resources', 'src/integrationTest/resources']
    }
    integrationTest {
        java.srcDirs = ['src/integrationTest/java']
    }
}
```

```
idea {
    module {
        testSourceDirs += sourceSets.integrationTest.java.srcDirs
    }
}
```

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

```
task integrationTest(type: Test) {
    testClassesDir = sourceSets.integrationTest.output.classesDir
    classpath = sourceSets.integrationTest.runtimeClasspath
}

check.dependsOn integrationTest
```

2. Method Two
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

```
test {
    exclude '**/*Test.class'
}
```

```
dependencies {
    compile("org.springframework.boot:spring-boot-starter-web")
    compile("org.springframework.boot:spring-boot-starter-data-jpa")
    testCompile("com.jayway.jsonpath:json-path")
    testCompile("org.springframework.boot:spring-boot-starter-test")
}
```

```
task unitTest(type: Test) {
    include '**/unit/*Test.class'
}

task integrationTest(type: Test) {
    include '**/intg/*IntegrationTest.class'
}

check.dependsOn unitTest
check.dependsOn integrationTest
```

### 实现

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


Unit test

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