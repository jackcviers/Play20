<!--- Copyright (C) 2009-2016 Lightbend Inc. <https://www.lightbend.com> -->
# Integrating with JPA

## Adding dependencies to your project

First you need to tell play that your project need javaJpa plugin which provide JDBC and JPA api dependencies.

There is no built-in JPA implementation in Play; you can choose any available implementation. For example, to use Hibernate, just add the dependency to your project:

```
libraryDependencies ++= Seq(
  javaJpa,
  "org.hibernate" % "hibernate-entitymanager" % "4.3.9.Final" // replace by your jpa implementation
)
```

## Exposing the datasource through JNDI

JPA requires the datasource to be accessible via JNDI. You can expose any Play-managed datasource via JNDI by adding this configuration in `conf/application.conf`:

```
db.default.driver=org.h2.Driver
db.default.url="jdbc:h2:mem:play"
db.default.jndiName=DefaultDS
```

## Creating a persistence unit

Next you have to create a proper `persistence.xml` JPA configuration file. Put it into the `conf/META-INF` directory, so it will be properly added to your classpath.

Here is a sample configuration file to use with Hibernate:

```
<persistence xmlns="http://xmlns.jcp.org/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_1.xsd"
             version="2.1">

    <persistence-unit name="defaultPersistenceUnit" transaction-type="RESOURCE_LOCAL">
        <provider>org.hibernate.jpa.HibernatePersistenceProvider</provider>
        <non-jta-data-source>DefaultDS</non-jta-data-source>
        <properties>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
        </properties>
    </persistence-unit>

</persistence>
```

Finally you have to tell Play, which persistent unit should be used by your JPA provider. This is done by the `jpa.default` property in your `application.conf`.

```
jpa.default=defaultPersistenceUnit
```

## Deploying Play with JPA

Running Play in development mode while using JPA will work fine, but in order to deploy the application you will need to add this to your `build.sbt` file.

@[jpa-externalize-resources](code/jpa.sbt)

Since Play 2.4 the contents of the `conf` directory are added to the classpath by default. This option will disable that behavior and allow a JPA application to be deployed. Note that the content of conf directory will still be available in the classpath due to it being included in the applications jar file.


## Annotating JPA actions with `@Transactional`

Every JPA call must be done in a transaction so, to enable JPA for a particular action, annotate it with `@play.db.jpa.Transactional`. This will compose your action method with a JPA `Action` that manages the transaction for you:

```
@Transactional
public static Result index() {
  ...
}
```

If your action runs only queries, you can set the `readOnly` attribute to `true`:

```
@Transactional(readOnly=true)
public static Result index() {
  ...
}
```

## Using the `play.db.jpa.JPA` helper

At any time you can retrieve the current entity manager from the `play.db.jpa.JPA` helper class:

```
public static Company findById(Long id) {
  return JPA.em().find(Company.class, id);
}
```

## Enabling Play database evolutions

Read [[Evolutions]] to find out what Play database evolutions are useful for, and follow the setup instructions for using it.
