# spring-tutorial-22rd
CEOS 백엔드 23기 스프링 튜토리얼

# **IoC**
IoC는 Inversion of Control의 약어로, “제어의 역전”이라는 뜻을 가진다. 객체에 대한 통제권(생성, 참조해제 등..)을 개발자가 쥐는 것이 아니라 IoC 컨테이너에서 담당하는, 디자인패턴의 일종이다.

개발자는 객체 생성과 생명주기 관리를 신경쓰지 않고 비즈니스 로직만을 작성하면 된다는 장점이 있으며, 추가로 객체끼리 서로 직접 객체를 생성하지 않으므로 객체 간 결합도가 낮아진다는 장점도 있다. (객체지향에서 궁극적으로 지향하는 바)

IoC를 구현하는 방법으로는 컨테이너가 의존성이 필요한 시점에서 객체를 직접 넣어주는지, 아니면 개발자가 요청해야하는지에 따라 DI와 DL로 나뉜다.

**DI (Dependency Injection, 의존성 주입)** 은 의존성이 필요한 시점에서 컨테이너가 직접 객체를 생성하고 전달하는, 의존성을 주입시키는 과정이다.

**DL (Dependency Lookup, 의존성 검색)** 은 개발자가 의존성이 필요한 시점에서 컨테이너에 요청을 보내는 방식이다.


## 스프링에서

스프링에서는 스프링 컨테이너가 IoC 컨테이너의 역할을 담당한다. 스프링의 관리를 받는 모든 객체들은 스프링 컨테이너가 생성하고, 객체의 생명주기를 관리한다.

```java
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory,
    HierarchicalBeanFactory, MessageSource,
    ApplicationEventPublisher, ResourcePatternResolver { ... }
```

스프링에서 가장 기본으로 사용되는 컨테이너는 ApplicationContext다. 그리고 ApplicationContext의 주요 기능인 객체를 저장하고, 컨테이너에서 가져오는 기능들은 모두 BeanFactory 인터페이스에서 기인한다.

## Bean 이란?

스프링 컨테이너에 저장되는 객체를 Bean 이라고 칭한다. 스프링은 빈과 함께 빈에 대한 메타데이터도 컨테이너에 저장하는데, 이 때 메타데이터가 담기는 객체는 BeanDefinition 객체다.


- 패키지 이름 + 클래스 이름
- 스코프, 생애주기와 같은 빈의 활동에 대한 설정
- 빈이 의존하는 다른 객체
- 기타 정보

개발자는 아래에 나올 어노테이션이나 Configuration 클래스 등을 통해 위와같은 정보들을 설정할 수 있다. 스프링은 이런 정보들을 BeanDefinition 객체에 저장하며, 스프링 컨테이너에서는 BeanDefinition을 보고 Bean을 다룬다.

## ApplicationContext

### 구조

```jsx
┌───────────────┐
│               │                   ┌───────────────┐    ┌───────────────┐
│  BeanFactory  │                   │  Application  │    │ResourcePattern│
│  (interface)  │ → → → →           │ EventPublisher│    │    Resolver   │
│               │          ↓        │  (interface)  │    │  (interface)  │
└───────────────┘          ↓        └───────────────┘    └───────────────┘
        ↓                  ↓             ↓                      ↓
┌───────────────┐    ┌───────────────┐   ↓    ┌───────────────┐ ↓  ┌───────────────┐
│    Listable   │    │  Hierarchical │   ↓    │  Environment  │ ↓  │ MessageSource │
│  BeanFactory  │    │  BeanFactory  │   ↓    │    Capable    │ ↓  │  (interface)  │
│  (interface)  │    │  (interface)  │   ↓    │  (interface)  │ ↓  │               │
└───────────────┘    └───────────────┘   ↓    └───────────────┘ ↓  └───────────────┘
        ↓                   ↓            ↓           ↓          ↓          ↓
				↓										↓	┌───────────────┐      ↓          ↓          ↓
				↓										↓	│  Application  │      ↓          ↓          ↓
				  → → → → → → → → → →	│    Context    │ ← ← ← ← ← ← ← ← ← ← ← ← ← ←
															│  (interface)  │
															└───────────────┘
```

이런 구조로 이루어져있다.

**BeanFactory**가 빈을 등록, 관리, 의존성을 주입하는 모든 기능을 총괄한다.

### 컴포넌트 스캔

스프링이 시작되고 빈에 등록할 클래스를 찾는 과정을 컴포넌트 스캔 (Component Scan)이라고 칭한다.
컴포넌트 스캔은 먼저 가장 기본이 되는 클래스(@ComponentScan 어노테이션이 있는 클래스)를 시작으로 같은 패키지에 있는 클래스의 메타데이터를 토대로 BeanDefinition 객체를 만들고 컨테이너에 등록한다. 이후 BeanDefinition 객체를 토대로 객체를 생성 및 컨테이너에 등록한다.

그래서 컴포넌트 스캔을 하기 위해서는 먼저 BeanDefinition들을 조사하여 컨테이너에 넣는 작업이 필요하다. 그리고 그 전에 ComponentScan이 있는, 가장 기본이 되는 클래스를 찾는 것이 필요하다.

```java
SpringApplication.run(DemoApplication.class, args);

↓

//SpringApplication 클래스
public ConfigurableApplicationContext run(String... args) {
    ...
		ConfigurableApplicationContext context = null;
		...
		try {
			...
			context = createApplicationContext();
			...
			prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner); //여기
			refreshContext(context);
			...
		}
		...
}
```
가장 먼저 main 메서드를 통해 SpringApplication.run() 메서드가 실행되면, ApplicationContext가 생성되고, prepareContext() 메서드가 호출된다.

```java
private void prepareContext(DefaultBootstrapContext bootstrapContext, ConfigurableApplicationContext context,
			ConfigurableEnvironment environment, SpringApplicationRunListeners listeners,
			ApplicationArguments applicationArguments, @Nullable Banner printedBanner) {
		context.setEnvironment(environment);
		...
		if (!AotDetector.useGeneratedArtifacts()) {
			// Load the sources
			Set<Object> sources = getAllSources();
			Assert.state(!ObjectUtils.isEmpty(sources), "No sources defined");
			load(context, sources.toArray(new Object[0])); //여기서
		}
    ...
}
```
prepareContext 메서드는 ApplicationContext의 초기화를 담당하며, 이후 load() 메서드를 호출한다.

load 메서드에게 넘기는 source 인자가 main 메서드가 속한 클래스다. (getAllSources() 메서드가 이 역할을 담당한다)

```java
//SpringApplication 클래스
protected void load(ApplicationContext context, Object[] sources) {
    ...
    loader.load();
}
	
↓

//BeanDefinitionLoader 클래스
void load() {
    for (Object source : this.sources) {    
        load(source);
    }
}

↓

private void load(Object source) {
    if (source instanceof Class<?> type) {
        load(type);
        return;
    }
    ...
}	
	
↓

private void load(Class<\?> source) {
    if (isEligible(source)) {
        this.annotatedReader.register(source);
    }
}

//AnnotatedBeanDefinitionReader 클래스
public void register(Class<\?>... componentClasses) {
    for (Class<\?> componentClass : componentClasses) {
        registerBean(componentClass);
    }
}
	
↓

public void registerBean(Class<\?> beanClass) {
    doRegisterBean(beanClass, null, null, null, null);
}
	
↓

private <T> void doRegisterBean(Class<T> beanClass, @Nullable String name,
			Class<? extends Annotation> @Nullable [] qualifiers, @Nullable Supplier<T> supplier,
			BeanDefinitionCustomizer @Nullable [] customizers) {

    AnnotatedGenericBeanDefinition abd = new AnnotatedGenericBeanDefinition(beanClass);
    if (this.conditionEvaluator.shouldSkip(abd.getMetadata())) {
        return;
    }

    abd.setAttribute(ConfigurationClassUtils.CANDIDATE_ATTRIBUTE, Boolean.TRUE);
    abd.setInstanceSupplier(supplier);
    ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(abd);
    abd.setScope(scopeMetadata.getScopeName());
    String beanName = (name != null ? name : this.beanNameGenerator.generateBeanName(abd, this.registry));

    ...

    BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(abd, beanName);
    definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
    BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, this.registry);
}

```
이를 통해 AnnotatedBeanDefinitionReader 클래스의 doRegisterBean() 메서드가 최종적으로 호출되어 BeanDefinition 객체를 만들어 Registry에 저장한다. 이 BeanDefinition 객체는 메인클래스의 정보를 담고있다.

여기서는 메인클래스의 BeanDefinition도 넣었지만 이외의 스프링 내부적으로 사용되는 빈도 BeanDefinition 객체로 만들어져서 빈에 들어간다.

그 다음, SpringApplication 클래스의 run 메서드로 다시 돌아가면, refresh() 메서드가 실행되게된다.

```java
@Override
public void refresh() throws BeansException, IllegalStateException {
    try {
		    ...
        try {
			      ...
   			    invokeBeanFactoryPostProcessors(beanFactory);
   	        ...
        }
    }
}

↓

protected void invokeBeanFactoryPostProcessors (
ConfigurableListableBeanFactory beanFactory) {
    PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors(beanFactory,
        getBeanFactoryPostProcessors());
    ...
}

↓

final class PostProcessorRegistrationDelegate {
    public static void invokeBeanFactoryPostProcessors(
        ...
        registryProcessor.postProcessBeanDefinitionRegistry(registry);
		    ...
    }
}

↓

public class ConfigurationClassPostProcessor implements BeanDefinitionRegistryPostProcessor ... {
@Override
    public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) {
		    ...
		    processConfigBeanDefinitions(registry);
    }
}

↓

public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {

    ...
    
    public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
    List\<BeanDefinitionHolder\> configCandidates = new ArrayList\<\>();
    String[] candidateNames = registry.getBeanDefinitionNames();

    for (String beanName : candidateNames) {
        BeanDefinition beanDef = registry.getBeanDefinition(beanName);
        if (...) {
            ...
        }
        else if (...)) {
            configCandidates.add(new BeanDefinitionHolder(beanDef, beanName));
        }
    }

		// Return immediately if no @Configuration classes were found
    if (configCandidates.isEmpty()) {
        return;
    }
		
    ...

    // Parse each @Configuration class
    ConfigurationClassParser parser = new ConfigurationClassParser(
        this.metadataReaderFactory, this.problemReporter, this.environment,
        this.resourceLoader, this.componentScanBeanNameGenerator, registry);
        ...
		
    do {
        ...
        parser.parse(candidates);
        ...
    }
    ...	
}

```
이 흐름을 타고 parse() 메서드가 호출된다. ConfigurationClassParser는 BeanDefinitionRegistry를 받는데, 여기에 위에서 등록한 BeanDefinition들이 있다.

매개변수 candidates에는 BeanDefinition들을 BeanDefinitionRegistry에서 빼서 BeanDefinitionHolder에 저장하고, 이 BeanDefinitionHolder 객체 여러 개를 Set 객체로 만든 것이다.

```java
//ConfigurationClassParser 클래스, parse 메서드가 processConfigurationClass를 호출함

// Search for locally declared @ComponentScan annotations first.
Set\<AnnotationAttributes\> componentScans = AnnotationConfigUtils.attributesForRepeatable(sourceClass.getMetadata(), ComponentScan.class, ComponentScans.class, MergedAnnotation::isDirectlyPresent);

// Fall back to searching for @ComponentScan meta-annotations (which indirectly
// includes locally declared composed annotations).
if (componentScans.isEmpty()) {
    componentScans = AnnotationConfigUtils.attributesForRepeatable(sourceClass.getMetadata(),
    ComponentScan.class, ComponentScans.class, MergedAnnotation::isMetaPresent);
}
```
먼저 현재 클래스 (보통 메인클래스)에서 @ComponentScan 어노테이션을 찾는다. sourceClass (보통은 메인클래스) 와 찾을 ComponentScan, ComponentScans 클래스를 넘겨서 해당 어노테이션이 있는지 찾는다.

다만 메인클래스에서 @SpringBootApplication 어노테이션 한 개만 사용할 경우 @ComponentScan 클래스를 찾지 못할 때가 있는데, 이 때를 위해서 어노테이션을 찾는 로직도 있다.

```java
if (!componentScans.isEmpty()) {
    ...
    for (AnnotationAttributes componentScan : componentScans) {
        // The config class is annotated with @ComponentScan -> perform the scan immediately
				Set<BeanDefinitionHolder\> scannedBeanDefinitions = this.componentScanParser.parse(componentScan, sourceClass.getMetadata().getClassName());

        // Check the set of scanned definitions for any further config classes and parse recursively if needed
				for (BeanDefinitionHolder holder : scannedBeanDefinitions) {
				    BeanDefinition bdCand = holder.getBeanDefinition().getOriginatingBeanDefinition();
				    if (bdCand == null) {
						    bdCand = holder.getBeanDefinition();
				    }

				    if (ConfigurationClassUtils.checkConfigurationClassCandidate(bdCand, this.metadataReaderFactory)) {
				        parse(bdCand.getBeanClassName(), holder.getBeanName());
				    }
        }
			}
		}
```
그 후에 본격적으로 스캔을 돌린다. @ComponentScan이 있는 패키지부터 스캔을 돌리기 시작한다.

```java
//ComponentScanAnnotationParser 클래스
public Set<BeanDefinitionHolder> parse(AnnotationAttributes componentScan, String declaringClass) {
    ClassPathBeanDefinitionScanner scanner = new ClassPathBeanDefinitionScanner(this.registry, componentScan.getBoolean("useDefaultFilters"), this.environment, this.resourceLoader);
    ...
    Set<String\> basePackages = new LinkedHashSet<\>();
    ...
		return scanner.doScan(StringUtils.toStringArray(basePackages));
	}
```

componentScanParser의 parse 메서드는 ClassPathBeanDefinitionScanner 클래스를 생성하고, 내부설정을 마친 후에 doScan() 메서드를 통하여 본격적으로 스캔을 실시한다. basePackages는 컴포넌트 스캔을 시작할 패키지다. Scanner의 설정과정에서 결정된다.

```java
public @interface ComponentScan {

	/**
	 * Alias for {@link #basePackages}.
	 * <p>Allows for more concise annotation declarations if no other attributes
	 * are needed &mdash; for example, {@code @ComponentScan("org.example")}
	 * instead of {@code @ComponentScan(basePackages = "org.example")}.
	 */
	@AliasFor("basePackages")
	String[] value() default {};
```

ComponentScan 어노테이션에는 “basePackeages”라는 이름을 가진 속성이 있는데, 이 속성을 통해 basePackeages를 가져오며, 기본값은 빈 배열이다. 만약 빈 배열일 경우에는 스프링이 자동으로 @ComponentScan 어노테이션이 있는 클래스에서 가져온다.

```jsva
protected Set<BeanDefinitionHolder\> doScan(String... basePackages) {
    ...
		Set<BeanDefinitionHolder\> beanDefinitions = new LinkedHashSet<\>();
		
		for (String basePackage : basePackages) {
		    Set<BeanDefinition\> candidates = findCandidateComponents(basePackage);
		    ...
		}

		return beanDefinitions;
}
```

doScan() 메서드가 본격적으로 정보를 스캔을 시작한다. @ComponentScan 어노테이션이 존재하는 패키지를 findCandidateComponents 메서드를 호출해 패키지 전체를 조사한다.

```java
public Set<BeanDefinition> findCandidateComponents(String basePackage) {
    ...
		return scanCandidateComponents(basePackage);
}
```

```java
private Set<BeanDefinition> scanCandidateComponents(String basePackage) {
    Set<BeanDefinition\> candidates = new LinkedHashSet<\>();
		
		try {
		    String packageSearchPattern = ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX + resolveBasePackage(basePackage) + '/' + this.resourcePattern;
		    ...		

        Resource[] resources = getResourcePatternResolver().getResources(packageSearchPattern);

        ...

        for (Resource resource : resources) {
			      String filename = resource.getFilename();
				    ...
				    try {
                MetadataReader metadataReader = getMetadataReaderFactory().getMetadataReader(resource);

                if (isCandidateComponent(metadataReader)) {
						        ScannedGenericBeanDefinition sbd = new ScannedGenericBeanDefinition(metadataReader);
						        sbd.setSource(resource);
						
						        if (isCandidateComponent(sbd)) {
							          ...
							          candidates.add(sbd);
						        }
						    else {
                    ...
							  }
                }
					}
					...
         }
         ...
    }
					
		return candidates;
}
```

scanCandidateComponents 메서드에서 컴포넌트 스캔의 핵심적인 역할을 담당한다.

basePackage를 기반으로 패키지의 하위클래스에 전부 스캔을 한다. 만약 패키지가 com.example 이라면

```java
String packageSearchPattern =
    "classpath*:" +
    resolveBasePackage(basePackage) +
    "/**/*.class";
```

```java
classpath*:com/example/**/*.class
```

이렇게 패키지의 하위 클래스를 전부 조사하며, ResourcePatternResolver가 이 역할을 담당하는 클래스다.

```java
	protected boolean isCandidateComponent(MetadataReader metadataReader) throws IOException {
		for (TypeFilter filter : this.excludeFilters) {
			if (filter.match(metadataReader, getMetadataReaderFactory())) {
				return false;
			}
		}
		
		for (TypeFilter filter : this.includeFilters) {
			if (filter.match(metadataReader, getMetadataReaderFactory())) {
			    registerCandidateTypeForIncludeFilter(metadataReader.getClassMetadata().getClassName(), filter);
			    return isConditionMatch(metadataReader);
			}
		}
		return false;
	}
```

가져온 클래스들은 먼저 isCandidateComponent 메서드에 들어간다. 첫 번째 반복문에서 빈에 넣지 말아야할 특정한 조건들이 있는지 여부를 검사하고, 두 번째 반복문에서 includerFilters로 필터링 과정에서 본격적으로 @Component나, @Service와 같이 빈으로 등록할만한 클래스인지 조사한다.

```java
protected boolean isCandidateComponent(AnnotatedBeanDefinition beanDefinition) {
		AnnotationMetadata metadata = beanDefinition.getMetadata();
		return (metadata.isIndependent() && (metadata.isConcrete() ||
				(metadata.isAbstract() && metadata.hasAnnotatedMethods(Lookup.class.getName()))));
	}
```

그리고 1차적으로 필터링을 거친 후 isCandidateComponent 메서드(위의 메서드에서 오버로딩함)에 BeanDefinition을 넘기고 본격적으로 클래스를 조사하게된다.

```java
//ConfigurationClassParser 클래스
for (AnnotationAttributes componentScan : componentScans) {
				// The config class is annotated with @ComponentScan -> perform the scan immediately
				Set<BeanDefinitionHolder> scannedBeanDefinitions =
						this.componentScanParser.parse(componentScan, sourceClass.getMetadata().getClassName());
				// Check the set of scanned definitions for any further config classes and parse recursively if needed
				for (BeanDefinitionHolder holder : scannedBeanDefinitions) {
					BeanDefinition bdCand = holder.getBeanDefinition().getOriginatingBeanDefinition();
					if (bdCand == null) {
						bdCand = holder.getBeanDefinition();
					}
					if (ConfigurationClassUtils.checkConfigurationClassCandidate(bdCand, this.metadataReaderFactory)) {
						parse(bdCand.getBeanClassName(), holder.getBeanName());
					}
				}
			}
```

- isIndependent() → 익명클래스 혹은 내부클래스인지 여부를 조사한다.
- isConcrete() → 구현체인지 확인한다. 인터페이스나 추상클래스는 해당되지 않는다.
- isAbstract() + Lookup.class() → 추상클래스더라도 @Lookup이 있다면 빈으로 등록할 수 있기 때문에 이 여부를 조사한다.


```java
protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet<>();

    for (String basePackage : basePackages) {
        Set<BeanDefinition> candidates = findCandidateComponents(basePackage);
		
        for (BeanDefinition candidate : candidates) {
            ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);
            candidate.setScope(scopeMetadata.getScopeName());
            String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);
			...
            
            if (checkCandidate(beanName, candidate)) {
                BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
                definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
                beanDefinitions.add(definitionHolder);
                registerBeanDefinition(definitionHolder, this.registry);
            }
        }
    }
    return beanDefinitions;
}
```
다시 doScan() 메서드로 돌아가면 여기서 반환된 candidates에 있는 BeanDefinition을 하나씩 ApplicationConext에 등록한다.

```java
//AnnotationConfigurableApplicationContext	
@Override
public void refresh() throws BeansException, IllegalStateException {
    ...
    try {
        ...
        try {
            ...
            invokeBeanFactoryPostProcessors(beanFactory);
            //invokeBeanFactoryPostProcessors를 통해서 위의 과정에서의
            //BeanDefinition 등록이 일어나게된다.
				
            ...
            finishBeanFactoryInitialization(beanFactory);
            ...
        }
    }
}
	
↓

protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
    ...
    beanFactory.preInstantiateSingletons();
}

↓

//DefaultListableBeanFactory 클래스
@Override
public void preInstantiateSingletons() throws BeansException {
    ...
    List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);
    ...
    try {
        List<CompletableFuture<\?>> futures = new ArrayList<\>();
        for (String beanName : beanNames) /* BeanDefinition 이름들의 배열 */  {
            RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);

            if (!mbd.isAbstract() && mbd.isSingleton()) {
                CompletableFuture<\?> future = preInstantiateSingleton(beanName, mbd);
                ...
            }
        ...
        }
    }
}
```

여기까지 진행했으면 다시 ApplicationContext 클래스로 돌아간다. 그리고 finishBeanFactoryInitialization 메서드가 호출되고, 여기서 호출되는 BeanFactory 클래스의 preInstantiateSingletons 메서드가 빈을 실제로 생성한다.

```java
private @Nullable CompletableFuture<\?> preInstantiateSingleton(String beanName, RootBeanDefinition mbd) {
   ...

    if (!mbd.isLazyInit()) {
        try {
            instantiateSingleton(beanName);
        }
    ...
    
↓

        private void instantiateSingleton(String beanName) {
            if (isFactoryBean(beanName)) {
                Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);

                if (bean instanceof SmartFactoryBean<?> smartFactoryBean && smartFactoryBean.isEagerInit()) {
                    getBean(beanName);
                }
            }
            else {
                getBean(beanName);
            }
        }

↓

//AbstractBeanFactory 클래스
public Object getBean(String name) throws BeansException {
    return doGetBean(name, null, null, false);
}
	
↓

protected <T> T doGetBean(
    String name, @Nullable Class<T> requiredType, @Nullable Object @Nullable [] args, boolean typeCheckOnly)
    throws BeansException {

    ...

            // Create bean instance.
            if (mbd.isSingleton()) {
                sharedInstance = getSingleton(beanName, () -> {
                    try {
                        return createBean(beanName, mbd, args);
                    }
                });
            }
```

여기서 본격적으로 컴포넌트 스캔 과정에서 얻은 BeanDefinition을 토대로 Bean을 생성한다. BeanDefinition을 기반으로 Bean을 생성해 ApplicationContext에 등록하게된다.

```java
//AbstractAutowireCapableBeanFactory 클래스
@Override
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object @Nullable [] args)
throws BeanCreationException {
...

    try {
        Object beanInstance = doCreateBean(beanName, mbdToUse, args);
        ...
        return beanInstance;
    }
    ...
}

↓

protected Object doCreateBean(String beanName, RootBeanDefinition mbd, @Nullable Object @Nullable [] args)
throws BeanCreationException {

    // Instantiate the bean.
    BeanWrapper instanceWrapper = null;
    ...
    if (instanceWrapper == null) {
        instanceWrapper = createBeanInstance(beanName, mbd, args);
    }
    
    Object bean = instanceWrapper.getWrappedInstance();

    ...

    // Initialize the bean instance.
    Object exposedObject = bean;
    try {
        populateBean(beanName, mbd, instanceWrapper);
        ...
    }
    ...
}

//이 윗과정에서는 생성된 Bean 토대로 BeanWrapper 클래스를 만든 후에 넘긴다

↓

protected void populateBean(String beanName, RootBeanDefinition mbd, @Nullable BeanWrapper bw) {
...

    if (hasInstantiationAwareBeanPostProcessors()) {
        ...
			
        for (InstantiationAwareBeanPostProcessor bp : getBeanPostProcessorCache().instantiationAware) {
            PropertyValues pvsToUse = bp.postProcessProperties(pvs, bw.getWrappedInstance(), beanName);
                ...
        }
    }

    boolean needsDepCheck = (mbd.getDependencyCheck() != AbstractBeanDefinition.DEPENDENCY_CHECK_NONE);
    if (needsDepCheck) {
        PropertyDescriptor[] filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
        checkDependencies(beanName, mbd, filteredPds, pvs);
    }
    ...
}
```
createBean -> doCreateBean -> populateBean() 메서드로 이어지게된다.

doCreateBean() 메서드에서는 본격적으로 빈을 생성한다. createBeanInstance() 메서드를 통해서 빈을 생성한다.

```java
if (mbd.getFactoryMethodName() != null) {
    return instantiateUsingFactoryMethod(beanName, mbd, args);
}

...

// Candidate constructors for autowiring?
Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
if (ctors != null || mbd.getResolvedAutowireMode() == AUTOWIRE_CONSTRUCTOR ||
    mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args)) {
    return autowireConstructor(beanName, mbd, ctors, args);
}

// Preferred constructors for default construction?
ctors = mbd.getPreferredConstructors();
if (ctors != null) {
    return autowireConstructor(beanName, mbd, ctors, null);
}

// No special handling: simply use no-arg constructor.
return instantiateBean(beanName, mbd);
}
```
createBeanInstance() 메서드 내부. 각각 팩토리메서드, Constructor Injection, No-args Constructor 메서드다.


```java
@Configuration
class AppConfig {

    @Bean
    public RestTemplate restTemplate() {
        RestTemplate rt = new RestTemplate();
        rt.setRequestFactory(...);
        return rt;
    }
}
```
이렇게 스프링이 관리하지 않는 특정 객체를 커스텀한 뒤 빈으로 등록하는 경우가 있는데, 이 경우는 팩토리 메서드에 해당한다고한다.



먼저 hasInstantiationAwareBeanPostProcessors 를 조사한다. 여기서는 Bean을 생성하기 전에 수행해야할 특정한 로직이 있는지 여부를 조사하며, 스프링 AOP에 사용된다.

```java
@Override
public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) {
		InjectionMetadata metadata = findAutowiringMetadata(beanName, bean.getClass(), pvs);
		try {
		    metadata.inject(bean, beanName, pvs);
		}
		..
		return pvs;
}
```

postProcessProperties 메서드는 findAutowwiringMetadata 메서드를 통해서 클래스 내의 @Value, @Autowired 가 붙은 필드 또는 메서드를 찾는다. 즉, 어디에 DI를 해야할지 조사하여 메타데이터로 가져온다.

```java
protected void inject(Object target, @Nullable String requestingBeanName, @Nullable PropertyValues pvs) throws Throwable {

    if (!shouldInject(pvs)) {
		    return;
    }
			
		if (this.isField) {
				Field field = (Field) this.member;
				ReflectionUtils.makeAccessible(field);
				field.set(target, getResourceToInject(target, requestingBeanName));
		} else {
				try {
				    Method method = (Method) this.member;
					  ReflectionUtils.makeAccessible(method);
					  method.invoke(target, getResourceToInject(target, requestingBeanName));
				}
				catch (InvocationTargetException ex) {
				    throw ex.getTargetException();
				}
			}
		}
```

PropertyValues는 클래스 내에서 DI를 주입할 후보가 되는 각 필드를 뜻한다. 첫 번째 조건문을 통해서 이미 필드에 값이 주입되면 별다른 값을 주입하지 않고 넘어간다.

두 번째 조건문에서는 필드주입인지, 메서드주입인지를 나눠서 조사한다. 필드주입은 리플렉션을 통해 private 필드에도 접근가능하게하여 빈을 주입한다. 매개변수로 전해지는 빈의 이름은 자기자신을 주입하는 것을 방지하기 위해 전달하는 것이다.

빈은 주입할 객체를 필드의 타입을 기반으로 찾는다. 만약 필드의 타입에서 주입할 객체를 고를 수 없다면 @Primary → @Qualifier 어노테이션 순으로 타입을 결정한다. 만약 하나의 인터페이스를 구현한 서비스가 여러 개일 때에는 직접 구현체를 명시하거나, 위의 타입결정 순서대로 타입을 매칭한다. 그래도 매칭하지 못한 경우 에러가 발생한다. (NoSuchBeanDefinitionException)

이 과정을 통해서 스프링은 Bean을 등록하고, 필드에 의존성을 주입한다. 이 과정은 모두 비즈니스 로직이 시작되기 전에 일어나게된다.

이 직후에 postProcessAfterInitialization 어노테이션이 호출되어서 @PostConstruct 메서드가 실행되고, 애플리케이션이 종료되면 ApplicationContext도 같이 제거되고 그 때 빈도 같이 제거된다. 만약 @PreDestroy 어노테이션이 적용된 메서드가 있다면 애플리케이션이 끝나기 전에 호출되어 실행된다.

## AOP

AOP는 프로그램 로직을 핵심적, 부가적 관점으로 나누고, 각 관점을 기준으로 모듈화하여 프로그래밍하는 방법이다. 즉, 핵심적인 기능과 부가적이 기능이 섞여있던 기존 프로그래밍 방법과 달리, 핵심적인 기능만을 작성하고 부가적인 기능은 따로 만들어둔 후 그때그때 불러와서 진행하는 것이다.

그래서 AOP의 특징은 로직이 수행되기 직전 또는 직후에 특정한 기능을 추가할 수 있다는 것이다. 트랜잭션 처리, 로깅, 보안 등과 관련된, 필수적이지만 핵심적이지는 않은 로직을 보다 유연하게 수행할 수 있다.

주요 개념으로는
Aspect: 여러 모듈에 거쳐서 적용시킬 모듈의 단위이다. 그래서 횡단 관심사라고도 칭한다.
Join point: Aspect가 실행될 수 있는 지점들
Pointcut: join point에서 Aspect가 실행될 기준을 뜻한다. 스프링에서는 특정한 표현식 제공하여 Pointcut를 지정할 수 있다.
Advice: join point에서 aspect가 취하는 동작이다.

Proxy: 프록시는 대리인, 대리자의 뜻을 가지고있다. 앞서서 특정 로직이 수행되기 직전 또는 직후에 특정한 기능을 추가할 수 있다고했는데, 그래서 객체가 다른 객체를 호출할 때, 객체가 호출하는게 아니라 호출하는 객체와 똑같은 프록시 객체가 다른 객체를 호출하게된다. 그래서 프록시 객체에 추가적인 로직을 추가할 수 있는 것.

```java
class CustomLogger{
    public static void log() { ... }
}
```

```java
service.doSomething() {
    CustomLogger.log();
    // 비즈니스 로직
}
```

기존 OOP 방식에서는 중복되는 로직을 하나의 클래스로 분리한 후에 외부에서 호출하는 방식을 사용했어야했으나, 직접 호출해야함과 더불어 코드 길이도 늘어나고, 호출을 빼먹을 수도 있다.

```java
@Around("execution(* Service.*(..))")
log()
```

```java
service.doSomething() {
    // 비즈니스 로직
}
```

하지만 AOP를 사용해 매 service 클래스마다 로직을 수행하게한다면 개발자가 신경쓰지않고도 자동으로 특정 로직이 수행된다. 이렇게 자주 사용되지만 부가적인 기능을 분리하여 자동으로 특정 로직의 전 또는 후에 실행되게해 코드도 줄이며 변경에 유연하도록하는 방법을 관점 지향 프로그래밍, AOP라고 칭한다.

## Spring MVC

MVC 패턴은 Model - View - Controller로 이루어진 패턴이다. Model은 서비스의 비즈니스 로직 전반을 처리하는 서비스계층에 해당하고, View는 유저에게 보여지는 프론트엔드에, Controller는 Model와 View를 이어주어 유저의 요청을 모델 파트로 전달하고, 모델 측에서 내려온 정보를 뷰 파트로 보내 유저에게 보여지게 하는 등의 역할을 한다.

그래서 순수한 MVC 패턴은 요청이 들어오면, 컨트롤러가 이를 모델층으로 넘기고, 모델층에서 데이터를 내려주면 이를 컨트롤러가 다시 뷰층으로 데이터를 넘기는 역할을 수행했다.

스프링 MVC는 이와 다른데, 먼저 DispatcherServlet이 요청을 받아서 수행한다.

```java
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		...
		try {
			...

			try {
				...

				// Determine handler for the current request.
				mappedHandler = getHandler(processedRequest);
				
				...

				// Determine handler adapter and invoke the handler.
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

				...
				
		finally {
			...
		}
	}
```

DispatcherServlet은 모든 요청을 받아서 컨트롤러에게 넘기는 작업을 수행한다.

메서드는 크게 getHandler와 getHandlerAdapter로 나뉘는데, 먼저 각 요청마다 수행할 컨트롤러를 가져온다. 그리고 요청과 컨트롤러 매핑 방법이 HttpRequestHandler, Servlet, 그리고 스프링의 @Controller + @RequestMapping 어노테이션 등 여러 방법이 있기 때문에 그에 맞춰 핸들러 어댑터를 제공해, 하나의 통일된 컨트롤러 연결을 수행할 수 있게된다.

```java
@Override
public void handle(HttpExchange exchange) throws IOException {
    //로그인 기능
    if(exchange.getRequestURI().toString().startsWith("/login")){
        if (exchange.getRequestMethod().equals("POST")) {
            LoginService loginService = new LoginService(connection);

            ...    
            ResponseEntity<HttpResponseDTO\> response = loginService.login(loginRequestDTO);
        }
    }
}
```

만약 DispatcherServlet 없이 그대로 컨트롤러로 요청이 전해지면, 직접 비즈니스 로직에 도달하기 직전에 요청을 낚아채서 URL과 이를 다룰 컨트롤러를 직접 매칭해주어야한다.

이후 컨트롤러는 요청을 수행하고, 추가적으로 컨트롤러는 Model&View라는, 모델단에서 내려온 데이터와 어느 view 단으로 데이터를 보여줄지에 대한 데이터를 모두 다루고 있다. 그 후에 ViewResolver를 통해 View 객체를 찾아서 데이터를 보여주면 끝.

그래서 스프링MVC는 기본 MVC 패턴과 다르게 DispatcherServlet이 하나의 중개자 역할을 수행하는 것이 큰 차이가 있다. 기존에는 컨트롤러 - 서비스 - 뷰가 서로 소통하는 방식이었다면, 스프링에서는 DispatcherServlet가 MVC를 이루는 멤버들과 로직을 수행한다.


추가로 스프링은 톰캣이라는 서버를 기본제공한다. 따라서 스프링을 동작하면 톰캣서버 위에서 웹서비스가 이루어진다.

클라이언트가 요청을 보내면 톰캣서버로 보내지고, 톰캣서버는 몇 가지 필터들을 통해 요청을 필터링한 후, 스프링에게 요청을 넘긴다. 이 요청을 받는 역할을 DispatcherServlet이 수행하고, DispathcerServlet이 다시 Controller 측으로 데이터를 넘긴다.

이렇게 클라이언트로부터 HTTP 요청을 받아서 간단한 정적 웹페이지를 반환하는 프로그램을 **서버**라고 칭한다. 하지만 데이터베이스 조회, 부가적인 로직 처리 등 정적 웹페이지가 아닌 더 복잡한 동적인 웹페이지를 반환해야할 때가 있는데, 이 때 이 부가적인 로직을 처리하는 미들웨어를 **WAS**(Web Application Server)라고 칭한다. 서버와는 다른 개념이다.

스프링은 웹 서버의 구동을 더 쉽게 도와주는 프레임워크, 톰캣서버는 스프링을 구동할 수 있는 WAS, 아파치 서버는 요청을 받아서 WAS가 구동되도록하는 웹 서버에 해당한다.

# CGV DB 모델링
https://www.erdcloud.com/d/vXew5ExzkDE48pxDi
