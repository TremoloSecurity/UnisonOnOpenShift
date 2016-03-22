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

## Step-by-Step Instructions

### Persistent Volumes

Prior to deploying Unison five persistent volumes must be built and integrated with OpenShift.

| Volume | Function | Persistant Volume | Persistent Volume Claim |
| ------ | -------- | ----------------- | ----------------------- |
| apps/proxy/auth | Stores jsp files used in authentication | pv-*-apps-proxy-auth.yaml | pvc-*-apps-proxy-auth.yaml |
| apps/proxy/WEB-INF | Stores Unison's configuration and keystores | pv-*-apps-proxy-webinf.yaml | pvc-*-apps-proxy-webinf.yaml |
| apps/tremoloadmin/WEB-INF | Stores Unison's admin system configuration | pv-*-apps-tremoloadmin-webinf.yaml | pvc-*-apps-tremoloadmin-webinf.yaml |
| apps/webservices/WEB-INF | Stores Unison's web services configuration and keystores | pv-*-apps-tremoloadmin-webinf.yaml | pvc-*-apps-tremoloadmin-webinf.yaml |
| conf | The Unison server configuration | pv-*-conf.yaml | pvc-*-conf.yaml |
| ext-lib | Additional libraries uploaded to Unison | pv-*-extlib.yaml | pvc-*-extlib.yaml |

The only files you will edit are the pv-*.yaml files as these will point to where the volume should be mounted from.  If the volume will come from an NFS mount, then simply changing the path and server variables should suffice.  If using another backend (ie GlusterFS or CIFS) the pv-*.yaml files will need to be updated to reflect where the persistent volume will be stored.

Even though the Docker file defines a logs volume, this can be ignored as all logs are sent through standard out.

### Services

The templates do not define services as there can be any number of services based on host names.  A single Unison server can host any number of host names so its left to the developer to determine which hosts are best for the project.

### Deploying Unison

Once the persistent volumes are defined, the volumes and claims can be deployed.  Since OpenShift currently has no way to explicitly bind a claim to a volume its VERY IMPORTANT that the volume and the claim get created in the correct order:

`````bash
$ ./oc login
$ ./oc project my-unison-master
$ ./oc create -f /path/to/pv-master-apps-proxy-auth.yaml
$ ./oc create -f /path/to/pvc-master-apps-proxy-auth.yaml
$ ./oc create -f /path/to/pv-master-apps-proxy-webinf.yaml
$ ./oc create -f /path/to/pvc-master-apps-proxy-webinf.yaml
$ ./oc create -f /path/to/pv-master-apps-tremoloadmin-webinf.yaml
$ ./oc create -f /path/to/pvc-master-apps-tremoloadmin-webinf.yaml
$ ./oc create -f /path/to/pv-master-apps-webservices-webinf.yaml
$ ./oc create -f /path/to/pvc-master-apps-webservices-webinf.yaml
$ ./oc create -f /path/to/pv-master-apps-conf.yaml
$ ./oc create -f /path/to/pvc-master-apps-conf.yaml
$ ./oc create -f /path/to/pv-master-apps-extlib.yaml
$ ./oc create -f /path/to/pvc-master-apps-extlib.yaml
$ ./oc create -f /path/to/pod-master-unison.yaml
$ ./oc create -f /path/to/service-master-unison.yaml
`````

At this point OpenShift will download and deploy the Unison Docker image from Dockerhub.  You can create a route for the admin service from OpenShift admin console and access the Unison admin console through that route.  For instance if you define a route with the host name unison-admin.openshift.mydomain.com then you would access the admin system through https://unison-admin.openshift.mydomain.com/.  You wouldn't go directly to port 9090 because the route and the service will do that for you.
The master will need write access to the persistent volumes so the access modes is different.  Its also important to note that admins will be able to 
