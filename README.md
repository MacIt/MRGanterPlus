# MRGanter+: Distributed FCA Algorithm Based on Twister #

<br>Release 1.3
<br>By Biao Xu at TSSG, WIT

## Intoduction##

MRGanter+is a distributed Formal Concept Analysis algorithm based on Gater's algorithm (as known as NextClosure) and an iterative MapReduce framework, [Twister](http://www.iterativemapreduce.org/).
NextClosure calculates closures in lectic ordering to ensure every concept appears only once. This approach allows a single concept to be tested with the closure validation condition during each iteration. This is efficient when the algorithm runs on a single machine. For multi-machine computation, the extra computation and redundancy resulting from keeping  only one concept after each iteration across many machines is costly. We modify NextClosure to reduce the number of iterations and name the corresponding distributed algorithm MRGanter+.

Rather than using redundancy checking, we keep as many closures as possible in each iteration; All closures are maintained and used to generate the next batch of closures. MRGanter+ has a Map method which calculates local concepts by working on previous concept and local data partition. The Reduce method in MRGanter+ merges local closures first, and then recursively examines if they already exist in the set of global formal concepts H. The set H is used to fast index and search a specified closure; it is designed as a two-level hash table to reduce its costs. The first level is indexed by the head attribute of the closure, while the second level is indexed by the length of the closure.

For more details about MRGanter+, please see our recent publication: 
	
	Distributed Formal Concept Analysis Algorithms Based on an Iterative MapReduce Framework:
	@incollection{,
	year={2012},
	isbn={978-3-642-29891-2},
	booktitle={Formal Concept Analysis},
	volume={7278},
	series={Lecture Notes in Computer Science},
	editor={Domenach, Florent and Ignatov, DmitryI. and Poelmans, Jonas},
	doi={10.1007/978-3-642-29892-9_26},
	title={Distributed Formal Concept Analysis Algorithms Based on an Iterative MapReduce Framework},
	url={http://dx.doi.org/10.1007/978-3-642-29892-9_26},
	publisher={Springer Berlin Heidelberg},
	keywords={Formal Concept Analysis; Distributed Mining; MapReduce},
	author={Xu, Biao and Fréin, Ruairí and Robson, Eric and Ó Foghlú, Mícheál},
	pages={292-308}
	}

at http://www.springerlink.com/content/02p8282703rx0m78/. And you can download the author's self-archiving version of this publication from http://arxiv.org/abs/1210.2401.
<br>To know how MapReduce works please refer to "MapReduce Simplified Data Processing on Large Clusters" at http://static.usenix.org/event/osdi04/tech/full_papers/dean/dean.pdf.
<br>Or you can find a presentaion introducing MRGanter+ which we made on ICFCA 2012: http://www.econ.kuleuven.be/ICFCA/present/Biao_Xu.pdf

## Sample Data ##

To be used by MRGanter+, we patition a single file data in a horizontal way and specify their number of objects/records and the number of attributes. Below is a simple example.

* Data partition 1

		3 
		7 
		1 1 0 1 0 1 0 
		1 0 1 0 1 0 1 
		0 1 1 1 0 1 1 

* Data partition 2

		3 
		7 
		0 1 0 1 1 0 0 
		1 0 0 1 1 1 0 
		0 1 1 0 0 1 1 

For testing purpose, we attached the well-known mushroom data, which is partitioned to 2 files, with source file. Appearently, you need to send each of them to a node when you have 2 nodes.

## Requirments ##

This algorithm implementation is tested with Twister 0.8 which requires Linux operating system. To compile MRGanterPlus some Java libraries are needed.

		Twister-0.8.jar
		NaradaBrokering.jar
		jug-asl-2.0.0
		jug-uuid.jar
		junit.jar

## Installation and Configuration ##

Few steps are needed in order to run MRGanter+ on your machine(s).

1. Ensure you are using Linux system or Mac OS. We tested MRGanter+ on Ubuntu 11.04.
2. Enable the ssh on your system and it should be configured to be connecting between machines without password. A sample solution can be viewed at http://www.linuxquestions.org/questions/linux-newbie-8/ssh-with-password-533684/.
	For the case of single machine, eventaully you should be able to access itself by "ssh localhost". Don't forget to install ssh-server, otherwise you will get an error like this: "ssh: connect to host localhost port 22: Connection refused".
3. Set up NaradaBrokering
<br>Before Twister installation, first download NaradaBrokering and unzip it to another directory. We call this $NBHOME. Configure this environment variable in /etc/environment by adding: NBHOME="/home/username/NaradaBrokering-4.2.2". The official tutorial of Twister suggests you to configure this in .bashrc file. However it turns out to be incorrect for Ubuntu system.
4. Setting up Twister
<br>Unzip the Twister.zip file to some directory. Then set environment variable named TWISTER_HOME pointing to this directory. As we did to NaradaBrokering, you should also add the following line to /etc/environment.
		
		TWISTER_HOME ="/home/username/twister-0.8"
<br>Now you need to set few configuration parameters as follows.
	* Edit `$TWISTER_HOME/bin/twister.properties` file as follows:
	<br>nodes_file = /home/username/twister-0.8/bin/nodes
	<br>daemons_per_node = 1
	<br>workers_per_daemon = 2
	<br>`app_dir = /home/username/twister-0.8/apps`
	<br>`data_dir = /home/username/twister-0.8/data`
	* Edit `$TWISTER_HOME/bin/nb.properties` file and set broker_host to the IP of the machine where you setup NaradaBrokering. They are could be the same if you like.
	+ Edit `$TWISTER_HOME/bin/nodes` file and add your local IP address to this file. In a single machine setup you should have only one IP address in this file, in the cluster setup this file will have all the IP addresses of the compute nodes.
5. JVM
<br>By default, the shell files such as start_twister.sh and stop_twister.sh in `$TWISTER_HOME/bin` are not runnable. You need to  change their properties as by choosing "Allow executing file as program" (on Ubuntu). 
<br>Also, you have to adjust the JVM parameters for Twister runtime. Edit `$TWISTER_HOME/bin/stimr.sh` file, and change `-Xmx40000m` to whatever value suits your situation.

## How to Run ##

1. Export MRGanter+ and everything in the directory as runnable JAR file. If you use eclipse, please select "Package required libraries into generated JAR" for Library handling option. Of course you could use `MRGanter+.jar` file coming with source code. Now Place `MRGanter+.jar` to `$TWISTER_HOME/apps`.
2. Start NaradaBrokering by running startbr.sh at its home directory.
3. Start Twister runtime by running:
		`$TWISTER_HOME/bin/start_twister.sh`
	Note that, "sh start_twister.sh" will not work.
4. Run `$TWISTER_HOME/bin/create_partition_file.sh` to create partition files for testing datasets. Take mushtoom for example,

		./create_partition_file.sh ../data mushroom01 ../data/mushroom.pf

	Note that, you need to change the filter (the middle parameter) to mushroom if you want to create partition files for many data partitions.

	Now you are ready to run MRGanter+. Go to `$TWISTER_HOME/apps/` and execute `run_MRGanter+.sh` in below way:

		./run_MRGanter+.sh 1 ../data/mushroom.pf

	For more details of parameters you can see the source code.

## More Resource ##

For the details of Twister, go to http://www.iterativemapreduce.org/#Papers_and_Presentation for the paper:
> Jaliya Ekanayake, Hui Li, Bingjing Zhang, Thilina Gunarathne, Seung-Hee Bae, Judy Qiu, Geoffrey Fox, Twister: A Runtime for Iterative MapReduce," The First International Workshop on MapReduce and its Applications (MAPREDUCE'10) - HPDC2010


More information about setting up Twister environment, see 
+ http://www.iterativemapreduce.org/userguide.html
+ http://salsahpc.indiana.edu/content/programming-iterative-mapreduce-applications-using-twister
+ http://salsahpc.indiana.edu/tutorial/twister_install.htm

For the use of Twister API, please refer to http://www.iterativemapreduce.org/docs/index.html

## Feedback ##

You are welcome to give any feedback and contributes. We're glad to hear from you about any bug. 
<br>You can reach me by softwind.xb@gmail.com.