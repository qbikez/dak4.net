* alternative approach to copying all src files - use a mounted volume
* splitting "home page" to a separate container - what about shared in-mem state?
* how to safely remove data owned by new microservice from database (someone may still be using it, i.e. stored procedures, reports, etc)

traefik is a revroxy that understands docker containers - its connecting to docker streaming API and listens for events (i.e. container was created, etc.). Can do the same with Kube.

breaking up by business feature.

# escape=` in dockerfile