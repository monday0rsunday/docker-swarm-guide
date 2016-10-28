### Setup environment

I use Vagrant for create virtual machine, all you need is installing Vagrant and VirtualBox. After that, just execute ```./build``` and use ```./vagrant ANY_OPTION``` instead of ```vagrant ANY_OPTION```

### create global overlay

Use central key-value store Consul. We need at least 3 VM instances, and a host machine(all must have installed Docker)

* Step 1: Create Consul instance (host machine)

```
docker run -d -p 8500:8500 --restart always progrium/consul -server -bootstrap
```

* Step 2: Setup Docker daemon in all VM instances (remember to leave Swarm mode )

```
cat >> /etc/default/docker <<EOF
DOCKER_OPTS="-H tcp://0.0.0.0:2376 --log-level=debug --cluster-store=\"consul://${host_ip}:8500\" --cluster-advertise=\"${vm_ethx_interface}:2376\""
EOF
```

* Step 3: Restart all Docker daemon

```
service docker restart
```

### create Docker Swarm

* Step 1: Create Swarm manager in one VM

```
docker -H :2376 run --name swarm-master --publish 4000:4000 -d swarm:latest manage -H tcp://0.0.0.0:4000 --strategy spread --advertise ${vm_ip}:4000 consul://${host_ip}:8500
```

* Step 2: Join Swarm node

```
docker -H :2376 run --name swarm-agent -d swarm:latest join --advertise ${vm_ip}:2376 consul://${host_ip}:8500
```

* Step 3: Verify that Swarm is installed success in any VM

```
docker -H tcp://${manager_ip}:4000 info
docker -H :2376 run --rm swarm list consul://${host_ip}:8500
```

* Step 4: Start any container. If there's any problem with daemon tcp lookup dns, rememeber correct DNS server of VMs in /etc/resolv.conf and restart corresponsding Docker

```
docker -H tcp://${manager_ip}:4000 run --rm nginx
```
