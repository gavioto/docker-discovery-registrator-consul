# docker-discovery-registrator-consul

Service discovery library for JVM based applications running in Docker containers that use the Registrator service registry bridge with Consul as a backend. 

The purpose of this library is for "self-discovery" from within your JVM based Docker application where you need to discover what
your accessible docker-host bound IP and mapped port(s) are. This is critical if your container has to do further peer discovery
for other services it provides or clustering groups it must form.

* [Status](#status)
* [Releases](#releases)
* [Requirements](#requirements)
* [Maven/Gradle install](#mavengradle)
* [Features](#features)
* [Usage](#usage)
* [Build from source](#building)
* [Unit tests](#tests)
* [Related Info](#related)
* [Todo](#todo)
* [Notes](#notes)
* [Docker info](#docker)


## <a id="status"></a>Status

Beta code. Master branch available only.

## <a id="releases"></a>Releases

No official releases yet.

* MASTER - in progress, this README refers to what is in the master branch. Switch to relevant RELEASE tag above to see that versions README

## <a id="requirements"></a>Requirements

* Java 6+
* Your application is running in a Docker container, and launched with Registrator consumed variables `-e SERVICE_[port]_NAME=[name]` and `-e SERVICE_TAGS=[tags..]`
* Your Docker host has a [Registrator](https://github.com/gliderlabs/registrator) container running
* The Registrator container is configured to use [Consul](https://consul.io/) as its registry backend

## <a id="mavengradle"></a>Maven/Gradle

To use this discovery strategy in your Maven or Gradle project use the dependency samples below. (coming soon)

### Gradle:

coming soon

### Maven:

coming soon

## <a id="features"></a>Features

* Permits a JVM based container application to self-discover all of its mapped ports and accessible ip address, as well of that as all of its peer "services" that share the same service name, ports and/or service tags, as set by Registrator in Consul.


## <a id="usage"></a>Usage

* Ensure your project has the `docker-discovery-registrator-consul` artifact dependency declared in your maven pom or gradle build file as described above. Or build the jar yourself and ensure the jar is in your project's classpath.

* Have Consul running and available somewhere on your network, start it such as:
```
consul agent -server -bootstrap-expect 1 -data-dir /tmp/consul -config-dir /path/to/consul.d/ -ui-dir /path/to/consul-web-ui -bind=0.0.0.0 -client=0.0.0.0
```

* On your Docker host ensure Registrator is running such as:
```
docker run -d --name=registrator --net=host --volume=/var/run/docker.sock:/tmp/docker.sock  gliderlabs/registrator:latest consul://localhost:8500
```

* In your JVM based application that will run in a container you can do the following:

```
ConsulDiscovery c = new ConsulDiscovery()
						.setConsulIp(System.getProperty("CONSUL_IP"))
						.setConsulPort(System.getProperty("CONSUL_PORT"))
						.setServiceName(System.getProperty("MY_SERVICE_NAME")) 
						.setMyNodeUniqueTagId(System.getProperty("MY_UNIQUE_TAG"))
						.addPortToDiscover(8080)
						.addPortToDiscover(8443)
						.addMustHaveTag("dev")
						.setServiceNameStrategyClass(MultiServiceNameSinglePortStrategy.class);


Collection<ServiceInfo> myServices = c.discoverMe();
for (ServiceInfo info : myServices) {
	System.out.println(info);
}
```

* Create your Docker image and run it as two containers on the same Docker host that Registrator is running on (for this example it would EXPOSE 8080,8443)

```
docker run -e "SERVICE_TAGS=dev,myUniqueId001" --rm=true -P myDockerApp:latest java -DMY_SERVICE_NAME=myDockerApp -DMY_UNIQUE_TAG=myUniqueId001 -DCONSUL_IP=127.0.0.1 -DCONSUL_PORT=8500 -jar myApp.jar

docker run -e "SERVICE_TAGS=dev,myUniqueId002" --rm=true -P myDockerApp:latest java -DMY_SERVICE_NAME=myDockerApp -DMY_UNIQUE_TAG=myUniqueId002 -DCONSUL_IP=127.0.0.1 -DCONSUL_PORT=8500 -jar myApp.jar  
```

* You should get output something like the following as the code above discovers itself:

```
{"serviceName":"myDockerApp","serviceId":"default:silly_jennings:8080","exposedAddress":"192.168.0.99","exposedPort":32797,"mappedPort":8080,"tags":"[dev, myUniqueId001]"}
{"serviceName":"myDockerApp","serviceId":"default:silly_jennings:8080","exposedAddress":"192.168.0.99","exposedPort":32798,"mappedPort":8443,"tags":"[dev, myUniqueId001]"}
```

## <a id="building"></a>Building from source

* From the root of this project, build a Jar : `./gradlew assemble`

* Include the built jar artifact located at `build/libs/docker-discovery-registrator-consul-[VERSION].jar` in your JVM based project

* If not already present in your hazelcast application's Maven (pom.xml) or Gradle (build.gradle) dependencies section; ensure that these dependencies are present (versions may vary as appropriate):

```
compile group: 'com.orbitz.consul', name: 'consul-client', version:'0.10.0'
compile 'javax.ws.rs:javax.ws.rs-api:2.0.1'
compile 'org.glassfish.jersey.core:jersey-client:2.22.2'
compile 'org.slf4j:slf4j-api:1.7.19'
```


## <a id="tests"></a>Unit-tests

Coming soon

## <a id="related"></a>Related info

* https://www.consul.io
* https://github.com/gliderlabs/registrator

## <a id="todo"></a>Todo

* Coming soon

## <a id="notes"></a> Notes

* Coming soon