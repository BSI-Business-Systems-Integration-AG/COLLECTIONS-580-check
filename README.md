# COLLECTIONS-580-check

Simple web application to test vulnerability of web container for apache commons collection issue COLLECTIONS-580.

With bug [COLLECTIONS-580](https://issues.apache.org/jira/browse/COLLECTIONS-580) a security related issue was 
reported that introduces a major vulnerability for affected servers and allows arbitrary remote code execution.

The simple test servlet tool in this repository may be used to check if your java web container is affected by 
this issue or not. Running the servlet in your container will check for the existence and accessibility of class 
`org.apache.commons.collections.functors.InvokerTransformer`.

## Background

### Preconditions for possible Attacks

An affected Apache Commons Collection (ACC) is avilable on the classpath of a Java application
  * version 3.0 or later, up to the fixed version 3.2.2
  * version 4.0 or later, up to the fixed version 4.1

For this to happen one of the following scenarios may be true
  * The application itself is using an affected ACC
  * The application is running in an affected web container

### Potential Risks

Using `Runtime.exec` the vulnerability may be used as a remote code execution (RCE) hole. Using such an approach 
allows the attacker for example to read/modify/delete files and to use this to access passwords stored in config files.

Using `System.exit` the vulnerability may be used for denial of service (DOS) attacks. The web container can simply
be shut down by the attacker.

### Affected Applications

A list of potentially affected applications/web containers is provided below:

* WebSphere (management port 8880)
* WebLogic (port 7001, redirect port)
* JBoss (JMXInvokerServlet)
* Jenkins (CLI ports)
* Applications using RMI

### Are Eclipse Scout Applications affected?

Scout applications are potentially affected if an attacker is in possession of the login credentials of the Scout application and one of the following two conditions is true:

1. Your application is running in an affected web container
2. Your application is using Eclipse Scout version 5.1 (or older) or directly includes the affected ACC

To check for the first possibility you may use the Test Servlet Tool according to <a href="#tool">these instructions</a>.

To check if your application is affected you may either include the servlet provided in this repository in your application 
or you need to verify the complete classpath of your deployed application.

<h2 id="tool">How to use the Test Servlet Tool</h2>

1. Build the WAR file
2. Deploy the WAR file to your web container
3. Check the output of the tool

### Build the WAR file

Use `mvn clean install` in the project directory to build the WAR file. After a successful build, the WAR file
can be found in sub-folder `target`.

### Deploy the WAR file to your web container

Follow your standard procedures to deploy the WAR file.

### Check the output of the tool

The outcome of this test is reported as shown below:

**Case 1: No InvokerTransformer is found**

![alt text](https://github.com/BSI-Business-Systems-Integration-AG/COLLECTIONS-580-check/raw/master/container_no_collection_save.png "Your container is save.")

Your container is save, no action regarding the container is necessary. Make sure that the individual applications deployed to the container are safe too.

**Case 2: A non-affected InvokerTransformer is found**

![alt text](https://github.com/BSI-Business-Systems-Integration-AG/COLLECTIONS-580-check/raw/master/container_save_collection.png "Your container is save.")

Your container is save, no action regarding the container is necessary. Make sure that the individual applications deployed to the container are safe too.

**Case 3: A non-affected InvokerTransformer is found**

![alt text](https://github.com/BSI-Business-Systems-Integration-AG/COLLECTIONS-580-check/raw/master/container_affected_collection.png "Your container is unsafe.")

**Warning**: In this case your container is affected by the security vulnerability. 

Check the output of the servlet for the location of jav file that introduces this vulnerability and remove this jar 
file from the class path of the web container. Alternatively replace the affected jar file with a patched jar file 
(version 3.2.2 or 4.1) available from [Apache](https://commons.apache.org/proper/commons-collections/download_collections.cgi).

Once your web container is patched, repeat the test to verify that your change has been successful.

Once your container is save, make sure that the individual applications deployed to the container are safe too.

