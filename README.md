# MongoDB
- It is a NoSQL database, more specifically a document type database that uses JSON.
- One of the important differences between NoSQL and relational databases is that you don’t need to deal with any table/database schema.
- Major advantage of using NoSQL databases is scaling.
  - But it comes with its own major disadvantage name “eventual consistency” unlike relational database where immediate consistency is maintained.
  - So, use it only when needed, especially when there are not much inter-connected relations.
- Mongo store documents in a format called BSON, Binary JSON
- #### Relational vs Mongo
  | Relational  | Mongo      |
  | ----------- | ---------- |
  | Database    | Database   |
  | Tables      | Collections|
  | Rows        | Documents  |
  | Columns     | Fields     |
  | Primary Key | Id         |
- #### Query Execution
  | Without Indexes                             | With Indexes                                 |
  | ------------------------------------------- | -------------------------------------------- |
  | Collection scan, each document is evaluated | Does not scan every document in collection   |
  | Slow searches	                              | Fast searches                                |
  | Fast inserts, updates                       | Slower inserts/updates                       |
- Booting up MongoDB server in windows:
  - mongod --dbpath "C:\Program Files\MongoDB\Server\5.0\data\db" - to start mongodb in windows. This is okay for compass or for url. If you want to have console do the following along with this
  - mongo --shell after starting the server using the above command to bring up shell.
- Queries:
  - Create a user:
    ```
    db.createUser(
      {
        user:"mani",
        pwd: "1234",
        roles: ["readWrite", "dvAdmin"]
      }
	); 
    ```
  - `show dbs` - list all databases
  - `use mycustomers` - use mycustomers databases. Creates one if doesn't exist
  - Data can look like anything, there need not to be consistent between rows. i.e., schema may or may not differ from one row to another. That is the beauty of NoSQL. It should be in JSON format as shown below
    ```
    {
      first_name:"John",
      last_name: "Doe",
      memberships: ["mem1", "mem2"],
      address: {
        street : "4 main st",
        city: "Boston"
      },
      contacts: [
        {name: "Brad", relationship: "friend"},
        {name: "John", relationship: "friend"}
      ]
	}
    ```
  - Create a collection:
	- Collections are similar to what tables in relational databases. They hold documents which are rows. `db.createCollection('customers');`
	  - Remember the collection name here. It's gonna be used from here on.
  - Insert into collection:
    ```
    db.customers.insert(
      {
        first_name: "John", last_name: "Doe"
      }
	);
    ```
  - Bulk Insert into collection:
    ```
    db.customers.insert(
      {
        first_name: "Steven", last_name: "Smith",
        first_name: "John", last_name: "JohnSon"
      }
    );
    ```
  - You can also add extra field, as NoSQL doesn't follow any schema.
    ```
    db.customers.insert(
      {
        first_name: "Yerr", last_name: "Son", gender: "female"
      }
    );
    ```
  - See documents in collections:
    - `db.customers.find();` - this will return all documents in the collections.
      - Also note an automatic _id is created which acts as key for this entry.
		  - use pretty() with find() for JSON formatted output
  - Updating a document where first_name is "John":
    ```
    db.customers.update({first_name: "John"},
      {
        first_name: "John",
        last_name: "Doe",
        gender: male
      }
	);
    ```
    - What this does is that it replaces the entire value of that key whose first_name is "John". That is the reason why you added first_name and last_name even though that document has them
    - There is a way around if you do not want to replace everything and just add a new field.
      ```
      db.customers.update({first_name: "Steven"},
	    { 
		  $set: {gender: "male"}
	    }
      );
      ```
      - This will add gender field to steven without replacing everything, but instead just appends this new data.
  - Increment a value
	  - Let us first add a document with age, so that we can use it to use increment operator.
	    ```
		db.customers.update({first_name:"Steven"}, {$set: {age: 45}}); - then
	    ```
      ```
      db.cust`omers.update({first_name: "Steven"}, {$inc: {age: 5}}); - this will increment Steven's age by 5.
      ```
  - Remove a field:
	  ```
      db.customers.update({first_name: "Steven"}, {$unset: {age: 1}}); - 1 here denotes as a boolean indicator.
      ```
  - Try updating something that is not there:
    ```
    db.customers.update({first_name: "Mani"}, {last_name: "Ankam"});
    ```
    - If there is not a match - then you will not see any error but you get a reply with all 0 as shown: `WriteResult({ "nMatched" : 0, "nUpserted" : 0, "nModified" : 0 })`.
  - If you want to insert when updating something that doesn't exist, then use upsert boolean as shown below\
	```
    db.customers.update({first_name: "Mani"}, {last_name: "Ankam"}, {upsert: true});
    ```
    - Now you can find this entry using db.customers.find()
  - Rename:
	```
    db.customers.update({first_name: "Steven"}, {$rename: {"gender": "sex"}});
	```
    - Renames the gender field to sex.
  - To Remove documents:
    ```
    db.customers.remove({first_name: "Steven"});
    ```
	- This removes all documents whose first_name is Steven. So be careful.	
	- You can limit the number of deletions
	  ```
      db.customers.remove({first_name: "Steven"}, {$justOne: true})
      ```
      - this will delete the first occurent of this match.
  - Now, let say we have something a bit more complex in our database as shown below:
    ```
    {
      first_name:"John",
      last_name: "Doe",
      gender : "female",
      age: 35,
      memberships: ["mem1", "mem2"],
      address: {
        street : "4 main st",
        city: "Boston",
        state: "MA"
      },
      balance: 99.9
	}
    ```
  - Find with a where clause:
    ```
    db.customers.find({first_name" "John"});
    ```
    - Or operator
      ```
      db.customers.find({$or: [{first_name: "Sharon"}, {first_name" "John"}]});
      ```
      - notice the array for muliple or parameters
    - less than operator
      ```
      db.customers.find({age:{$lt: 40}});
      ```
      - Similarly
        - gt -> greater than
		- lte -> less than or equal to
		- gte -> greater than or equal to
    - Find everyone that lives in the city of boston:
      ```
      db.customers.find({"address.city" : "Boston"});
      ```
      - notice the quotes for field names unlike other places, as th city is nested inside address. Will not work otherwise.
    - Find everyone that has mem1 in their memberships:
      ```
      db.customers.find({memberships: "mem1"});
      ```
  - Sorting:
    ```
    db.customers.find().sort({lastname: 1}); -> ascending order
	db.customers.find().sort({lastname: -1}); -> for descending order
    ```
  - Count:
	```
    db.customers.find().count(); -> returns number of documents
    ```
  - Count with where
	```
    db.customers.find({gender: "male"}).count()
    ```
  - Find with limit:
	```
    db.customers.find().limit(4); returns the first 4
    ```
  - Combinations will also work
	```
    db.customers.find().limit(4).sort(){last_name: 1};
    ```
  - For each:
    ```
    db.customers.find().forEach(function(doc){print("Customer Name : " + doc.first_name)});
    ```
  - Text search:
    - Adding an index on the field will improve performance in this case, so first create an index on first_name
      ```
      db.customers.createIndex({first_name: 'text'}); -> here text is the name of index to use that index
      ```
    - Now search will be faster as we have an index
      ```
      db.customers.find(
      {
        $text : { $search: "\" Steve\""}
      }).pretty();
      ```
  - Sub-document:
    - Let us assume you have a document inside an array of a document as shown below:
      ```
      db.customers.update({ first_name: 'Steven' },
      {
        $set: {
        phones: [
          {
          number: 78954098,
          type: 'mobile',
          extension: '+91'
          },
          {
          number: 78096738,
          type: 'home',
          extension: '+91'
          }
        ]
        }
      });
      ```
    - Now, to find by an element in array - we need to use $elemMatch as shown below
      ```
      db.posts.find({
        phones: {
         $elemMatch: {
           type: 'mobile'
           }
        }
        }
      );
      ```   
# Using MongoDB within your Spring application
- #### Spring annotations used for model objects:
  - @Document - Used on classes just like entity with JPA. Name of the collection is deduced from class name. Or you can also use collection attribute just like @Document(collection= "name")
	- @Id - Used on primary field. This field is by default indexed.
	- @Field - used like @Column. Use as @Field(name= "name")
	- @Transient - use on field that does not need to be persisted. This is usually used on internal field that are not part of document.
	- @Indexed - use this on fields that needs indexing for better performance. @Indexed(direction=IndexDirection.ASCENDING, unique=false)
	- @TextIndexed - Just like @Indexed but should be used only on strings.
	- @CompoundIndex - used on multiple fields at the class level.
	- @DbRef - Used on fields. Acts somehow like joins. Used to link one document in an another one.
- #### Mongo Query Object:
  - Is most powerful and flexible component for Mongo CRUD operations
  - Query Components:
    - Criteria for filtering data
    - Sorting definition for ordering data - optional
    - Paging definition for splitting data - optional
  - This is how a sample spring query for mongo looks like
    ```
    Query byAge = Query.query(Criteria.where(“age”).gt(18))
    .with(Sort.by(Sort.Direction.DESC, “age”))
    .with(PageRequest.of(1, 10));
    ```
  - Mongo Filter Operators
    - ls/ne – for equal to and not equal to
    - lt/lte – for less than and less than or equal to
    - gt/gte
    - in – for value in list
    - exists – has value
    - regex – for regular expression patterns
  - MongoTemplate
    - Used for executing a query in spring as shown below
      ```
      List<Person> people = this.mongoTemplate.find(byAge, Person.class);
      ```
    - A simple injectable class following the standard template pattern in Spring
    - It allows us to perform CRUD operations on data
    - It allows us to execute commands against a Mongo Database
    - It’s very powerful, but also low level
  - Some more example of Mongo query object for Spring
    - Using multiple criteria:
      ```
      Query bigAricraftAnd737 = Query.query(new criteria()
        .andOperator(Criteria.where(“family”).is(“737”),
        Criteria.where(“nbSeats”).get(250)
      )
      );
      ```
    - Finding in sub documents:
      ```
      Query engineNeedingMaintenance = Query.query(
        Criteria.where(“engine.needsMaintenance”).is(true)
      );
      ```
  - Implementing Full Text Search:
    - Test Indexes
      - Bits and pieces that makes full text search possible.
      - Works on properties of type string or arrays of strings.
      -	You can text index properties across the whole object graph
      -	Pay attention to weights as they may change the order or relevance of a found document
      -	In spring, the process starts with @TextIndexed annotation
      -	It is important to provide these annotations with their weights when creating a document class in your spring application.
        - For example, let us assume you have title and aboutMe fields in person document and you used @TextIndexed annotation without any weights. In this situation when you search for a text, results are sorted based on internal score provided by MongoDB.
        - Now let us assume that you want to get results based on aboutMe first and then others when searching, in which case you need to provide more weight to than annotation as shown below `@TextIndexed(weight=2)`
    - Executing a Full Text Search
      ```
      TextCriteria textCriteria = TextCriteria.forDefaultLanguage().matching(text);
      Query byFreeText = TextQuery.queryText(textCriteria).sortByScore().with(PageRequest.of(0, 3));
      return mongoTemplate.find(byFreeText, FlightInformation.class);
      ```
- #### Transactions: Used for inserting, update and delete:
  - Single Document transactions:
    - An operation on a single document is atomic i.e., it either succeeds or fails. Because relationships are embedded in a single document, this mostly eliminates the need for multi-document transactions.
  - Multi document transaction:
    - For situations that require atomicity to multiple documents (in single or multiple collections), Mongo 4+ supports multi document transaction.
  - ***Though multiple document transactions are supported, it is recommended not to overuse them.*** Because Mongo is not designed for it.
  - We can use MongoTemplate for inserting as shown below
    - **Inserting data**
      ```
      Person p = new Person(“Mani”, 18);
      mongoTemplate.insert(p);
      ```
      - In this case, an automatic id is provided. You can provide your own id, but make sure that it is unique. Otherwise you will get a duplicate key error.
    - For inserting multiple documents:
      - The naïve way is to use for loop over mongoTemplate.insert()
      - Or we can combile everything to list, and use *insertAll()* method on mongo template.
    - **Updating data**
      - Single document:
        - `mongoTemplate.save(person);` - after getting this person by Id.
        - Save is more like insert if not found kind of operation.
        - Also remember, the complete object will be saved and not just the element you want to add. So carefully deal with objects when using save.
      - Comparision between insert and save:
        | Insert	                                                             |  Save                                                                | 
        | -------------------------------------------------------------------- | -------------------------------------------------------------------- |
        | With no ID, generates one and inserts the document in the collection | With no ID, generates one and inserts the document in the collection |
        | With existing ID, if not present, inserts document in collection	   | With existing ID, if not present, inserts document in collection     |
        | With existing ID, throws an error	                                   | With existing ID, overwrites the whole document with new values      |
        | Has batch operation support(insertMany)	                             | Doesn’t have batch operation support                                 |
      - Use insert for new documents and save for updates. Do not use save for both operations just because it feels easy. Otherwise, you may face subtle problems.
      - Though Mongo doesn’t support batch updates, MongoTemplate does offer support for batch updates, but to use it you need to
        - Retrieve the documents you want to update using the query object.
        -	Define the fields that you want to update along with the new values.
        -	Update those fields using the “updateMulti()” on mongo Template
        -	Here is an example of how it looks like
          ```
          Query airportsInRomania = Query.query(Criteria.where(“country”).is(“Romania”));
          Update setWeather = Update.update(“proximityWeather”, Weather.CLOUDS);
          mongoTemplate.updateMulti(airportsInRomania, setWeather, Airport.class);
          ```
      - Now, because of the nature of Java objects, when using update* methods, only fields defined in the update definition are affected, not the whole document.
    - **Deleting data**
      - We can delete single document, multiple documents, or an entire collection.
      - To delete single `mongoTemplate.remove(airport);`
      - To delete multiple documents
        ```
        Query franceAirports = Query.query(Criteria.where(“country”).is(“France”));
        mongoTemplate.findAllAndRemove(franceAirports, Airport.class);
        ```
      - To delete all documents in a collections there are multiple ways.
        - First generate a query that gets all documents and the use *findAllAndRemove* method like below
          ```
          Query query = new Query();
          mongoTemplate.findAllAndRemove(query, Airport);
          ```
        - Or the simplest way is to use dropCollection method as shown below
          ```
          mongoTemplate.dropCollection(Airport.class);
          ```
- #### Mongo Custom Converters:
  - Though spring knows how a field in Java to be converted to MongoDB variant, sometimes you might want to change that behavior. You can do that using converter
  - Converter creation process:
    - Create write converter (from Java type to Mongo type)
    - Create read converter (from Mongo to Java type)
    - Register converts as a spring bean.
  - Let us assume you want to store address value of person. You have decided to store it as comma separated value in the database, but in your spring application, you want it as an address object. Here is how converters for those looks like
    ```
    AddrWriteConverter.java:
      public class AddrWriteConverter implements Converter<Address, String>
      {
        @Override
        public String convert(Address address)
        {
        return address.getCity() + “,” + address.getCountry();
      }
      }
    ```
    ```
    AddrReadConverter.java:
    public class AddrReadConverter implements Converter<String, Address>
    {
      @Override
      public Address convert(String s)
      {
        String[] parts = s.split(“,”) +
        return new Address(s[0], s[1]);
    }
    }
    ```
  - You can register the converters in any `@Configuration` class.
  - In spring boot, it can be the main application or define a new class and annotate it with @Configuration and add the following bean.
    ```
    @Bean
    public MongoCustomConversions customConversions()
    {
      List<Converter<?, ?>> converters = new ArrayList<>();
      converters.add(new AddrReadConverter());
    converters.add(new AddrWriteConverter());
    return new MongoCustomConversions(converters);	
    }
    ```
- #### Mongo Repositories:
  - An injectable interface that can be used in Spring applications.
  - Provide basic CRUD operations capabilities out of the box
  - Can be expanded with ease
    ```
    Repository<T, ID> # by spring
              ^
              |
    CrudRepositroy<T, ID> # by spring
              ^
              |
    PagingAndSortingRepository<T, ID> # by spring
              ^
              |
    MongoRepository<T, ID> # by mongo
    ```
  - `@Repository` annotation is used over these repository classes.
  - At this point you might be wondering, how does extending MongoRepository in your repository interface will work as there is no implementation provided for those method by you. The answer is spring factory bean. Spring knows the implementation for all methods of before mentioned interfaces and maintains them as beans.
  - **Query Methods:*
    - We really cannot forget about Query methods when discussing about repositories
    - Query methods are declarative way to add functionality to a spring repository.
    - Query methods depend on return type, prefix, field names and filter(s). Look at the following query method
      - `List<Airport> findByFlightsPerDayGreaterThan(int value);`
        - Here List<Airport> is the return type
        - findBy – is the method prefix
        - FlightsPerDay – is the field name on this document
        - GreaterThan – is the filter,
      - Based on these, spring provides the implementation automatically.
      - Some more examples of query methods
        ```
        List<Airport> findByFlightsPerDayBetween(int min, int max);
        List<Airport> findByFlightsPerDayGreaterThan(int value);
        List<Airport> findByFlightsPerDayGreaterThanEqual(int value);
        List<Airport> findByFlightsPerDayLessThanEqual(int value);
        ```
      - Query methods for String properties:
        ```
        List<Airport> findByNameLike(String airportName);
        List<Airport> findByNameNotNull();
        List<Airport> findByNameNull();
        Optional<Airport> findByName (String airportName); - no filter, matches exact value provided
        List<Airport> findByName (String airportName); - - no filter, matches exact value provided
        ```
      - Query methods for boolean properties
	      ```
        List<Airport> findByClosedTrue();
	      List<Airport> findByClosedfasle();
        ```
      - Query for complete query methods
	      - List<Airport> findByClosedTrueAndFlightsPerDayGreaterThan(int minFlights);
    - What about even more complex queries:
      - We can use `@Query` annotation inside which you need to provide true mongo query. (Remember the fields names provided in these queries are not java fields names and should always use field names in the document as in mongo db)
      - This way you can make repository methods execute custom Mongo query language constructs.
      - Here is how they look:
        ```
        @Query(“{‘location.city’: ?0}”)
        List<Airport> findByCity(String city);

        @Query(“{‘flightsPerDay’ : {$lte: 50}}”)
        List<Airport> findSmallAirports();
        ```
      - Prefer standard query methods instead of @Query where possible. Use @Query just for more complex queries.
- #### Mongo Document References:
  - A way to link together multiple documents that are related.
  - Since Mongo DB is NoSQL, for most cases the denormalized model where is stored in a single document is optimal. So, try to minimize the number of relations between documents.
  - But since enterprise applications are complex, there occurs a situation where you need to relate two documents. For example, let us consider for a banking application, bank transactions are related to customer. But instead of providing any relationship, you designed the application to store customer information in every transaction. Now, when the customer information is updated, you also need to find all transactions that relate to the customer an update it which is a bad design. So, you need to have relationship between transaction and customer.
  - Relating Documents is of two types:
    - Manual: where you maintain id of the related document and process based on it.
    - DBRefs: Link documents together by using “_id” field, collection name and data base name. This is like manual reference on steroids.
      - Can link document across collections or across different databases.
      - To resolve DBRefs, the application must perform additional queries.
      - Cascading is not supported.
    - DBRef format:
      - $ref: the name of the collection where the linked document resides.
      - $id: the value of the _id field of the referenced document.
      - $db: the name of the database where the referenced document resides.
    - Example:
      ```
      {
        “_id”: ObjectId(“4e98734f-2845-43b6-ae71-e4098237hel34”),
        “name”: 737,
        “capacity”: 234,
        “engine”: {
          “$ref”: “engines”,
          “$id”: ObjectId(“23df098-23409-4e549-1-e-04rf-e-2-”)
          “$db”: “atm” //optional
        }
      }
      ```
    - *Note:* The order of fields in the DBRef matters and must be used in the correct sequence.
    - DBRefs are not “smart” like relational database. So know the limits.
  - `@DBRef` annotation is used in Spring
  - Rememer DbRef in Mongo is just some json data and Mongo doesn’t fetch the document automatically. But in Spring, the process is automatic. Spring Data MongoDB fetches documents annotated with @DBRef for us automatically.
	- Important points:
    - Cascading on save doesn’t work by default. Which mean when you insert a parent object, the child object is not inserted but the parent object does without any error.
    - So, to make this work, we first need to insert the child object, assign this to parent object and then save the parent object.
    - Alternatively, you can use @DBRef(lazy=true) in the model class.
      - Useful for bi-directional references to avoid a cyclic dependency.
  - *Note*: The most important things to remembers is, @Query with document references fields will not work, as it is not directly available on the parent document. So, keeping these things in mind, design your application accordingly.
- #### Spring Mongo Lifecycle Events:
  - Events that your application can respond to by registering special beans in the spring application context.
  - “Before” events:
    - onBeforeConvert
    - onBeforeSave
    - onBeforeDelete
  - “After” events:
    - onAfterConvert
    - onAfterSave
    - onAfterLoad
    - onAfterDelete
  - Save/Update document events:
    - You usually call insert or save in MongoTemplate
    - onBeforeConvert is called before the java object is converted to a document by the MongoConverter
    - onBeforeSave is called before inserting or saving the document in the database
    - onAfterSave is called after the document was inserted or saved in the database
  - Load documents events:
    - You usually call find, findOne, findAndRemove in the mongo template
    - onAfterLoad is called after the document has been retrieved from the database.
    - onAfterConvert is called after the document has been converted into java object.
  - Delete document events:
    - Call remove on MongoTemplate
      - onBeforeDelete is called just before the document gets deleted from the database
      - onAfterDelete is called after the document has been deleted from the database.
  - Event behavior:
    - Lifecycle events are emitted only for root level types. Sub documents are not subject to event publication unless they are annotated with @DBRef
    - It is all happening in an async fashion. We have no guarantees to when an event is processed.
  - When do you need them?
    - Trying to implement cascade on save
    - Trigger some job/action in different systems
    - Security auditing.
  - How to use them?
    - Start with
      ```
      public abstract class AbstractMongoEventListener<E> implements ApplicationListener<MongoMappingEvents<?>> {
	      // has all the methods for above mentioned methods.
      }
      ```
    - You need to extend the above class and then override the corresponding event method.
      - You also need to register this as bean using @Component on the class or create a bean in configuration class.
    - Suggestion, try to create an event listener per feature. Like CascadingEventListener, or AuditingEventListener instead of a general one for all.
- #### Data Migrations in MongoDB:
  - No matter how future proof your design is, data changes over time and you need to reconcile these changes between your MongoDB and spring application.
  - You have options like creating your own application/scripts manually to handle these changes or use a framework than does things automatically.
  - MongoBee is one such framework in Java that automates the process of migration from one version to another.
  
  
  
References: 
[1] https://www.youtube.com/watch?v=-56x56UppqQ&t=0s
[2] https://app.pluralsight.com/library/courses/spring-framework-data-mongodb/table-of-contents
