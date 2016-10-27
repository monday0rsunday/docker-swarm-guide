### Setup environment

I use Vagrant for create virtual machine, all you need is installing Vagrant and VirtualBox. After that, just execute ```./build``` and use ```./vagrant ANY_OPTION``` instead of ```vagrant ANY_OPTION```

### create global overlay

Use central key-value store Consul. We need at least 3 VM instances, and a host machine(all must have installed Docker)

* Step 1: Create Consul instance (host machine)

```
docker run -d -p 8500:8500 progrium/consul -server -bootstrap
```

* Step 2: Setup Docker daemon in all VM instances (remember to leave Swarm mode )

```
cat >> /etc/default/docker <<EOF
DOCKER_OPTS="--log-level=debug --cluster-store=\"consul://${host_ip}:8500\" --cluster-advertise=\"${vm_ethx_interface}:2376\" --label=node=${vm_name}"
EOF
```

* Step 3: Restart all Docker daemon

```
service docker restart
```

### create Docker Swarm

* Step 1: Create Swarm manager in one VM

```
docker run --name swarm-master -d swarm:latest manage -H tcp://0.0.0.0:3376 --strategy spread --advertise ${vm_ip}:2376 consul://${host_ip}:8500
```

* Step 2: Join Swarm node

```
docker run --name swarm-agent -d swarm:latest join --advertise ${vm_ip}:2376 consul://${host_ip}:8500
```

* Step 3: Verify that Swarm is installed success in any VM

```
docker run --rm swarm list consul://${host_ip}:8500
```
