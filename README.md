# mongodb-kafka-workshop
Learn more about MongoDB and Kafka and how to connect the two in a local environment! 

## Objective
> [!TIP]
> The goal of this workshop is to gain some familiarity with Mongo, Kafka and how we can transfer data from Mongo into Kafka.
> We will be covering the following:
>    * Try out MongoDB
>    * Try out Kafka
>    * Connect MongoDB to Kafka using a Kafka connector


## Emoji Key
* :thought_balloon: Something to think about or keep in mind as you’re going through this workshop.
* :question: A question. An answer will be expandable if available. Try to think of your answer before you expand.
* :star2:  An important idea or resource that may be relevant in multiple places throughout this workshop.
* :magic_wand: A hint or helpful info.

## Getting Started:
There are two tools we will need for this workshop. If you do not already have the following, go ahead and get these installed on your machine.

* [Download Docker Desktop](https://www.docker.com/products/docker-desktop/)
* [Download a GUI for MongoDB](https://www.mongodb.com/try/download/compass)
  * _We will be using Mongo Compass in this Workshop. Feel free to use what you’re comfortable with, but we can’t guarantee good tech support on other GUI options._
    
* Pull down this repo which has a folder called `CR-workshop-2024`. This folder contains the necessary docker components for this workshop.
* Download additional sample datasets for MongoDB by pulling down this repo: https://github.com/neelabalan/mongodb-sample-dataset
  * We will specifically be using the sample_mflix movies dataset, but feel free to explore more of the sets. Each of these are provided by MongoDB and get used in MongoDB University courses.
 
## Troubleshooting Notes:
<details>
  <summary>Need a fresh start with your Kafka components?</summary>
  
  * `docker compose up -d --force-recreate`
  * This will tear down most of the Kafka work you have done. The data within your local mongo would be unaffected due to the volumes that were created.
</details>

<details>
  <summary>Need a fresh start with your mongo data?</summary>

  * Delete the mongo volumes in Docker Desktop
    * Make sure you 🛑 _Stop_ 🛑 your docker containers first
    *  _Delete_ the mongo containers `[mongo-node1, mongo-node2, mongo-node3]`
      ![image](https://github.com/samifrank/mongodb-kafka-workshop/assets/84085490/a131d831-d295-4364-b887-7b9de2ec7b30)
    * _Delete_ the mongo volumes
    
      ![image](https://github.com/samifrank/mongodb-kafka-workshop/assets/84085490/4c28d09b-d39d-4543-a155-78316c7189d8)
</details>

<details>
  <summary>Docker containers not spinning up?</summary>

  * If you are seeing an error message like the following:
    ![image (2)](https://github.com/user-attachments/assets/de9a20bd-0db2-450f-99e0-4b4ba6977fba)

  * Check if you are signed into Docker Desktop. If you are logged in - **log out**
      
</details>

## Get The Workshop Running

> [!WARNING]  
> In order to properly connect to your local mongo in this `docker-compose` file, **we need to be able to resolve the mongo host created in the docker container on our machine** (outside of the docker environment).
> 
> _Add a new host definition for your local IP address to include mongo in your machines_ `/etc/hosts` file to include:
> 
> `127.0.0.1       mongo-node1`
> 
> _note_: `mongo-node1` is the main mongo container created in the provided file `docker-compose.yml`
> 
> Using a normal text editor: find your `/etc/hosts` and add the definition noted above.
> 
> In a terminal:
>   1. navigate to the `/etc` directory on your machine
>      
>      a. On a Windows Machine: `C:\Windows\System32\drivers\etc`
>
>      b. On a Mac: `~./etc`
>      
>   2. `sudo nano hosts`
>      
>      a. This command will allow you to edit your hosts file. Add the definition noted above.
>      ![image](https://github.com/samifrank/mongodb-kafka-workshop/assets/84085490/a00dc743-ecf4-41e9-b8a6-19115d89ec5b)
>    
>   3. To save your changes: `Control + X` -> `Y`


1. **Start up Docker containers**: In a terminal, navigate to wherever you downloaded the files needed for this workshop. Run the command `docker compose up -d`
   
   _Take a look in Docker Desktop to see all of the needed containers for this workshop start to spin up._
 
> [!NOTE]  
> If you see containers that are not running and grey in color, press the play button to try and spin them up.

![image](https://github.com/samifrank/mongodb-kafka-workshop/assets/84085490/a16afa4c-7be2-41c6-8dc5-c7d0c74840b2)

2. Once you see the 3 mongo contains running, we can connect to it through MongoDB Compass
   ![image](https://github.com/samifrank/mongodb-kafka-workshop/assets/84085490/17a29230-d7ed-4f6c-b2f6-ea0ad94a24a0)

   a. `mongodb://mongo-node1:27017/?replicaSet=rs0`


## Mongo 
> [!NOTE]  
> #### **MongoDB Terms**
>   * **mongod**: The actual process that is running your mongodb instance. It handles data requests, manages access, performs background tasks, etc.
>   * **mongosh**: The mongo shell is a Javascript environment that is used to interact with the mongod process. It is available on many MongoDB GUIs as well as downloadable for use on terminal.
>   * **replica set**: An odd number of mongod instances that maintain the same set of data.

### Getting Things Set Up in MongoDB
 1. 🌟 Create a Database and a collection using the sample_mflix movie data set.
    
    a. In the demo, we call the database `netflix` and the collection `movies`. We also import the other data sets in the sample_mflix as well and store them as other collections. The demo only utilizes the movies dataset, though.
2. Add data to this new collection by importing a JSON file.
    
    a. :magic_wand: When you perform an import in Compass, the tool leverages a MongoDB command called [mongoimport](https://www.mongodb.com/docs/database-tools/mongoimport/) . If you were to load data from the terminal, this is the exact command you would use it the mongosh.
3. Congratulations! You officially have a three node [replica set](https://www.mongodb.com/docs/manual/replication/) populated with data!

### Exploring the Replica Set
1. Find the `>_MONGOSH` at the bottom of your Compass screen.
   ![image](https://github.com/samifrank/mongodb-kafka-workshop/assets/84085490/7f33234c-1092-4740-bafd-b4d1f4b316a9)

2. In the shell, type in: `rs.status()`  and scroll to the members section of the results. It should looks something like:  
  ![image](https://github.com/samifrank/mongodb-kafka-workshop/assets/84085490/2839133f-c98a-4740-a986-abba1c9b01d9)

Note which of your nodes is currently the Primary node.  A primary node in a replica set is the node that receives all of the writes. 

3. In the shell, type in: `rs.conf()` and look at the members section. Notice the priority of your primary node as well as the priority and votes attributed to the secondary nodes in the cluster.
4. Let’s change who is primary. Type in:
   ```
    conf = rs.conf()
    conf.members[1].priority = 2
    rs.reconfig(conf)
   ```
5. Now, run `rs.status()`. ❓ Which node is now primary? If you rerun `rs.conf()`, you should also see the change in priority reflected. 

> [!NOTE]  
> Datasite has two different architectures for replica sets: the global set and the regional set. 
>
> Global  
>   * US East: 2 nodes, US Central: 1 node, EU 1: 1 node, EU 2: 1 node, AU 1: 1 node, AU 2: 1 node
>     
> Regional (US)
>  * US East: 2 nodes, US Central: 1 node
>
> ❓❓❓
> 
> * What happens in the event that US East loses one of its nodes?
> * What happens if the entire US East data center goes down?
> * What could Datasite do to increase its resiliency to make the loss of the US East data center less impactful for the regional nodes?
>   
> ❓❓❓

### Exploring the Dataset
1. Try to find your favorite movie in the data set! Note, this is an older data set and the “latest” movies in the set are from 2016.
   
    a. If you want to remain in the mongosh, type `use neflix` to switch databases. Then, type `db.movies.find({ title: “Your movie title here“ })`.

    b. You can also use the query builder withing MongoDB Compass to browse the data.

![image](https://github.com/user-attachments/assets/1b76ce2a-f322-4770-b858-7e16068ad118)

2. Notice if you click on options in the query builder, you can leverage:
   
    a. **project**: this reduces the number of fields returned by the cursor
   
    b. **sort**: orders the results based on the field(s) indicated (-1 desc sort, 1 asc sort)
   
    c. **collation**: applies a language to the query search
   
    d. **index hint**: used when a query has trouble using the appropriate index
   
    ![image](https://github.com/samifrank/mongodb-kafka-workshop/assets/84085490/2cc27be2-c9e9-4bb9-899b-01ae5b90b489)

    You can do a similar query in the shell:

  ![image](https://github.com/user-attachments/assets/da4d371b-4474-4d6c-9f17-dc9f05822ddc)


3. See if you can answer the following questions about the data set using the `find` operator:

     🪄 If you have never worked with MongoDB queries, you may have to find some helpful operators from the MongoDB documentation (or a Google search). 

   <details>
      <summary> ❓ What is the title of the oldest movie in the dataset? What is the newest? </summary>
      
      * Oldest: Newark Athlete, 1891
      
      * Newest: The Masked Saint, 2016
      * Query: `db.movies.find({released: {$exists: true}}, {released: 1, title: 1}).sort({released: -1}).limit(2)`

    </details>

    <details>
      <summary> ❓ Which movie won the most awards?</summary>
      
      * 12 Years a Slave (267 awards)
      * Query: `db.movies.find({}, {"awards.wins": 1, "title": 1}).sort({"awards.wins": -1}).limit(1)`

    </details>

    <details>
      <summary> ❓How many movies had an imbd rating greater than or equal to 9?</summary>
      
      * 31
      * Query: `db.movies.find({"imdb.rating": {$exists: true}, "imdb.rating": {$gte: 9}}).count()`

    </details>

    <details>
    <summary> ❓What was the most recent movie to receive an imbd rating greater than or equal to 9?</summary>
    
    * “A Brave Heart: The Lizzie Velasquez Story”, 2015
    * Query: `db.movies.find({"imdb.rating": {$exists: true}, "imdb.rating": {$gte: 9}}, {title: 1}).sort({released: -1}).limit(1)`

  </details>

  * If you are comfortable with basic MongoDB operators and you want some aggregation practice: 

    <details>
      <summary> ❓ Bucket the movies by their runtime. Use the boundaries 1, 50, 100, 200, 300, 400, 500, 1000, 1500. Which range has the most movies?</summary>
      
      Most movies fall into the 100-200 range. 

      Query: `db.movies.aggregate([{$match: {runtime: {$exists: true}}}, {$bucket: { groupBy: "$runtime", boundaries: [1, 50, 100, 200, 300, 400, 500, 1000, 1500], output: {count: {$sum: 1}}}}])`

    </details>

    <details>
      <summary> ❓ What are the three most common genres in the dataset? Which three are least popular?</summary>
      
      Drama, comedy, romance are most popular. Least popular are talk-show, news, and film-noir. 

      `db.movies.aggregate([{$unwind: "$genres"}, {$group: {_id: "$genres", count: {$sum: 1}}}, {$sort: {count: 1}}])`

    </details>

### Setting Up a Change Stream

1. Let’s set up our collection for pre and post imaging. Go back to your `mongosh`, make sure you are using the netflix database, and enter the following: 
    ```
      db.runCommand({collMod: "movies", changeStreamPreAndPostImages: {enabled: true}})
    ```
2. Still in the shell, run the following:
   ```
     db.movies.updateMany({released: {$gte: ISODate("1967-01-01")}}, {$set: {colorRelease: true}})
     db.movies.updateMany({released: {$lt: ISODate("1967-01-01")}}, {$set: {colorRelease: false}})
   ```
3. Navigate to the config database in the shell using `use config`.
4. Run: `db.system.preimages.findOne()`.  ❓ Do you see a record of one of the documents you just updated?
5. Now, let us see if we can run a basic change stream. Set a watch on your cursor in your current shell using `use netflix`:
   ```
   watchCursorFullDocumentBeforeChange = db.movies.watch(
     [],
     { fullDocumentBeforeChange: "whenAvailable" }
   )
   ```
6. Open another window in MongoDB compass and open a new `mongosh` at the bottom. Navigate to your netflix db using `use netflix` and run the following update:
     `db.movies.updateMany({"imbd.rating": {$gte: 9}}, {$set: { addToWatchList: true }})`

Now, in your first window where you were watching the cursor, you should see 31 records that look something like this: 
![image](https://github.com/samifrank/mongodb-kafka-workshop/assets/84085490/cbd2cc8c-e5eb-4d95-920d-2d9d2d81962c)

💭 These are change stream entries for an update. They denote that the field addToWatchList was updated to true as well of what each document looked like prior to change! In our case, we opened up a watch on a cursor, but you can also leverage change streams with connectors, including one for Kafka. 


## Kafka:

### Exploring Kafka - General:
> [!NOTE]
> The docker compose file has a container ( `kafka-ui-data-demo` ) that is running a GUI for our local Kafka setup (Kafka UI). 🪄 You can access at: http://localhost:8426 

1. Take a look at what already exists in Kafka UI before you continue.
   <details>
      <summary> ❓ Do you notice certain topics that exist? Can you make any general conclusions to what those topics do based on their names? </summary>
      

   * You should see 3 topics during the initial start up of this workshop. At a high level, these three topics are helping manage 🌟 the connectors that exists in the `connect-data-demo` container (Kafka Connect) and the data those connectors will bring into the `broker-data-demo` container (Kafka broker). 

   * They’ll look pretty bare bones at first until we start moving data with a connector.

     💭 Take a look back at these topics after you get a **connector** running to see what kind of data they hold.

     `docker-connect-configs`: Stores the connector and task configurations. Connect worker nodes will listen to changes here in order to pick up and use the latest configs.
  
     `docker-connect-offsets`: Stores the offsets of the where the connectors are at in a particular stream. If a task fails and needs to restart, it can continue processing from where it left off by reading the stored offset.
  
     `docker-connect-status`: Stores the status of the connectors and tasks. Allows all worker nodes to know a connectors status so workers can stay in sync.

     **Want to learn more?** [Kafka Connect Concepts](https://docs.confluent.io/platform/current/connect/index.html#kafka-connect-concepts) 
    </details>

2. Create a topic

   ![image](https://github.com/samifrank/mongodb-kafka-workshop/assets/84085490/7fddb20d-2f15-4b86-b3f6-8c5be926cca7)

3. Poke around the UI in the topic you just created
  
   a. 🌟 Turn on ‘Live Mode’ - _This is a Kafka Consumer option that will read topic messages as they appear. Feel free to play around with the other options._

    ![image](https://github.com/samifrank/mongodb-kafka-workshop/assets/84085490/defbdeae-48cb-4dcc-bc76-c4a014451841)
   
   b. Produce a few messages to the topic, inspect the message

    ![image](https://github.com/samifrank/mongodb-kafka-workshop/assets/84085490/9c4c326d-8919-463e-9f44-f89382f9f0d0)

<details>
  <summary> ❓ Can you produce a message without a key or a value? </summary>
  
  * _Yes. There are a couple cases when you might forgo sending a key or a value._
    * Not sending a `key` (but including a `value`), would publish the message to a partition in the Kafka topic in a round robin fashion.
    * Not sending a `value` (but including a `key` ), is an indicator of a [tombstone record](https://medium.com/@damienthomlutz/deleting-records-in-kafka-aka-tombstones-651114655a16) in [compact Kafka topics](https://docs.confluent.io/kafka/design/log_compaction.html#topic-compaction-video)
  
</details>

<details>
  <summary> ❓  What might be the value of having a dedicated place in a topic message for a key? </summary>
  
  * Keying your message can allow you to logically group certain messages. This is helpful if your use case needs to _retain the order of incoming messages. Keep in mind, ordering can only be guaranteed at a key level._
    * When a record with a Key is produced to a topic, Kafka will decide what partition based on the following formula `partition # = hash(key) % total_partitions`
    * [More on keying and partitions](https://developer.confluent.io/courses/apache-kafka/partitions/#kafka-partitioning)
  
</details>


### Exploring Kafka Connect
_Now that you’re a little more familiar with MongoDB and with Kafka, lets try to connect the two!_

> [!NOTE]
> Kafka Connect is a key component that allows us an easy way to configure a connection between a data source and Kafka.
>
> There are **two** types of Kafka connectors:
>
> **Source** - _pulls_ data from a data source into Kafka
>
> **Sink** - _pushes_ data from Kafka into a data source
>
> 💭 What type of connector do you think we’ll use to hook up Mongo and Kafka?
>

> [!IMPORTANT]
> Kafka allows users to interact with the connect component through a Rest API
>
> There are several options, we will be focused on the `/connectors` set of endpoints in this workshop. Take a look at whats available:
>
> 🌟 [Kafka Connect REST Guide](https://docs.confluent.io/platform/current/connect/references/restapi.html#get--connectors-(string-name)-status)
>

1. In Docker Desktop, navigate to the container called `connect-data-demo` . In order to use these connect endpoints in this workshop, **we need to exec into the docker container where Kafka connect is running**. This gives us access to run commands on the running container.

  ![image](https://github.com/samifrank/mongodb-kafka-workshop/assets/84085490/ac9cf17e-ea0e-40af-a330-c3dfdd65e140)

2. Lets execute the `GET` [endpoint to view what connectors (if any) are currently added to Kafka](https://docs.confluent.io/platform/current/connect/references/restapi.html#get--connectors-(string-name)-status)

   `curl http://connect-data-demo:8083/connectors`

  > [!IMPORTANT]
  > 
  > `connect-data-demo` is the name of the connect container in our `docker-compose.yml` file. It is also the containers _advertised host name_ that we must use when making these REST calls. 

   <details>
    <summary> ❓  Was something returned? </summary>
    
   At first, we wont have any connectors and the return will be an empty list. This means that we have no Kafka connectors configured - which means there are no connections between Kafka and other data stores 

   lets fix that!
    
  </details>

3. We can use the `/connectors` endpoint with a `PUT` action [to add or adjust a connector](https://docs.confluent.io/platform/current/connect/references/restapi.html#put--connectors-(string-name)-config)
  
    a. Set up a connector configuration that will look at that new mongo collection you created earlier. Below is a sample to format your connector initially. 
      
      Please review and update the fields within {} before executing.  💭 What do you think they should be?
  
    b.
     ```
      curl -X  PUT -H "Content-Type:application/json" \
        http://connect-data-demo:8083/connectors/{NAME_YOUR_CONNECTOR}/config \
        -d '{"connector.class":"com.mongodb.kafka.connect.MongoSourceConnector",
             "connection.uri":"mongodb://mongo-node1:27017/",
             "database":"{DB_NAME}",
             "collection":"{COLLECTION_NAME}",
             "value.converter": "org.apache.kafka.connect.storage.StringConverter",
             "key.converter": "org.apache.kafka.connect.storage.StringConverter",
             "copy.existing": "true"
             }'
     ```

   <details>
    <summary> ❓ What do you think each configuration setting does? </summary>
    
     ```
      {"connector.class":"com.mongodb.kafka.connect.MongoSourceConnector", //defines what type of connector this is
        "connection.uri":"mongodb://mongo-node1:27017/", //how the connector should connect to the data source
        "database":"{DB_NAME}", //The database the connector should pull data from
        "collection":"{COLLECTION_NAME}", //The collection the connector should pull data from
        "value.converter": "org.apache.kafka.connect.storage.StringConverter", //How the connector should deserialize the key of the message
        "key.converter": "org.apache.kafka.connect.storage.StringConverter", //How the connector should deserialize the value of the message
        "copy.existing": "true" //Specific to MongoSourceConnector. This setting when true will pull every record from the collection upon initial start up 
      }
     ```
    
  </details>

4. Re-run the `GET` endpoint from step 2.  ❓ What do you see now?
5. At this point you should have a Kafka connector returned! Lets poke around some of the other `/connectors` endpoints that are available to us. 

    a. Check the `status` of the connector you just created

    b. Validate the `config` of the connector you just created

    <details>
      <summary> 🪄 Need Help? </summary>
      
     View Status: `curl http://connect-data-demo:8083/connectors/{NAME_YOUR_CONNECTOR}/status`
  
     View Config: `curl http://connect-data-demo:8083/connectors/{NAME_YOUR_CONNECTOR}/config`
      
    </details>

6. Go back to Kafka UI and take a look at the topics again. ❓ Do you see a new topic?
  
    a. Investigate some of the messages on your new topic
    
    b. Turn on 'Live Mode'
    
    c. Go back to Mongo Compass and update a record. ❓ Do you see that update in Kafka? Does the data in that update look different from other records you looked at in 6a.?
  
7. Explore the different connector configurations and see if you can notice the changes that happen because of them.  
  🪄 _The below configurations can be added to your existing connectors configuration that you created in Step 3. Make additional updates in your Mongo collection to investigate changes_

  > [!IMPORTANT]
  > 
  > Feel free to explore any of the options that Mongo allows: [Mongo Connector Configs](https://www.mongodb.com/docs/kafka-connector/current/source-connector/configuration-properties/all-properties/)
  >
  > Provided below are a few settings that may be more interesting and exist in the mentioned documentation.

  a. `change.stream.full.document`

  b. `change.stream.full.document.before.change`

   ❓ What are the differences between settings a. and b.?  🪄 view attached mongo documentation 

  c. `pipeline`

  
  <details>
    <summary> 🪄 Pipeline hints and examples? </summary>
    
   * You might need to [restart your connector & tasks](https://docs.confluent.io/platform/current/connect/references/restapi.html#post--connectors-(string-name)-restart) after you’ve applied a pipeline change. 🪄  look at the endpoints query parameters.

   * Examples:

     * Lets say you only want to pull in updates where the field rating changes

       * `"pipeline": "[{\"$match\": {\"updateDescription.updatedFields.rating\": {\"$exists\": true}}}]"`

     * Maybe you only care about DELETE events

       * `"pipeline": "[{\"$match\": {\"operationType\": \"delete\"}}]"`

   💭 What other aggregations might you want to filter your topic on? 
    
   </details>

  d. `topic.prefix` & `topic.separator`  
  * 🪄 To test this, you will need to create a new connector to see this change. Using _a different connector name_ in `{NAME_YOUR_CONNECTOR}` will generate a new connector instance. 













