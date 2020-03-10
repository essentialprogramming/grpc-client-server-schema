### 1. Introduction
[gRPC](https://www.grpc.io/) is a high performance, open source RPC framework initially developed by Google.
It helps in eliminating boilerplate code and helps in connecting polyglot services in and across
data centers.  
You can define how you want the information you’re serializing to be structured by defining protocol
buffer message types in ```.proto``` files. Each protocol buffer message is a small logical record of information, containing a series of name-value pairs.
### 2. Download project using git
Use the following command to clone this repository:

    git clone https://github.com/essentialprogramming/grpc-client-server-schema.git


### 3. Defining the Service

The Service definition is already defined in ```proto``` file. The ```.proto``` file contains the description of some methods
that can be called remotely with their parameters and return types.  This is done in the ```.proto``` file using the protocol buffers. Protocol Buffers are also used for 
describing the structure of the payload messages.  



### 4. Generate Java code based on .proto files

Navigate to ```grpc-client``` folder at ```pom.xml``` level and execute the following command ```mvn clean install``` in order to:  
* build the project (generate executable uber-jar)
* generate the classes specified in ```.proto``` file

> run the newly generated jar with the following command(from the same path) :  
```java -jar target/grpc-0.0.1-SNAPSHOT-shaded.jar```

Navigate to ```grpc-server``` folder at ```pom.xml``` level and execute the following command ```mvn clean install``` in order to:  
* build the project (generate executable uber-jar)
* generate the classes specified in ```.proto``` file  
> run the newly generated jar with the following command(from the same path) :  
```java -jar target/grpc-0.0.1-SNAPSHOT-shaded.jar```


### 5. Create the Server 

After the source code in generated in *target/generated-sources/protobuf* , navigate to ```grpc-server``` and create a class in main folder called ```HelloWorldServiceImpl``` and populate it as follows : 
> We shall extend this class and override the hello() method mentioned in our service definition:
  

```java
public class HelloServiceImpl extends HelloServiceImplBase {
 
    @Override
    public void hello(
      HelloRequest request, StreamObserver<HelloResponse> responseObserver) {
 
        String greeting = new StringBuilder()
          .append("Hello, ")
          .append(request.getFirstName())
          .append(" ")
          .append(request.getLastName())
          .toString();
 
        HelloResponse response = HelloResponse.newBuilder()
          .setGreeting(greeting)
          .build();
 
        responseObserver.onNext(response);
        responseObserver.onCompleted();
    }
}
```  
> The default implementation of the abstract class HelloServiceImplBase is to throw runtime exception io.grpc.StatusRuntimeException saying that the method is unimplemented.  


 In order to run the server we need to start the gRPC server to listen for incoming requests.  
 Create a new class called  ```GrpcServer``` and populate it as follows:  
 ```java
public class GrpcServer {
    public static void main(String[] args) {
        Server server = ServerBuilder
          .forPort(8080)
          .addService(new HelloServiceImpl()).build();
 
        server.start();
        server.awaitTermination();
    }
}

```
In this code snippet we use the builder to create a gRPC server on specified port (8080) and we add the service that we previously defined ```HelloServiceImpl```  
The *start()* method is starting the server and we are calling *awaitTermination()* to keep the server running in foreground blocking the prompt.  

### 6. Create the Client  
   gRPC has a channel construct that abstracts the details like load balancing, connection, connection pulling.
   The channel is created using the class *ManagedChannelBuilder* where the only detail that is specified is the address and the port.  
   
   After the source code in generated in *target/generated-sources/protobuf* , navigate to ```grpc-client``` and create a class in main folder called ```GrpcClient``` and populate it as follows :  
   ```java
public class GrpcClient {
    public static void main(String[] args) {
        ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost", 8080)
          .usePlaintext()
          .build();
 
        HelloServiceGrpc.HelloServiceBlockingStub stub 
          = HelloServiceGrpc.newBlockingStub(channel);
 
        HelloResponse helloResponse = stub.hello(HelloRequest.newBuilder()
          .setFirstName("John")
          .setLastName("Smith")
          .build());
 
        channel.shutdown();
    }
}
```

### 7. Run the newly generated classes  
*  Navigate to grpc-server folder at pom.xml level and execute the following command ```mvn clean install``` in order to  build the project that uses the newly generated classes (generate executable uber-jar).  
  Run the newly generated jar with the following command *(from the same path)* :  
  ```java -jar target/grpc-0.0.1-SNAPSHOT-shaded.jar ```
  

* Navigate to grpc-client folder at pom.xml level and execute the following command ```mvn clean install``` in order to  build the project that uses the newly generated classes (generate executable uber-jar).  
Run the newly generated jar with the following command *(from the same path)* :  
  ```java -jar target/grpc-0.0.1-SNAPSHOT-shaded.jar```





