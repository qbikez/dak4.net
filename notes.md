* alternative approach to copying all src files - use a mounted volume
* splitting "home page" to a separate container - what about shared in-mem state?
* how to safely remove data owned by new microservice from database (someone may still be using it, i.e. stored procedures, reports, etc)
* UI needs to be updated for async flow
* architecture diagram should have labels

* what's the advantage of going from app service to kubectl?
* rolling updates: if there's ongoing request, does kube wait for it to finish? 
* canary updates?

* dependencies - what if some dependency die over time?
* metrics - better to split platform from app (i.e. appinsights)

traefik is a revroxy that understands docker containers - its connecting to docker streaming API and listens for events (i.e. container was created, etc.). Can do the same with Kube.

breaking up by business feature.

how to store kibana/es config


# escape=` in dockerfile

nats: an in-mem message queue (no persistence)
* good for interprocess comm between containers
* think of failure modes - maybe there's no need for persitence (at least for some messages)

dependencie: if running in a cluster, check your dependencies and if they're not available, just die - orchestrator will take care of restarting

docker-compose pull/push

docker image should have a set of default configuration (makes things just work for dev)
Look in config files or env variables for overrides

readiness: check my dependencies and die if some are not available. let platform (k8s) handle that.


metrics: docker exposes them in Prometheus format


when using 3rd party image, better to create own image from it and bake in own defaults for config