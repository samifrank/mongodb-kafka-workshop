# mongodb-kafka-workshop
Learn more about MongoDB and Kafka and how to connect the two in a local environment

#todo
- link to confluence guide?
- export of guide published to readme - once more complete
  


# test markdown

> [!TIP]
>  Helpful advice for doing things better or more easily.

> [!NOTE]  
> Highlights information that users should take into account, even when skimming.

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

* Download Docker Desktop
  * https://www.docker.com/products/docker-desktop/
* Download a GUI for MongoDB
  * _We will be using Mongo Compass in this Workshop. Feel free to use what you’re comfortable with, but we can’t guarantee good tech support on other GUI options._
  * https://www.mongodb.com/try/download/compass
* Pull down this repo which has a folder called `CR-workshop-2024`. This folder contains the necessary docker components for this workshop.
* Download additional sample datasets for MongoDB by pulling down this repo: https://github.com/neelabalan/mongodb-sample-dataset
  * We will specifically be using the Netflix movies dataset, but feel free to explore more of the sets. Each of these are provided by MongoDB and get used in MongoDB University courses.
 
## Troubleshooting Notes:
<details>
  <summary>Need a fresh start with your Kafka components?</summary>
  
  * `docker compose up -d --force-recreate`
  * This will tear down most of the Kafka work you have done. The data within your local mongo would be unaffected due to the volumes that were created
</details>

<details>
  <summary>Need a fresh start with your mongo data??</summary>

  * Delete the mongo volumes in Docker Desktop
    * Make sure you 🛑 _Stop_ 🛑 your docker containers first
    *  _Delete_ the mongo containers `[mongo-node1, mongo-node2, mongo-node3]`
      ![image](https://github.com/samifrank/mongodb-kafka-workshop/assets/84085490/a131d831-d295-4364-b887-7b9de2ec7b30)
    * _Delete_ the mongo volumes
    
      ![image](https://github.com/samifrank/mongodb-kafka-workshop/assets/84085490/4c28d09b-d39d-4543-a155-78316c7189d8)
</details>

## Mongo 
> [!NOTE]  
> ### **MongoDB Terms**
>   * **mongod**: The actual process that is running your mongodb instance. It handles data requests, manages access, performs background tasks, etc.
>   * **mongosh**: The mongo shell is a Javascript environment that is used to interact with the mongod process. It is available on many MongoDB GUIs as well as downloadable for use on terminal.
>   * **replica set**: An odd number of mongod instances that maintain the same set of data.

> [!WARNING]  
> In order to properly connect to your local mongo in this `docker-compose` file, **we need to be able to resolve the mongo host created in the docker container on our machine** (outside of the docker environment)
> Add a new host definition for your local IP address to include mongo in your laptops `/etc/hosts` file to include:
> `127.0.0.1       mongo-node1`
> _note_: `mongo-node1` is the main mongo container created in the provided file `docker-compose.yml`
> Using a normal text editor: find `/etc/hosts` and add the definition noted above.
> In a terminal:
>   1. navigate to the `/etc` directory on your machine
>      a. On a Windows Machine: `C:\Windows\System32\drivers\etc\hosts`
>      b. On a Mac: `/etc/hosts`
>   2. `sudo nano hosts`
>      a. This command will allow you to edit your hosts file. Add the definition noted above.
>      ![image](https://github.com/samifrank/mongodb-kafka-workshop/assets/84085490/a00dc743-ecf4-41e9-b8a6-19115d89ec5b)
>      
>   3. To save your changes: `Control + X` -> `Y`

# TODO maybe agjust 
### Getting Things Set Up in MongoDB
1. In a terminal, navigate to wherever you downloaded the files needed for this workshop. Run the command `docker compose up -d`
   
   _Take a look in Docker Desktop to see all of the needed containers for this workshop start to spin up._
 
   > [!NOTE]
   > If you see containers that are not running and grey in color, press the play button to try and spin them up.
   > ![image](https://github.com/samifrank/mongodb-kafka-workshop/assets/84085490/a16afa4c-7be2-41c6-8dc5-c7d0c74840b2)

2. Once you see the 3 mongo contains running, we can connect to it through MongoDB Compass
   ![image](https://github.com/samifrank/mongodb-kafka-workshop/assets/84085490/17a29230-d7ed-4f6c-b2f6-ea0ad94a24a0)

   a. `mongodb://mongo-node1:27017/?replicaSet=rs0`

3. Add data to mongo
   a. :star2: Create a Database and a collection using the Netflix movies data set.
      i. In the demo, we call the database netflix and the collection movies. We also import the other data sets in the Netflix sample as well and store them as other collections. The demo only utilizes the movies dataset, though.
   b. Add data to this new collection by importing a JSON file. 
      i. :magic_wand: When you perform an import in Compass, the tool leverages a MongoDB command called [mongoimport](https://www.mongodb.com/docs/database-tools/mongoimport/) . If you were to load data from the terminal, this is the exact command you would use it the mongosh.
   c. Congratulations! You officially have a three node [replica set](https://www.mongodb.com/docs/manual/replication/) populated with data!

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

❓❓❓
> Datasite has two different architectures for replica sets: the global set and the regional set. 
> Global
>     US East: 2 nodes, US Central: 1 node, EU 1: 1 node, EU 2: 1 node, AU 1: 1 node, AU 2: 1 node
> Regional (US)
>     US East: 2 nodes, US Central: 1 node
> * What happens in the event that US East loses one of its nodes?
> * What happens if the entire US East data center goes down?
> * What could Datasite do to increase its resiliency to make the loss of the US East data center less impactful for the regional nodes? 
❓❓❓





