Setting up your local RabbitMQ server to use SSL
================================================

Create the rabbitmq.config file in the appropriate rabbitmq directory.  On my local rabbitmq server, it is located at /usr/local/etc/rabbitmq/rabbitmq.config  
You can use the rabbitmq.config.sample file as a template.  Update the references to the following:
	* cacertfile - Certificate created in the step below titled "Generating Server Certificate Authority"
	* certfile - The .crt file created in the step below titled "Generating for Rabbit Server SSL cert/key, signed by root-muledev-ca"
	* keyfile - The .key file created in the step below titled "Generating for Rabbit Server SSL cert/key, signed by root-muledev-ca"




Generating the key and certification:
=====================================

* Generating Server Certificate Authority:
-----------------------------------------

openssl genrsa -aes256 -out root-muledev-ca.key 4096
openssl req -x509 -new -nodes -key root-muledev-ca.key -days 1825 -out root-muledev-ca.crt

	Country Name (2 letter code) [AU]:US
	State or Province Name (full name) [Some-State]:Utah
	Locality Name (eg, city) []:West Jordan	
	Organization Name (eg, company) [Internet Widgits Pty Ltd]:Me
	Organizational Unit Name (eg, section) []:
	Common Name (e.g. server FQDN or YOUR name) []:localhost
	Email Address []:ericparshall@gmail.com


* Generating for Rabbit Server SSL cert/key, signed by root-muledev-ca:
-----------------------------------------------------------------------

openssl genrsa -out localhost.key 2048
openssl req -new -key localhost.key -out localhost.csr
	
	Country Name (2 letter code) [AU]:US
	State or Province Name (full name) [Some-State]:Utah
	Locality Name (eg, city) []:West Jordan
	Organization Name (eg, company) [Internet Widgits Pty Ltd]:Me
	Organizational Unit Name (eg, section) []:
	Common Name (e.g. server FQDN or YOUR name) []:localhost
	Email Address []:ericparshall@gmail.com
	
	Please enter the following 'extra' attributes
	to be sent with your certificate request
	A challenge password []:heehaw
	An optional company name []:

openssl x509 -req -in localhost.csr -CA root-muledev-ca.crt -CAkey root-muledev-ca.key -CAcreateserial -out localhost.crt -days 1825

openssl pkcs12 -export -in localhost.crt -inkey localhost.key -out localhost.p12 -name localhost -CAfile root-muledev-ca.crt -caname root
	Enter Export Password: heehaw
	Verifying - Enter Export Password: heehaw
	
keytool -importkeystore -destkeystore localhost.ks -srckeystore localhost.p12 -srcstoretype pkcs12 -alias localhost
	Enter destination keystore password: heehaw
	Re-enter new password: heehaw
	Enter source keystore password: heehaw


* Generating Client cert/key, signed by root-muledev-ca:
-------------------------------------------------------

openssl genrsa -out localhost-client.key 2048

openssl req -new -key localhost-client.key -out localhost-client.csr
	Country Name (2 letter code) [AU]:US
	State or Province Name (full name) [Some-State]:Utah
	Locality Name (eg, city) []:West Jordan
	Organization Name (eg, company) [Internet Widgits Pty Ltd]:Me
	Organizational Unit Name (eg, section) []:
	Common Name (e.g. server FQDN or YOUR name) []:localhost-client
	Email Address []:ericparshall@gmail.com
	
	Please enter the following 'extra' attributes
	to be sent with your certificate request
	A challenge password []:heehaw
	An optional company name []:
	
openssl x509 -req -in localhost-client.csr -CA root-muledev-ca.crt -CAkey root-muledev-ca.key -CAcreateserial -out localhost-client.crt -days 1825
	Enter pass phrase for root-muledev-ca.key: heehaw
	
openssl pkcs12 -export -in localhost-client.crt -inkey localhost-client.key -out localhost-client.p12 -name localhost-client -CAfile root-muledev-ca.crt -caname root
	Enter Export Password: heehaw
	Verifying - Enter Export Password: heehaw
	
keytool -importkeystore -destkeystore localhost-client.ks -srckeystore localhost-client.p12 -srcstoretype pkcs12 -alias localhost-client
	Enter destination keystore password: heehaw
	Re-enter new password: heehaw
	Enter source keystore password: heehaw
	
	
* Adding Server Certificate to trust store:
------------------------------------------

keytool -importkeystore -srckeystore localhost.ks -destkeystore localhost-client.ks -srcstorepass "heehaw" -deststorepass "heehaw" -srcalias localhost -destalias localhost



* Verify that both certificates are now in the keystore:
--------------------------------------------------------

keytool -list -v -keystore localhost-client.ks

	You should see:
		Your keystore contains 2 entries

		Alias name: localhost-client
		...
		Alias name: localhost