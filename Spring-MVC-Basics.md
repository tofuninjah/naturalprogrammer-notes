## Spring MVC Basics 

### Using Views

##### Request Handlers
Responsible for handling the request coming from the Web Client

1. Request -> Spring Dispatcher Servlet -> picks a Handler and use it to send the response.

##### HomeController

    @Controller
    public class HomeController {
        @RequestMapping("/")
        public String hello() {
            return "home"; // Logical View Name
        }
    }

*Spring will use what is called 'ViewResolver' which looks at the return value and try to resolve the view.  Spring will know how to add the suffix and prefix by defining it in the application.properties*

    spring.mvc.view.prefix: /WEB-INF/jsp/
    spring.mvc.view.suffix: .jsp

##### @ResponseBody Annotation

	@Controller
	public class HomeController {
	    @RequestMapping("/")
	    @ResponseBody
	    public String home() {
	        return "home";
	    }
	}

*Whatever is returned ("home"), is not the Logical View Name, but it will be directly written to the 'Response'. It will be the same as having @RestController, instead of @Controller at the top of the HomeController Class. @ResponseBody can be used inside a @Controller class (Instead of just using @RestController) if you need to return plain text, or json, instead of a view.*

##### View Controllers

ViewController Component should be added with ViewConrollerRegistry. Package -> Config -> new Class -> MvcConfig

    @Configuration
    public class MvcConfig implements WebMvcConfigurer {
        @Override
        public void addViewControllers(ViewControllerRegistry registry) {
            registry.addViewController("/").setViewName("home");
        }
    }

With the above implementation,, you are now able to remove a HomeController that maps to the home view.

##### Static Content (Js, css, images, etc)

*In Summary, all static content should go in either /static, /public,  /resources, or /META-INF/resources. Can also seperate out Header and Footer as includes:*

    <%@include file ="includes/header.jsp"%>
    <%@include file ="includes/footer.jsp"%>

### Receiving and Displaying Data

    @Controller
    public class HelloController {
        @RequestMapping("/")
        public String hello(Model model) {
        	model.addAttribute("name", "Sanjay");
        
            return "home"; // Logical View Name
        }
    }

Using Model Object, Spring will provide us with a reference to a Model Object that we can use to add Attributes that will then be passed to the view!

	Hello, ${name}!

**Another way we can do it is with ModelAndView!**

	@RequestMapping("/hello")
	public ModelAndView hello() {
	    ModelAndView mav = new ModelAndView("hello"); // View Name
	    mav.addObject("name", "Sanjay");
	    return mav;
	}

*Where are the Attributes of the Model Object stored?  It's initially  stored in a Java Map.  And after the Controller method finishes excecuting, but before the View takes over, the Model Attributes gets copied over the the HTTP Request Object or Session Object if you tell it to do so.*

https://www.intertech.com/Blog/understanding-spring-mvc-model-and-session-attributes/

##### Internationalization

    messages-fr.properties
    messages.properties (Default)

Inside messages.properties:

	hello: Hi

Include the spring tag lib

	...
	<%@ taglib prefix="spring" uri="http://www.springframework.org/tags" %>
	<spring:message code="hello" />, ${name}!
	...

How does Spring search for a specific message?  It starts with the most specific properties file. For example if it is going to search for 'hello' in the english local, it will search:

1. messages_en_US.properties
	2. messages_en.properties
		3. messages.properties
	

How would you use this in a controller? -> MessageSource
​	
​    org.springframework.context.MessageSource;
​    org.springframework.context.i18n.LocaleContextHolder;
​    
​    @Controller
​    public class HelloController {
​    
​        @Autowired
​        private MessageSource messageSource;
​    
​        @RequestMapping("/hello")
​        public String hello(Model model) {
​        	model.addAttribute("name", messageSource.getMessage("name", null, LocaleContextHolder.getLocale()));
​    
​        	return "home"; // Logical View Name
​        }
​    }

The null in the getMessage method is a placeholder. You can use it like below:

	text: Visit {0} if you are below 18, else visit {1}!
	
	Object[] urls = {
	    "http://below18.example.com",
	    "http://above18.example.com"
	}
	
	messageSource.getMessage("text", urls, LocaleContextHolder.getLocale());
	
	Output: Hi, Visit http://below18.example.com if you are below 18, else visit http://above18.example.com!

###### You can also create a utility class

	@Component
	public class MyUtils {
		
		private static MessageSource messageSource;
		
		public MyUtils(MessageSource messageSource) {
	        MyUtils.messageSource = messageSource;
		}
	    
	    public static String getMessage(String messageKey, Object... args) {
	        return messageSource.getMessage(messageKey, args, LocaleContextHolder.getLocale());
	    }
	}
	
	@Controller
	public class HelloController {
	
	    @RequestMapping("/hello")
	    public tring hello(Model model){
	        model.addAttribute("name", MyUtils.getMessage("text", 	"http://below18.example.com",
	            "http://above18.example.com"), urls, LocaleContextHolder.getLocale());
	    }
	}

##### Retreiving Request Data

Request Parameters, Path variables, Headers, Cookies

Example: http://localhost:8080/hello/5?name=Sanjay

	@RequestMapping("/hello/{id}")
	public String helloId(Model model, @PathVariable("id") int id,
	@RequestParam("name") String name) {
	
		model.addAttribute("id", id);
		model.addAttribute("name", name);
	
	    return "hello-view"; // in view -> ${name}, ${id}
	}

*In Java 8, the name parameter is not needed if it is compiled with the -parameters flag: @PathVariable int id, @RequestParam String name, and it still should work*

	<plugins>
	    <plugin> 
	        <groupId>org.apache.maven.plugins</groupId>
	        <artifactId>maven-compiler-plugin</artifactId>
	        <configuration>
	            <compilerArgs>
	                <arg>-parameters</arg>
	            </compilerArgs>
	        </configuration>
	    </plugin>
	</plugins>

* Marking request parameters as optional

@RequestParam("name", required=false) String name 
​	- Mark Request paramater as optional


##### Rendering and Receiving Forms

	@Controller
	public class SignupController {
		@RequestMapping(value="/signup", 
			method=ReqeustMethod.POST)
	   		public String doSignup(@RequestParam String email,
	    @RequestParam String name, @RequestParam String password) {
			return "hello";
		}
	}

##### Pojo Value Object

UserCommandVO.class

	public class UserCommandVO {
	    private String email;
	    private String name;
	    private String password;
	    
	    public String getEmail() {
	        return email;
	    }
	    public String getName() {
	        return name;
	    }
	    public String getPassword() {
	        return password;
	    }
	    public String setEmail(String email){
	        this.email = email;
	    }
	    public String setName(String name){
	        this.name = name;
	    }
	    public String setPassword(String password){
	        this.password = password;
	    }
	}
	
	/**
	 * Now we can use it as a Parameter!!! 
	 */
	
	@Controller
	@RequestMapping("/signup")
	public class SignupController {
	    private static Log log = LogFactory.getLog(SignupController.class);
	    
	    @GetMapping
	    public String signup() {
	        return "signup";
	    }
	    
	    @PostMapping String doSignup(UserCommandVO user) {
	        log.info("Email: " + user.getEmail() + ";  Name: " + user.getName() + "; Password: " + user.getPassword());
	    
	    	return "redirect:/";
	        
	    }
	}

**When Spring sees the UserCommandVO user Object, it will try to fill that in with the Request parameters!**

##### Form Validation

Using the Bean Validation API - and Annotations

UserCommandVO.class

	public class UserCommandVO {
		@NotBlank
		@Email
		@Size(min=4, max=250)
	    private String email;
	    
	    @NotBlank
	    @Size(min=1, max=100)
	    private String name;
	    
	    @NotBlank
	    @Size(min=6, max=32)
	    private String password;
	    
	    ... // getter/setters
	}

Annotate the Controller class with @Validated, _then_ Bind the results with BindingResult result.  If there are any errors, Spring will put those Errors into the results variable.
​	
​	@PostMapping String doSignup(@Validated UserCommandVO user, 
​	BindingResult result) {
​		
		// If there are errors
		if(result.hasErrors()){
	    	return "signup";    
		}
		
	    log.info("Email: " + user.getEmail() + ";  Name: " + user.getName() + "; Password: " + user.getPassword());
	    
	    	return "redirect:/";
	        
	}

**Spring Automatically adds to the Model Object any Pojo Object passed into Controller, so that we can show the old data!  It also adds any erros in the results object!!  So only thing to do is add code in JSP View to display the errors** 

`<%@ taglib uri="http://www.springframework.org/tags/form" prefix="form" %>`

    <form:form modelAttribute="userCommandVO">
    	
    	<form:errors cssClass="error" />
    	
    	<div class="form-group">
            <form:label path="email"></form:label>
            <form:input path="email" type="email"/>
            <form:errors path="email" cssClass="error" />
    	</div>
    	<div class="form-group">
    		<form:label path="name"></form:label>
            <form:input path="name" type="name"/>
            <form:errors path="name" cssClass="error" />
    	</div>
    </form:form>
    
##### Custom Error Messages



    
    
    
    
    
    
    
    
    
    
    
    