# Quickstart: Java

The following guide will get you up and running locally as quickly as possible
using the Java.

### Download starter project

```
curl -LO https://github.com/tigrisdata/tigrisdb-starter-java/archive/refs/heads/main.zip
unzip main.zip
```

# Build and generate models

```
cd tigrisdb-starter-java-main
mvn clean install
```

# Launch

Open it in your favorite IDE and launch
the `com.tigrisdata.starter.TigrisDBStarterApplication`
For example using IntelliJ IDEA

![launcher window image](/img/screenshots/launcher_window.png)

A successful launch of the application will end with log line

`[main] INFO c.t.s.spring.TigrisDBInitializer - Finished initializing TigrisDB`

and you can play around with the rest API
at [http://localhost:8080/swagger.html](http://localhost:8080/swagger.html).

# Code walk through

## Schema to Java model generation

TigrisDB provides a maven plugin and a CLI that helps generate Java models from
TigrisDB JSON schema. In this starter project maven-plugin is configured in
[`pom.xml`](https://github.com/tigrisdata/tigrisdb-starter-java/blob/main/pom.xml)

```xml

<plugin>
    <groupId>com.tigrisdata.tools.code-generator</groupId>
    <artifactId>maven-plugin</artifactId>
    <!-- we are still pre-release -->
    <version>1.0-SNAPSHOT</version>

    <executions>
        <execution>
            <goals>
                <goal>generate-sources</goal>
            </goals>
        </execution>
    </executions>
    <configuration>
        <!-- location from where to read the schema files -->
        <schemaDir>src/main/resources/tigrisdb-schema</schemaDir>
        <!-- generated Java model's package name -->
        <packageName>com.tigrisdata.starter.generated</packageName>
        <!-- output directory for Java models, they will be added to
        compilation unit automatically -->
        <outputDirectory>target/generated-sources</outputDirectory>
    </configuration>
</plugin>
```

## Client initialization

```java
// configuration
TigrtisDBConfiguration tigrisDBConfiguration = TigrisDBConfiguration.
        newBuilder(baseUrl).build();

// client
TigrisDBClient client = StandardTigrisDBClient.getInstance(
                            new TigrisAuthorizationToken(token),
                            tigrisDBConfiguration
                        );
```

In this starter project above initialization is located in [`com.tigrisdata. starter.spring.TigrisDBSpringConfiguration`](https://github.com/tigrisdata/tigrisdb-starter-java/blob/main/src/main/java/com/tigrisdata/starter/spring/TigrisDBSpringConfiguration.java)

## Retrieve database

```java
TigrisDatabase myDatabase = client.getDatabase(dbName);
```

In this starter project above initialization is located in [`com.tigrisdata. starter.spring.TigrisDBSpringConfiguration`](https://github.com/tigrisdata/tigrisdb-starter-java/blob/main/src/main/java/com/tigrisdata/starter/spring/TigrisDBSpringConfiguration.java)

## Create collections

```java
myDatabase.createCollectionsInTransaction(new File("src/main/resources/tigrisdb-schema"));
```

This will read all the schema files in this directory and register
collections per schema file. In the starter project this code is located in
[`com.tigrisdata.starter.spring.TigrisDBInitializer`](https://github.com/tigrisdata/tigrisdb-starter-java/blob/main/src/main/java/com/tigrisdata/starter/spring/TigrisDBInitializer.java)

## Retrieve collection

```java
TigrisCollection<User> userCollection = myDatabase.getCollection(User.class)
```

Note that `User` is generated model and it automatically implements `com. tigrisdata.db.client.model.TigrisCollectionType`. In this starter project this
initialization is located in [`com.tigrisdata.starter.controller.UserController`](https://github.com/tigrisdata/tigrisdb-starter-java/blob/main/src/main/java/com/tigrisdata/starter/controller/UserController.java)

## CRUD

```java
// insert 3 users (alice, lucy & emma) into user collection
User alice = new User().withId(1).withName("Alice").withBalance(100);
User lucy = new User().withId(2).withName("Lucy").withBalance(85);
User emma = new User().withId(3).withName("Emma").withBalance(105);

userCollection.insert(alice);
userCollection.insert(lucy);
userCollection.insert(emma);

// read alice from user collection
User alice = userCollection.readOne(Filters.eq("id", 1));

// update emma's name in user collection
userCollection.update(
        Filters.eq("id", emma.getId()),
        UpdateFields.newBuilder().set(
            UpdateFields.SetFields.newBuilder().set("name", "Dr. Emma").build()
        ).build()
);

// delete lucy from user collection
userCollection.delete(Filters.eq("id", lucy.getId()));
```

In this starter project similar code references are located in [`com. tigrisdata.starter.controller.UserController`](https://github.com/tigrisdata/tigrisdb-starter-java/blob/main/src/main/java/com/tigrisdata/starter/controller/UserController.java)

## Perform a transaction

```java
TigrisDatabase myDatabase = client.getDatabase(dbName);
TransactionOption transactionOptions = new TransactionOption();
TransactionSession session = myDatabase.beginTransaction(transactionOptions);
try {
    TransactionTigrisCollection<User> userCollection = session.getCollection(User.class);
    User emma = userCollection.readOne(Filters.eq("id", emma.getId()));
    User lucy = userCollection.readOne(Filters.eq("id", lucy.getId()));

    // substract emma's balance by 10
    userCollection.update(
        Filters.eq("id", emma.getId()),
        UpdateFields.newBuilder().set(
            UpdateFields.SetFields
                .newBuilder()
                .set(
                    "balance",
                    emma.getBalance() - 10
                ).build()
        ).build()
    );

    // add lucy's balance by 10
    userCollection.update(
        Filters.eq("id", lucy.getId()),
        UpdateFields.newBuilder().set(
            UpdateFields.SetFields
                .newBuilder()
                .set(
                    "balance",
                    lucy.getBalance() + 10
                ).build()
        ).build()
    );
    // commit transaction
    session.commit();
} catch(Exception ex) {
    // handle ex
    // in case of an error, rollback
    session.rollback();
}
```

In this starter project similar code references are located in [`com.tigrisdata.starter.controller.OrderController`](https://github.com/tigrisdata/tigrisdb-starter-java/blob/main/src/main/java/com/tigrisdata/starter/controller/OrderController.java)
