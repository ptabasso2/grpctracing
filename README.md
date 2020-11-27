# grpctracing

## Instructions


A detailed step-by-step showing how tracing can be implemented for Grpc.
This tutorial is based on the example from the official grpc-go repo. 

**_Preliminary tasks and first time steps_**

Pull image (docker pull pejdd/grpc:v0) and run the container:

``` 
MacOSX:grpctracing - root$ docker run -it --name grpc -h grpc -d pejdd/grpc:v0
bc60db3f5bf7d1d25f79d2cbbfe1288233ffb95de8a6cdb06e62b9f3052f8c83
MacOSX:grpctracing - root$ docker exec -it grpc bash

[root@grpc:~/go/grpc]$ tree
.
|-- go.mod
|-- go.sum
|-- google.golang.org
|   `-- grpc
|       `-- examples
|           `-- helloworld
|               `-- helloworld
|                   `-- grpc.pb.go
|-- grpc.proto
|-- grpc_client
|-- grpc_client.go
|-- grpc_server
|-- grpc_server.go
`-- grpchw.tar.gz

5 directories, 9 files


```

At this stage the grpc server is already running and waiting for the client to connect.

**_Spin up the Datadog Agent (Provide your API key  to the  below command)_** 


```
COMP10619:Kafka pejman.tabassomi$ DOCKER_CONTENT_TRUST=1 docker run -d --rm --name datadog_agent -h datadog \ 
-v /var/run/docker.sock:/var/run/docker.sock:ro -v /proc/:/host/proc/:ro -v /sys/fs/cgroup/:/host/sys/fs/cgroup:ro \
-p 8126:8126 -p 8125:8125/udp -e DD_API_KEY=<Api key to enter> -e DD_APM_ENABLED=true \
-e DD_APM_NON_LOCAL_TRAFFIC=true -e DD_PROCESS_AGENT_ENABLED=true -e DD_DOGSTATSD_NON_LOCAL_TRAFFIC="true" \ 
-e DD_LOG_LEVEL=debug datadog/agent:7
```


**_Create a bridge network_**

```
MacOSX:grpctracing - root$ docker network create nw0
```


**_connect both containers to the newly created network_**

```
MacOSX:grpctracing - root$ docker network connect nw0 grpc
MacOSX:grpctracing - root$ docker network connect nw0 datadog_agent
```

**_ssh into the grpc container and run the grpc_client_**

```
MacOSX:grpctracing - root$ docker exec -it grpc bash
[root@grpc:~/go/grpc]$ ./grpc_client
...
2020/11/27 11:09:14 Datadog Tracer v1.27.0 INFO: DATADOG TRACER CONFIGURATION {"date":"2020-11-27T11:09:14Z","os_name":"Linux (Unknown Distribution)","os_version":"18.04.4 LTS (Bionic Beaver)","version":"v1.27.0","lang":"Go","lang_version":"go1.14.12","env":"","service":"my-grpc-client","agent_url":"http://datadog_agent:8126/v0.4/traces","agent_error":"Post \"http://datadog_agent:8126/v0.4/traces\": dial tcp: lookup datadog_agent on 192.168.65.1:53: no such host","debug":true,"analytics_enabled":true,"sample_rate":"NaN","sampling_rules":null,"sampling_rules_error":"","tags":{"runtime-id":"8d67f3c6-59f0-446e-84e2-4f65432de144"},"runtime_metrics_enabled":false,"health_metrics_enabled":false,"dd_version":"","architecture":"amd64","global_service":"","lambda_mode":"false"}
2020/11/27 11:09:14 Greeting: Hello world
2020/11/27 11:09:16 Datadog Tracer v1.27.0 DEBUG: Sending payload: size: 433 traces: 1

```


<br>


