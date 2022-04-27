# training_plan 

<img src="plan.png">

### kubernetes namepsace resources deletion 

```
kubectl  get  all
No resources found in ashu-oci namespace.
fire@ashutoshhs-MacBook-Air ~ % kubectl  delete all --all
No resources found
fire@ashutoshhs-MacBook-Air ~ % 

```

### MULTI stage dockerfile 

### dockerfile with java app example 

```
FROM oraclelinux:8.4 AS Stage1 
LABEL email=ashutoshh@linux.com
RUN yum  install java-1.8.0-openjdk.x86_64 java-1.8.0-openjdk-devel.x86_64 maven  -y  ## layer ID 3453094545
COPY  . /javaweb/
WORKDIR /javaweb
RUN mvn clean package 
# above step for building javawebapp to .war file 

FROM tomcat 
LABEL name=ashutoshh
COPY --from=Stage1  /javaweb/target/WebApp.war /usr/local/tomcat/webapps/
# default location for java apps
EXPOSE 8080
```


