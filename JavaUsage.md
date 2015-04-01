# Java Usage #

The following example shows a server and a client using the `com.googlecode.protobuf.socketrpc` Java API.

Note that this example uses delimited rpc channel factories which is new in version 2.0 and is not compatible with 1.x versions. Use undelimited rpc channel factories if you need to communicate with 1.x versions.

## Server side ##

```
// Start server
ServerRpcConnectionFactory rpcConnectionFactory = SocketRpcConnectionFactories
    .createServerRpcConnectionFactory(port);
RpcServer server = new RpcServer(rpcConnectionFactory, 
    Executors.newFixedThreadPool(threadPoolSize), true);
server.registerService(new MyServiceImpl()); // For non-blocking impl
server.registerBlockingService(MyService
    .newReflectiveBlockingService(new MyBlockingInterfaceImpl())); // For blocking impl
server.run();
```

## Client Side (Blocking) ##

```
// Create channel
RpcConnectionFactory connectionFactory = SocketRpcConnectionFactories
    .createRpcConnectionFactory(host, port);
BlockingRpcChannel channel = RpcChannels.newBlockingRpcChannel(connectionFactory);

// Call service
BlockingInterface service = MyService.newBlockingStub(channel);
RpcController controller = new SocketRpcController();
MyResponse myResponse = service.myMethod(controller, myRequest);

// Check success
if (rpcController.failed()) {
  System.err.println(String.format("Rpc failed %s : %s", rpcController.errorReason(),
      rpcController.errorText()));
}
```

## Client Side (non-blocking) ##

```
// Create a thread pool
ExecutorService threadPool = Executor.newFixedThreadPool(1);

// Create channel
RpcConnectionFactory connectionFactory = SocketRpcConnectionFactories
    .createRpcConnectionFactory(host, port);
RpcChannel channel = RpcChannels.newRpcChannel(connectionFactory, threadPool);

// Call service
MyService myService = MyService.newStub(channel);
RpcController controller = new SocketRpcController();
myService.myMethod(rpcController, myRequest,
    new RpcCallback<MyResponse>() {
      public void run(MyResponse myResponse) {
        System.out.println("Received Response: " + myResponse);
      }
    });
    
// Check success
if (rpcController.failed()) {
  System.err.println(String.format("Rpc failed %s : %s", rpcController.errorReason(),
      rpcController.errorText()));
}
```