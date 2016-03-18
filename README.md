# Unison On OpenShift

The stock Unison image on Dockerhub (https://hub.docker.com/r/tremolosecurity/unison/) runs on OpenShift out-of-the-box without modification.  This repo contains example yaml files for deploying Unison on OpenShift.  This build was tested on OpenShift Origin 3.2.  

## Master/Slave & Clustering

Unison on OpenShift can not use the standard push mechanism for clustering as pod deployments will be dynamic and IPs are not static.  In order to manage Unison its recommended to run a Master project (with read write access to the persistent volumes) on non-public nodes and Slaves (read-only access to persistent volumes) on "public" nodes.  Once Unison is installed, its important to enable the shared file system under "Admin System".  After making updates to the master, push the changes the way you would in non OpenShift deployments.  The push will leave markers on the file systems that will tell the slave Unisons to pull the configuration from disk.

## Routes

When deploying the Unison master its important to create a route to the service on 9090 so that you are able to access the Unison admin system.  Routes should also be created for each "host" in Unison.  For instance if an application URL defines 3 hosts that can be used by users, then one route for each host must be created.  Finally, configure the route to use "Passthrough" for TLS.

## Logging

All logging is sent to standard out per OpenShfit and Docker best practices

## Persistent Volumes

At this time OpenShift does not provide a capability to explicitly bind a persistent volume to a persistent volume claim without first created the persistent volume and then IMMEDIATELY created the claim on that volume.  This will be a feature in the next version of OpenShift.  Until then, its important to create the persistent volume and its claim in tandum to ensure that the claim binds to the volume correctly.
