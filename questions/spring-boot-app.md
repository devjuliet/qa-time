# How does SpringApplication.run() method work?
Okay, we used it to run our application. But, how does it work internally?
```java
@SpringBootApplication
public class SpringBootDemo {
   public static void main(String args[]) {
      SpringApplication.run(SpringBootDemo.class, args);
   }
}
```
Basically, it main purpose is to create a new context for our application. But let's go a little deeper with the implementation. 

```java
public ConfigurableApplicationContext run(String... args) {
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		DefaultBootstrapContext bootstrapContext = createBootstrapContext();
		ConfigurableApplicationContext context = null;
		configureHeadlessProperty();
		SpringApplicationRunListeners listeners = getRunListeners(args);
		listeners.starting(bootstrapContext, this.mainApplicationClass);
		try {
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
			ConfigurableEnvironment environment = prepareEnvironment(listeners, bootstrapContext, applicationArguments);
			configureIgnoreBeanInfo(environment);
			Banner printedBanner = printBanner(environment);
			context = createApplicationContext();
			context.setApplicationStartup(this.applicationStartup);
			prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
			refreshContext(context);
			afterRefresh(context, applicationArguments);
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
			}
			listeners.started(context);
			callRunners(context, applicationArguments);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, listeners);
			throw new IllegalStateException(ex);
		}

		try {
			listeners.running(context);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, null);
			throw new IllegalStateException(ex);
		}
		return context;
	}
```

The first two lines basically allows for timing of a number of tasks, exposing total running time and running time for each named task.

```java
StopWatch stopWatch = new StopWatch();
stopWatch.start();
```

Next, it provides facilities to configure an application context in addition to the application context client methods in the ApplicationContext interface.
```java
ConfigurableApplicationContext context = null;
```


The most important part comes next, since we are going to create the context.
Following the next steps:
1. Application Context is started.
2. Using application context autodiscovery occurs: @ComponentScan
3. All default configurations are set up based on dependencies.
4. An embedded servlet container is started. 

 ```java
context = createApplicationContext();
context.setApplicationStartup(this.applicationStartup);
prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);
refreshContext(context);
afterRefresh(context, applicationArguments);
stopWatch.stop();
listeners.started(context);
callRunners(context, applicationArguments);
 ```
We create the ApplicationContext. By default this method will respect any explicitly set application context class. Mainly, creates the Spring container.
 ```java
context = createApplicationContext();
context.setApplicationStartup(this.applicationStartup);
 ```
 It preprocesses the Spring container.
```java
prepareContext(bootstrapContext, context, environment, listeners, applicationArguments, printedBanner);

```
Refresh the container and post processes the Spring container.

```java
refreshContext(context);
afterRefresh(context, applicationArguments);
```

Ends the timer and print. This is the display time of the console after we start.
```java
stopWatch.stop(); 
``` 

Finally:

1. Issue start end event
2. It executes the run method to implement `ApplicationRunner` and `CommandLineRunner`
3. Publish application started event
```java
listeners.started(context);
callRunners(context, applicationArguments);
```

Finally, we return the contex to the Application.
```java
return context
```