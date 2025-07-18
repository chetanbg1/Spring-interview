# Spring-interview

 1. What is Spring ORM? How does it integrate with Hibernate?
Answer:
Spring ORM is a module in the Spring Framework that simplifies the integration of popular ORM frameworks like Hibernate, JPA, JDO, and iBATIS with Spring applications. It provides a consistent way to handle persistence and transaction management across different ORM tools.

Integration with Hibernate:
Spring manages the SessionFactory (Hibernate's core object) as a bean.
You can use HibernateTemplate or Spring Data JPA to simplify Hibernate data access code.
Spring handles transactions via @Transactional and the PlatformTransactionManager.

✅ 2. How is SessionFactory managed in Spring?
Answer:
In a traditional Hibernate setup, SessionFactory is created manually. In Spring, it's managed as a bean, usually configured via:

XML Configuration:
<bean id="sessionFactory" class="org.springframework.orm.hibernate5.LocalSessionFactoryBean">
    <property name="dataSource" ref="dataSource" />
    <property name="hibernateProperties">
        <props>
            <prop key="hibernate.dialect">org.hibernate.dialect.MySQLDialect</prop>
            <prop key="hibernate.show_sql">true</prop>
        </props>
    </property>
    <property name="packagesToScan" value="com.example.model" />
</bean>
Java Config (Recommended in Spring Boot):
@Bean
public LocalSessionFactoryBean sessionFactory() {
    LocalSessionFactoryBean sessionFactory = new LocalSessionFactoryBean();
    sessionFactory.setDataSource(dataSource());
    sessionFactory.setPackagesToScan("com.example.model");
    sessionFactory.setHibernateProperties(hibernateProperties());
    return sessionFactory;
}
Spring then injects this SessionFactory wherever required.

✅ 3. What is HibernateTemplate in Spring? Is it recommended now?
Answer:
HibernateTemplate is a helper class provided by Spring to reduce boilerplate Hibernate code such as opening/closing sessions, transactions, etc.
Example:
public class UserDao {
    private HibernateTemplate hibernateTemplate;

    public void saveUser(User user) {
        hibernateTemplate.save(user);
    }
}
However, it’s now considered deprecated and not recommended in new projects. Instead, use Spring Data JPA or the JPA EntityManager directly with @Transactional.

✅ 4. How is transaction management handled in Spring ORM?
Answer:
Spring supports declarative transaction management using @Transactional. It integrates with Hibernate’s SessionFactory and manages transactions using PlatformTransactionManager.
@Service
public class EmployeeService { 
    @Autowired
    private EmployeeRepository repository;

    @Transactional
    public void saveEmployee(Employee emp) {
        repository.save(emp);
    }
}
Spring automatically:
Begins a transaction
Commits it if no exceptions occur
Rolls back on runtime exceptions
Note: You need to configure HibernateTransactionManager bean in XML or Java Config if not using Spring Boot.

✅ 5. What is LazyInitializationException and how do you solve it?
Answer:
LazyInitializationException occurs when you try to access a lazily loaded association outside of an active Hibernate session.
Employee emp = employeeService.findById(1);
emp.getDepartment().getName(); // LazyInitializationException here if session is closed
Solutions:
Use @Transactional on the service method to keep the session open.
Use fetch joins in your query:

@Query("SELECT e FROM Employee e JOIN FETCH e.department WHERE e.id = :id")
Change fetching strategy to EAGER (not always recommended for performance).

✅ 6. What are the advantages of using Spring with Hibernate?
Answer:
Declarative Transaction Management using @Transactional
Easy session and resource handling
Exception translation (DataAccessException)
Integration with Spring Data JPA for cleaner, boilerplate-free code
Dependency injection for SessionFactory and DAOs

✅ 7. How does Spring convert Hibernate exceptions into Spring exceptions?
Answer:
Spring translates Hibernate-specific exceptions (e.g., org.hibernate.ObjectNotFoundException) into Spring’s unified unchecked exceptions (e.g., org.springframework.dao.DataAccessException) using the @Repository annotation.

This happens through:
PersistenceExceptionTranslationPostProcessor

@Repository
public class EmployeeDao {
    // Hibernate exception translated into Spring's exception
}
This abstraction allows you to decouple your DAO layer from Hibernate-specific exceptions.

✅ 8. How does Spring Boot simplify Hibernate configuration?
Answer:
Spring Boot:
Auto-configures DataSource, SessionFactory, and TransactionManager
Automatically detects entity classes annotated with @Entity
Manages transaction boundaries with @Transactional
Uses application.properties or application.yml for configuration:

spring.datasource.url=jdbc:mysql://localhost/test
spring.datasource.username=root
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
No need for boilerplate XML or Java config for Hibernate setup.

✅ 9. How does @Transactional behave with different propagation types in Hibernate integration?
Answer:
Propagation defines how transactions relate to each other. Common types:
REQUIRED (default): Use existing transaction or create new
REQUIRES_NEW: Suspend current, start a new transaction
NESTED: Create nested transaction (savepoint)
SUPPORTS: Run with or without a transaction

@Transactional(propagation = Propagation.REQUIRES_NEW)
public void saveAuditLog(Audit audit) {
    auditRepo.save(audit);  // runs in its own transaction
}
✅ 10. Can we use Spring Data JPA with Hibernate as a provider?
Answer:
Yes, Spring Data JPA is an abstraction built over JPA (Java Persistence API), and Hibernate is the most commonly used JPA provider.
In Spring Boot:
properties

spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect
spring.jpa.show-sql=true
Repositories can be defined as:
public interface EmployeeRepository extends JpaRepository<Employee, Long> {}
Spring Data takes care of implementation behind the scenes using Hibernate.

1. What is Spring Data JPA?
Answer:
Spring Data JPA is a part of the larger Spring Data project that aims to simplify data access using the Java Persistence API (JPA). It provides:
Repository support (e.g., JpaRepository, CrudRepository)
Automatic query method derivation
Paging and sorting
Integration with underlying JPA providers (like Hibernate)
Benefits:
No need to write boilerplate DAO code.
Simplifies custom query creation with @Query and query methods.

2. What is the difference between JPA and Spring Data JPA?
Answer:
Feature	                          JPA (Java Persistence API)	                            Spring Data JPA
Specification                      	A Java specification (JSR-338)                      	A Spring module that implements JPA
Provider	                        Needs a provider like Hibernate	                        Built on top of JPA and integrates provider
Boilerplate Code	                Requires EntityManager, transactions	                  Eliminates boilerplate with repositories
Repository Support	              Manual implementation                                  	Auto-generated interfaces (JpaRepository)
Querying	                        JPQL or Criteria API	                                  Method name queries, JPQL, Native, Criteria

3. What is JpaRepository and how is it different from CrudRepository?
Answer:
JpaRepository is an interface in Spring Data JPA that provides full JPA functionality, including:
Basic CRUD methods (from CrudRepository)
Pagination and sorting (from PagingAndSortingRepository)
JPA-specific methods like flush() and saveAndFlush()
Hierarchy:
JpaRepository → PagingAndSortingRepository → CrudRepository
Example:
public interface UserRepository extends JpaRepository<User, Long> {
}
4. How are queries derived in Spring Data JPA?
Answer:
Spring Data JPA can automatically generate queries based on method names.
Examples:
List<User> findByName(String name);
User findByEmailAndStatus(String email, String status);
List<User> findTop5ByOrderByCreatedDateDesc();
Spring parses the method name and generates the appropriate JPQL or SQL query.

5. What is the use of @Query annotation?
Answer:
@Query is used to define custom JPQL or native SQL queries when method name derivation isn't sufficient.
@Query("SELECT u FROM User u WHERE u.status = ?1")
List<User> findByStatus(String status);
With Native SQL:
@Query(value = "SELECT * FROM users WHERE status = ?1", nativeQuery = true)
List<User> findByStatusNative(String status);
6. What is the difference between getOne() and findById()?
Answer:
Method	Behavior
findById()	Returns Optional<T> immediately; executes the query
getOne()	Returns a proxy; query is deferred until accessed (Lazy)
In Spring Boot 2.0+, getOne() is replaced by getReferenceById().

7. What is the purpose of @Modifying annotation?
Answer:
When executing update/delete queries using @Query, you must annotate the method with @Modifying.
@Transactional
@Modifying
@Query("UPDATE User u SET u.status = ?1 WHERE u.id = ?2")
int updateUserStatus(String status, Long id);
It tells Spring Data that the query modifies data and is not a select.

8. How do you perform pagination and sorting in Spring Data JPA?
Answer:
Use the Pageable and Sort interfaces.
Page<User> findByStatus(String status, Pageable pageable);
Usage:
Pageable pageable = PageRequest.of(0, 10, Sort.by("name").descending());
Page<User> page = userRepository.findByStatus("ACTIVE", pageable);
9. What is the N+1 problem in JPA, and how can Spring Data JPA help resolve it?
Answer:
N+1 Problem: Occurs when JPA executes one query to get parent entities and then N additional queries for each child entity due to lazy loading.
Solutions:
Use JOIN FETCH in a @Query
Use EntityGraph to fetch associations eagerly
@Query("SELECT u FROM User u JOIN FETCH u.roles")
List<User> findAllUsersWithRoles();
10. How to use projections in Spring Data JPA?
Answer:
Projections allow fetching partial data using interfaces or DTOs.
Interface Projection:
public interface UserView {
    String getName();
    String getEmail();
}

List<UserView> findByStatus(String status);
DTO Projection:
@Query("SELECT new com.example.UserDTO(u.name, u.email) FROM User u")
List<UserDTO> findAllUserDTOs();
11. What are specifications in Spring Data JPA?
Answer:
Specification<T> is part of Spring Data JPA for dynamic query construction using the JPA Criteria API.

public static Specification<User> hasStatus(String status) {
    return (root, query, cb) -> cb.equal(root.get("status"), status);
}
Used with JpaSpecificationExecutor:
List<User> users = userRepository.findAll(hasStatus("ACTIVE"));
12. What is the role of EntityManager in Spring Data JPA?
Answer:
Even with Spring Data JPA, you can use the lower-level EntityManager to:
Create dynamic queries (Criteria API)
Flush/persist entities manually
Manage transactions programmatically (rare in Spring)
@Autowired
private EntityManager entityManager;

public void customSave(User user) {
    entityManager.persist(user);
}
13. How does Spring Data JPA handle transactions?
Answer:
Spring uses @Transactional annotation to manage transactions.
Read-only by default on repository methods
Use @Transactional at service layer for write operations
@Transactional
public void saveUser(User user) {
    userRepository.save(user);
}
14. Can you explain how audit fields (createdDate, modifiedDate) are handled?
Answer:
Spring Data JPA supports automatic auditing.
Steps:
Add fields with annotations:
@CreatedDate
private LocalDateTime createdDate;
@LastModifiedDate
private LocalDateTime modifiedDate;
Annotate entity with @EntityListeners(AuditingEntityListener.class)
Enable auditing in configuration:
@EnableJpaAuditing

15. How do you handle soft deletes in Spring Data JPA?
Answer:
Soft delete means marking records as inactive instead of deleting them physically.
Approach:
Add a field deleted: boolean
Override repository methods with condition:
List<User> findByDeletedFalse();
For custom delete:
@Modifying
@Query("UPDATE User u SET u.deleted = true WHERE u.id = ?1")
void softDeleteById(Long id);


1. What is the Spring Framework?
Answer:
Spring is a lightweight, open-source Java framework for building enterprise-level applications. It provides comprehensive infrastructure support for developing Java applications, especially loosely coupled, testable, and maintainable code.

Key features:
Dependency Injection (DI)
Aspect-Oriented Programming (AOP)
MVC framework
Data access integration
Transaction management

2. What are the core modules of the Spring Framework?
Answer:
Spring is divided into multiple modules. Major ones include:

Module	                            Description
Core Container                    	Provides core features (Beans, DI, Factory, Context)
Spring AOP                          Supports Aspect-Oriented Programming
Data Access / Integration	          JDBC, ORM, JMS, Transactions
Web / MVC	                          Web and RESTful services using Spring MVC
Test	                              Support for testing Spring components using JUnit/TestNG

3. What is Inversion of Control (IoC)?
Answer:
IoC is a principle where the control of object creation and dependency management is delegated to the Spring container instead of the programmer. This is mainly achieved through Dependency Injection (DI).

4. What is Dependency Injection (DI)?
Answer:
DI is a design pattern where an object’s dependencies are injected at runtime rather than being created internally.
Types in Spring:
Constructor Injection
Setter Injection
Field Injection (via annotations)

Example (Setter Injection):

@Component
public class Car {
    @Autowired
    private Engine engine;
}

5. What are the different types of Spring containers?
Answer:
BeanFactory – Basic container providing DI
ApplicationContext – Advanced container, provides:
BeanFactory features
Event propagation
Internationalization
AOP support

6. What is a Spring Bean?
Answer:
A Spring Bean is any Java object that is managed by the Spring container and instantiated, assembled, and otherwise managed at runtime.

Beans are defined in:
@Component, @Service, @Repository, or @Bean
XML configuration

7. What is the difference between @Component, @Service, @Repository, and @Controller?
Answer:
Annotation            	Usage
@Component	            Generic bean
@Service	              Business logic
@Repository            	Data access layer (adds exception translation)
@Controller            	Web controller (used in Spring MVC)
All are stereotype annotations and are detected during component scanning.

8. What is the Spring Bean lifecycle?
Answer:
Bean instantiation
Dependency injection
BeanNameAware and BeanFactoryAware callbacks
InitializingBean.afterPropertiesSet() or @PostConstruct
Custom init-method
Bean is ready
DisposableBean.destroy() or @PreDestroy
Custom destroy-method

9. What is Autowiring in Spring?
Answer:
Spring autowires beans automatically by:
Type (@Autowired)
Qualifier (@Qualifier)
Name (@Autowired + @Qualifier)
Constructor-based autowiring

10. What is the difference between Singleton and Prototype bean scope?
Answer:
Scope	Description
singleton	Only one instance per Spring container (default)
prototype	New instance each time the bean is requested

@Scope("prototype")
@Component
public class MyBean { }

11. What is the use of @Configuration and @Bean annotations?
Answer:
@Configuration: Marks a class that provides bean definitions.
@Bean: Declares a method that returns a Spring bean.

@Configuration
public class AppConfig {
    @Bean
    public MyService service() {
        return new MyServiceImpl();
    }
}
12. What is Spring AOP?
Answer:
Aspect-Oriented Programming (AOP) allows separation of cross-cutting concerns like logging, security, and transactions.
Spring AOP concepts:
Aspect – Common logic
Advice – Action (Before, After, Around)
Join Point – Where advice applies (method)
Pointcut – When advice applies
Weaving – Applying advice to target object

13. What is the difference between ApplicationContext and BeanFactory?
Answer:
Feature                  	BeanFactory                            	ApplicationContext
Basic container          	Yes                                    	Yes
Eager loading	            No (lazy)	                              Yes
Internationalization	    No	                                    Yes
AOP Support	              No                                    	Yes
Event publishing	        No	                                    Yes

14. What is a proxy in Spring AOP?
Answer:
Spring AOP creates proxies to apply advice to target objects. Types:
JDK Dynamic Proxy – Interface-based
CGLIB Proxy – Subclass-based (used when no interfaces)

15. How is Spring different from other frameworks (like EJB)?
Answer:
Feature                        	Spring                      	EJB
Lightweight                    	Yes	                          No
Dependency Injection	          Yes	                          Partial/Manual
POJO-based	                    Yes                          	No
Testability                    	High	                        Low
Container	                      Any Servlet container	        Requires JEE container


✅ Spring AOP (Aspect-Oriented Programming)
What is AOP? Use Cases?

Terminologies: Aspect, Join Point, Advice, Pointcut, Weaving

Types of Advice: @Before, @After, @Around, etc.

Using @Aspect and @EnableAspectJAutoProxy





🔸 1. What is DispatcherServlet in Spring MVC?
Answer:
DispatcherServlet is the central front controller in the Spring MVC architecture. It intercepts all incoming HTTP requests and delegates them to the appropriate handlers (like controllers), views, or exceptions.
It's defined in web.xml (Spring < Boot) or auto-configured in Spring Boot.

🔸 2. Describe the complete workflow of DispatcherServlet.
Answer:
Here’s a step-by-step breakdown of how a request is processed by DispatcherServlet:

🧭 DispatcherServlet Workflow

Browser → DispatcherServlet → HandlerMapping → Controller → Service → DAO (if any)
          ↓                     ↓             ↓
       ViewResolver ← ModelAndView ← View ← Response to Browser
Detailed Steps:
Step 1: Request Entry
A client sends an HTTP request to a URL like /users.
This URL is mapped to the DispatcherServlet in web.xml or auto-configured in Spring Boot.

<servlet>
  <servlet-name>spring</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
</servlet>
Step 2: DispatcherServlet Initialization
On startup, DispatcherServlet loads configuration from web.xml or application.properties/application.yml (in Spring Boot).
Initializes core components:
HandlerMapping
HandlerAdapter
ViewResolver
ExceptionResolver

Step 3: Request Handling Begins
DispatcherServlet receives the HTTP request and consults HandlerMapping to find the correct Controller (@Controller) method.

@Controller
public class UserController {
    @GetMapping("/users")
    public String getUsers(Model model) {
        model.addAttribute("users", userService.findAll());
        return "user-list";
    }
}
Step 4: HandlerAdapter Calls Controller
Once the handler is found, HandlerAdapter invokes the controller method.
The controller prepares a ModelAndView object (data + view name).

Step 5: View Resolution
DispatcherServlet gives the view name to the ViewResolver.
The ViewResolver resolves the logical view name (user-list) to an actual JSP/Thymeleaf/HTML file.
@Bean
public InternalResourceViewResolver viewResolver() {
    InternalResourceViewResolver resolver = new InternalResourceViewResolver();
    resolver.setPrefix("/WEB-INF/views/");
    resolver.setSuffix(".jsp");
    return resolver;
}
Step 6: Render the View
The resolved view (e.g., /WEB-INF/views/user-list.jsp) is rendered with model data and sent back as an HTTP response.
Step 7: Response Sent
Finally, the HTML response is sent back to the browser, completing the request.

🔸 3. What are the key components of Spring MVC involved in this process?
Component	Description
DispatcherServlet	Front controller to handle all requests
HandlerMapping	Maps request URL to handler method
HandlerAdapter	Executes the handler method
Controller	Contains business logic
ModelAndView	Contains view name + data
ViewResolver	Resolves logical view name to actual resource
View	Renders the output

🔸 4. How is DispatcherServlet different from a normal servlet?
DispatcherServlet                          	Regular Servlet
Part of Spring MVC                        	Part of standard Java EE
Supports DI, AOP, and all                   Spring features	Manual configuration required
Works with HandlerMapping & ViewResolver  	No MVC abstraction
Easily testable and modular                	Less flexible and harder to test

🔸 5. How does DispatcherServlet know which controller to invoke?
It uses HandlerMapping which checks:
URL pattern
HTTP method (GET, POST)
Annotations like @RequestMapping, @GetMapping, etc.

🔸 6. What is ModelAndView?
It’s a wrapper class that carries:
The model (data to pass to view)
The view name (like user-list)
return new ModelAndView("user-list", "users", userService.findAll());

🔸 7. What happens if no view is resolved?
If no view is found, DispatcherServlet throws an exception like HttpMediaTypeNotAcceptableException or ViewResolutionException.

🔸 8. What is the default URL pattern for DispatcherServlet?
Common patterns:
/ — to capture all requests
*.do, *.action — in older projects
In Spring Boot, it’s auto-mapped to /

@RequestMapping, @GetMapping, etc.

ModelAndView, @ModelAttribute, @RequestParam, @PathVariable

Exception Handling (@ControllerAdvice, @ExceptionHandler)

File Uploading/Downloading

Validation (@Valid, @Validated)

🔹 1. What is Spring Boot? Why is it used?
Answer:
Spring Boot is a project built on top of the Spring Framework that simplifies application development by:
Eliminating boilerplate configuration.
Providing embedded servers (like Tomcat, Jetty).
Offering production-ready features (metrics, health checks).
Enabling fast development with “convention over configuration.”

🔹 2. Describe the architecture of Spring Boot.
Answer:
Spring Boot architecture includes the following layers and components:
🧱 Spring Boot Architecture Layers
Client (Browser, Postman)
        ↓
Controller Layer (@RestController)
        ↓
Service Layer (@Service)
        ↓
Repository Layer (@Repository – Spring Data JPA)
        ↓
Database (e.g., MySQL, MongoDB)
🔧 Key Components of Spring Boot Architecture
Component                              	Description
Spring Boot                            Starter	Predefined dependencies for features (e.g., spring-boot-starter-web, starter-data-jpa)
Auto Configuration	                   Automatically configures beans based on dependencies (@EnableAutoConfiguration)
Spring Boot CLI	                       Command-line tool to quickly develop apps
Spring Initializr	                     Web tool to generate Spring Boot project structure
Embedded Server	                       No need to deploy WARs, runs on embedded Tomcat/Jetty
Spring Boot Actuator	                 Adds production-ready features like health, metrics, env
Spring Boot DevTools	                 Enables hot reloading and live reload during dev
Application.properties / YAML	         Centralized and easy configuration

🔹 3. What is auto-configuration in Spring Boot?
Answer:
Auto-configuration is a feature that automatically configures Spring Beans based on the classpath and defined properties.
Enabled using: @EnableAutoConfiguration or @SpringBootApplication
Example: If spring-boot-starter-data-jpa is on the classpath, Spring auto-configures:
DataSource
JPA Vendor
Transaction Manager

🔹 4. What is a Spring Boot Starter?
Answer:
A starter is a dependency that groups other dependencies needed for a specific feature.
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
Includes:
Spring MVC
Jackson (JSON)
Embedded Tomcat

🔹 5. How does Spring Boot run without a web.xml file?
Answer:
Spring Boot uses Java-based configuration, not XML. It uses:
@SpringBootApplication (which includes @Configuration, @ComponentScan, and @EnableAutoConfiguration)
Embedded servers

@SpringBootApplication
public class MyApp {
  public static void main(String[] args) {
    SpringApplication.run(MyApp.class, args);
  }
}
🔹 6. What is the role of application.properties or application.yml?
Answer:
Used for centralized configuration like:
Server port
DB credentials
Logging levels
Custom properties

server.port=8081
spring.datasource.url=jdbc:mysql://localhost:3306/test
logging.level.org.springframework=DEBUG

🔹 7. What is SpringApplication.run()?
Answer:
This is the entry point of a Spring Boot app. It:
Boots the application context
Starts the embedded server
Initializes beans
SpringApplication.run(MyApp.class, args);

🔹 8. How is embedded Tomcat integrated?
Spring Boot includes Tomcat as a dependency (via spring-boot-starter-web). At runtime, it launches Tomcat as an embedded server, so no need to deploy WARs to an external server.

🔹 9. Can we disable auto-configuration in Spring Boot?
Yes, using:
@EnableAutoConfiguration(exclude = {DataSourceAutoConfiguration.class})
Or spring.autoconfigure.exclude in properties

🔹 10. What is Spring Boot DevTools?
Answer:
DevTools adds:
Hot swapping (code reload without restart)
Automatic restart on file changes
LiveReload support for browser auto-refresh
Useful for faster development and testing.

🔹 11. What is Spring Boot Actuator?
Answer:
Actuator exposes production-ready endpoints like:
/actuator/health
/actuator/info
/actuator/metrics
To monitor application health and metrics.

🔹 12. Can Spring Boot be used for non-web applications?
Yes. Spring Boot supports:
Web apps (REST APIs, web UI)
Command-line batch apps
Scheduled jobs
You can exclude the web server dependency and use just core features.
Auto-configuration
Spring Boot Starters
Embedded Tomcat
Application Properties/YAML
Custom Configuration (@ConfigurationProperties)
CommandLineRunner & ApplicationRunner
Actuator Endpoints
Spring Boot DevTools

1. What is Spring Security?
Answer:
Spring Security is a powerful and highly customizable authentication and access control framework. It is the de facto standard for securing Spring-based applications. It supports:
Authentication and Authorization
Protection against CSRF
Session management
Password encoding
Integration with OAuth2, LDAP, JWT, etc.

2. How does Spring Security work internally?
Answer:
Spring Security uses a filter chain (DelegatingFilterProxy) to intercept incoming HTTP requests. Key filters in the chain include:
UsernamePasswordAuthenticationFilter – for form-based login
BasicAuthenticationFilter – for basic auth
SecurityContextPersistenceFilter – loads/stores security context
ExceptionTranslationFilter – handles security-related exceptions
Authentication is performed using AuthenticationManager, and decisions are based on user roles and access rules defined in configuration.

3. What is the difference between Authentication and Authorization?
Answer:
Concept	                Description
Authentication	        Who you are (e.g., verifying username/password)
Authorization          	What you can do (e.g., access control based on roles)

4. What are the different ways to configure Spring Security?
Answer:
XML Configuration (legacy)
Java-based Configuration using @EnableWebSecurity and WebSecurityConfigurerAdapter (till Spring Security 5.7)
Lambda DSL style (from Spring Security 5.4+)

5. Explain WebSecurityConfigurerAdapter and why it's deprecated.
Answer:
WebSecurityConfigurerAdapter was used to configure custom security logic by overriding configure(HttpSecurity http) method. From Spring Security 5.7 onwards, it’s deprecated in favor of a component-based approach and builder-style SecurityFilterChain beans for better flexibility and readability.

6. What is CSRF and how does Spring Security handle it?
Answer:
Cross-Site Request Forgery (CSRF) is an attack where a user is tricked into submitting malicious requests. Spring Security protects against CSRF by:
Generating a CSRF token for each session.
Validating the token on state-changing requests (POST, PUT, DELETE).
You can disable it with:
http.csrf().disable();
But it should only be done for APIs (especially stateless).

7. What is the difference between Basic Authentication and Form Login?
Answer:
Feature                          	Basic Auth	                                  Form Login
UI	                              Browser popup	                                Custom HTML login form
Credential handling	              Sent with every request (Base64)	            Sent once, stored in session
Use Case	                        APIs or internal systems	                    Web applications
Security	                        Less secure (unless over HTTPS)	              More flexible with customization

8. How do you secure REST APIs using Spring Security?
Answer:
Use stateless sessions (sessionCreationPolicy.STATELESS)
Prefer JWT for authentication
Disable CSRF
Use role-based access with .antMatchers() or @PreAuthorize

Example:

  .csrf().disable()
  .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
  .authorizeRequests()
  .antMatchers("/api/public/**").permitAll()
  .anyRequest().authenticated();
9. What is the use of UserDetailsService?
Answer:
UserDetailsService is a core interface used to retrieve user-related data. It loads the user by username from any data source (like DB) and returns a UserDetails object containing username, password, and roles.

10. What is the difference between @PreAuthorize and @Secured?
Answer:
Annotation          	Description
@Secured	            Accepts only roles (e.g., @Secured("ROLE_ADMIN"))
@PreAuthorize	        Supports SpEL expressions (e.g., @PreAuthorize("hasRole('ADMIN') and #user.name == authentication.name"))

@PreAuthorize is more powerful and preferred in newer applications.


1. What is JWT and why is it used in Spring Security?
Answer:
JWT (JSON Web Token) is a compact, self-contained token format used to securely transmit information between parties. It is often used in stateless authentication mechanisms.
In Spring Security, JWT allows:
Stateless authentication (no server-side session)
Scalability across distributed systems
Easy integration with frontend apps (e.g., Angular, React)

Structure of JWT:
Header.Payload.Signature

2. What are the components of a JWT?
Answer:
Header: Specifies the algorithm (e.g., HS256) and token type.
Payload: Contains claims such as sub, exp, roles.
Signature: Ensures integrity; generated using secret key + algorithm.


{
  "alg": "HS256",
  "typ": "JWT"
}.
{
  "sub": "chetan",
  "roles": ["ADMIN"],
  "exp": 1695017721
}.
[Signature]

3. How does JWT-based authentication work in Spring Security?
Answer:
User logs in with credentials.
Backend authenticates using AuthenticationManager.
JWT is generated and returned to the client.
Client sends JWT in Authorization: Bearer <token> header on each request.
Spring Security validates JWT using a filter and sets the Authentication object in the context.

4. How do you configure Spring Security for JWT?
Answer:
You create a custom filter to:
Intercept requests
Extract and validate the JWT
Set SecurityContext with authenticated user
Key components:
JwtAuthenticationFilter
JwtUtils (token generator and validator)
SecurityConfig (to register filter)

http.csrf().disable()
    .authorizeRequests().antMatchers("/auth/**").permitAll()
    .anyRequest().authenticated()
    .and()
    .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);
    
5. How do you generate and validate a JWT in Spring Boot?
Answer:
Generate:

String token = Jwts.builder()
    .setSubject(username)
    .claim("roles", roles)
    .setIssuedAt(new Date())
    .setExpiration(new Date(System.currentTimeMillis() + JWT_EXPIRATION))
    .signWith(SignatureAlgorithm.HS256, SECRET_KEY)
    .compact();

Validate:

Claims claims = Jwts.parser()
    .setSigningKey(SECRET_KEY)
    .parseClaimsJws(token)
    .getBody();
    
6. What are some common JWT security concerns?
Answer:
Token leakage: Tokens stored in localStorage can be vulnerable to XSS.
No built-in revocation: Tokens can’t be invalidated easily.
Expiration must be short: Long-lived tokens can be misused.
Use HTTPS: Always encrypt tokens in transit.

7. How do you handle JWT token expiration?
Answer:
Short-lived access tokens (e.g., 15 mins)
Long-lived refresh tokens for reissuing access tokens
Spring Boot apps usually provide a /refresh-token endpoint to issue a new access token when the old one expires.

8. What is the difference between access token and refresh token?
Feature	                      Access Token	                          Refresh Token
Lifespan	                    Short-lived (minutes)	                  Long-lived (days/weeks)
Purpose	                      Authorize API access	                  Generate new access token
Stored where?	                Client/browser memory	                  Secure store (HttpOnly cookie)
Sent to server	              Every request                          	Only to refresh endpoint

9. How to add roles and permissions in JWT?
Answer:
When generating the token:

.setClaims(Map.of("roles", List.of("ADMIN", "USER")))
While validating:
String role = claims.get("roles", List.class);
In filter:
UsernamePasswordAuthenticationToken auth = 
  new UsernamePasswordAuthenticationToken(username, null, getAuthorities(roles));
  
10. Can you revoke a JWT?
Answer:
JWT is stateless, so tokens cannot be revoked directly.
Workarounds:
Use a token blacklist or token versioning
Maintain token state in DB/cache
Shorten token expiration + use refresh tokens
OAuth2 / SSO integration
Custom authentication filters


Ask ChatGPT

Basic Auth vs Form Login

Custom UserDetailsService

Role-based access (@PreAuthorize, @Secured)

JWT Authentication

Security Filters & Custom Authentication Providers

OAuth2 Integration

What is REST?
REST stands for Representational State Transfer. It is an architectural style for designing networked applications, particularly web services, that interact over HTTP.
🔍 Key Characteristics of REST
Stateless
Each request from the client to the server must contain all the information needed to understand and process the request.
The server does not store anything about the client's session between requests.
Client-Server Architecture
Separation of concerns: The client handles the user interface, and the server handles data and logic.
Uniform Interface
Standardized way of communication using HTTP methods:
GET – Retrieve resource
POST – Create resource
PUT – Update resource
DELETE – Delete resource
Resource-Based
Everything is considered a resource (e.g., users, orders).
Each resource is identified by a unique URI (Uniform Resource Identifier).

Example:
GET /api/users/1 → Fetch user with ID 1

Representation-Oriented
Resources are transferred in the form of representations (commonly JSON or XML).
The client can modify the resource by sending a new representation.
Cacheable
Responses must define whether they are cacheable or not to improve performance and scalability.
Layered System
REST allows an architecture to be composed of hierarchical layers by restricting component behavior.

✅ Example of a RESTful API in Spring
@RestController
@RequestMapping("/api/products")
public class ProductController {

    @GetMapping("/{id}")
    public Product getProduct(@PathVariable Long id) {
        return productService.getProductById(id);
    }

    @PostMapping
    public Product createProduct(@RequestBody Product product) {
        return productService.save(product);
    }
}
✅ Why REST?
Simple and easy to understand
Widely adopted using HTTP/HTTPS
Scalable and stateless
Works well with cloud and microservices architectures


1. What is Spring REST?
Answer:
Spring REST is a module of the Spring Web MVC framework that helps in creating RESTful web services. It uses HTTP verbs (GET, POST, PUT, DELETE) and provides annotations to easily expose REST APIs.
Spring REST is built on top of Spring MVC and simplifies:
JSON/XML serialization with Jackson or JAXB
HTTP status handling
Exception handling via @ControllerAdvice
Content negotiation via produces and consumes

2. How do you create a RESTful web service using Spring Boot?
Answer:
You annotate your controller with @RestController and use @RequestMapping or HTTP-specific annotations.
@RestController
@RequestMapping("/api/users")
public class UserController {

    @GetMapping("/{id}")
    public User getUser(@PathVariable Long id) {
        return userService.getUserById(id);
    }

    @PostMapping
    public ResponseEntity<User> createUser(@RequestBody User user) {
        return ResponseEntity.status(HttpStatus.CREATED)
                             .body(userService.saveUser(user));
    }
}

3. What is the difference between @Controller and @RestController?
Answer:
Annotation	          Behavior
@Controller	          Returns view (e.g., JSP/HTML). Used for MVC apps.
@RestController	      Combines @Controller and @ResponseBody. Returns data (JSON/XML) directly.

So, @RestController is ideal for building RESTful APIs.

4. How do you handle HTTP status codes in Spring REST?
Answer:
Using ResponseEntity:
return new ResponseEntity<>(user, HttpStatus.OK);

Global Exception Handling:

@ControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<String> handleNotFound(UserNotFoundException ex) {
        return new ResponseEntity<>(ex.getMessage(), HttpStatus.NOT_FOUND);
    }
}

5. How does Spring handle JSON/XML serialization?
Answer:
Spring Boot auto-configures Jackson (for JSON) or JAXB (for XML).
Annotate your model class with standard Java annotations like @JsonIgnore, @XmlElement, etc.
Spring handles content negotiation automatically via Accept and Content-Type headers.

6. What is Content Negotiation in Spring REST?
Answer:
Content negotiation determines the format (JSON, XML) of the response returned to the client based on:
Accept header (application/json, application/xml)
URL extension (e.g., /user.json)
Query param (e.g., /user?format=xml)
Spring handles it via ContentNegotiationManager.

7. How do you secure Spring REST APIs?
Answer:
Use Spring Security with Basic Auth, JWT, OAuth2.
Configure HTTP security to protect specific endpoints.
Use role-based access control with @PreAuthorize.
Example:
@PreAuthorize("hasRole('ADMIN')")
@GetMapping("/admin")
public String adminEndpoint() {
    return "Only accessible by admin";
}

8. How do you validate incoming request data in REST APIs?
Answer:
Use @Valid with @RequestBody and add validation annotations to your model class.

public class User {
    @NotNull
    private String name;

    @Email
    private String email;
}
Controller:
@PostMapping
public ResponseEntity<User> createUser(@Valid @RequestBody User user) {
    return ResponseEntity.ok(userService.save(user));
}
Exception handling with @ControllerAdvice for MethodArgumentNotValidException.

9. How do you test REST APIs in Spring Boot?
Answer:
Use Postman, cURL, or RestTemplate for manual testing.
For automated tests, use:
MockMvc for controller unit tests
@WebMvcTest for testing only web layer
TestRestTemplate or WebTestClient for integration tests

Example with MockMvc:

mockMvc.perform(get("/api/users/1"))
       .andExpect(status().isOk())
       .andExpect(jsonPath("$.name").value("Chetan"));
       
10. What is HATEOAS in Spring REST?
Answer:
HATEOAS (Hypermedia As The Engine Of Application State) is a constraint of REST where the server provides links in the response to guide the client on possible actions.

Spring supports HATEOAS using spring-hateoas.

Example:

json
Copy
Edit
{
  "id": 1,
  "name": "Chetan",
  "_links": {
    "self": { "href": "/api/users/1" },
    "orders": { "href": "/api/users/1/orders" }
  }
}

Versioning APIs

HATEOAS basics

Swagger/OpenAPI integration

✅ Spring Testing
Unit Testing with @WebMvcTest, @DataJpaTest, @SpringBootTest

Mockito + Spring

Test slices and embedded DBs (H2)

✅ Spring Cloud (for Microservices)
Spring Cloud Config

Spring Cloud Netflix (Eureka, Ribbon, Feign)

Spring Gateway

Circuit Breaker Interview Questions with Detailed Answers
1. What is the Circuit Breaker pattern? Why is it used?
Answer:
The Circuit Breaker is a design pattern used in distributed systems to prevent the system from repeatedly trying to execute an operation that is likely to fail. It's especially useful when integrating with remote services. The pattern improves the system's stability and resilience by allowing it to fail fast and recover gracefully.
When a service call repeatedly fails (e.g., due to timeout or server errors), the circuit breaker trips and further calls to the service are not made. Instead, a fallback response can be returned. After a predefined time, the circuit allows limited test requests to check if the service has recovered.

2. What are the different states of a circuit breaker?
Answer:
A circuit breaker typically has three states:
Closed: The default state. Calls are allowed and monitored. If the failure rate exceeds a threshold, it moves to Open.
Open: Calls are blocked to prevent further failures. After a specified time, it transitions to Half-Open.
Half-Open: Limited calls are allowed. If they succeed, the circuit returns to Closed. If they fail, it goes back to Open.

This mechanism prevents system overload and enables self-healing.

3. How is Circuit Breaker implemented in Spring Boot using Resilience4j?
Answer:
Resilience4j is a lightweight fault tolerance library. To implement a circuit breaker in Spring Boot:
Step 1: Add the dependency:

<dependency>
  <groupId>io.github.resilience4j</groupId>
  <artifactId>resilience4j-spring-boot2</artifactId>
  <version>1.7.1</version>
</dependency>

Step 2: Annotate your method:

@CircuitBreaker(name = "userService", fallbackMethod = "fallbackUser")
public User getUser(String id) {
    // Call to remote service
}

public User fallbackUser(String id, Throwable t) {
    return new User("default", "Fallback User");
}

Step 3: Configure in application.yml:

resilience4j.circuitbreaker:
  instances:
    userService:
      slidingWindowSize: 5
      failureRateThreshold: 50
      waitDurationInOpenState: 10s
      permittedNumberOfCallsInHalfOpenState: 2

4. What is the difference between Hystrix and Resilience4j?
Feature                	Hystrix	                                                  Resilience4j
Design	                Uses thread pools (isolation via separate threads)	      Uses decorators and lambdas
Java 8 Features	        Limited support	Fully supports                            Java 8 lambdas and functional interfaces
Modular                	Monolithic design                                        	Highly modular, each feature is in a separate dependency (circuit breaker, rate                                                                                       limiter, retry, etc.)
Resilience4j is a modern alternative to Hystrix with support for modular features (circuit breaker, rate limiter, retry, bulkhead).

5. What is fallback method in circuit breaker?
Answer:
A fallback method is used to provide an alternative response when the original method fails or the circuit is open. This ensures that your application doesn’t crash and can still serve degraded responses.
Example:

public String getData() {
    return restTemplate.getForObject("http://remote-service/data", String.class);
}

public String fallbackData(Throwable t) {
    return "Default Data";
}

6. What parameters can you configure in Resilience4j circuit breaker?
Answer:
slidingWindowSize: Number of calls in the window for calculation.
failureRateThreshold: % of failures that will trip the circuit.
waitDurationInOpenState: Duration to wait before transitioning from open to half-open.
permittedNumberOfCallsInHalfOpenState: Trial calls in half-open state.
minimumNumberOfCalls: Minimum calls before calculating failure rate.

7. Can Circuit Breakers be used with asynchronous or reactive calls?
Answer:
Yes. Resilience4j supports both synchronous and asynchronous/reactive calls (e.g., using CompletableFuture or Reactor). This is useful in non-blocking microservices using WebFlux or reactive programming.
Example:

@CircuitBreaker(name = "reactiveService")
public Mono<String> getReactiveData() {
    return webClient.get().uri("/data").retrieve().bodyToMono(String.class);
}

8. What are common mistakes when using Circuit Breakers?
Answer:
Not configuring thresholds properly (causing either too frequent or too rare trips).
Not monitoring or logging fallback usage.
Forgetting to handle exceptions in fallback methods.
Using blocking code in fallback or reactive flows.


 1. What is MongoTemplate in Spring Data MongoDB?
Answer:
MongoTemplate is the central class in Spring Data MongoDB for performing MongoDB operations. It provides a high-level abstraction to interact with the Mongo database by using methods for common operations like insert(), find(), update(), remove(), and more.
It is the lower-level alternative to using repository interfaces (MongoRepository or CrudRepository), and gives you more control over queries and operations.

✅ 2. How is MongoTemplate different from MongoRepository?
Aspect              	MongoRepository                                          	MongoTemplate
Abstraction          	High-level	                                              Low-level
Usage                	Declarative, interface-based	                            Programmatic, class-based
Custom Queries	      Through method names or @Query	                          Using Query, Criteria
Flexibility	          Less	                                                    More
Learning Curve	      Easier	                                                  Steeper
Conclusion:
Use MongoRepository for quick CRUD; use MongoTemplate for complex, fine-grained operations.

✅ 3. How do you create a custom query using MongoTemplate?
Answer:

Query query = new Query();
query.addCriteria(Criteria.where("name").is("John").and("age").gte(25));
List<User> users = mongoTemplate.find(query, User.class);
Query represents the MongoDB query.
Criteria helps build the condition.
mongoTemplate.find() executes the query.

✅ 4. How do you update documents using MongoTemplate?
Answer:

Query query = new Query(Criteria.where("email").is("test@example.com"));
Update update = new Update().set("name", "Updated Name");

mongoTemplate.updateFirst(query, update, User.class);
updateFirst() updates only the first matching document.
Use updateMulti() to update all matching records.
Use upsert() to update if exists or insert if not.

✅ 5. How do you perform aggregation using MongoTemplate?
Answer:
Aggregation agg = Aggregation.newAggregation(
    Aggregation.match(Criteria.where("status").is("ACTIVE")),
    Aggregation.group("department").count().as("total"),
    Aggregation.sort(Sort.Direction.DESC, "total")
);
AggregationResults<DepartmentStats> results = 
    mongoTemplate.aggregate(agg, "users", DepartmentStats.class);
Aggregation is used to build MongoDB aggregation pipelines.
DepartmentStats is a DTO to hold results.

✅ 6. How do you insert or save a document using MongoTemplate?
Answer:
mongoTemplate.insert(new User("John", 30));  // Insert only new
mongoTemplate.save(user);                    // Insert or update
insert() throws an error if the document exists.
save() checks if _id is present and updates or inserts accordingly.

✅ 7. What is the difference between findOne() and findById() in MongoTemplate?
Method	Description
findById(id, class)	Searches by _id field only
findOne(query, class)	Allows arbitrary query criteria
Example:
mongoTemplate.findById("123", User.class);
mongoTemplate.findOne(Query.query(Criteria.where("email").is("abc@example.com")), User.class);
✅ 8. How to check if a document exists using MongoTemplate?

Query query = Query.query(Criteria.where("email").is("abc@example.com"));
boolean exists = mongoTemplate.exists(query, User.class);
✅ 9. How do you delete documents with MongoTemplate?
Query query = Query.query(Criteria.where("status").is("INACTIVE"));
mongoTemplate.remove(query, User.class);
✅ 10. What are some real-world use cases where MongoTemplate is preferred over MongoRepository?
Answer:
Complex queries with multiple AND/OR conditions
Dynamic query construction at runtime
Multi-collection joins using aggregation
Complex update operations (like inc, unset, push)
Fine-tuned performance control over bulk operations

✅ 1. What is RestTemplate in Spring?
Answer:
RestTemplate is a synchronous client provided by Spring for consuming RESTful web services. It simplifies HTTP communication by providing methods to perform operations like GET, POST, PUT, DELETE, etc., abstracting away low-level details.
It’s part of spring-web and is often used in microservices architecture to call other services.

✅ 2. What are some common methods provided by RestTemplate?
Method	Description
getForObject()	Sends a GET request and returns the response body as an object
getForEntity()	Returns full ResponseEntity including status and headers
postForObject()	Sends a POST request and returns the result
postForEntity()	Returns full ResponseEntity for POST request
exchange()	Fully customizable request with HTTP method, headers, etc.
put()	Sends a PUT request
delete()	Sends a DELETE request

✅ 3. What is the difference between getForObject() and getForEntity()?
Feature	getForObject()	getForEntity()
Return Type	Deserialized body only	ResponseEntity<T>
Status Code	Not accessible	Accessible
Headers	Not accessible	Accessible
Example:
User user = restTemplate.getForObject(url, User.class);
ResponseEntity<User> response = restTemplate.getForEntity(url, User.class);
HttpStatus status = response.getStatusCode();

✅ 4. How do you send custom headers using RestTemplate?
Answer:
Use HttpHeaders and HttpEntity with the exchange() method:
HttpHeaders headers = new HttpHeaders();
headers.set("Authorization", "Bearer token123");
HttpEntity<String> entity = new HttpEntity<>(headers);
ResponseEntity<String> response = restTemplate.exchange(
    url, HttpMethod.GET, entity, String.class
);

✅ 5. How do you post a JSON request with a body using RestTemplate?
Answer:

HttpHeaders headers = new HttpHeaders();
headers.setContentType(MediaType.APPLICATION_JSON);
User user = new User("john", "john@gmail.com");
HttpEntity<User> request = new HttpEntity<>(user, headers);
ResponseEntity<String> response = restTemplate.postForEntity(url, request, String.class);

✅ 6. How do you handle errors in RestTemplate?
Answer:
Implement a custom ResponseErrorHandler:
restTemplate.setErrorHandler(new ResponseErrorHandler() {
    public boolean hasError(ClientHttpResponse response) {
        return response.getStatusCode().series() == HttpStatus.Series.CLIENT_ERROR
            || response.getStatusCode().series() == HttpStatus.Series.SERVER_ERROR;
    }

    public void handleError(ClientHttpResponse response) {
        // Custom error handling
    }
});

✅ 7. Why is RestTemplate considered deprecated in Spring 5?
Answer:
Spring 5 introduced the WebClient (from spring-webflux) as a modern, non-blocking, and reactive alternative to RestTemplate.
RestTemplate is blocking (each thread waits for the response).
WebClient is non-blocking and supports asynchronous and streaming responses.
However, RestTemplate is still widely used and supported in existing codebases.

✅ 8. How do you use exchange() method for PUT or DELETE?
Answer:

HttpHeaders headers = new HttpHeaders();
headers.setContentType(MediaType.APPLICATION_JSON);
User user = new User("newName", "email@test.com");
HttpEntity<User> request = new HttpEntity<>(user, headers);

// PUT
restTemplate.exchange(url, HttpMethod.PUT, request, Void.class);

// DELETE
restTemplate.exchange(url, HttpMethod.DELETE, request, Void.class);

✅ 9. How do you send query parameters with RestTemplate?

String url = "http://localhost:8080/users?name={name}&age={age}";
Map<String, String> params = Map.of("name", "john", "age", "30");
User user = restTemplate.getForObject(url, User.class, params);

✅ 10. Can you log or intercept RestTemplate requests and responses?
Answer:
Yes, by customizing ClientHttpRequestInterceptor:
restTemplate.setInterceptors(List.of((request, body, execution) -> {
    System.out.println("Request URI: " + request.getURI());
    return execution.execute(request, body);
}));

✅ 1. What is WebClient in Spring?
Answer:
WebClient is a non-blocking, reactive HTTP client introduced in Spring 5 as part of the Spring WebFlux module. It replaces RestTemplate for asynchronous and streaming-based HTTP communication.
Key features:
Non-blocking I/O (asynchronous)
Reactive Streams support (uses Reactor)
Supports streaming data, WebSockets, SSE
Works with both annotated (@RestController) and functional endpoints

✅ 2. How does WebClient differ from RestTemplate?
Feature              	RestTemplate                                	WebClient
Blocking            	Yes	                                          No (non-blocking)
Async support	        Limited	                                      Full async with reactive streams
Streaming	            Not supported	                                Supported
Thread usage	        Tied to servlet thread	                      Event loop, fewer threads
Future support      	Deprecated in favor of                        WebClient	Preferred for new dev

✅ 3. Basic example of a GET request using WebClient

WebClient webClient = WebClient.create();
Mono<String> response = webClient
        .get()
        .uri("http://localhost:8080/api/data")
        .retrieve()
        .bodyToMono(String.class);

// To actually get data (blocking for simplicity)
String result = response.block();

✅ 4. How to send a POST request with a request body?

WebClient webClient = WebClient.create();

Mono<ResponseEntity<String>> response = webClient
    .post()
    .uri("http://localhost:8080/users")
    .header(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
    .bodyValue(new User("chetan", "mail@example.com"))
    .retrieve()
    .toEntity(String.class);
    
✅ 5. What is the role of Mono and Flux in WebClient?
Mono<T>: A reactive publisher that emits 0 or 1 item (used for most API calls).
Flux<T>: A reactive publisher that emits 0 or more items (used for streaming APIs, like SSE).
These are part of Project Reactor and allow you to build reactive pipelines.

✅ 6. How to handle errors in WebClient?

webClient.get()
        .uri("http://localhost:8080/users/123")
        .retrieve()
        .onStatus(HttpStatus::is4xxClientError, response -> 
            Mono.error(new RuntimeException("Client Error!")))
        .onStatus(HttpStatus::is5xxServerError, response -> 
            Mono.error(new RuntimeException("Server Error!")))
        .bodyToMono(String.class);
        
✅ 7. What is .exchange() vs .retrieve() in WebClient?
Method	Description
retrieve()	Shortcut for basic responses, directly extracts body
exchangeToMono() / exchangeToFlux()	Gives access to full ClientResponse, more flexible
webClient.get()
    .uri("/data")
    .retrieve() // easy to use
    .bodyToMono(String.class);

webClient.get()
    .uri("/data")
    .exchangeToMono(response -> {
        if (response.statusCode().is2xxSuccessful()) {
            return response.bodyToMono(String.class);
        } else {
            return Mono.error(new RuntimeException("Failed!"));
        }
    });
    
✅ 8. How to add global headers or base URL?

WebClient webClient = WebClient.builder()
        .baseUrl("http://localhost:8080")
        .defaultHeader(HttpHeaders.AUTHORIZATION, "Bearer token123")
        .build();
        
✅ 9. How do you send query parameters using WebClient?

webClient.get()
    .uri(uriBuilder ->
        uriBuilder.path("/search")
                  .queryParam("name", "john")
                  .queryParam("age", 30)
                  .build())
    .retrieve()
    .bodyToMono(String.class);
    
✅ 10. How to log WebClient request and response?
WebClient webClient = WebClient.builder()
    .baseUrl("http://localhost:8080")
    .filter(ExchangeFilterFunction.ofRequestProcessor(clientRequest -> {
        System.out.println("Request: " + clientRequest.method() + " " + clientRequest.url());
        return Mono.just(clientRequest);
    }))
    .filter(ExchangeFilterFunction.ofResponseProcessor(clientResponse -> {
        System.out.println("Response Status: " + clientResponse.statusCode());
        return Mono.just(clientResponse);
    }))
    .build();
    
✅ 11. How do you handle timeouts in WebClient?
HttpClient httpClient = HttpClient.create()
    .responseTimeout(Duration.ofSeconds(5));

WebClient webClient = WebClient.builder()
    .clientConnector(new ReactorClientHttpConnector(httpClient))
    .build();
    
✅ 12. How to consume streaming responses (e.g. Server-Sent Events)?
webClient.get()
    .uri("/sse-stream")
    .accept(MediaType.TEXT_EVENT_STREAM)
    .retrieve()
    .bodyToFlux(String.class)
    .subscribe(System.out::println);

1. What is OpenFeign?
Answer:
OpenFeign is a declarative HTTP client that allows you to define REST API calls using Java interfaces. With Spring Cloud integration, it simplifies communication between microservices by abstracting the HTTP client logic.
Instead of writing boilerplate code using RestTemplate, you can just write:

@FeignClient(name = "user-service")
public interface UserClient {
    @GetMapping("/users/{id}")
    User getUser(@PathVariable Long id);
}
✅ 2. How is OpenFeign different from RestTemplate and WebClient?
Feature                	RestTemplate                        	WebClient	                                  OpenFeign
Programming	            Imperative                          	Reactive (non-blocking)                    	Declarative
Boilerplate            	More	                                Moderate	                                  Very little
Load Balancing        	Manual                              	Manual                                    	Built-in via Spring Cloud
Circuit Breaker        	Manual config	                        Manual config                              	Easy fallback support
Reactive	No	Yes	No

✅ 3. How to enable OpenFeign in a Spring Boot application?
Steps:
Add dependency:
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
Enable Feign clients:
@SpringBootApplication
@EnableFeignClients
public class App { ... }
Create interface:

@FeignClient(name = "order-service")
public interface OrderClient {
    @GetMapping("/orders/{id}")
    Order getOrder(@PathVariable Long id);
}

✅ 4. How do you pass headers to a Feign client?
Option 1: Using @RequestHeader

@GetMapping("/user")
User getUser(@RequestHeader("Authorization") String token);
Option 2: Global headers using RequestInterceptor
@Component
public class FeignHeaderInterceptor implements RequestInterceptor {
    public void apply(RequestTemplate template) {
        template.header("Authorization", "Bearer " + getToken());
    }
}
✅ 5. How do you configure a fallback with OpenFeign?
Use the fallback attribute of @FeignClient.
@FeignClient(name = "user-service", fallback = UserFallback.class)
public interface UserClient {
    @GetMapping("/users/{id}")
    User getUser(@PathVariable Long id);
}

@Component
class UserFallback implements UserClient {
    public User getUser(Long id) {
        return new User("fallback", "unknown@example.com");
    }
}
Make sure you also configure a circuit breaker library like Resilience4j.

✅ 6. How to log requests and responses in OpenFeign?
Step 1: Set log level bean
@Bean
Logger.Level feignLoggerLevel() {
    return Logger.Level.FULL;
}
Step 2: Enable debug logs
logging.level.feign.client=DEBUG

✅ 7. Can OpenFeign support multipart file upload?
Yes, but it's tricky. Use @RequestPart:

@PostMapping(value = "/upload", consumes = MediaType.MULTIPART_FORM_DATA_VALUE)
String uploadFile(@RequestPart("file") MultipartFile file);
Ensure your encoder supports multipart (like SpringFormEncoder).

✅ 8. How does Feign work under the hood?
Feign uses JDK dynamic proxies.
At runtime, it builds the HTTP request from the annotations (@GetMapping, etc.).
If Spring Cloud is used, it integrates with:
Eureka for service discovery
Ribbon or Spring Cloud LoadBalancer for client-side load balancing
Hystrix/Resilience4j for circuit breakers

✅ 9. How do you send POST requests using OpenFeign?

@PostMapping("/users")
User createUser(@RequestBody User user);

✅ 10. What are common errors with Feign and how to fix them?
Error	Cause	Solution
404 Not Found	Wrong endpoint	Check path, service name
503 Service Unavailable	Eureka can’t find service	Verify service registration
IllegalStateException: No fallbackFactory	Missing fallback class	Define fallback/factory
Content type error	Header mismatch	Use correct Content-Type

✅ 11. How can you pass dynamic base URLs to a Feign client?
Use @FeignClient(name = "x", url = "${dynamic.url}") and set value via application.yml.
You can also use a FeignClientFactoryBean with a custom configuration.

✅ 12. Can Feign call external APIs?
Yes. Just specify the url attribute.
@FeignClient(name = "google-api", url = "https://maps.googleapis.com")
public interface GoogleClient {
    @GetMapping("/maps/api/something")
    String getMapData();
}
✅ 13. How does Feign handle load balancing?
If you're using Spring Cloud and Eureka, Feign automatically picks up service instances and load balances using Ribbon or Spring Cloud LoadBalancer.
You don’t need to specify the full URL—just the service name registered with Eureka.

✅ 14. What are alternatives to Feign?
RestTemplate: Verbose, but widely used
WebClient: Reactive and modern, supports streaming
Retrofit: Another declarative HTTP client
Spring Cloud Gateway (for APIs)

✅ 15. Can you customize error handling in Feign?
Yes, using ErrorDecoder:
@Component
public class CustomErrorDecoder implements ErrorDecoder {
    @Override
    public Exception decode(String methodKey, Response response) {
        if (response.status() == 404) {
            return new NotFoundException("Resource not found");
        }
        return new RuntimeException("Generic error");
    }
}
 
