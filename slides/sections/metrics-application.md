# Exporting Application Metrics

---

The next level of detail is application-level metrics, recording details about what your app is doing. You surface those through a metrics API in the same way as the runtime metrics.

We'll add application metrics to the message handlers, so we can see the flow of messages through the system.

---

## Expose metrics from the message handlers

The message handlers already have code to record metrics when they handle messages. 

You can see this in the [Program.cs](./src/SignUp.MessageHandlers.SaveProspect/Program.cs) file for the SQL Server handler, and the [QueueWorker.cs](./src/SignUp.MessageHandlers.IndexProspect/Workers/QueueWorker.cs) file for the Elasticsearch handler.

> Both handlers use a community Prometheus package on NuGet, [prometheus-net](https://github.com/prometheus-net/prometheus-net). It's a .NET Standard library, so you can use it from .NET Framework and .NET Core apps.

---

## About Prometheus metrics

Prometheus uses a time-series database - it grabs metrics on a schedule and stores every value along with a timestamp. You can aggregate across dimensions or drill down to specific values.

You should record metrics at a fairly coarse level - "Event count" in this example. Then add detail with labels, like the processing status and the hostname of the handler. 

---

## Build new versions of the handlers

There's a new [Dockerfile for the save handler](./docker/metrics-application/save-handler/Dockerfile) and a new [Dockerfile for the index handler](./docker/metrics-application/index-handler/Dockerfile). They package the same code, but they set default config values to enable the metrics API.

```
docker image build -t dak4dotnet/save-handler:v2 `
  -f .\docker\metrics-application\save-handler\Dockerfile . ; `

docker image build -t dak4dotnet/index-handler:v2 `
  -f .\docker\metrics-application\index-handler\Dockerfile .
```

> The build should be super-fast, because of the cache.

---

## Run the new save message handler

You can run containers with the new message handler apps to see what sort of metrics they expose.

_Run the new version of the SQL Server handler:_

```
docker container run -d  `
  -e ConnectionStrings:SignUpDb='Server=signup-db;Database=SignUp;User Id=sa;Password=DockerCon!!!' `
  --name save-v2 dak4dotnet/save-handler:v2
```

---

## Check the save metrics

The save message handler is a .NET Framework console app. The Prometheus NuGet package adds a self-hosted HTTP server for the metrics API.

The run command didn't publish any ports, but you can still browse to the metrics endpoint using the container's IP address.

_Check out the metrics:_

```
$ip = docker container inspect --format '{{ .NetworkSettings.Networks.nat.IPAddress }}' save-v2; `

firefox "http://$($ip):50505/metrics"
```

> Port `50505` isn't standard, it's just the port I've chosen. 

---

## Run the new index message handler

The index message handler records similar metrics about messages handled, and the processing status.

_Run the new version of the Elasticsearch handler:_

```
docker container run -d --name index-v2 dak4dotnet/index-handler:v2
```

---

## Check the index metrics

The save message handler is a .NET Core console app. The same Prometheus NuGet package publishes the metrics API with a self-hosted web server.

_Check out the metrics using the container's IP address:_

```
$ip = docker container inspect --format '{{ .NetworkSettings.Networks.nat.IPAddress }}' index-v2; `

firefox "http://$($ip):50505/metrics"
```

> The raw data is very basic. Prometheus will make it more useful.

---

## Tidy up

Now we know how the metrics look, let's remove the new containers:

```
@('save-v2', 'index-v2') | foreach { docker container rm -f $_}
```

> This is just some fancy PowerShell to loop over a list of container names and remove them.

---

## Application metrics

Metrics about what your application is actually doing give you useful insight into your application health, and how work is being distribuited among containers.

You need to write code to get this level of detail, but all the major languages have Prometheus client libraries which make it very easy to capture and export metrics.
