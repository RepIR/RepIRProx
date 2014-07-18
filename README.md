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

1. In a folder, we will clone the RepIR repository, to build the required jar files. In a folder of your choosing, download the parent pom with resource `git clone git@github.com:RepIR/RepIRProx.git`.
2. Do not create the directories yet, but in the end the directory structure will look like:
2. Open `RepIRProx/pom.xml`, and look for `hadoop.core`. Modify the version to whatever version of Hadoop you use. 
2. Download `rr.tar.gz`, and unpack this using `tar -zxvf rr.tar.gz`, which gives you a directory `rr` in which RepIR stores its files on the gateway.
2. Download `bin.tar.gz`, and unpack this using `tar -zxvf bin.tar.gz`, in some place that you include in your $PATH. You mostly need `rr` and `rrconfig` to launch MR jobs, but there are a few additional tools included.
3. The tools in the bin are made for MR. If you use MR2, in `hd` you probably want to replace `hadoop` by `yarn`, and in `dfs` you replace `hd` by `hdfs`. 
4. Now we will download the RepIR libraries with git, and use maven to compile, and copy the jars RepIRTools, RepIR, RepIRApps and RepIRProximity to `rr/lib/$VERSION` and the jars of the dependencies in `rr/lib/libs`. Let $VERSION be the latest RepIR version in trunk, then repeat the following for `$FILE = {RepIRTools, RepIR, RepIRApps, RepIRProximity}`:
  - `git clone git@github.com:RepIR/$FILE.git`
  - `cd $FILE`
  - `mvn install`
  - `cp target/$FILE-$VERSION.jar ../rr/lib/$VERSION/$FILE-$VERSION.jar`
  - `mvn dependency:copy-dependencies -DoutputDirectory=../rr/lib/libs`
5. Now we are going to adjust the configuration to your environment. Open `rrconfig` in the `bin` directory of step 2. 
  - `rrdir` must point to wherever you placed the rr folder in step 3. 
  - `rrversion` must contain the exact version of RepIR you installed. 
  - `CLASSPATH` must contain the folders with the RepIR jars, jars for the dependencies, hadoop, hadoop configuration xml's and hadoop's dependent libraries. Change these to whatever is correct for you environment.
6. Open `rr/settings/clustersettings`. In the bottom section, `rr.lib` must contain all jars that are required for MapReduce jobs: by default the RepIR jars, antlr and backport. If you followed the defaults, the configuration is ok, otherwise chnage these to your environment. In the top section, `cluster.nodes` is actually the number of slots the cluster has for jobs (this name may change later). `retriever.tempdir` should point to a correct folder on HDFS that can be used to store temorary files. The configuration files can include any Hadoop configuration settings, so if your cluster requires you to submit to a queue you can include that here.
7. To test, we will setup the `tiny` repository. Download `enwiki.trec`, which is a TREC format archive of the first ~1300 Wikipedia pages. Copy this file to HDFS, for instance in the folder `input/tiny/enwiki.trec`. `dfs -mkdir input/tiny` and `hdput enwiki.trec input/tiny`.
8. Modify `rr/settings/tiny` to your envirnoment. `repository.inputdir` points to the folder of `enwiki.trec` on HDFS, and `repository.dir` points to the location were the repository is created. The root folder should exists, so if you store it in output/tiny, create it with `dfs -mkdir output/tiny`.
9. We are going to create a repository with features that are useful for Information Retrieval type tasks. RepIRApps contains standard programs for standard tasks. From this, we use the vocabulary and repository builder. We first need to extract a dictionary using `hdvoc tiny`. Note that most supplied apps take the configuration file as the first parameter. If all goes well, a Hadoop job is launched, and when it finished successful it has created output/tiny/tiny.master and a folder output/tiny/repository with feature files inside. You can view the tiny.master, it is plain text and contains settings used during creation and soem corpus statistics. The stored feature files in the repository folder are usually write once read many, in a structured binary file format, and should only be accessed ny obtaining the corresponding feature from the Repository (which is really a breeze, since Repository only needs the name of the configfile and getFeature only needs the classname which is the second part of the filename of a feature).
10. Now create the index, using `hdindex tiny`. After the Hadoop job finishes successfully, there will be features added to the repository.
11. Now you are ready to try it out. For tiny repositories, there is a fast retrieval tool for test purposes that does not use MapReduce. `qq tiny Albert Einstein`, by default returns the top-10 documents using a Dirichlet smoothed Language Model.
12. For IR experiments, you can easily execute test sets over MR. For the tiny collection, we have created am example topics and qrels file in `rr/adhoc`. These are flat text files, but to ensure you have correctly modified the configuration files to your local settings, you can view the topics with `showtopics tiny`. When that works, you can run the set of topics on the cluster `testset tiny kld`. When the job is finished you should have a results file `rr/eval/tiny.kld`. You can compute the map score for the results file `map tiny kld`.  

The structure of files could look like:
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

If this was successful, you can try other test collections. `rr/settings` contains the settings we used for TREC1-8 and WebTrack 2009-2013, but you need to supply the collection and download the topic and qrel files yourself. You will have to modify the configuration files for any environment dependent settings. To replicate the exact results from our study, you can setup variants of configuration files like `trec1sdm` which have a corresponding file in `rr/settings/tuned` that contains the parameter settings obtained through cross validation (see paper).

Note: For large repositories with many partitions (e.g. Web crawls), you should create a feature `PartitionLocation` (RepIR calls everything stored a feature), which when available avoids costly lookup of the location of files on HDFS, greatly reducing the startup time of jobs. For `tiny`, you do not need that, but you may get warnings that it does not exists. You can create it with `hdploc tiny`.
