Seam Booking Example (Java EE 5)
================================

This example demonstrates the use of Seam 3 in a Java EE 6 environment (or a
Java EE 5 environment enhanced with JSR-299 [Web Beans], JSF 2.0 and Bean
Validation). Contextual state management and dependency injection are handled
by JSR-299. Transaction and persistence context management is handled by the
EJB 3 container. Validation of input fields is handled by Bean Validation.

= Prerequisites

Please consult the Web Beans reference documentation for instructions on how to
deploy the Web Bean implementation to JBoss AS 5. To upgrade the JSF libraries
to JSF 2.0, go to the Seam jsf-upgrade-tool module and follow the instructions
contained there in pom.xml. You also need to have Bean Validation installed in 
the container (or added to the classpath) in order for UI validation to work. 
Installing Bean Validation is as simple as putting the JARs in the library 
directory of the application server.

You'll also need the 1.0.0-SNAPSHOT of the Web Beans logger extension. You can
find it in the Web Beans extensions source tree. Run 'mvn install' in that
directory to have it installed in your local Maven 2 repository.

= First steps

This example uses a Maven 2 build. To build the EJB and WAR and package them
inside an EAR, execute the following command:

 mvn

Now you're ready to deploy.

= Pointing Maven at your JBoss AS installation

First, set the JBOSS_HOME environment variable to the location of a JBoss AS 5
installation and start the server. You can optionally set the jboss.home Maven
property in an active profile defined in the $HOME/.m2/settings.xml file.

Maven assumes JBoss AS is running on port 8080. (This can be changed in the
plugin configuration). Once that's setup, you can deploy the application to
JBoss AS using Maven.

= Deploying an packaged or exploded archive to JBoss AS (the wicked fast way)

Since this build is based on Maven 2, you're probably thinking you'll need to
take a stroll to the kitchen for coffee while you wait for the build to run.
Well, you better have your coffee ready because this build is fast...and easy!

The secret? The Maven 2 CLI plugin: http://github.com/mrdon/maven-cli-plugin.
This plugin gives you a command console to execute maven goals and plugins.
Even better, it supports aliases, so no more having to type long Maven commands
with switches and properties. Here's how to get started.

First, rev up the command console:

 mvn cli:execute-phase

You'll be presented with a prompt:

maven2> 

At the prompt, you can type any of the following commands (a description is
provide for each command, but don't type that of course):

explode   -> deploy as an exploded archive to JBoss AS)
deploy    -> deploy as a packaged archive to JBoss AS
undeploy  -> undeploy the exploded or packaged archive from JBoss AS
restart   -> restart the exploded or packaged archive deployed to JBoss AS
reexplode -> undeploy then explode

You can prefix any command with "clean" if you want to do a clean build:

maven2> clean explode

Leave the console running and enjoy extremely fast deployments ;)

But fast isn't for all of us.

= Deploying a packaged archive to JBoss AS (the Slowskys' way)

Why work fast when you can work slow? Perhaps your boss is annoying you and you
want to irritate him with long builds. Or maybe you just fear speed. If that's
the case, this section is for you (or if you just want to know what all those
commands are doing above).

Putting JBOSS_HOME aside for the moment, you can deploy the application to
JBoss AS via JMX by executing this command:

 mvn -o -f seam-booking-ear/pom.xml jboss:deploy

You can undeploy the application via JMX using this command:

 mvn -o -f seam-booking-ear/pom.xml jboss:undeploy

Here's the chained restart command via JMX:

 mvn -o -f seam-booking-ear/pom.xml jboss:undeploy && mvn -o package && mvn -o -f seam-booking-ear/pom.xml jboss:deploy

If you would rather deploy more traditional way by copying the archive directly
to the deploy directory of the JBoss AS domain, use this command instead:

 mvn -o -f seam-booking-ear/pom.xml jboss:harddeploy

But likely during development, you're only interested in exploded archives.

= Deploying a packaged archive to JBoss AS (the Slowskys' way)

It's much better to use the antrun plugin over the jboss plugin since it's
smarter about what it copies to the server. The antrun plugin is bound to the
end of the package goal when the explode profile is active:

 mvn -o package -Pexplode

This profile executes an series of Ant tasks that copy the exploded WAR,
EJB-JAR, and EAR to the JBoss AS deploy directory.

You can force a restart of the application by activating the restart profile,
which executes an Ant task bound to the validate phase:

 mvn -o validate -Prestart

You can remove the archive by activating the undeploy profile, which also
executes an Ant task bound to the validate phase:

 mvn -o validate -Pundeploy

Finally, you can undeploy and explode all in one command using both the
undeploy and explode profiles with a standard package build:

 mvn -o package -Pundeploy,explode

Note that the -o puts Maven in offline mode so that it doesn't perform time
consuming update checks. The -o flag is included in the aliases configured for
the Maven CLI plugin documented above.

= Example highlights

- 3 module Maven 2 reactor project (ejb-jar, war, ear)
- establishes a standard for Seam 3 examples
- repeat elements are kept to a minimum in the Maven POM files (DRY)
- JBoss datasource is deployed with the EAR to simplify packaging
  (-ds.xml doesn't have to be deployed to JBoss AS separately)
- supports both a packaged and exploded deployment to JBoss AS

= Known issues

(1) Clicking on logout throws an exception
    javax.context.ContextNotActiveException: No active contexts for scope type javax.context.RequestScoped
       at org.jboss.webbeans.ManagerImpl.getContext(ManagerImpl.java:739)
       at org.jboss.webbeans.bean.proxy.ClientProxyMethodHandler.getProxiedInstance(ClientProxyMethodHandler.java:116)
       at org.jboss.webbeans.bean.proxy.ClientProxyMethodHandler.invoke(ClientProxyMethodHandler.java:96)
       at org.jboss.webbeans.conversation.ConversationImpl_$$_javassist_213.isLongRunning(ConversationImpl_$$_javassist_213.java)

(2) Ajax is not working on blur in p:edit form fields (had to disable)
    error malformedXML

(3) No list of workspaces

(4) Cannot use <f:view> in template or else it will remove the conversation id token from the view root

(5) @AfterTransactionSuccess observer does not work

(6) Can't read properties from Hotel object in history list. Get this exception:
    javax.inject.IllegalProductException: Cannot return null from a non-dependent producer method
       at org.jboss.webbeans.bean.AbstractProducerBean.checkReturnValue(AbstractProducerBean.java:209)
       at org.jboss.webbeans.bean.AbstractProducerBean.create(AbstractProducerBean.java:349)
       at org.jboss.webbeans.context.AbstractMapContext.get(AbstractMapContext.java:98)
       at org.jboss.webbeans.bean.proxy.ClientProxyMethodHandler.getProxiedInstance(ClientProxyMethodHandler.java:117)
       at org.jboss.webbeans.bean.proxy.ClientProxyMethodHandler.invoke(ClientProxyMethodHandler.java:96)
       at org.jboss.seam.examples.booking.model.Hotel_$$_javassist_227.getAddress(Hotel_$$_javassist_227.java)

= Open questions

- How do I clear a contextual bean from a scope, in particular the session scope? I've had to do workarounds.

- How do I inject an Event object into a stateful component? I get an error that there is a reference to a
  non-serializable object from a bean declaring a non-passivating scope. I have to use the Manager to fire an event
  instead.

- Should the parent project be adjacent to ear/ejb-jar/war?

= TODO

- secure pages (likely will use <f:event type="beforeRenderView"/>

- get status messages from default or resource bundle (not just hardcoded defaults)

- demonstrate use of exception handling in JSF 2

- auto-detect which files have @NotNull or @NotEmpty to determine whether to put the * in <p:edit>

- use Cargo plugin to support deployment to other Java EE servers (GlassFish)

- refactor the password/confirm password into a reusable component (needed on
  registration and change password)

- consider using unpackTypes in the ear plugin (which works incrementally) to replace portions of the antrun logic

- use a resource to define persistence context
<EntityManager>
   <PersistenceContext>
      <unitName>booking</unitName>
   </PersistenceContext>
   <booking:BookingDatabase/>
</EntityManager>
