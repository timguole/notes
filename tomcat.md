# Tomcat Basics

## Installation

### Setup JDK

Download and setup JDK

```shell
tar -zxf jdk-22_linux-x64_bin.tar.gz
echo <<EOF >> ~/.bashrc
export JAVA_HOME=/home/gle/jdk-22.0.1
export JRE_HOME=/home/gle/jdk-22.0.1
export PATH=$PATH:$JAVA_HOME/bin;
EOF
source .bashrc
java -version # this will print java version information
```

### Setup tomcat

```shell
tar -zxf apache-tomcat-10.1.24.tar.gz
```

## Start & stop tomcat

```shell
TOMCAT/bin/starup.sh
TOMCAT/bin/shutdown.sh
```

### Deploy web applications

```shell
cp webapp.war TOMCAT/webapps/
```



## Configure tomcat

#### Configure HTTPS

First, generate a certificate with keytool

```shell
# use keytool (provided by JDK) to generate a x509 certificate
keytool -genkey -alias tomcat -keyalg RSA -validity 365
```

Then, configure tomcat to use the certificate. Edit file: TOMCAT/conf/server.xml

```xml
<Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
		maxThreads="150" SSLEnabled="true"
		maxParameterCount="1000">
	<UpgradeProtocol className="org.apache.coyote.http2.Http2Protocol" />
	<SSLHostConfig>
		<Certificate certificateKeystoreFile="${user.home}/.keystore"
				certificateKeystorePassword="tomcat10"
				type="RSA" />
	</SSLHostConfig>
</Connector>
```



#### security-constraint

Here shows how to use the `<security-constraint>`. In TOMCAT/conf/web.xml, add the following under `<web-app>` tag.

> IMPORTANT:
>
> It is very important to understand how security-constraint works before action. The security-constraint consists of two parts: web-resource-collection and auth-constraint. A security-constraint means resources (URLs and http method on those URLs) defined by web-resource-collection is controlled by auth-constraint. The tag http-method means this method on the URL is controlled by auth-constraint (like a blacklist, all other methods are not affected). The tag http-method-omission means this method on URL is not controlled by the auth-constraint (like a whitelist, all other methods are affected). These two tags can not be used at the same time.
>
> For example, the following snippet means accessing URLs using all methods except GET and POST is denied. (because auth-constrainst is empty)

```xml
	<security-constraint>
		<web-resource-collection>
			<web-resource-name>all</web-resource-name>
			<url-pattern>/*</url-pattern>
			<http-method-omission>GET</http-method-omission>
			<http-method-omission>POST</http-method-omission>
			<!-- The below lines means all these methods needs authentication
			<http-method>GET</http-method>
			<http-method>POST</http-method>
			<http-method>PUT</http-method>
			<http-method>DELETE</http-method>
			<http-method>PATCH</http-method>
			<http-method>OPTIONS</http-method>
			<http-method>HEAD</http-method>
			<http-method>TRACE</http-method>
			-->
		</web-resource-collection>
		<auth-constraint>
			<!-- If use the below role-name tag, the security-role is at the end of this snippet is needed. And users in this role can be defined in TOMCAT/conf/tomcat-users.xml.
			<role-name>httpbasicauth</role-name>
			-->
		</auth-constraint>
		<user-data-constraint>
			<!-- transport-guarantee can be CONFIDENTIAL, INTEGRAL, or NONE -->
			<transport-guarantee>NONE</transport-guarantee>
		</user-data-constraint>
	</security-constraint>
	<!-- This tag works together with role-name above.
	<security-role>
		<role-name>httpbasicauth</role-name>
	</security-role>
	-->
```

If user authentication is needed, uncomment the role-name and security-role tags above and define users under the tomcat-users tag in TOMCAT/conf/tomcat-users.xml.

```xml
<role rolename="httpbasicauth"/>
  <user username="tom" password="tom" roles="httpbasicauth"/>
  <user username="tim" password="tim" roles="httpbasicauth"/>
  <user username="alice" password="alice" roles="httpbasicauth"/>
```

