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

1. Download `rr.tar.gz`, and unpack this using `tar -zxvf rr.tar.gz`, which gives you a directory `rr` in which RepIR stores its files on the gateway.
2. Download `bin.tar.gz`, and unpack this using `tar -zxvf bin.tar.gz`, in some place that you include in your $PATH. You mostly need `rr` and `rrconfig` to launch MR jobs, but there are a few additional tools included.
3. Now we will download the RepIR libraries with git, and use maven to compile, and copy the jars RepIRTools, RepIR, RepIRApps and RepIRProximity to rr/lib/$VERSION and the jars of the dependencies in rr/lib/libs. Let $VERSION be the latest RepIR version in trunk, then repeat the following for $FILE = {RepIRTools, RepIR, RepIRApps, RepIRProximity}:
  - git clone git@github.com:RepIR/$FILE.git
  - cd $FILE
  - mvn install
  - cp target/$FILE-$VERSION.jar ../rr/lib/$VERSION/$FILE-$VERSION.jar
  - mvn dependency:copy-dependencies -DoutputDirectory=../rr/lib/libs
4. Now we are going to adjust the configuration to your environment. Open rrconfig in the bin directory of step 2. rrdir must point to wherever you placed the rr folder in step 3. rrversion must contain the exact version of RepIR you installed. CLASSPATH must contain the folders with the RepIR jars, jars for the dependencies, hadoop, hadoop configuration xml's and hadoop's dependent libraries. Change these to whatever is correct for you environment.
5. Open rr/settings/clustersettings. In the bottom section, rr.lib must contain all jars that are required for MapReduce jobs: by default the RepIR jars, antlr and backport. These dirs must check out. In the top section, cluster.nodes is actually the number of slots the cluster has for jobs (this name may change later). The configuration files can include any Hadoop configuration settings, so if your cluster requires you to submit to a queue you can include this here.
