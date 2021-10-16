# Springboot_autoconfig


# springboot**自动装配原理入门**

## 引导加载自动配置类

启动类注解@SpringBootApplication是三个注解的合成注解

![image-20210827142945213](/springboot.assets/image-20210827142945213-16300457863982.png)

## @SpringBootConfiguration

@Configuration。代表当前是一个配置类

![image-20210827142659206](/springboot.assets/image-20210827142659206-16300456201351.png)

Indicates that a class provides Spring Boot application **@Configuration**. Can be used as an alternative to the Spring's standard @Configuration annotation **so that configuration can be found automatically** (for example in tests).

使用它的目的：实现自动装配

Application should only ever include one **@SpringBootConfiguration** and most idiomatic Spring Boot applications will inherit it from @SpringBootApplication.

一个应用只有一个，并且大多数应用程序从@SpringBootApplication继承它

## @ComponentScan

指定扫描那些，spring注解

**重点：**

## @EnableAutoConfiguration

@EnableAutoConfiguration由@AutoConfigurationPackage和@Import注解合成

![image-20210827143125206](/springboot.assets/image-20210827143125206-16300458863023.png)

+ @AutoConfigurationPackage

点进去

![image-20210827143707464](/springboot.assets/image-20210827143707464-16300462284304.png)

```java
@Import(AutoConfigurationPackages.Registrar.class)
//给容器导入一些组件
public @interface AutoConfigurationPackage {}
//利用registrar给容器中导入一系列组件
//将指定的一个包下的所有组件导入进来
```

再点击Registrar

![image-20210827143754197](/springboot.assets/image-20210827143754197-16300462750665.png)

to store the base package from the importing configuration.

存储导入配置中的基本包

```java
register(registry, new PackageImports(metadata).getPackageNames().toArray(new String[0]));
```

在register打一个断点，DEBUG

![image-20210827144244255](/springboot.assets/image-20210827144244255-16300465662266.png)

计算new PackageImports(metadata).getPackageNames()

发现结果就是我们程序的包名com.lorry

**原因：**注解标注在我们的主类上，返回主类所在的包，把包名封装在数组里面然后注册进去。registrar的作用就是把注解所在的包下的组件批量注册进来。

**总结：**

**@AutoConfigurationPackage**

原理：利用Registrar给容器中导入一系列组件 

将指定的一个包下的所有组件导入进来，Springboot02ConfigApplication所在包下。

作用：自动配置包，制定了默认的包规则

2. @Import(AutoConfigurationImportSelector.class)

AutoConfigurationImportSelector下有一个方法叫做selectImports，这个函数将所有需要的配置组件转成String数组返回出去

selectImport调用了getAutoConfigurationEnrty方法

![image-20210827145502983](/springboot.assets/image-20210827145502983-16300473040067.png)

getAutoConfigurationEnrty上图的这一行代码

![image-20210827145734841](/springboot.assets/image-20210827145734841-16300474562108.png)

包下刚开始会初始化全部的configurations 原始数量有131个

![image-20210827145823242](/springboot.assets/image-20210827145823242-16300475040269.png)

过滤后只剩下25个

![image-20210827150401379](/springboot.assets/image-20210827150401379-163004784223810.png)

step into

会跳到AutoConfigurationImportSelector.java的getCandidateConfigurations上

点loadFactoryNames()这个方法，这个方法返回loadSpringFactories()会跳转到SpringFactoriesLoader.java上的![image-20210827151031758](/springboot.assets/image-20210827151031758-163004823277811.png)

得到一个重要的方法：

```java
Map<String, List<String>> loadSpringFactories(ClassLoader classLoader)
```

利用工厂加载上述方法，

作用：得到所有的组件

我们在这个方法上打一个断点

![image-20210827151419470](/springboot.assets/image-20210827151419470.png)

这行代码获取资源文件，FACTORIES_RESOURCE_LOCATION代表资源文件的位置

```java
Enumeration<URL> urls = classLoader.getResources(FACTORIES_RESOURCE_LOCATION);
```

![image-20210827151610454](/springboot.assets/image-20210827151610454-163004857145512.png)

从META-INF/spring.factories位置来加载一个文件， 默认扫描我们当前系统里面所有META-INF/spring.factories位置的文件

例如 

![image-20210827151907297](/springboot.assets/image-20210827151907297-163004874818213.png)

==最核心的包==

![image-20210827152123126](/springboot.assets/image-20210827152123126-163004888393714.png)

点进这个包下的spring.factories里

![image-20210827152337053](/springboot.assets/image-20210827152337053-163004901900615.png)

有一个EnableAutoConfiguration，从它下面到

![image-20210827152427586](/springboot.assets/image-20210827152427586-163004906831316.png)

都是我们的自动配置类

156-26+1正好等于我们上面131个configurations

![image-20210827145734841](/springboot.assets/image-20210827145734841-16300474562108-163004933982217.png)

文件写死了springboot一启动就要给容器加载的所有配置类

```xml
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,\
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,\
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.context.LifecycleAutoConfiguration,\
org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,\
org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,\
org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ReactiveElasticsearchRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.elasticsearch.ReactiveElasticsearchRestClientAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jdbc.JdbcRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.ldap.LdapRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jReactiveDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jReactiveRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.neo4j.Neo4jRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.r2dbc.R2dbcDataAutoConfiguration,\
org.springframework.boot.autoconfigure.data.r2dbc.R2dbcRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,\
org.springframework.boot.autoconfigure.elasticsearch.ElasticsearchRestClientAutoConfiguration,\
org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,\
org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,\
org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,\
org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,\
org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,\
org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration,\
org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration,\
org.springframework.boot.autoconfigure.http.codec.CodecsAutoConfiguration,\
org.springframework.boot.autoconfigure.influx.InfluxDbAutoConfiguration,\
org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration,\
org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration,\
org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration,\
org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration,\
org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,\
org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,\
org.springframework.boot.autoconfigure.jsonb.JsonbAutoConfiguration,\
org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration,\
org.springframework.boot.autoconfigure.availability.ApplicationAvailabilityAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration,\
org.springframework.boot.autoconfigure.ldap.LdapAutoConfiguration,\
org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,\
org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,\
org.springframework.boot.autoconfigure.neo4j.Neo4jAutoConfiguration,\
org.springframework.boot.autoconfigure.netty.NettyAutoConfiguration,\
org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
org.springframework.boot.autoconfigure.quartz.QuartzAutoConfiguration,\
org.springframework.boot.autoconfigure.r2dbc.R2dbcAutoConfiguration,\
org.springframework.boot.autoconfigure.r2dbc.R2dbcTransactionManagerAutoConfiguration,\
org.springframework.boot.autoconfigure.rsocket.RSocketMessagingAutoConfiguration,\
org.springframework.boot.autoconfigure.rsocket.RSocketRequesterAutoConfiguration,\
org.springframework.boot.autoconfigure.rsocket.RSocketServerAutoConfiguration,\
org.springframework.boot.autoconfigure.rsocket.RSocketStrategiesAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.UserDetailsServiceAutoConfiguration,\
org.springframework.boot.autoconfigure.security.servlet.SecurityFilterAutoConfiguration,\
org.springframework.boot.autoconfigure.security.reactive.ReactiveSecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.reactive.ReactiveUserDetailsServiceAutoConfiguration,\
org.springframework.boot.autoconfigure.security.rsocket.RSocketSecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.saml2.Saml2RelyingPartyAutoConfiguration,\
org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration,\
org.springframework.boot.autoconfigure.session.SessionAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.client.servlet.OAuth2ClientAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.client.reactive.ReactiveOAuth2ClientAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.resource.servlet.OAuth2ResourceServerAutoConfiguration,\
org.springframework.boot.autoconfigure.security.oauth2.resource.reactive.ReactiveOAuth2ResourceServerAutoConfiguration,\
org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration,\
org.springframework.boot.autoconfigure.sql.init.SqlInitializationAutoConfiguration,\
org.springframework.boot.autoconfigure.task.TaskExecutionAutoConfiguration,\
org.springframework.boot.autoconfigure.task.TaskSchedulingAutoConfiguration,\
org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,\
org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,\
org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration,\
org.springframework.boot.autoconfigure.web.client.RestTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.HttpHandlerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.ReactiveWebServerFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.WebFluxAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.error.ErrorWebFluxAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.function.client.ClientHttpConnectorAutoConfiguration,\
org.springframework.boot.autoconfigure.web.reactive.function.client.WebClientAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration,\
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.reactive.WebSocketReactiveAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketMessagingAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration,\
org.springframework.boot.autoconfigure.webservices.client.WebServiceTemplateAutoConfiguration
```

**利用getAutoConfigurationEntry(annotationMetadata);给容器中批量导入一些组件的完整过程：**

1. 利用getAutoConfigurationEntry(annotationMetadata);给容器中批量导入一些组件 
2. 调用List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes)获取到所有需要导入到容器中的配置类 
3. 利用工厂加载 Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader)；得到所有的组件
4.  从META-INF/spring.factories位置来加载一个文件。  默认扫描我们当前系统里面所有META-INF/spring.factories位置的文件    spring-boot-autoconfigure-2.3.4.RELEASE.jar包里面也有META-INF/spring.factories

## 按需开启自动配置项

虽然我们131个场景的所有自动配置启动的时候默认全部加载。xxxxAutoConfiguration 按照条件装配规则（@Conditional），最终会按需配置。

例如：我们点开

![image-20210827153547550](/springboot.assets/image-20210827153547550-163004974842618.png)

以AopAutoConfiguration为例子：

首先说明是一个配置类，@Configuration(proxyBeanMethods = false)代表每个@Bean方法被调用多少次返回的组件都是新创建的

![image-20210827180447448](/springboot.assets/image-20210827180447448-163005868847420.png)

注意==@ConditionalOnClass==意思我们内路径存在Advice.class我们的配置才生效

![image-20210827153602837](/springboot.assets/image-20210827153602837-163004976361919.png)

```java
@ConditionalOnProperty(prefix = "spring.aop", name = "auto", havingValue = "true", matchIfMissing = true)
```

判断配置文件是否存在spring.aop.auto这个配置 的配置，并且值为true那么下面值就生效，matchIfMissing,如果你没配，默认值为true

下面有两个class方法

```java
@Configuration(proxyBeanMethods = false)
	@ConditionalOnClass(Advice.class)
	static class AspectJAutoProxyingConfiguration {}
/*
 @Configuration(proxyBeanMethods = false)是一个配置类
 @ConditionalOnClass(Advice.class)有对应的Advice.class才生效
*/
```

```java
@Configuration(proxyBeanMethods = false)
	@ConditionalOnMissingClass("org.aspectj.weaver.Advice")
	@ConditionalOnProperty(prefix = "spring.aop", name = "proxy-target-class", havingValue = "true",
			matchIfMissing = true)
	static class ClassProxyingConfiguration {}
/*
 @Configuration(proxyBeanMethods = false)是一个配置类
 @ConditionalOnMissingClass("org.aspectj.weaver.Advice")
 没有这个Advice才生效，并且说明了对应的配置方法
*/
```



总结：

- SpringBoot先加载所有的自动配置类  xxxxxAutoConfiguration
- 每个自动配置类按照条件进行生效，默认都会绑定配置文件指定的值。xxxxProperties里面拿。xxxProperties和配置文件进行了绑定

- 生效的配置类就会给容器中装配很多组件
- 只要容器中有这些组件，相当于这些功能就有了

- 定制化配置

- - 用户直接自己@Bean替换底层的组件
  - 用户去看这个组件是获取的配置文件什么值就去修改。

**xxxxxAutoConfiguration ---> 组件  --->** **xxxxProperties里面拿值  ----> application.properties（配置文件）**


