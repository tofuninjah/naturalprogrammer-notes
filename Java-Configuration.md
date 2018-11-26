## Java Configuration Classes
### How to configure the Beans, and put Objects into the Application Context

**Example Annotations**
1. @Controller
2. @Service
3. @Repository
4. @Configuration
5. @SpringBootApplication

**@SpringBootApplication will tell Spring to scan the current package for Beans.  You can also tell Spring to scan different packages by using `scanBasePackageClasses`: 

    @SpringBootApplication(scanBasePackageClasses = {NpSpring5TutorialApplication.class, AnotherClass.class})
    public class NpSpring5TutorialApplication {
        public static void main(String[] args) {
            SpringApplication.run(NpSpring5TutorialAplication.class, args);
        }
    }

*Let's say we have a JAR in our path, and need to create a Bean from Mock and Smtp Mail Sender.. But with no source code access to add the @Component annotations.  How would we create a Bean in this instance?*

##### Creating a Bean with a Configuration class
*When Spring sees that this is an Annotation (Configuration) class with @Bean Annotation, it will call those methods and the return Object will be stored in the Application Context as Beans.  The name of those Beans will be the method names, `mockMailSender` of type MockMailSender, and `smtpMailSender` of type SmtpMailSender*

* MailConfig

        @Configuration
        public class MailConfig {
            @Bean
            public MailSender mockMailSender() {
                return new MockMailSender();
            }
            @Bean
            public MailSender smtpMailSender() {
                return new SmtpMailSender();
            }
        }

###### Another way is to use XML Configurations to create the Beans.

### Using Application Properties (application.properties)

* application.properties

        app.name = Dev Environment

* ExampleController
*When instanciated, Spring will inject app.name to String appName variable*
        @RestController
        public class HelloController {
            
            @Value("${app.name}");
            private String appName;
            
            @RequestMapping("/hello")
            public String hello() {
                return "Hello, world" + appName; // Should return Hello, world Dev Environment
            }
        }

### Spring profiles
*Say for instance you are developing an application for a Book Shop as well as a Grocery Shop in 3 environments: Development, Test, and Prod.  You can plan to have 5 profiles: Book, Grocery, Dev, Test, Prod*

* application.properties

    app.name = Dev Environment
    spring.profiles.active: book, dev
    
*Suppose we want to to use the smtp in prod and mock in development.*    

###### We can use @Profile to tell Spring only to use a Bean in the 'Dev' environment
https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-profiles.html
https://www.baeldung.com/spring-profiles

* MailConfig

        @Configuration
        public class MailConfig {
            @Bean
            @Profile("dev")
            public MailSender mockMailSender() {
                return new MockMailSender();
            }
            @Bean
            @Profile("!dev")
            public MailSender smtpMailSender() {
                return new SmtpMailSender();
            }
        }    

### Conditional Annotations
https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-developing-auto-configuration.html

*Say we want to configure the smtp mail sender when we find the property `spring.mail.host`, otherwise use the mock mail seder*

* application.properties

    spring.mail.host: smtp.gmail.com









