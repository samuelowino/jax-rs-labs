# jax-rs-labs
JAX-RS: Java API for RESTful Web Services is a Java programming language API spec that provides support in creating web services according to the Representational State Transfer architectural pattern.

The REST paradigm has been around for quite a few years now and it's still getting a lot of attention.

A RESTful API can be implemented in Java in a number of ways: you can use Spring, JAX-RS, or you might just write your own bare servlets if you're good and brave enough. All you need is the ability to expose HTTP methods – the rest is all about how you organize them and how you guide the client when making calls to your API.


As you can make out from the title, this article will cover JAX-RS. But what does “just an API” mean? It means that the focus here is on clarifying the confusion between JAX-RS and its implementations and on offering an example of what a proper JAX-RS webapp looks like.

2. Inclusion in Java EE
JAX-RS is nothing more than a specification, a set of interfaces and annotations offered by Java EE. And then, of course, we have the implementations; some of the more well known are RESTEasy and Jersey.

Also, if you ever decide to build a JEE-compliant application server, the guys from Oracle will tell you that, among many other things, your server should provide a JAX-RS implementation for the deployed apps to use. That's why it's called Java Enterprise Edition Platform.

Another good example of specification and implementation is JPA and Hibernate.

2.1. Lightweight Wars
So how does all this help us, the developers? The help is in that our deployables can and should be very thin, letting the application server provide the needed libraries. This applies when developing a RESTful API as well: the final artifact should not contain any information about the used JAX-RS implementation.

Sure, we can provide the implementation (here‘s a tutorial for RESTeasy). But then we cannot call our application “Java EE app” anymore. If tomorrow someone comes and says “Ok, time to switch to Glassfish or Payara, JBoss became too expensive!“, we might be able to do it, but it won't be an easy job.

If we provide our own implementation we have to make sure the server knows to exclude its own – this usually happens by having a proprietary XML file inside the deployable. Needless to say, such a file should contain all sorts of tags and instructions that nobody knows nothing about, except the developers who left the company three years ago.

2.2. Always Know Your Server
We said so far that we should take advantage of the platform that we're offered.

Before deciding on a server to use, we should see what JAX-RS implementation (name, vendor, version and known bugs) it provides, at least for Production environments. For instance, Glassfish comes with Jersey, while Wildfly or Jboss come with RESTEasy.

This, of course, means a little time spent on research, but it's supposed to be done only once, at the beginning of the project or when migrating it to another server.

3. An Example
If you want to start playing with JAX-RS, the shortest path is: have a Maven webapp project with the following dependency in the pom.xml:

1
2
3
4
5
6
<dependency>
    <groupId>javax</groupId>
    <artifactId>javaee-api</artifactId>
    <version>7.0</version>
    <scope>provided</scope>
</dependency>
We're using JavaEE 7 since there are already plenty of application servers implementing it. That API jar contains the annotations that you need to use, located in package javax.ws.rs. Why is the scope “provided”? Because this jar doesn't need to be in the final build either – we need it at compile time and it is provided by the server for the run time.

After the dependency is added, we first have to write the entry class: an empty class which extends javax.ws.rs.core.Application and is annotated with javax.ws.rs.ApplicationPath:

@ApplicationPath("/api")
public class RestApplication extends Application {
}
We defined the entry path as being /api. Whatever other paths we declare for our resources, they will be prefixed with /api.

Next, let's see a resource:

@Path("/notifications")
public class NotificationsResource {
    @GET
    @Path("/ping")
    public Response ping() {
        return Response.ok().entity("Service online").build();
    }
 
    @GET
    @Path("/get/{id}")
    @Produces(MediaType.APPLICATION_JSON)
    public Response getNotification(@PathParam("id") int id) {
        return Response.ok()
          .entity(new Notification(id, "john", "test notification"))
          .build();
    }
 
    @POST
    @Path("/post/")
    @Consumes(MediaType.APPLICATION_JSON)
    @Produces(MediaType.APPLICATION_JSON)
    public Response postNotification(Notification notification) {
        return Response.status(201).entity(notification).build();
    }
}
We have a simple ping endpoint to call and check if our app is running, a GET and a POST for a Notification (this is just a POJO with attributes plus getters and setters).

Deploy this war on any application server implementing JEE7 and the following commands will work:

curl http://localhost:8080/simple-jaxrs-ex/api/notifications/ping/
 
curl http://localhost:8080/simple-jaxrs-ex/api/notifications/get/1
 
curl -X POST -d '{"id":23,"text":"lorem ipsum","username":"johana"}'
  http://localhost:8080/simple-jaxrs-ex/api/notifications/post/
  --header "Content-Type:application/json"
Where simple-jaxrs-ex is the context-root of the webapp.

This was tested with Glassfish 4.1.0 and Wildfly 9.0.1.Final. Please note that the last two commands won't work with Glassfish 4.1.1, because of this bug. It is apparently a known issue in this Glassfish version, regarding the serialization of JSON (if you have to use this server version, you'll have to manage JSON marshaling on your own)

4. Conclusion
At the end of this article, just keep in mind that JAX-RS is a powerful API and most (if not all) of the stuff that you need is already implemented by your web server. No need to turn your deployable into an unmanageable pile of libraries.

This write-up presents a simple example and things might get more complicated. For instance, you might want to write your own marshalers. When that's needed, look for tutorials that solve your problem with JAX-RS, not with Jersey, Resteasy or other concrete implementation. It's very likely that your problem can be solved with one or two annotations.
