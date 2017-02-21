# OpenShift Blue/Green Pipeline Demo

This demo sets up a pipeline within OpenShift that can be shows how to do a Blue/Green Deployment into a production OpenShift environment.

It uses a very simple container from DockerHub that simply draws a square in a given color and with a given text. The color can be set through a parameter.

To deploy in a Blue/Green fashion we are following the *Two Services/One Route* approach. Note that this is just of many possible approaches to Blue/Green deployment - but it *is* the simplest one to set up. We have two deployment configurations that are running at all times. Both expose the application as a service. There is *one* route that points to the currently active service.

The Blue/Green deployment happens as follows:

* Determine which application is active (Blue or Green) by examining to which service the route currently points.
* Deploy the *other* application with the latest code (or in our case environment variable)
* Typically there would be some acceptance tests at this point - for sake of simplicity these are omitted in this demo
* Stop the Pipeline and ask for permission to switch the route over. This requires manual input. It would also be possible to do this automatically based on various tests of the newly deployed application.
* When the user approves the switch, go ahead and switch the route from the previous service to the newly deployed/updated service with no service interruption.

To set this demo up use the following commands:
[source,bash]
----
oc new-project bluegreen
oc new-app jenkins-persistent --param MEMORY_LIMIT=1Gi --param VOLUME_CAPACITY=2Gi
oc new-app openshift/deployment-example:v1 --name=example-green
oc new-app openshift/deployment-example:v2 --name=example-blue
oc set triggers dc/example-green --remove-all
oc set triggers dc/example-blue --remove-all
oc expose svc/example-green --name=example
----

* First we create a new project (*bluegreen*).
* In this project we create a Jenkins instance with persistent storage. Technically this would not be necessary - OpenShift would launch an ephemeral version of Jenkins every time a pipeline is run - but that's quite inefficient when running a pipeline multiple times.
* We create two versions of the same application. One named *example-green* and one named *example-blue*. Also one prints *v1* on the web page while the other prints *v2*.
* Since the deployment is triggered by the pipeline we turn off all triggers from our deployment configurations. This way the pipeline can control exactly what gets deployed when.
* And finally we expose the green application through a route.
