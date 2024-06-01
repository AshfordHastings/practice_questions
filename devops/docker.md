# Docker Questions

## Networking

### What does setting external=True on a Docker network do?  

Setting external=True on a Docker network configuration in a docker-compose.yml file indicates that the network is an existing one created outside the docker-compose context. Docker Compose will not attempt to create or manage this network; it will simply use it if it already exists. This allows for integration with pre-defined networks and facilitates communication between containers managed by different Compose files or Docker setups.

