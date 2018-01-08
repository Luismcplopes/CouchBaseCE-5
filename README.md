#Couchbase Server 5.0.1 Community Edition on Ubuntu 16.4

allow connections to the following ports: 11211, 11210, 11209, 4369,
8091, 8092, 18091, 18092, 11214, 11215 and from 21100 to 21299.


`$ docker run -d -p 8091-8093:8091-8093 -p 11210:11210 -v /opt/couchbase/var/lib/couchbase/data:/opt/couchbase/var/lib/couchbase/data couchbase/server:community-5.0.0`

### Iniciar Ubuntu 16.04
```
$ docker run -d -p 8091-8093:8091-8093 -p 11210:11210 -v /opt/couchbase/var/lib/couchbase/data:/opt/couchbase/var/lib/couchbase/data 
$ docker container exec <containerid>   /bin/bash  /opt/couchbase/entrypoint.sh
```






This README will guide you through running Couchbase Server with Docker Containers.

[Couchbase Server](http://www.couchbase.com/nosql-databases/couchbase-server) is a NoSQL document database with a distributed architecture for performance, scalability, and availability. It enables developers to build applications easier and faster by leveraging the power of SQL with the flexibility of JSON.

For additional questions and feedback, please visit the [Couchbase Forums](https://forums.couchbase.com/) or [Stack Overflow](http://stackoverflow.com/questions/tagged/couchbase).

# QuickStart with Couchbase Server and Docker

Here is how to get a single node Couchbase Server cluster running on Docker containers:

**Step - 1 :** Run Couchbase Server docker container

`docker run -d --name db -p 8091-8094:8091-8094 -p 11210:11210 `

**Step - 2 :** Next, visit `http://localhost:8091` on the host machine to see the Web Console to start Couchbase Server setup.

![setup splash screen](https://raw.githubusercontent.com/cihanb/docker/master/generate/resources/couchbase-server/images/setup-initial.jpg)

Walk through the Setup wizard and accept the default values.

-	Note: You may need to lower the RAM allocated to various services to fit within the bounds of the resource of the containers.
-	Enable the beer-sample bucket to load some sample data.

![setup step-1 screen](https://raw.githubusercontent.com/cihanb/docker/master/generate/resources/couchbase-server/images/setup-step1.jpg)

**Note :** For detailed information on configuring the Server, see [Initial Couchbase Server Setup](https://developer.couchbase.com/documentation/server/5.0/install/init-setup.html).

## Running A N1QL Query on the Couchbase Server Cluster

N1QL is the SQL based query language for Couchbase Server. Simply switch to the Query tab on the Web Console at `http://localhost:8091` and run the following N1QL Query in the query window:

```SELECT name FROM `beer-sample` WHERE brewery_id ="mishawaka_brewing";```

You can also execute N1QL queries from the commandline. To run a query from command line query tool, run the interactive shell on the container:

`docker exec -it db sh`

Then, navigate to the `bin` directory under Couchbase Server installation and run cbq command line tool and execute the N1QL Query on `beer-sample` bucket

`/opt/couchbase/bin/cbq`

```cbq> SELECT name FROM `beer-sample` WHERE brewery_id ="mishawaka_brewing";```

For more query samples, refer to the [Running your first N1QL query](https://developer.couchbase.com/documentation/server/5.0/getting-started/try-a-query.html) guide.

## Connect to the Couchbase Server Cluster via Applications and SDKs
Couchbase Server SDKs comes in many languages: C SDK 2.4/2.5 Go, Java, .NET, Node.js, PHP, Python. Simply run your application through the Couchbase Server SDK of your choice on the host, and point it to http://localhost:8091/pools to connect to the container.

For running a sample application, refer to the [Running a sample Web app](http://developer.couchbase.com/documentation/server/4.5/travel-app/index.html) guide.

# Requirements and Best Practices

## Container Requirements

Official Couchbase Server containers on Docker Hub are based on Ubuntu 14.04.

**Docker Container Resource Requirements :** For minimum container requirements, you can follow [Couchbase Server minimum HW recommendations](http://developer.couchbase.com/documentation/server/current/install/pre-install.html) for development, test and production environments.

## Best Practices

**Avoid a Single Point of Failure :** Couchbase Server's resilience and high-availability are achieved through creating a cluster of independent nodes and replicating data between them so that any individual node failure doesn't lead to loss of access to your data. In a containerized environment, if you were to run multiple nodes on the same piece of physical hardware, you can inadvertently re-introduce a single point of failure. In environments where you control VM placement, we advise ensuring each Couchbase Server node runs on a different piece of physical hardware.

**Sizing your containers :** Physical hardware performance characteristics are well understood. Even though containers insert a lightweight layer between Couchbase Server and the underlying OS, there is still a small overhead in running Couchbase Server in containers. For stability and better performance predictability, It is recommended to have at least 2 cores dedicated to the container in development environments and 4 cores dedicated to the container rather than shared across multiple containers for Couchbase Server instances running in production. With an over-committed environment you can end up with containers competing with each other causing unpredictable performance and sometimes stability issues.

**Map Couchbase Node Specific Data to a Local Folder :** A Couchbase Server Docker container will write all persistent and node-specific data under the directory /opt/couchbase/var by default. It is recommended to map this directory to a directory on the host file system using the `-v` option to `docker run` to get persistence and performance.

-	Persistence: Storing `/opt/couchbase/var` outside the container with the `-v` option allows you to delete the container and recreate it later without losing the data in Couchbase Server. You can even update to a container running a later release/version of Couchbase Server without losing your data.
-	Performance: In a standard Docker environment using a union file system, leaving /opt/couchbase/var inside the container results in some amount of performance degradation.

> NOTE for SELinux : If you have SELinux enabled, mounting the host volumes in a container requires an extra step. Assuming you are mounting the `~/couchbase` directory on the host file system, you need to run the following command once before running your first container on that host:
>
> `mkdir ~/couchbase && chcon -Rt svirt_sandbox_file_t ~/couchbase`

**Increase ULIMIT in Production Deployments :** Couchbase Server normally expects the following changes to ulimits:

```console
ulimit -n 40960        # nofile: max number of open files
ulimit -c unlimited    # core: max core file size
ulimit -l unlimited    # memlock: maximum locked-in-memory address space
```

These ulimit settings are necessary when running under heavy load. If you are just doing light testing and development, you can omit these settings, and everything will still work.

To set the ulimits in your container, you will need to run Couchbase Docker containers with the following additional --ulimit flags:

`docker run -d --ulimit nofile=40960:40960 --ulimit core=100000000:100000000 --ulimit memlock=100000000:100000000 --name db -p 8091-8094:8091-8094 -p 11210:11210 `

Since "unlimited" is not supported as a value, it sets the core and memlock values to 100 GB. If your system has more than 100 GB RAM, you will want to increase this value to match the available RAM on the system.

> Note:The --ulimit flags only work on Docker 1.6 or later.

**Network Configuration and Ports :** Couchbase Server communicates on many different ports (see the [Couchbase Server documentation](http://docs.couchbase.com/admin/admin/Install/install-networkPorts.html)). Also, it is generally not supported that the cluster nodes be placed behind any NAT. For these reasons, Docker's default networking configuration is not ideally suited to Couchbase Server deployments. For production deployments it is recomended to use `--net=host` setting to avoid performance and reliability issues.

# Multi Node Couchbase Server Cluster Deployment Topologies

With multi node Couchbase Server clusters, there are 2 popular topologies.

## All Couchbase Server containers on one physical machine

This model is commonly used for scale-minimized deployments simulating production deployments for development and test purposes. Placing all containers on a single physical machine means all containers will compete for the same resources. Placing all containers on a single physical machine also eliminates the built-in protection against Couchbase Server node failures with replication when the single physical machine fail, all containers experience unavailability at the same time loosing all replicas. These restrictions may be acceptable for test systems, however it isn’t recommended for applications in production.

You can find more details on setting up Couchbase Server in this topology in Couchbase Server [documentation](https://developer.couchbase.com/documentation/server/5.0/install/docker-deploy-multi-node-cluster.html).

	┌──────────────────────────────────────────────────────────┐
	│                     Host OS (Linux)                      │
	│                                                          │
	│  ┌───────────────┐ ┌───────────────┐  ┌───────────────┐  │
	│  │ Container OS  │ │ Container OS  │  │ Container OS  │  │
	│  │   (Ubuntu)    │ │   (Ubuntu)    │  │   (Ubuntu)    │  │
	│  │ ┌───────────┐ │ │ ┌───────────┐ │  │ ┌───────────┐ │  │
	│  │ │ Couchbase │ │ │ │ Couchbase │ │  │ │ Couchbase │ │  │
	│  │ │  Server   │ │ │ │  Server   │ │  │ │  Server   │ │  │
	│  │ └───────────┘ │ │ └───────────┘ │  │ └───────────┘ │  │
	│  └───────────────┘ └───────────────┘  └───────────────┘  │
	└──────────────────────────────────────────────────────────┘


## Each Couchbase Server container on its own machine

This model is commonly used for production deployments. It prevents Couchbase Server nodes from stepping over each other and gives you better performance predictability. This is the supported topology in production with Couchbase Server 4.5 and higher.

You can find more details on setting up Couchbase Server in this topology in Couchbase Server [documentation](https://developer.couchbase.com/documentation/server/5.0/install/docker-deploy-single-node-cluster.html).

	┌───────────────────────┐  ┌───────────────────────┐  ┌───────────────────────┐
	│   Host OS (Linux)     │  │   Host OS (Linux)     │  │   Host OS (Linux)     │
	│  ┌─────────────────┐  │  │  ┌─────────────────┐  │  │  ┌─────────────────┐  │
	│  │  Container OS   │  │  │  │  Container OS   │  │  │  │  Container OS   │  │
	│  │    (Ubuntu)     │  │  │  │    (Ubuntu)     │  │  │  │    (Ubuntu)     │  │
	│  │  ┌───────────┐  │  │  │  │  ┌───────────┐  │  │  │  │  ┌───────────┐  │  │
	│  │  │ Couchbase │  │  │  │  │  │ Couchbase │  │  │  │  │  │ Couchbase │  │  │
	│  │  │  Server   │  │  │  │  │  │  Server   │  │  │  │  │  │  Server   │  │  │
	│  │  └───────────┘  │  │  │  │  └───────────┘  │  │  │  │  └───────────┘  │  │
	│  └─────────────────┘  │  │  └─────────────────┘  │  │  └─────────────────┘  │
	└───────────────────────┘  └───────────────────────┘  └───────────────────────┘

# Additional References

-	[Couchbase Server and Containers](http://www.couchbase.com/containers)
-	[Getting Started with Couchbbase Server and Docker](https://developer.couchbase.com/documentation/server/5.0/install/getting-started-docker.html)
-	Detailed Walk-through for [Deploying Couchbase Server on Docker Containers](https://developer.couchbase.com/documentation/server/5.0/install/deploy-with-docker-hub.html)

# Licensing

Couchbase Server comes in 2 Editions: Enterprise Edition and Community Edition. You can find details on the differences between the 2 and licensing details on the [Couchbase Server Editions](https://developer.couchbase.com/documentation/server/5.0/introduction/editions.html) page.

-	Enterprise Edition -- free for development, testing and POCs. Requires a paid subscription for production deployment. Please refer to the [subscribe](http://www.couchbase.com/subscriptions-and-support) page for details on enterprise edition agreements.

-	Community Edition -- free for unrestricted use for community users.

By default, the `latest` Docker tag points to the latest Enterprise Edition. If you want the Community Edition instead, you should add the appropriate tag, such as

```console
docker run couchbase:community-4.0.0
```
