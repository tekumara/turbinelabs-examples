# CloudFormation + NLB + Envoy with Rotor

Forked from [turbinelabs/examples](https://github.com/turbinelabs/examples/tree/master/rotor-nlb)

[cloud-formation-server.yaml](https://github.com/turbinelabs/examples/blob/master/rotor-nlb/cloud-formation-server.yaml) - creates an EC2 instance with an application on port 8080 that returns a color in hex, eg: `1B9AE4`. The EC2 instance has tags:

|Key                            |Value |
|-------------------------------|------|
|tbn:cluster:server:8080        |server|
|tbn:cluster:server:8080:stage  |prod  |
|tbn:cluster:server:8080:version|0.18.2|

[cloud-formation-client.yaml](https://github.com/turbinelabs/examples/blob/master/rotor-nlb/cloud-formation-client.yaml) - creates an EC2 instance with an application on port 8080 that returns an HTML web client that read from the server. The EC2 instance has tags:

|Key                            |Value |
|-------------------------------|------|
|tbn:cluster:client:8080        |client|
|tbn:cluster:client:8080:stage  |prod  |
|tbn:cluster:client:8080:version|0.18.2|

[cloud-formation-envoy.yaml](https://github.com/turbinelabs/examples/blob/master/rotor-nlb/cloud-formation-envoy.yaml) creates
* Two envoy EC2 instances (running [envoy-simple](https://github.com/turbinelabs/envoy-simple)) in an ASG with a NLB in front of them. The Envoy admin interface is configured on port 9999 (access is disabled by default by the Security Group)
* A [Rotor](https://github.com/turbinelabs/rotor) instance. Rotor reads [EC2 instance tags](https://github.com/turbinelabs/rotor#ec2) of the format `tbn:cluster:<cluster-name>:<port-name>` and uses that to configure Envoy.

All EC2 instances are created with public IPs, although restricted by Security Groups and your VPC Network ACLs / routes.

# Ops

Uses [coreos](https://coreos.com/os/docs/latest/booting-on-ec2.html).

Use the `core` user to login, eg: `ssh core@....`

[Logs](https://coreos.com/os/docs/latest/reading-the-system-log.html) are available via journalctl, eg: 
```
journalctl -u tbnproxy.service
journalctl -u Rotor.service
```

Access the apps by specifying the `Host` header so Envoy knows which instance to route to 
```
curl -v -H 'Host: client' ip-172-25-206-164.ap-southeast-2.compute.internal
curl -v -H 'Host: server' ip-172-25-206-164.ap-southeast-2.compute.internal
```

# TODO

* AZ awareness
* Stats

# References

* https://www.learnenvoy.io/articles/front-proxy.html
* https://docs.turbinelabs.io/introduction/#houston
* https://docs.turbinelabs.io/advanced/ec2.html