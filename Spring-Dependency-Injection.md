## The Natural Programmer
###### https://www.naturalprogrammer.com/

## Dependency Injection

*We created a instance of MockMailsender().  What if we wanted to use SmptMailSender instead?  We would have to change the
code of the controller class (MailController).  What if we have a development environment that uses Mock and Prod environment that
will use Smtp? We cand address this issue using DI, or Dependency Injection.*

* Interface MailSender

        public interface MailSender {
            void send(String to, String subject, String body);
        }

* Implementation of MockMailSender

        import org.apache.commons.logging.Log;
	    import org.apache.commons.logging.LogFactory;
	    public class MockMailSender implements MailSender {
		    private static Log log = LogFactory.getLog(MockMailSender.class);

    		@Override
    		public void send(String to, String subject, String body) {
    			Log.info("Sending MOCK mail to " + to);
    			Log.info("with Subject " + subject);
    			Log.info("and body " + body);
    		}
	    }

* Implementation of StpMailSender

	    import org.apache.commons.logging.Log;
	    import org.apache.commons.logging.LogFactory;
	    public class SmtpMailSender implements MailSender {

		    private static Log log = LogFactory.getLog(SmtpMailSender.class);

		    @Override
		    public void send(String to, String subject, String body) {
			    Log.info("Sending Smtp mail to " + to);
			    Log.info("with Subject " + subject);
			    Log.info("and body " + body);
		    }
	    }

* MailController
 
        import com.naturalprogrammer.spring5tutorial.mailMailSender;
	    import com.naturalprogrammer.spring5tutorial.MockMailSender;
	    @RestController
	    public class MailController {
		    // Creating an instance variable
		    private MailSender mailSender = new MockMailsender();
		
		    @RequestMapping("/mail")
		    public String mail() {
			    mailSender.send("mail@example.com", "A test mail", "Body of the test mail");
			    return "Mail sent!";
		    }
	    }

##### Solving using DI.
*Added @Component on the MockMailSender Implementation.  Then @Autowired the mailSender. The @Component tells Spring to create an instance of the MockMailSender (these are also called Beans) and store it in the Application Context.  Since there is also a @RequestController, 
an instance of MailController will also be created by Spring and will see that there is an @Autowired within and search in the Application Context for the MailSender and assign it to the mailSender variable.*

* MockMailSender	

        import org.apache.commons.logging.Log;
	    import org.apache.commons.logging.LogFactory;
	    @Component
	    public class MockMailSender implements MailSender {

		    private static Log log = LogFactory.getLog(MockMailSender.class);

		    @Override
		    public void send(String to, String subject, String body) {
			    Log.info("Sending MOCK mail to " + to);
			    Log.info("with Subject " + subject);
			    Log.info("and body " + body);
		    }
	    }

* MailController

    	import com.naturalprogrammer.spring5tutorial.mailMailSender;
    	import com.naturalprogrammer.spring5tutorial.MockMailSender;
    	@RestController
    	public class MailController {
    		@Autowired
    		private MailSender mailSender;
    		// Removed -> private MailSender mailSender = new MockMailsender();
    		
    		@RequestMapping("/mail")
    		public String mail() {
    			mailSender.send("mail@example.com", "A test mail", "Body of the test mail");
    			return "Mail sent!";
    		}
    	}

*@Autowired can be used not only on instance variables `private MailSender mailSender`, but any instance methods as well.
@Autowired can be used on constructors as well (where the method name is same as class name), but in this case, @Autowired is optional,as Spring will _Automatically_ inject it as well as all the parameters.*

Example on a constructor:

    	@Autowired // Optional in this case, mailSender will be auto-injected
    		public void setMailSender(MailSender mailSender) {
    		this.mailSender = mailSender;
    	}

###### Solving the Multiple bean probelm | What if there are TWO beans of type MailSender?
*In our example we have TWO MailSenders.  One is MockMailSender, and the other is SmtpMailSender. So when Spring tries to inject `(MailSender mailSender)` it will find two in the Application Context*

* By default each bean will have a name applied automatically, which will be the name of the class in camel case, ex: SmtpMailSender will be named smtpMailSender (small first letter, camelCased).

To remedy:

	public MailController(MailSender smtpMailSender) {
	  this.mailSender = smtpMailSender;
	}
	
	-= OR =-

	public MailController(MailSender mockMailSender) {
	  this.mailSender = smtpMailSender;
	}
	
**To add custom name, just use @Component("smtp").  This Bean will now how a name of 'smtp'**

	public MailController(MailSender smtp) {
		this.mailSender = smtp;
	}

*Another way to solve the multiple Bean problem is using the @Primary Notation.  This notation if put on for example the
MockMailSender class below @Component notation will tell Spring to preference this as the default Bean*

Example:

    	import org.apache.commons.logging.Log;
	    import org.apache.commons.logging.LogFactory;
	    @Component
	    @Primary
	    public class MockMailSender implements MailSender {

		    private static Log log = LogFactory.getLog(MockMailSender.class);

		    @Override
		    public void send(String to, String subject, String body) {
			    Log.info("Sending MOCK mail to " + to);
			    Log.info("with Subject " + subject);
			    Log.info("and body " + body);
		    }
	    }

*As well as a name, each bean will also have a 'Qualifier' associated with it automatically*

Example:

	@Component
	@Qualifier("smtpMail")
	public class SmtpMailSender implements MailSender {
		private MailSender mailSender;
		public MailController(@Qualifier("smtpMail") Mailsender smtp) {
			this.mailSender = smtp;
		}
	}


#### Summary
1. We can use @Autowired notations directly on the instance variables, or on setter methods, or on contructors!  
Which should we preffer? -> General recommendation is we should preffer Contructor injections!!!!!
2. Simlar to @Autowired, we also have more annotations such as @Resource and @Inject which is similar to @Autowired, 
but @Autowired has become the standard.
