- This is a copy of Module 2 of REST With Spring: The Starter Class

- I'm writing this in order to write this application from the ground up, rather than
  following along with the examples; I think there is much more I can learn by taking
  this approach than by purely following along with the lessons on a ready-made project
  that I don't fully understand.
  
Steps I took to rewrite the project...

POM.XML
- Since this is a multi-module project, I started out by writing the parent pom.
  - I just created a folder on my laptop and added a pom.xml file. I copied over some boilerplate
    to that pom, like the <project>, <modelVersion>, <groupId>, <artifactId>, <version>...
  - And then added two more tags: <packaging> and <modules>
  - The <packaging> and <modules> tags are important as they are the ones that indicate that this
    is a parent pom. Since the packaging is 'pom' and not jar, war, ear... this says that this is
    simply a pom. 
  - The <modules> tag goes with the <packaging> tag and it indicates the names (artifactId, or 
    <build><finalName>final-component-name</finalName></build>) of the sub-modules of this parent
    pom.
  - The parent pom then contains the 'dependency' to spring-boot-starter-parent as well as many
    other properties that affect all sub-modules.
  - Note how there are no <dependencies> in the parent pom, only <build>, <properties>, and other
    similar tags.
    
 - Inside the folder that contains the parent pom, I created a folder for each of the sub-modules.
 - Inside each of those sub-module folders, I created a Maven Project through Eclipse. During the
   sub-module creation process, I came to a screen that asked for the group, artifact,... and below
   those was a parent section. That's where I added the info pertaining to the parent pom.
 - After creating the Maven sub-module project, Eclipse linked those up so that the parent project
   now had folders belonging to each of the sub-module projects I created.
 - Also, after creating each Maven sub-module project, I right-clicked on that project and 
   Maven > Update Project; this created the folder structure of a Maven project.
 - I added the <dependencies> and <build> tag items from the Baeldung project poms.
 - Once I was done with the poms of the sub-modules, I right-clicked on the parent projects and
   Maven > Update Project; the sub-modules now show the Spring Elements among other things.
   
 main()
 - Initially, the project had two submodules, and one had a dependency on the other. The webapp
   had a dependency on the common module, which from a high level makes sense, since it's a common
   module.
 - The pom of the common module contained a <relativePath>..</relativePath> in the <parent> tag.
   The webapp did not, which I don't fully understand. Also, the webapp is the project that has
   the main(). So there might be some relationship between those two. I'll see when I add another
   module later.
 - The main() method class in the webapp had a lot of things I've never seen before...
   - There are three things that I'll try to figure out:
   1. The @Import statement and the files provided
   2. The two @Bean methods involving the DispatcherServlet
   3. The SpringBootServletInitializer super-class
 - First, here is an explanation of what Servlets are from a Baeldung page on servlets:
 "Servlets are under the control of another Java application called a Servlet Container. When an 
 application running in a web server receives a request, the Server hands the request to the Servlet 
 Container – which in turn passes it to the target Servlet."
   - So, Request -> Server --Request--> Servlet Container --Request--> Servlet
 1. @Import - This annotation allows you to import (or include) other @Configuration classes into your
              @Import annotated @Configuration class. It makes complete sense if the @Configuration
              classes included in the @Import group is not included in the component-scan. It makes less
              sense though if those classes are included in the component-scan. I wonder if some sort
              of configuration order is created where the @Beans in the @Configuration classes that are 
              in the @Import group are initialized first, so they are ready for the class containing
              the @Import statement. (I'm not clear on how @Beans in @Configuration classes are initialized
              as it's typical to have dependencies between @Beans, and therefore Spring must somehow
              identify that it needs to initialize @Bean A before @Bean B, if @Bean B relies on @Bean A)
 2. Dispatcher Servlet - So this is the famous Servlet which routes requests to the appropriate handle
                         method. This is a .war file and while the SpringBootServletInitializer is required
                         to be extended by the main() method containing class, you are not required to
                         initialize the DispatcherServlet on account of it being a .war. You do have the
                         option to configure the DispatcherServlet (DS) however, but doing so will disable 
                         the auto-configuration for the DS. The steps to take to achieve this configuration
                         of the DS is to declare a DS @Bean, and then use a ServletRegistrationBean to
                         add the configurations you want. Note that the dispatcherServletRegistration()
                         method maps all requests to /api, so /api/*, to the DispatcherServlet (an obvious
                         question here is can I map other base URIs to the same DS @Bean, or is another 
                         required?), adds the configurations I'm looking for (I actually don't know what
                         the three configuration values used are for), and also sets a startup priority
                         for the DS registered with the ServletRegistrationBean (again what if there were
                         more than one servlets registered, would they all be started up at the same time,
                         or would I create another ServletRegistrationBean(s) for the other Servlet(s),
                         and include their load priority as 2 or 3...?
 3. SpringBootServletInitializer - As taken directly from the Spring docs:
 			  (docs.spring.io/spring-boot/docs/current/reference/html/hotwto-traditional-deployment.html) so 
 			  section 92...
 			  
              - "The first step in producing a deployable war file is to provide a SpringBootServletInitializer
                subclass and override its configure() method. Doing so makes use of Spring Framework's Servlet
                3.0 support and lets you configure your application when it is launched by the servlet container.
                Typically, you should update your application's main class to extend SpringBootServletInitializer
    			   as" it done in the UmApp.java class. 
    			  - The next step is to update your build configuration such that your project produces a war file rather
    			    than a jar file: <Packaging>war</packaging> or for gradle: apply plugin: 'war'
    			  - The final step in the process of creating a deployable war file is to ensure that the embedded servlet
    			    container does not interfere with the servlet container to which the war file is deployed. To do so,
    			    you need to mark the embedded servlet container dependency as 'being provied':
    			    <dependencies>
    			    ...
    			      <dependency>
    			        <groupId>org.springframework.boot</groupId>
    			        <artifactId>spring-boot-starter-tomcat</artifactId>
    			        <scope>provided</scope>
    			      </dependency>
    			    ...
    			    </dependencies>
    			  for gradle, you'd do:
    			    dependencies {
    			    ...
    			      providedRuntime 'org.springframework.boot:spring-boot-starter-tomcat
			    ...
			    }
			    
 			 - The war pom of the application as it stands, does not include this tomcat scope provided
 			   dependency. I wonder if I would have to include this dependency if I were to deploy the
 			   application to a server/servlet-container that's not the default embedded Tomcat one.
 			 - The @Overriden SpringBootServletInitializer.configure() method contains something else 
 			   that's new; the creation of a SpringApplicationBuilder object. 
 			 - I'm trying to understand what's going on here. So the application kicks off in main();
 			   from there a SpringApplicationBuilder object is created, taking in as an arg the UmApp
 			   class with the main() and the @Import @Configuration group. The ApplicationContext is
 			   replaced by the ConfigurableApplicationContext which is the same as the ApplicationContext
 			   but you can manage its lifecycle (initialization and destruction).  This takes place in
 			   the MyApplicationContextInitializer class. So the SpringApplicationBuilder.initializers()
 			   method takes an ApplicationContextInitializer which is the MyApplicationContextInitializer
 			   class. That class just sets the active profiles to those listed by spring.profiles.active;
 			   it's interesting how this is done differently than how spring.profiles.active is used in
 			   the change-rgl code (I need to understand the difference). Then SpringApplicationBuilder.
 			   listeners() is called with no listeners provided. I'm reading the 23.5 Application Events
 			   and Listeners section in the spring doc listed above, and it talks about how the Spring
 			   Application sends application events in addition to the Spring Framework events. Those 
 			   additional events can be listened to by Listeners registered in the listeners() method.
 			   "Some events are triggered before the ApplicationContext is created, so you cannot register
 			   a listener on those as a @Bean. You can register them with the SpringApplication.addListeners()
 			   or SpringApplicationBuilder.listeners() methods."
 			   The type of events send by the Spring Applications are: ApplicationStartingEvent,
 			   ApplicationEnvironmentPreparedEvent, ApplicationPreparedEvent, ApplicationStartedEvent,
 			   ApplicationReadyEvent, and ApplicationFailedEvent.
 			   I'm not sure why the .listeners() method is called with no params tho, and don't quite
 			   understand the Javadoc explanation.
 
 Other @Configuration classes...
 
 -- UmContextConfig.java --
 - This class appears unnecessary. It's purpose is to set up a bean of @PropertySourcesHolderConfigurer.
   The purpose of this class (which is no longer needed after Spring 4.3RC2) is to initialize that 
   @PropertySourceHolderConfigurer @Bean in order to access properties in .properties files. Since 
   Spring 4.3.RC2, this appears to be built in, so you only have to declare @PropertySource; if you're
   only using the application.properties file and not application-<env>.properties then you can ignore  
   this annotation as it's no longer needed.
   
 -- UmPersistenceJpaConfig.java --
 - @EnableTransactionManagement is an annotation I don't know too much about except what the name implies.
   Haven't worked with Transactions really, especially Jpa Transactions which I assume this is tied to
   due to the @EnableJpaRepositories annotation below.
 - @EnableJpaRepositories is an annotation that is only needed when the classes implementing JpaRepository
   are not under the main package of the @SpringBootApplication class (and its @EnableAutoConfiguration 
   annotation). If the JpaRepositories were under that same package, then they would be instantiated
   along with the rest of the @Beans in the Application Context.
 - I assume that the JpaTransactionManager @Bean that is the main @Bean created in this @Configuration class
   is associated with the @EnableTransactionManagement annotation somehow but I don't really know. The 
   DataSource @Bean is also created here and used by the JpaTransactionManager.
 
 -- UmWebConfig.java --
 - This class sets a couple of properties on the Jackson Message Converter. It appears that the Jackson
   Message Converter is part of the start up of the application somehow. It's not clear at all that this
   is the case but it's implied. I assume Jackson is part of the spring-boot-starter-web dependency. I
   don't understand how some beans are initialized and how the whole application startup process works,
   since this class is annotated with @EnableWebMvc and that must somehow work with the WebMvcConfigurerAdapter.
   From the @EnableWebMvc Javadoc: 
   - Adding this annotation to an @Configuration class imports the Spring MVC configuration from 
     WebMvcConfigurationSupport. 
   - To customize the imported configuration, implement the interface WebMvcConfigurer or more likely extend 
     the empty method base class WebMvcConfigurerAdapter and override individual methods.
   So I'm guessing that the Spring MVC Configuration import is related to the arguments of the implemented
   method extendMessageConverters() which takes a List<HttpMessageConverter<?>> converters. Somehow this
   method is linked and called thereby getting passed in this List of HttpMessageConverters.
   
 -- UmServiceConfig.java --
 - This class is basically empty at this point.
 
 - I started the Spring Application at this point after copying over the UmPersistenceJpaConfig and the
   UmWebConfig classes along with the MyApplicationContextInitializer too. Also copied over application.properties
   and the persistence-<env>.properties files. The application started up just fine but I couldn't hit any
   endpoint at localhost:8082/api which makes sense since I haven't defined any endpoints at that base url.
   
 
 		 
 			 
 			 
 			 
 			 
 			 
 			 
 			 
 			 
 			 
 			 
 			 
 			 
 			 
 			 
 			 
 			 
 			 
 			 
 			 
 			 
 			 
 			 
 			 
 			 
 			 
 			 
 			 
 			 
 			   
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 