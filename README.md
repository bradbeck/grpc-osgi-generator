# grpc-osgi-generator
This project implements a protoc plugin to generate a Java service interface class and supporting classes.

For example, consider the HealthCheck service declaration in the following [health.proto](https://github.com/ECF/grpc-RemoteServicesProvider/blob/master/examples/org.eclipse.ecf.examples.provider.grpc.health.api/src/main/proto/health.proto) snippet:

```proto

service HealthCheck {
  // Unary method
  rpc Check(HealthCheckRequest) returns (HealthCheckResponse);
  // Server streaming method
  rpc WatchServer(HealthCheckRequest) returns (stream HealthCheckResponse);
  // Client streaming method
  rpc WatchClient(stream HealthCheckRequest) returns (HealthCheckResponse);
  // bidi streaming method
  rpc WatchBidi(stream HealthCheckRequest) returns (stream HealthCheckResponse);
}
```
If [protoc](https://developers.google.com/protocol-buffers) is run with this proto file as input, along with the [grpc-java](https://github.com/grpc/grpc-java) and [reactive-grpc plugin](https://github.com/salesforce/reactive-grpc), along with this grpc-osgi-generator plugin, will generate the following service interface

```java
package io.grpc.health.v1;

import io.reactivex.Flowable;
import io.reactivex.Single;

@javax.annotation.Generated(
value = "by grpc-osgi-generator - A protoc plugin for ECF's grpc remote services distribution provider at https://github.com/ECF/grpc-RemoteServiceSProvider ",
comments = "Source: health.proto.  ")
public interface HealthCheckService {
    
    default Single<io.grpc.health.v1.HealthCheckResponse> check(Single<io.grpc.health.v1.HealthCheckRequest> requests)  {
        return null;
    }
    
    /**
     * <pre>
     *  Server streaming method
     * </pre>
     */
    default Flowable<io.grpc.health.v1.HealthCheckResponse> watchServer(Single<io.grpc.health.v1.HealthCheckRequest> requests)  {
        return null;
    }
    
    /**
     * <pre>
     *  Client streaming method
     * </pre>
     */
    default Single<io.grpc.health.v1.HealthCheckResponse> watchClient(Flowable<io.grpc.health.v1.HealthCheckRequest> requests)  {
        return null;
    }
    
    /**
     * <pre>
     *  bidi streaming method
     * </pre>
     */
    default Flowable<io.grpc.health.v1.HealthCheckResponse> watchBidi(Flowable<io.grpc.health.v1.HealthCheckRequest> requests)  {
        return null;
    }
}
```
As can be seen above, this interface class has a method corresponding to each type of [grpc](https://grpc.io/) method type: unary, server-streaming, client-streaming, and bi-directional streaming.  The streaming methods use the Reactive Java types (Flowable and Single) in the service interface.

Note that the generated service interface uses the 'default' keyword provided by Java8 compiler, and so depends upon using Java8 (or higher) target Java runtime environment.

This service interface can then be used for exposing [OSGi Remote Services](https://docs.osgi.org/specification/osgi.cmpn/7.0.0/service.remoteservices.html) in [OSGi](https://www.osgi.org) environments, since all OSGi Services are based upon the service interface class.  [ECF has an implementation](https://wiki.eclipse.org/OSGi_Remote_Services_and_ECF) of a [OSGi Remote Services Distribution Provider](https://github.com/ECF/grpc-RemoteServicesProvider) that exports and imports (along with discovery) of such grpc-based services using the [OSGi Remote Service Admin](https://docs.osgi.org/specification/osgi.cmpn/7.0.0/service.remoteserviceadmin.html) specification.

The net effect of running protoc with these three protoc plugins is that the following java classes will be produced

1. Classes representing the protobuf messages and options (HealthCheckRequest/Response) -- *via protocol buffers compiler*
1. Classes providing access to grpc-based unary service methods (HealthCheckGrpc) -- via grpc-java plugin that is part of [grpc](https://github.com/grpc/)
1. Classes providing access to reactive-grpc APIs for streaming service methods (RxHealthCheckGrpc) - via [reactive-grpc](https://github.com/salesforce/reactive-grpc)
1. A service interface class ([HealthCheckService](https://github.com/ECF/grpc-RemoteServicesProvider/blob/master/examples/org.eclipse.ecf.examples.provider.grpc.health.api/src/main/java/io/grpc/health/v1/HealthCheckService.java) class above) that references the message types and the reactive io.reactivex.Flowable and io.reactivex.Single classes -- via this grpc-osgi-generator plugin

[Here is the directory](https://github.com/ECF/grpc-RemoteServicesProvider/tree/master/examples/org.eclipse.ecf.examples.provider.grpc.health.api/src/main/java/io/grpc/health/v1) with all the generated source code.  Note that all the classes in this directory were generated by a single run of the protoc compiler with the grpc-java, reactive-grpc, and grpc-osgi-generator plugins added via the maven-protobuf-plugin with usage described below.

## Using this protoc plugin via [protobuf-maven-plugin](https://www.xolstice.org/protobuf-maven-plugin/)

Currently, the easiest way to run protoc including the grpc-java plugin, the reactive-grpc plugin, and the grpc-osgi-generator plugin is to use the [protobuf-maven-plugin](https://www.xolstice.org/protobuf-maven-plugin/) to invoke protoc and these 3 protoc plugins as part of a maven build.

For example, with these pom.xml properties to define the appropriate versions

```xml
<properties>
<!-- your other properties -->
	<rxjava.version>2.2.19</rxjava.version>
	<reactive.grpc.version>1.0.0</reactive.grpc.version>
	<grpc.contrib.version>0.8.0</grpc.contrib.version>
	<grpc.version>1.23.0</grpc.version>
	<grpc-osgi-generator.version>1.2.0-SNAPSHOT</grpc-osgi-generator>
</properties>
```
and these pom dependencies 

```xml
	<dependencies>
		<dependency>
			<groupId>io.reactivex.rxjava2</groupId>
			<artifactId>rxjava</artifactId>
			<version>${rxjava.version}</version>
		</dependency>
		<dependency>
			<groupId>com.salesforce.servicelibs</groupId>
			<artifactId>rxgrpc-stub</artifactId>
			<version>${reactive.grpc.version}</version>
		</dependency>
		<dependency>
			<groupId>io.grpc</groupId>
			<artifactId>grpc-netty</artifactId>
			<version>${grpc.version}</version>
		</dependency>
		<dependency>
			<groupId>io.grpc</groupId>
			<artifactId>grpc-protobuf</artifactId>
			<version>${grpc.version}</version>
		</dependency>
		<dependency>
			<groupId>io.grpc</groupId>
			<artifactId>grpc-core</artifactId>
			<version>${grpc.version}</version>
		</dependency>
		<dependency>
			<groupId>io.grpc</groupId>
			<artifactId>grpc-context</artifactId>
			<version>${grpc.version}</version>
		</dependency>
		<dependency>
			<groupId>io.grpc</groupId>
			<artifactId>grpc-stub</artifactId>
			<version>${grpc.version}</version>
		</dependency>
	</dependencies>
```

the following extension and the protobuf-maven-plugin...along with metadata for the 3 protobuf plugins should be in your build section (along with any other maven plugins needed for your build)

```xml
		<extensions>
			<extension>
				<groupId>kr.motd.maven</groupId>
				<artifactId>os-maven-plugin</artifactId>
				<version>1.6.2</version>
			</extension>
		</extensions>
		<plugins>
			<plugin>
				<groupId>org.xolstice.maven.plugins</groupId>
				<artifactId>protobuf-maven-plugin</artifactId>
				<version>0.6.1</version>
				<configuration>
					<protocArtifact>com.google.protobuf:protoc:${protoc.version}:exe:${os.detected.classifier}
					</protocArtifact>
					<pluginId>grpc-java</pluginId>
					<pluginArtifact>io.grpc:protoc-gen-grpc-java:${grpc.version}:exe:${os.detected.classifier}
					</pluginArtifact>
				</configuration>
				<executions>
					<execution>
						<goals>
							<goal>compile</goal>
							<goal>compile-custom</goal>
						</goals>
						<phase>generate-sources</phase>
						<configuration>
							<protocPlugins>
								<protocPlugin>
									<id>grpc-osgi-generator</id>
									<groupId>org.eclipse.ecf</groupId>
									<artifactId>grpc-osgi-generator</artifactId>
									<version>${grpc-osgi-generator.version}</version>
									<mainClass>org.eclipse.ecf.grpc.osgigenerator.OSGiGenerator
									</mainClass>
								</protocPlugin>
                                				<protocPlugin>
                                    					<id>rxgrpc</id>
                                    					<groupId>com.salesforce.servicelibs</groupId>
                                    					<artifactId>rxgrpc</artifactId>
                                    					<version>${reactive.grpc.version}</version>
                                    					<mainClass>com.salesforce.rxgrpc.RxGrpcGenerator</mainClass>
                                				</protocPlugin>
							</protocPlugins>
						</configuration>
					</execution>
				</executions>
			</plugin>
...your other maven plugins
            </plugins>
  </build>
```
Note in the above that the protoc + grpc-java plugin + reactive-grpc + grpc-osgi-generator (this project) will be run as part of a build (generate-sources phase) on **any .proto files in ./src/main/proto** directory and output to the **./target/generated-sources/java/** directory.  

