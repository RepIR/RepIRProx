RepIRProx
=========

Master pom for RepIR RepIRTools RepIRApps and RepIRProximity, to reproduce the results for "Distance Matters!"

Requirements:
- Hadoop cluster (tested with hadoop-cdh3u6)
- Collections TREC1-8 adhoc, ClueWeb09, ClueWeb12
- Topic and Qrels files for TREC1-8, WebTrack 2009-2013 (on TREC site)
- Maven

The results can either be replicated using version 0.23, which can be downloaded along with the sources from Maven Central. Here, we will discuss an easier setup using the latest snapshot version from trunk. We also assume you are setting up on a gateway server, from which you can directly access the Hadoop cluster.

Getting started:

1. Do not create the directories yet, but below the getting started is a tree that indicates what goes where. 
2. In current folder of your choice `.`, clone the repositories for $FILE={RepIRProx, RepIRTools, RepIR, RepIRApps, RepIRProximity} with `git clone git://github.com/RepIR/$FILE.git`.
3. `tar -zxvf RepIRProx/rr.tar.gz`, which gives you a directory `rr` in which RepIR stores the files used on the gateway.
4. `tar -zxvf RepIRProx/bin.tar.gz`, in some place that you include in your $PATH. You mostly need `rr` and `rrconfig` to launch MR jobs with RepIR, but there are some other tools included for your convenience and as examples.
5. Let $VERSION be the version of RepIR you cloned. Then to build the jars:
   - Open `RepIRProx/pom.xml`, and look for `hadoop.core`. Modify the version to whatever version of Hadoop you use.
   - `cd RepIRProx` which contains the parent pom.
   - `mvn install`
   - `mvn dependency:copy-dependencies -DoutputDirectory=../rr/lib/libs`
   - `cd ..`
   - for $FILE={RepIRTools, RepIR, RepIRApps, RepIRProximity}
      - `cp $FILE/target/*.jar ../rr/lib/$VERSION`
6. Adjust the configuration to your environment. Open `rrconfig` in the `bin` directory of step 5. 
  - `rrdir` must point to wherever you placed the rr folder in step 4. 
  - `rrversion` must contain the exact version of RepIR you installed. 
  - `CLASSPATH` must contain the folders with the RepIR jars, jars for the dependencies, hadoop, hadoop configuration xml's   - Note: The tools in the bin are made for MR. If you use MR2 (yarn), in `hd` you can replace `hadoop` by `yarn`, and in `dfs` you can replace `hd` by `hdfs` (not tested). 
7. Open `rr/settings/clustersettings`. In the bottom section, `rr.lib` must contain all jars that are required for MapReduce jobs: by default the RepIR jars, antlr and backport. If you followed the defaults, the configuration is ok, otherwise chnage these to your environment. In the top section, `cluster.nodes` is actually the number of slots the cluster has for jobs (this name may change later). `retriever.tempdir` should point to a correct folder on HDFS that can be used to store temorary files. The configuration files can include any Hadoop configuration settings, so if your cluster requires you to submit to a queue you can include that here.
8. To test, we will setup the `tiny` repository. Located in RepIRProx is `enwiki.trec`, which is a TREC format archive of the first ~1300 Wikipedia pages. Copy this file to HDFS, for instance in the folder `input/tiny/enwiki.trec`. Use your own hadoop skills, or `dfs -mkdir input/tiny` and `hdput enwiki.trec input/tiny`.
9. Modify `rr/settings/tiny` to your envirnoment. `repository.inputdir` points to the folder of `enwiki.trec` on HDFS, and `repository.dir` points to the location were the repository is created. The root folder should exists, so if you store it in output/tiny, create it with `dfs -mkdir output/tiny`.
10. Create the dictionary features using `hdvoc tiny`. Note that most supplied RepIR tools take the configuration file (which are in rr/settings) as the first parameter. If all goes well, a Hadoop job is launched, and when it finished successful it has created output/tiny/tiny.master and a folder output/tiny/repository with feature files inside (on HDFS). You can view the tiny.master, it is plain text and contains settings used during creation and some corpus statistics. The stored feature files in the repository folder are usually write-once-read-many, in a structured binary file format, and should only be accessed ny obtaining the corresponding feature from the Repository (which is really a breeze, since Repository only needs the name of the configfile and getFeature only needs the classname which is the second part of the filename of a feature).
11. Now create the index, using `hdindex tiny`. After the Hadoop job finishes successfully, index features have been added to the repository.
12. Now you are ready to try it out. For tiny repositories, there is a fast retrieval tool `qq` for test purposes that does not use MapReduce. `qq tiny Albert Einstein`, by default this returns the top-10 documents using a Dirichlet smoothed Language Model.
13. For IR experiments, we typically run test sets over MR. Please ignore that for the `tiny` example the Hadoop overhead is rediculous compared to stand-alone retrieval, we have created a tiny example topics and qrels file in `rr/adhoc` to try it out. These are flat text files, but to ensure you have correctly modified the configuration files to your local settings, you can view the topics with `showtopics tiny`. When that works, you can run the set of topics on the cluster with `testset tiny kld`. When the job is finished you should have a results file `rr/eval/tiny.kld`. You can compute the map score for the results file `map tiny kld`.

The structure of files could look like (after the build you can remove the git clones, and should be able to relocate bin and rr if you adjust the rr location in the configuration):
<pre>
+-- RepIRProx        master pom & resources 
+-- RepIR            core lib
+-- RepIRTools       low level tools used by RepIR
+-- RepIRApps        end user tools and apps
+-- RepIRProximity   implementations of proximity models for "Distance Matters!"
+-- rr               
|   +-- settings     configuration files
|   +-- adhoc        topics and qrels
|   +-- eval         result files for test batches
|   +-- lib          jar files
|       +-- $VERSION RepIR jar files
|       +-- libs     jars of dependencies
+-- bin              command line tools to operate RepIR
</pre>

When successful, you can try other test collections. `rr/settings` contains the settings we used for TREC1-8 and WebTrack 2009-2013, but you need to supply the collections yourself and download the topic and qrel files. You will have to modify the configuration files to your environment. To replicate the exact results from our study, you can setup variants of configuration files like `trec1sdm` which have a corresponding file in `rr/settings/tuned` that contains the parameter settings obtained through cross validation (see paper).

Note: For large repositories with many partitions (e.g. ClueWeb), you should create the `PartitionLocation` feature (RepIR calls everything a feature), which when available avoids costly lookup of the location of files on HDFS, greatly reducing the startup time of jobs. For `tiny`, you do not need that, but you may get warnings that it does not exists. You can create it with `hdploc tiny`.
