# Wildfly Hystrix and Turbine Example

__This example is based on the work of<br>
https://github.com/siamaksade/wildfly-swarm-hystrix-example <br>
This is the version of the original example for Wildfly/EAP (the original version is for Wildfly Swarm).
The implementation of Employee Service and Payroll Service is the same of the original version.
Thanks for the big work!__

Wildfly Hystrix sample is an implementation in java of the circuit breaker pattern.<br>
For additional details http://microservices.io/patterns/reliability/circuit-breaker.html

The circuit breaker is implemented using the Netflix Hystrix library https://github.com/Netflix/Hystrix

This example is composed of:
* __Employee REST Service:__ service that returns the list of employees. This service randomly generates error and timed-out responses.
* __Payroll REST Service:__ service that invokes Employee service using Netflix Hystrix
* __Turbine:__ Netflix components for aggregating streams of json data
* __Hystrix Dashboard:__ a dashboard for visualizing aggregated data streams


# Run on OpenShift

Employee Service, Payroll Service and Turbine will be created as new app in OCP using the eap7 image (you can also use a wildfly image)

Hystrix Dashboard will be created as new app in OCP using a docker image from docker hub: 
__mlabouardy/hystrix-dashboard:latest__

These are the commands to run on OCP to create the entire project:

	$ oc new-project hystrix-wildfly
	$ oc policy add-role-to-user cluster-reader system:serviceaccount:hystrix-wildfly:default

	$ oc new-app eap70-openshift~https://github.com/hifly81/wildfly-hystrix-sample --context-dir=employee-service --name=employee-app
    $ oc new-app eap70-openshift~https://github.com/hifly81/wildfly-hystrix-sample --context-dir=payroll-service --name=payroll-app -l hystrix.enabled='true'
    $ oc new-app eap70-openshift~https://github.com/hifly81/wildfly-hystrix-sample --context-dir=turbine --name=turbine
    $ oc new-app mlabouardy/hystrix-dashboard:latest --name=hystrix-dashboard

    $ oc expose service employee-app
    $ oc expose service payroll-app
    $ oc expose service turbine
    $ oc expose service hystrix-dashboard


Employee service and Payroll service will exposed at:<br>
http://employee-app-hystrix-wildfly.127.0.0.1.nip.io/employees<br>
http://payroll-app-hystrix-wildfly.127.0.0.1.nip.io/payroll


# Hystrix Dashboard

The dashboard will be available at<br>
http://hystrix-dashboard-hystrix-wildfly.127.0.0.1.nip.io/hystrix

Hystrix dashboard must be configured to add the stream from Turbine; the url of the stream is:<br>
http://<service_internal_ip>:8080/turbine-1.0.0-SNAPSHOT/turbine.stream

# How to test

In order to generate some load to Payroll Service you can use the command:

	$ ab -n 100 http://payroll-app-hystrix-wildfly.127.0.0.1.nip.io/payroll

Hystrix dashboard will show if the circuit is opened or closed during the execution of the test.
