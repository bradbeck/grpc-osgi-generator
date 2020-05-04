# grpc-osgi-generator
Service interface generator plugin to enhance the behavior of the [grpc-java](https://github.com/grpc/grpc-java) compiler.  Both this plugin and the grpc-java plugin are plugins for [Google's Protocol Buffers](https://developers.google.com/protocol-buffers) protoc code generator.  When protoc is run with this plugin and the grpc-java plugin the following 3 types of java classes are generated:

1 classes representing protobuf messages and options -- via protoc with java as generator output
1 classes providing access to grpc-based services -- via grcp-java compiler plugin
! classes providing support for using OSGi Remote Services -- via the plugin provided by this project

## What running protoc + grpc-osgi-generator plugin + [grpc-java](https://github.com/grpc/grpc-java) plugin does

[Here](https://raw.githubusercontent.com/ECF/grpc-RemoteServicesProvider/master/examples/org.eclipse.ecf.examples.provider.grpc.health.api/src/main/proto/health.proto) is a grpc example proto file input file:

```proto
// Copyright 2015 The gRPC Authors
//
// Licensed under the Apache License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License.
// You may obtain a copy of the License at
//
//     http://www.apache.org/licenses/LICENSE-2.0
//
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
// See the License for the specific language governing permissions and
// limitations under the License.

// The canonical version of this proto can be found at
// https://github.com/grpc/grpc-proto/blob/master/grpc/health/v1/health.proto

syntax = "proto3";

package grpc.health.v1;

option java_multiple_files = true;
option java_outer_classname = "HealthProto";
option java_package = "io.grpc.health.v1";

message HealthCheckRequest {
  string message = 1;
}

message HealthCheckResponse {
  enum ServingStatus {
    UNKNOWN = 0;
    SERVING = 1;
    NOT_SERVING = 2;
    SERVICE_UNKNOWN = 3;  // Used only by the Watch method.
  }
  ServingStatus status = 1;
}

service HealthCheck {
  // Unary method
  rpc Check(HealthCheckRequest) returns (HealthCheckResponse);
  // Streaming method
  rpc Watch(HealthCheckRequest) returns (stream HealthCheckResponse);
}
```

Note that HealthCheckRequest and HealthCheckResponse are protocol buffers messages, and that the HealthCheck service is declared with both a unary method (Check), and a streaming method (Watch).

With the above proto file as input to protoc with the grpc-osgi-generator and grpc-java plugins the following classes are generated:

AbstractHealthCheckServiceImpl - **grpc-osgi-generator**
HealthCheckGrpc - grpc-java compiler plugin
HealthCheckRequest - protoc java
HealthCheckRequestOrBuilder - protoc java
HealthCheckResponse - protoc java
HealthCheckResponseOrBuilder - protoc java
HealthCheckService - **grpc-osgi-generator**
HealthCheckProto - grpc-java

[Here](https://github.com/ECF/grpc-RemoteServicesProvider/tree/master/examples/org.eclipse.ecf.examples.provider.grpc.health.api/src/main/java/io/grpc/health/v1) is the directory in the example with the generated source code.

The two classes generated by this plugin are a service interface class **[HealthCheckService](https://github.com/ECF/grpc-RemoteServicesProvider/blob/master/examples/org.eclipse.ecf.examples.provider.grpc.health.api/src/main/java/io/grpc/health/v1/HealthCheckService.java)** 

and an abstract impl of this service interface class **AbstractHealthCheckServiceImpl**(https://github.com/ECF/grpc-RemoteServicesProvider/blob/master/examples/org.eclipse.ecf.examples.provider.grpc.health.api/src/main/java/io/grpc/health/v1/AbstractHealthCheckServiceImpl.java). 

All of the other classes were generated by either protoc java directly, or by the grcp-java compiler plugin.

For OSGi Services a service interface is required (aka objectClass) for service registration, and that's why this grpc-osgi-generator plugin generates **HealthCheckService** class.   **AbstractHealthCheckServiceImpl** is an abstract helper class that implements **HealthCheckService** that service implementers can extend.  See the [grcp-RemoteServicesProvider](https://github.com/ECF/grpc-RemoteServicesProvider) project.

## Building and Installing this Plugin into local Maven repo

To build and install this plugin into your local Maven repository, clone this to your system and then

>cd <grpc-osgi-generator repo-location>
>maven install
  

## Using this protoc plugin via protobuf-maven-plugin

Currently the easiest way to use protoc with both the grpc-java plugin and this plugin is to use the protobuf-maven-plugin.

It's typically easiset to declare versions from within the Maven pom.xml properties section

```maven
<properties>
<!-- your other properties -->
	<grpc.version>1.29.0</grpc.version>
	<protoc.version>3.11.4</protoc.version>
	<jprotoc.version>1.0.1</jprotoc.version>
</properties>
```

Then in the same pom.xml these dependencies should be present

```maven
	<dependencies>
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
			<groupId>com.salesforce.servicelibs</groupId>
			<artifactId>jprotoc</artifactId>
			<version>${jprotoc.version}</version>
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

Finally, the following extension and plugins should be in your build section (along with any other maven plugins like compile, clean, etc)

```maven
	<build>
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
									<version>1.0.0-SNAPSHOT</version>
									<mainClass>org.eclipse.ecf.grpc.osgigenerator.OSGiGenerator
									</mainClass>
								</protocPlugin>
							</protocPlugins>
						</configuration>
					</execution>
				</executions>
			</plugin>
		</plugins>  
  </build>
```
Note in the above that the protoc + grpc-java plugin + grpc-osgi-generator will be run as part of a build (generate-sources phase) on **any .proto files in ./src/main/proto** directory and output to the **./target/generated-sources/java/** directory.  

