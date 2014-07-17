RepIRProx
=========

Master pom for RepIR RepIRTools RepIRApps and RepIRProximity, to reproduce the results for "Distance Matters!"

Requirements:
- Hadoop cluster (tested with hadoop-cdh3u6)
- Collections TREC1-8 adhoc, ClueWeb09, ClueWeb12
- Topic and Qrels files for TREC1-8, WebTrack 2009-2013 (on TREC site)
- Maven

Assuming you are setting up on a gateway server, from which you can directly access the Hadoop cluster.

Getting started:

1. Download rr.tar.gz, and unpack this using "tar -zxvf rr.tar.gz", which gives you a directory rr in which RepIR stores its files on the gateway.
2. Download pom.xml of RepIRProx. This is the master pom, that should allow easy download and build of the entire project. Release 0.23 was used for the paper, and is available at maven central, or take the latest snapshot from this repo.
