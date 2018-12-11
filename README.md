# RuDiK

This repository contains a Java implementation of RuDiK, a system for discovering positive and negative logical rules over RDF knowledge graphs.
The prototype was first developed as a research project under the database group of Prof. Paolo Papotti at Ira A. Fulton Schools of Engineering, Arizona State University, and now moved to the Data Science group of EURECOM.

The current implementation works against a Knowledge Graph specified as a http SPARQL endpoint, please have a look at the src/main/config/Configuration.xml on how to specify the http endpoint and various parameters of the system.

The folder src/test/java/asu.edu.rule_miner.rudik.dbpedia contains some example tests on how to run the system against DBPedia Knowledge Graph.

# Technical Report

For a full technical report of RuDik and the algorithms involved, please refer to: http://www.eurecom.fr/en/publication/5321

# Run DBPedia Examples
 
 In this example, we are going to mine negative rules from DBPedia
 
## System Requirements

- Gradle >=4.10 
- Java 1.8 or above

## Installation

- Checkout git project `git clone git@github.com:stefano-ortona/rudik.git`
- See all defined tasks by executing `gradle tasks` in the poject folder.
- Build RuDik
  - `gradle build -x test` - Build RuDik wihtout executing the tests
- Assemble a jar file
  - `gradle jar` - Build a jar file that doesn't contain all dependencies
  - `gradle shadowJar` - Build a fatjar that contains rudik and all its dependencies.

## Run Spouse Example

We are going to run the test case `testSpouseNegative` of the test class `asu.edu.rule_miner.rudik.dbpedia.DBPediaClient`. This test case will mine negative rules for the target predicate _spouse_

### Configuration parameter

All configuration parameters are defined in an xml file. For this test case, we are using the specific file for DBPedia `src/main/config/DbpediaConfiguration.xml`. Have a look at this file, and in particular pay attention to the following parameters:

- **max_rule_lenght** -> Maximum number of atoms of the body of the output rules. The higher is this parameter, the bigger will be the search space (hence the slower the computation)
- **sparql_endpoint** -> This parameter defines the endpoint to query the target Knowledge Graph. In this example, the endpoint is set to the online accessible endpoint `http://dbpedia.org/sparql`. This parameter is necessary to query the external database
- **examples.positive(.negative)** -> These two parameters define the number of input positive (negative) examples to be used for the mining (currently set to 10 and 10). The higher are these two numbers, the bigger is the search space, but also the higher the chance to find valid rules. We found that an acceptable compromise between runtime and output quality can always be achieved with 500-1K examples, however in this spouse example we can mine qualitative rules with just 10 examples in input
- **include_literals** -> decide whether to include literal values or not in the output rules

For a full explanation of all parameters, check out the technical report.

### Run the example

Run the test case `testSpouseNegative` of the test class `asu.edu.rule_miner.rudik.dbpedia.DBPediaClient`. The computation, for this particular example, should take around 50 seconds (depending also on your network speed). At the end of the computation you should see an output like this:

```edu.rule_miner.rudik.ClientTest [INFO] ----------------------------COMPUTATION ENDED----------------------------
edu.rule_miner.rudik.ClientTest [INFO] Final computation time: 49.982 seconds.
edu.rule_miner.rudik.ClientTest [INFO] ----------------------------Final output rules----------------------------
edu.rule_miner.rudik.ClientTest [INFO] http://dbpedia.org/ontology/spouse(object,v0) & http://dbpedia.org/ontology/parent(subject,v0)
edu.rule_miner.rudik.ClientTest [INFO] http://dbpedia.org/ontology/successor(subject,object)
```

In the above log messages we found two negative output rules for the target predicate spouse. The first rule has 2 atoms in the body, while the second one has just 1 atom.

**NOTE:** The 10 examples we use in this test are randomly selected from a set of bigger examples, so it is possible that your output is different or empty. If that is the case and you want to see some real rules, try to re-run the test multiple times, or increase the number of input examples.

# Mine from your own Knowledge Graph

The above example mines rules against the publicly available version of DBPedia. If you wish to mine rules from your own Knowledge Graph, or simply you want to have a local version of DBPedia, you need to change the sparql endpoint in the configuration file, along with few other parameters.

Look inside the folder `src/main/config`, there you can find other configuration files for knowledge graphs such as _Yago_, _Freebase_, and _Wikidata_. We tested several RDF triple store and SPARQL engines, and we found that the fastest freely available is [Virtuoso](https://virtuoso.openlinksw.com/). To install Virtuoso on a Mac, simply use the brew package manager:

- brew install virtuoso (this will install Virtuoso in `/usr/local/Cellar/virtuoso`)
- use the Virtuoso bulk loader to insert your graph triples in Virtuoso http://vos.openlinksw.com/owiki/wiki/VOS/VirtBulkRDFLoader
- after loading the triples, access the Sparql endpoint at http://localhost:8890/sparql
- update the configuration file with the new Sparql endpoint and run your test cases

# Install Virtuoso with DBpedia files

The following steps may prompt for a sudo password and may need to be written in a virtuoso SQL client terminal.

- STEP 1: Installation of virtuoso server. (Borrowed from http://vos.openlinksw.com/owiki/wiki/VOS/VOSUbuntuNotes)

```sudo aptitude install  virtuoso-opensource```

- The above command triggers the download as well as the installation. It also prompts for a dba password

- STEP 2: Download the dbpedia ttl files. Let us say that the ttl files are downloaded in the user's home directory

```cd ~
mkdir dbpedia_ttl
cd dbpedia_ttl
wget http://downloads.dbpedia.org/2016-10/core/geo_coordinates_en.ttl.bz2
wget http://downloads.dbpedia.org/2016-10/core/persondata_en.ttl.bz2
wget http://downloads.dbpedia.org/2016-10/core/instance_types_transitive_en.ttl.bz2
wget http://downloads.dbpedia.org/2016-10/core/specific_mappingbased_properties_en.ttl.bz2
wget http://downloads.dbpedia.org/2016-10/core/infobox_properties_en.ttl.bz2
wget http://downloads.dbpedia.org/2016-10/core/mappingbased_literals_en.ttl.bz2
wget http://downloads.dbpedia.org/2016-10/core/mappingbased_objects_en.ttl.bz2
```

- STEP 3 : Uncompress the files. In case you don't have bzip2 install it using "sudo apt install bzip2" The following commands will delete the bz2 files after uncompressing. In case you want to preserve the zip files, issue the command as bzip2 -dk <filename>.bz2

```bzip2 -d geo_coordinates_en.ttl.bz2
bzip2 -d persondata_en.ttl.bz2
bzip2 -d instance_types_transitive_en.ttl.bz2
bzip2 -d specific_mappingbased_properties_en.ttl.bz2
bzip2 -d infobox_properties_en.ttl.bz2
```

 - STEP 4 : Modify the virtuoso.ini file with the filepath to the ttl files and set the number of buffers based on available RAM. Copy the virtuoso.ini file into the virtuoso db directory

```sudo cp /etc/virtuoso-opensource-6.1/virtuoso.ini /var/lib/virtuoso-opensource-6.1/db/virtuoso.ini
sudo cd /var/lib/virtuoso-opensource-6.1/db
sudo vi virtuoso.ini
```

Copy the path to the ttl files and append that to the DirsAllowed. For example, if home directory is /home/rudik, copy the ttl files as following:  DirsAllowed = ., /usr/share/virtuoso-opensource-6.1/vad, /home/rudik/dbpedia_ttls

The default number of buffers is 10000 and the maximum dirty buffers are set to 6000. Comment them depending on the available RAM, uncomment the appropriate list of options available in the virtuoso.ini file. Save the settings and close the file

- STEP 5: Stop the virtuoso server if it is already running by searching for the virtuoso server as "ps -ef | grep virtuoso-t". Start the virtuoso server with the updated settings in virtuoso.ini

```sudo /usr/bin/virtuoso-t```

- STEP 6: In a new terminal tab, start the virtuoso client

```sudo /usr/bin/isql-vt```

- STEP 7: In the SQL prompt, issue the following sparql command to load the triples from the ttl files into the virtuoso server

```OpenLink Interactive SQL (Virtuoso), version 0.9849b.
Type HELP; for help and EXIT; to exit.
SQL> ld_dir('/home/rudik/Documents/dbpedia_ttls', '*.ttl', 'http://dbpedia.org');
Connected to OpenLink Virtuoso
Driver: 06.01.3127 OpenLink Virtuoso ODBC Driver

Done. -- 17 msec.
SQL> rdf_loader_run();
```

Once the above steps are completed, the loader finishes and you are set to issue sparql queries in the client SQL terminal.


# RuDiK APIs

The APIs we currently expose are defined in the interface `asu.edu.rule_miner.rudik.rule_generator.HornRuleDiscoveryInterface`. The current implementation, explained in details in the technical report, is defined in the class `asu.edu.rule_miner.rudik.rule_generatorDynamicPruningRuleDiscovery`

# Running RuDiK using Gradle
The rule and instance generation can also be done using the [Gradle](https://gradle.org) build tool.
To get a list of all defined tasks, simply run `./gradlew tasks` in the root folder of the project.
Among many other tasks the task group `RuDik tasks` should also be displayed.

    RuDik tasks
    -----------
    generate_instances - Generates instances of RuDik rules
    generate_rules - Generates RuDik rules

These two tasks can be used to generate rules and corresponding instances.

## Setting up the backend
Since the generated rules and their instances are stored in a MongoDB, it is necessary to configure the connection parameters of the MongoDB before executing the tasks.
For this purpose, a file named backend.xml must be created in the [/src/main/config](https://github.com/milost/rudik/tree/develop/src/main/config) folder.
The folder already contains a preconfigured file ([`backend.default.xml`](https://github.com/milost/rudik/blob/develop/src/main/config/backend.default.xml)) with which it is possible to connect to a locally installed MongoDB instance.
**ATTENTION**: It is mandatory that the file has the name `backend.xml`.
If it is necessary to connect to a remote MongoDB instance, the commented parameters can be used.
An explanation of the individual connection parameters can be found [here](https://docs.mongodb.com/manual/reference/connection-string/)

As soon as the MongoDB configuration has been completed, the generation process can be started.

# Rule Generation
The rule generation can be started by executing the following command.

    ./gradlew generate_rules --args="[predicate_file_path] [rudik_configuration_file]"
    
Here the predicate file contains a list of predicates for which rules should be discovered.
[dbpedia_predicates.txt](https://github.com/milost/rudik/blob/develop/src/main/resources/dbpedia_predicates.txt) is an example of such a file.
The RuDik configuration file is the configuration file of RuDik. One such file is the [DbpediaConfiguration.xml](https://github.com/milost/rudik/blob/develop/src/main/config/DbpediaConfiguration.xml).
These two files can be used to start the rule generation process as follows:

    ./gradlew generate_rules --args="src/main/resources/dbpedia_predicates.txt src/main/config/DbpediaConfiguration.xml"

After the command has been issued, the rule generation should start. The discovered rules are written into the MongoDB for later use.

# Instance Generation
After some rules have been discovered, concrete instances can also be generated for these rules.
Instances are created for all rules in the MongoDB.
The process can be started as follows:

    ./gradlew generate_instances --args="[max_Instances] [rudik_configuration_file]"

Here, the `max_instances` parameter specifies the maximum number of instances to be generated for a rule.
As in the rule generation case, the rudik_configuration_file` specifies the RuDik configuration file to be used.
With these two specifications, the instance generation process can be started as follows.

    ./gradlew generate_instances --args="1000 src/main/config/DbpediaConfiguration.xml"

The generated instances are persisted in the MongoDB just like the rules before.

# Contacts

1. [Stefano Ortona](mailto:stefano.ortona@gmail.com)
2. [Paolo Papotti](mailto:change-with.last-name_@eurecom.fr)
3. [Vamsi Meduri](mailto.vamsikrishna1902@gmail.com)
