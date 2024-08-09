# Hands-on Shared Classes Cache Lab

In this lab, you will learn the benefits of shared classes case technology and how using Semeru Runtimes can help you run workloads that start 50% faster and consume 50% less memory.

This lab is divided into 5 sections that each explore a different topic.

The first two sections look at using the Open Liberty web server with Semeru
Runtimes to see the impact of using shared class cache or not.

The following three sections switch gears to look at the Apache Tomcat v10
server to measure the startup and memory use for both the [Eclipse Temurin JDK](https://adoptium.net/temurin/) - which is the JDK used in the official [Tomcat containers](`https://hub.docker.com/_/tomcat`), as well as the IBM Semeru Runtimes JDK.

# Steps:

- [Hands-on Shared Classes Cache Lab](#hands-on-shared-classes-cache-lab)
- [Steps:](#steps)
	- [1. Initial lab setup](#1-initial-lab-setup)
	- [2. Open Liberty without Shared Classes Cache](#2-open-liberty-without-shared-classes-cache)
		- [Summary](#summary)
	- [3. Open Liberty with Shared Classes Cache](#3-open-liberty-with-shared-classes-cache)
		- [Summary](#summary-1)
	- [4. Apache Tomcat with Eclipse Temurin JDK](#4-apache-tomcat-with-eclipse-temurin-jdk)
		- [Summary](#summary-2)
	- [5. Apache Tomcat with IBM Semeru Runtimes](#5-apache-tomcat-with-ibm-semeru-runtimes)
		- [Summary](#summary-3)
	- [6. Apache Tomcat with IBM Semeru Runtimes and Shared Classes Cache](#6-apache-tomcat-with-ibm-semeru-runtimes-and-shared-classes-cache)
		- [Summary](#summary-4)
	- [Conclusion](#conclusion)

## 1. Initial lab setup

We will run the entire lab out of one redhat/ubi9 container that preinstalls the required software.

To create the workshop container, run the following command:

```bash
cd LabSharedCache/techxchange-shared-cache-lab
./main.build.sh
```

Although it's not recommended as common practice, this lab will run more
smoothly for you if you invoke it as root. The safest way to do that is by
running a command like this one so that you can't accidentally leave a terminal window on your host with root permission lying around:

```bash
sudo ./main.run.sh
```

> **NOTE**: You can peek inside that script to see that it just runs the following command:
>```bash
>podman run --network=host --privileged --name=workshop-main --replace -it workshop/main /bin/bash
>```

## 2. Open Liberty without Shared Classes Cache

In this section of the lab, we'll set up a Liberty container with the "Getting Started" application and take some rough measurements of the start-up time and memory usage for this container. In this section, we will be intentially not using the Semeru Runtimes Shared Classes Cache to get an idea what the baseline is (note that this is not the default Liberty configuration, which automatically prepopulates a shared classes cache; we'll be trying that out in <b>Section_2</b>).

First, go to the directory for Section 1:

```bash
cd /Workshop_SharedCache/Section_1
```

Complete the following steps:

1. Build the getting started application in this directory:

	```bash
	mvn package
	```

2. Build an OpenLiberty container that contains the getting started application. There is a `Dockerfile.liberty_noscc` file provided that specifically disables the Semeru Runtimes shared classes cache when this container is built. Run the following command:

	```bash
	podman build --network=host -f Dockerfile.liberty_noscc -t liberty_noscc .
	```

3. Run the container. It will automatically start the OpenLiberty server with the loaded getting started application. Run the following commadn and wait for the server to start:

	```bash
	podman run -p 9080:9080 --name=liberty_noscc --replace -it liberty_noscc
	```

	Look for the elapsed time to start the server. You'll see a line that ends with something like:: `The defaultServer server started in 3.043 seconds.`

4. Go to another terminal window and log into the workshop container. Run the following command in another terminal window:

	```bash
	podman exec --privileged -it workshop-main /bin/bash
	```

	This will connect to the running workshop container so that you can run another command there while the Liberty server is running.

5. Use podman stats to observe the memory use of the container.

	```bash
	podman stats
	```

	This command shows various statistics about all containers running within the main workshop container. For example, you should see something like:

	```bash
	ID            NAME               CPU %       MEM USAGE / LIMIT  MEM %       NET IO      BLOCK IO    PIDS        CPU TIME    AVG CPU %
	b814b591a1da  liberty_noscc      3.44%       131MB / 2.047GB    6.40%       0B / 0B     0B / 0B     60          9.040596s   33.61%
	```

	This shows the Liberty server you started in Step 3 running with 131MB of memory. You can leave this podman stats command running for Step 7; it will update the list of active containers about every 5 seconds.

	>**IMPORTANT**: Leave this `podman stats` command running for the other parts of this lab so you can keep watching the statistics for the containers you use.

6. Hit `control-c` to stop the server that you started in Step 3.

  	**Optional**: Start and stop the server (Step 3 and 6) a few times to get a feeling for how the startup time and memory consumption varies in different server instances.

	You won't see exactly the same time and memory usage in different runs, but the server startup time usually falls within a few tenths of a second and the memory usage is typically within a few MB.

### Summary

This completes this part of the lab! Move on to the next Section to see how fast Open Liberty will load this application when the shared cache has been created!

## 3. Open Liberty with Shared Classes Cache

In this section of the lab, we'll be running the Liberty server in its default mode to run the Getting Started application. We'll find that the server starts much faster (in a little more than half the time) and can use less memory because the shared classes cache does not need to remain resident.

First, go to the directory for Section 2:

```bash
cd ../Section_2
```

Complete the following steps:

1. Build the getting started application in this directory:

	```bash
	mvn package
	```

2. Build an OpenLiberty container that contains the getting started application. There is a `Dockerfile.liberty_scc` file provided that enables and populates the Semeru Runtimes shared classes cache automatically when this container is built. Run the following command:

	```bash
	podman build --network=host -f Dockerfile.liberty_scc -t liberty_scc .
	```

3. Run the container. It will automatically start the OpenLiberty server with the loaded getting started application. Run the following command and wait for the server to start:

	```bash
	podman run --network=host --name=liberty_scc --replace -it liberty_scc
	```

	Look for the elapsed time to start the server. You'll see a line that ends with something like: `The defaultServer server started in 1.702 seconds.`

	Comparing to the Liberty server we started in <b>Section_1</b>, this server using the prepopulated shared classes cache starts in about 55% of the time (1.702 seconds versus 3.043 seconds), a dramatic improvement!

4. Use the `podman stats` window from <b>Section_1</b> to observe the memory use of the container.

	You should see something like:
	
	```
	ID NAME CPU % MEM USAGE / LIMIT MEM % NET IO BLOCK IO PIDS CPU TIME AVG CPU % f6ecefd2dace liberty_scc 3.51% 97.9MB / 2.047GB 4.78% 0B / 0B 0B / 0B 59 4.626379s 15.75%
	```

	This shows the Liberty server you started in Step 3 running with 98MB of memory. The memory use is lower because the shared memory used by the shared cache can be shared by multiple instances and so isn't counted as part of the memory use for the container. If you were to start a second, third, fourth, etc. server on the same machine connected to the same shared cache, there would be only one copy of the cache loaded in memory. Although it doesn't happen in this lab, it is pretty common in production deployments for multiple servers to be running on the same physical node so these savings are real even though completely subtracting it from the memory usage seems overboard especially with only one server running.

	>**IMPORTANT**: Leave the `podman stats` command running for the other parts of this lab so you can keep watching the statistics for the containers you use.

5. Hit `control-c` to stop the server that you started in Step 3. 
   
   **Optional**: Start and stop the server (Steps 3 and 5) a few times to get a feeling for how the startup time and memory consumption varies in different server instances.

	You won't see exactly the same time and memory usage in different runs, but the server startup time usually falls within a few tenths of a second and the memory usage is typically within a few MB.

### Summary

This completes the second section in the lab! Move on to the next section to see if we can use the Semeru Runtimes Shared Classes Cache to help another server to start faster!

## 4. Apache Tomcat with Eclipse Temurin JDK

In this section of the lab, we're going to establish the baseline startup time and memory usage for a different server technology: Apache Tomcat. We're also going to switch temporary away from IBM Semeru Runtimes to get a feel for how performance varies with different JDKs.

We'll be running a different application than the "Getting Started" application used in <b>Section_1</b> and <b>Section_2</b>, and tomcat does not include all the same kinds of support that the Liberty server does. For these reasons, and because tomcat is an extremely lightweight server, we'll see it will start a bit faster than Liberty and it will be using less memory after startup. In later sections of the lab, we'll further improve on these numbers by using IBM Semeru Runtimes and its shared classes cache technology.

We'll be using Tomcat v10 to stay with Java 17 consistently through the rest of the labs.

First, go to the directory for Section 3:

```bash
cd ../Section_3
```

Complete the following steps:

1. We'll be starting with the pre-built `sample.war` file provided by the tomcat community.

	Build a tomcat container that contains the sample application. There is a `Dockerfile.tomcat_temurin` file provided that copies the `sample.war` file into a standard Tomcat container that is built using the Eclipse Temurin JDK distribution. Eclipse Temurin is a straight JDK built from the source code released by the OpenJDK project (so it uses the HotSpot JVM).

	```bash
	podman build --network=host -f Dockerfile.tomcat_temurin -t tomcat_temurin
	```

2. Run the container. It will automatically start the Tomcat server with the sample application.

	```bash
	podman run --cpus=1 --network=host --name=tomcat_temurin --replace -it tomcat_temurin > log.cpu1
	```

	This starts tomcat using only a single CPU core and stores the output into the file `log.cpu1`.

3. Use the `podman stats` window from <b>Section_1</b> to observe the memory use of the container.

	```
	ID            NAME               CPU %       MEM USAGE / LIMIT  MEM %       NET IO      BLOCK IO    PIDS        CPU TIME    AVG CPU %
	671f11639e46  tomcat_temurin     0.32%       67.28MB / 2.047GB  3.29%       0B / 0B          0B / 0B     31          1.25609s    7.83%
	```

	which shows the Tomcat server you started in Step 2 running with 67MB of memory. Tomcat is a lightweight server but less capabilities are loaded here than into the Liberty server you started in Sections 1 and 2, which explains at least part of the different in memory usage.

	>**IMPORTANT**: Leave the `podman stats` command running for the other parts of this lab so you can keep watching the statistics for the containers you use.

4. Press `control-c` to stop the server that you started in Step 2. At this point, we can run the `startupTime.awk` script on the log to calculate the time it took from initiating the java command line from catalina.sh until the server posted its "Server started" message.

	```bash
	./startTime.awk log.cpu1
	Server initiated 1715355700320533504, up at 1715355701473000000
	Full start time is 1152.47 ms
	```

	So the server started in roughly 1.15 seconds.

	You can start the server a few more times, capturing the output to different log files so that you can get the start time for several runs, just to see the variation you experience. We aren't going to do a rigourous statistical analysis for this lab but that would obviously be important if you were going to start measuring server start time for your production workloads.

	You can also run with more cores just to see how the start time changes. For example:

	```bash
	podman run --cpus=2 --network=host --name=tomcat_temurin --replace -it tomcat_temurin > log.cpu2
	```

	**NOTE**: Hit `control-c` after checking the memory consumption in the stats output.

	```bash
	./startTime.awk log.cpu2
	Server initiated 1715356179088315648, up at 1715356179722000000
	Full start time is 633.684 ms
	```

	As you can see, adding a second core helps reduce the startup time to 633ms, but you may also find that the memory consumption increases a bit (perhaps to 73MB).

### Summary

You should have measured performance somewhat along these lines:

JDK Core limit Start time Memory usage after start Temurin 1 core 1152.47ms 68MB Temurin 2 cores 633.684ms 73MB

Move on to the next section to see what happens when we move to an IBM Semeru Runtimes JDK to run the Tomcat server with the same sample application.

## 5. Apache Tomcat with IBM Semeru Runtimes

In this section of the lab, we're going to try running Apache Tomcat with a different JDK (IBM Semeru Runtimes) than the Eclipse Temurin JDK that comes pre-installed. We have a small challenge: Tomcat comes in a container and Semeru Runtimes also comes in a container. To resolve this dilemma, our Dockerfile will use a multi-stage build to copy the JDK from the Semeru Runtimes container into our new container that will be based on (FROM) the Tomcat container.

We'll be using Tomcat v10 to stay with Java 17 consistently through the entire lab. In this section of the lab, we'll also be initially disabling the Semeru Runtime Shared Classes Cache.

First, go to the directory for Section 4:

```bash
cd ../Section_4
```

Complete the following steps:

1. Build a tomcat container that contains the sample application. There is a `Dockerfile.tomcat_semeru.noscc` file provided that first copies the Semeru Runtimes JDK into the container and then copies the `sample.war` file. This Dockerfile also adds the command-line option `-Xshareclasses:none` which completely disables any class sharing for Semeru Runtimes.
   
	```bash
	podman build --network=host -f Dockerfile.tomcat_semeru.noscc -t tomcat_semeru.noscc
	```

2. Run the container. It will automatically start the Tomcat server with the sample application. Run the following command and wait for the server to start:

	```bash
	podman run --cpus=1 --network=host --name=tomcat_semeru.noscc --replace -it tomcat_semeru.noscc > log.cpu1
	```

	Just like in the previous section, this starts the server with 1 CPU core.

3. Use the `podman stats` window from <b>Section_1</b> to observe the memory use of the container.

	```
	ID            NAME                CPU %      MEM USAGE / LIMIT  MEM %       NET IO      BLOCK IO    PIDS        CPU TIME    AVG CPU %
	ef1846553ffb  tomcat_semeru.noscc 41.53%     41.73MB / 2.047GB  2.04%       0B / 0B     0B / 0B     35          2.011569s   41.53%
	```

	This shows the Tomcat server you started in Step 2 running with only 42MB of memory. Compared to starting Tomcat with Eclipse Temurin (with the HotSpot VM), when running this container uses about 40% less memory just by replacing the Temurin JDK with the Semeru Runtimes JDK.

	>**IMPORTANT**: Leave the `podman stats` command running for the other parts of this lab so you can keep watching the statistics for the containers you use.

	Another way to think about that is that starting tomcat with Temurin consumes 62% more memory than with Semeru Runtimes.

	But keep in mind this measurement is only after startup and does not necessarily mean that the server will continue to use less memory under load (although that has been the general finding). We can't really test under load with the sample application, though, so you'll have to investigate this aspect further on your own if you're interested.


4. Press `control-c` to stop the server that you started in Step 2. Once you stop the server, you can run the `startTime.awk` script to find out how long it took to start the server. Let's see what that looks like:

	```bash
	./startTime.awk log.cpu1
	Server initiated 1715356585620868352, up at 1715356587598000000
	Full start time is 1977.13 ms
	```

	This message shows the server started in just under 2 seconds, which is really a LOT longer than with Temurin! Don't fear, however, this isn't the best Semeru Runtimes can do.

6. Before we improve on that time, however, first start and stop the server a few times and use the `startTime.awk` script to see how reliable this start time is. You can also try running with 2 cores to see what that does: 
  
	```bash
	podman run --cpus=2 --network=host --name=tomcat_semeru.noscc --replace -it tomcat_semeru.noscc > log.cpu2
	```

	With 2 cores, the podman stats are: 
	```
	ID NAME CPU % MEM USAGE / LIMIT MEM % NET IO BLOCK IO PIDS CPU TIME AVG CPU % 
	d6c64ea6596d tomcat_semeru.noscc 0.22% 42.09MB / 2.047GB 2.06% 0B / 0B 0B / 0B 36 2.186217s 11.56%
	```

	So not much change in memory usage, at least. How about the start time? 
		```bash
		./startTime.awk log.cpu2
		Server initiated 1715357245252879360, up at 1715357246401000000 Full start time is 1148.12 ms
		```

	Well, it improved dramatically, as you'd expect, and it's not as much behind Temurin as it was on one core. But it's still very far behind. Don't give up yet! In the next section, Semeru Runtimes will improve dramatically!

### Summary

In this section we added performance results like these:

```bash
JDK Core limit Start time Memory usage after start:
Temurin 1 core 1152.47ms 68MB Semeru NOSCC 1 core 1977.13ms 42MB
Temurin 2 cores 633.684ms 73MB Semeru NOSCC 2 cores 1148.12ms 42MB
```

Move on to the next section to see what happens when we activate the Shared Classes Cache in IBM Semeru Runtimes and run the Tomcat server with the same sample application. You should see this picture reverse!

## 6. Apache Tomcat with IBM Semeru Runtimes and Shared Classes Cache

In this section of the lab, we're going to try to use the Semeru Runtimes shared classes cache technology to accelerate the less than inspiring startup times we saw in the last section when we tried to start the Tomcat server with IBM Semeru Runtimes.

For consistency, we'll be using Tomcat v10 and Java 17 through the entire lab.

First, goto the direction for Section 5:

```bash
cd ../Section_5
```

Complete the following steps:

1. Build a tomcat container that contains the sample application. There is a `Dockerfile.tomcat_semeru.scc` file provided that first copies the Semeru Runtimes JDK into the container and then copies the `sample.war` file. Finally, it makes sure that the shared classes cache will be used by setting the environment variable `OPENJ9_JAVA_OPTIONS=-Xshareclasses` . Using this option is the simplest way to ensure the shared classes technology is activated. But you're going to see that the results will be unexpected using this overly simplistic approach.
   
	**NOTE**: This environment variable will only be read by the Eclipse OpenJ9 JVM that's in Semeru Runtimes, so the fact that this command-line option wouldn't be recognized by HotSpot shouldn't be a concern if you want to be able to run either JVM (because HotSpot will never look at this environment variable). Using this approach makes it easier for the same Tomcat configuration to be used to start a Tomcat server with either a HotSpot-based distribution like Eclipse Temurin or an OpenJ9-based distribution like IBM Semeru Runtimes.

2. Run the following command to build the container:
	```bash
	podman build --network=host -f Dockerfile.tomcat_semeru.scc -t tomcat_semeru.scc
	```

3. Run the container. It will automatically start the Tomcat server with the sample application. Run the following command and wait for the server to start:
	```bash
	podman run --cpus=1 --network=host --name=tomcat_semeru.scc --replace -it tomcat_semeru.scc > log.cpu1
	```

	As in earlier sections, we're initially confining the server to start with just 1 CPU core.

4. Use the `podman stats` window from <b>Section_1</b> to observe the memory use of the container.

	This command shows various statistics about all containers running within the main workshop container. For example, you should see something like:
	```bash
	ID            NAME               CPU %       MEM USAGE / LIMIT  MEM %       NET IO      BLOCK IO    PIDS        CPU TIME    AVG CPU %
	e564779194b2  tomcat_semeru.scc  0.57%       39.33MB / 2.047GB  1.92%       0B / 0B     8.192kB / 0B  35          5.275247s   45.62%
	```

	As you can see, IBM Semeru Runtimes runs in even less memory than earlier (39MB versus the 42MB we saw earlier in <b>Section_4</b>). That's not as dramatic an improvement as we saw with the Liberty server, but let's ignore that fact for now. With this new memory footprint baseline, the Temurin JDK with the HotSpot JVM consumes 75% more memory to start the server.

5. Press `control-c` to stop the server that you started in Step 2. Once you stop the server, we can check the start time using the `startTime.awk` script:
	```bash
	./startTime.awk log.cpu1
	Server initiated 1715357831549004032, up at 1715357834783000000
	Full start time is 3234 ms
	```

	Wow, what is going on here? That's already much slower than starting with Temurin and even slower than using Semeru Runtime without their (in?)famous shared cache technology! What's going on here?

6. Do a few runs to confirm your finding and that it's not just a fluke. You may see some variation because the times are much longer, but you should find the times are consistently large to a wildly unexpected degree.

	>**Sidebar**: Ask yourself what you would do at this point if had gone through this exercise outside of this lab on your own. Maybe this kind of result has even happened to you already. Would you continue to try to understand why the startup time got worse? Or would you give up, assuming that Semeru Runtimes reputation for fast startup is probably exaggerated or undeserved?

7. I'm afraid you've been set up. Let's consider how the shared cache technology works. In order to have fast startup time, you need a cache to help you go faster. That means you need to do one "cold" run to populate the cache with classes and compiled code. This "cold" run is expected to run more slowly because its purpose is to generate a high quality cache that will enable subsequent "warm" runs to start as quickly as possible. In fact, the Eclipse OpenJ9 JVM employs very different compilation heuristics in this cold run compared to even a normal run that doesn't use the shared classes cache, and that's why the start time you saw in Step 5 was even longer than the times we measured in the previous section when we weren't using the shared classes cache.

	So what happened in Step 1 is that all we did in the `Dockerfile.tomcat_semeru.scc` is to add the `-Xshareclasses` option. There is no cache in the container so the starting point for every run we did in Step 3 is to populate a new cache that's just going to be thrown away when the container stops (because by default results aren't persisted when a container runs)! Basically, we turned every run into a "cold" run that starts more slowly by design so that it builds a high quality cache.

	How do we fix it? Rather than just turning on `-Xshareclases` when we run the server, we need to actually start the server inside the build step to create a `prepopulated` cache. Once the cache is created in the build step, it will be present every time the container is run and every one of those runs will be a "warm" run that will start faster. To populate the cache with tomcat, we can start the tomcat server, sleep for a while until we're sure the server is started, then stop the server. You can see this at the end of `Dockerfile.tomcat_semeru.prepop_scc`. Let's build that container now:

	```bash
	podman build --network=host -f Dockerfile.tomcat_semeru.prepop_scc -t tomcat_semeru.prepop_scc .
	```

	If you pay very close attention, you'll see one more thing that's done at the very end of `Dockerfile.tomcat_semeru.prepop_scc`: we added a "readonly" suboption to `-Xshareclasses`. Since nothing written into the cache in a "warm" run will be saved by the container anyway, there is no point adding any other classes or compiled code after the build step. By updating the option to include "readonly", all accesses to the cache do not need synchronization and all the code paths that focus on adding to the cache will be disabled. The end result will be faster startup!

	Let's see how it works out by starting one of the new containers with the usual one CPU core: 
	```bash
	podman run --cpus=1 --network=host --name=tomcat_semeru.prepop_scc --replace -it tomcat_semeru.prepop_scc > log.prepop.cpu1
	```

	There is a memory improvement because the shared cache is now being used:
	```
	ID            NAME                      CPU %   MEM USAGE / LIMIT  MEM %       NET IO      BLOCK IO    PIDS        CPU TIME    AVG CPU %
	3e4994b07276  tomcat_semeru.prepop_scc  0.84%   35.94MB / 2.047GB  1.76%       0B / 0B     8.192kB / 0B  35        719.773ms   5.87%
	```

	And if we look at the start time: 
	```bash
	./startTime.awk log.prepop.cpu1 Server initiated 1715359356618435584, up at 1715359357211000000 Full start time is 592.564 ms
	```

	Now that's more like it! You can do the same runs with 2 cores: 
	```bash
	podman run --cpus=2 --network=host --name=tomcat_semeru.prepop_scc --replace -it tomcat_semeru.prepop_scc > log.prepop.cpu2
	```

	And see almost exactly the same memory usage: 
	```
	ID NAME CPU % MEM USAGE / LIMIT MEM % NET IO BLOCK IO PIDS CPU TIME AVG CPU % 3adfe1f7ded9 tomcat_semeru.prepop_scc 0.97% 35.91MB / 2.047GB 1.75% 0B / 0B 0B / 0B 36 718.876ms 6.49%
	```

	With even faster start time (under half a second!): 
	
	```bash
	./startTime.awk log.prepop.cpu2 Server initiated 1715359654960000256, up at 1715359655408000000 Full start time is 448 ms
	```

### Summary

This is the last section of this part of the lab, so you can stop the podman stats command running in the other terminal window at this point by hitting `control-c` in that window.

In this section, we initially found very poor results due to a fairly common mistake activating the shared cache technology in Semeru Runtimes that seemed to paint a very dim picture. But when we properly configured the shared classes cache and prepopulated the cache in the container build step, we saw dramatically better startup and memory use with Semeru Runtimes. When running with a single CPU core, we found that compared to using Semeru Runtimes, the Temurin JDK will consume 88% more memory (68MB versus 36MB) and will start 94% slower (1152ms versus 593ms).

Let's add the numbers to our performance summary table: 

```
JDK Core limit Start time Memory usage after start:
Temurin 1 core 1152.47ms 68MB 
Semeru NOSCC 1 core 1977.13ms 42MB 
Semeru SCC 1 core 3234ms 39MB 
Semeru Prepop SCC 1 core 593ms 36MB
Temurin 2 cores 633.684ms 73MB 
Semeru NOSCC 2 cores 1148.12ms 42MB 
Semeru SCC 2 cores 1624.24ms 39MB 
Semeru Prepop SCC 2 cores 448ms 36MB
```

## Conclusion

We hope you enjoyed this lab and learned more about how startup time and memory usage can be dramatically different depending on which JDK you use to deploy your Java workloads. You have also seen the most common misconfiguration of Semeru Runtimes's shared cache technology that results in much slower start-up times that may lead developers to the wrong conclusions about the worth of this technology. In the end, properly configuring the shared class cache technology, particularly when using containers, can led to dramatic savings in both start time and memory use. Faster start time means you can provide a more responsive and elastic infrastucture, whereas lower memory usage can translate into smaller VMs which cost less money.