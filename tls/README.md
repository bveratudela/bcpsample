# Prepare and Deploy SSL Certificates

Now for SSL/TLS we will start by running the scripts in <a href="bin">here</a> in a single node in order. You may inspect each to see what they do but in summary, we create our own Certificate Authority (root CA) along with an intermediate CA which issues the signed certificates, and create x509 and Java Keystore (JKS) formats for each, along with truststores in both formats. Once created, inspect the contents of each using keytool -list -keystore keystore.jks -storepass (for JKS) and openssl x509 -in cert.pem -text -noout. You can also verify the x509 certificate validity via openssl verify -CAfile cacerts cert.pem. Notice that the certificates created must have the extensions: 

```
keyUsage = digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth, clientAuth
subjectAltName = @alt_names
```

where subjectAltName is the fully qualified name of the hostname. If a load balancer will be used for services running on the host, you must also include the fully qualified name of the load balancer in the list of alternate names. For example:

```
subjectAltName = node1.mydomain.com,loadbalancer.mydomain.com
```

The order in which to run the scripts is the following but before you do, edit 0-SSLProps.sh to ensure HOSTS is the list of fully qualified domain names for your cluster nodes along with any load balancer names if high availability will be enabled. The format is "fqdn.host.1,fqdn.load.balancer fqdn.host.2,fqdn.load.balancer ... fqdn.host.n" although you can omit the load balancer name for nodes that will not be running any load balanced roles:

```
./1-Cleanup.sh
./2-RootCA.sh
./3-IntermediateCA.sh
./4-SignedCerts.sh
./5-CreateKeystore.sh
```

Then on each host, create the directory structure for the certificates and copy each certificate to the corresponding host, and the CAcerts and cdh.truststore to all nodes:

```
mkdir -p /opt/cloudera/security/jks
mkdir -p /opt/cloudera/security/x509
mkdir -p /opt/cloudera/security/truststore
mkdir -p /opt/cloudera/security/CAcerts

cp ${HOST}.jks /opt/cloudera/security/jks
cp ${HOST}.key /opt/cloudera/security/x509
cp ${HOST}.pem /opt/cloudera/security/x509
cp pkey.pass   /opt/cloudera/security/x509
cp cdh.truststore jssecacerts /opt/cloudera/security/truststore
cp cacerts rootca.pem intermediateca.pem /opt/cloudera/security/CAcerts

chmod 400 /opt/cloudera/security/x509/pkey.pass
chmod 400 /opt/cloudera/security/x509/${HOST}.key
chmod 444 /opt/cloudera/security/x509/${HOST}.pem
chmod 444 /opt/cloudera/security/jks/${HOST}.jks

ln -s /opt/cloudera/security/x509/${HOST}.key /opt/cloudera/security/x509/sslcert.key
ln -s /opt/cloudera/security/x509/${HOST}.pem /opt/cloudera/security/x509/sslcert.pem
ln -s /opt/cloudera/security/jks/${HOST}.jks /opt/cloudera/security/jks/cdh.keystore
