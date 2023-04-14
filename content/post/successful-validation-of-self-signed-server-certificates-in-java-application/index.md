---
title: "Successful Validation of self-signed Server certificates in Java Application" # Title of the blog post.
date: 2023-04-14T10:01:42+01:00 # Date of post creation.
description: "Tutorial how to handle self-signed server certificate in a Java application" # Description used for search engine.
featured: false # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
#featureImage: "/images/path/file.jpg" # Sets featured image on blog post.
#thumbnail: "/images/path/thumbnail.png" # Sets thumbnail image appearing inside card on homepage.
#shareImage: "/images/path/share.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Java
  - JVM
tags:
  - ssl
  - keytool
  - certificate
  - truststore
  - java
# comment: false # Disable comment if false.
---

When your Java application wants to communicate with a server that used self-signed certificate, you get very often a `javax.net.ssl.SSLHandshakeException`, because the validation of self signed public key fails. 
It is possible to ignore this validation. 
But this is not recommended because of it simplifies a man-in-the-middle-attack.

The best solution is to avoid using self-signed certificate, but sometimes it is not possible (I know, the reasons are very often nonsense, but this case is reality, so you should handle with that). 
For this case, you can add the public key of the self-signed server certificate in your truststore of your JVM, so that the Java application can validate the certificate successfully.


Download the public key of the server with OpenSSL
-------------------
First, you need the public key of the server in a PEM file format.
You can download it with the [OpenSSL](https://www.openssl.org/) tool.

````shell
openssl s_client -conntect server.example.com:443 | openssl x509 -text > server-public-key.pem
````

The above one-liner is doing three things. 
Let's explain it in an example:

The first part downloads the public key and other information about this secure connection.
````shell
➜ openssl s_client -connect www.google.de:443
CONNECTED(00000003)
depth=2 C = US, O = Google Trust Services LLC, CN = GTS Root R1
verify return:1
depth=1 C = US, O = Google Trust Services LLC, CN = GTS CA 1C3
verify return:1
depth=0 CN = www.google.de
verify return:1
---
Certificate chain
 0 s:CN = www.google.de
   i:C = US, O = Google Trust Services LLC, CN = GTS CA 1C3
   a:PKEY: id-ecPublicKey, 256 (bit); sigalg: RSA-SHA256
   v:NotBefore: Mar 28 16:54:30 2023 GMT; NotAfter: Jun 20 16:54:29 2023 GMT
 1 s:C = US, O = Google Trust Services LLC, CN = GTS CA 1C3
   i:C = US, O = Google Trust Services LLC, CN = GTS Root R1
   a:PKEY: rsaEncryption, 2048 (bit); sigalg: RSA-SHA256
   v:NotBefore: Aug 13 00:00:42 2020 GMT; NotAfter: Sep 30 00:00:42 2027 GMT
 2 s:C = US, O = Google Trust Services LLC, CN = GTS Root R1
   i:C = BE, O = GlobalSign nv-sa, OU = Root CA, CN = GlobalSign Root CA
   a:PKEY: rsaEncryption, 4096 (bit); sigalg: RSA-SHA256
   v:NotBefore: Jun 19 00:00:42 2020 GMT; NotAfter: Jan 28 00:00:42 2028 GMT
---
Server certificate
-----BEGIN CERTIFICATE-----
MIIEhTCCA22gAwIBAgIRAKSN7Wi9JF2/CsOwEh+bjLUwDQYJKoZIhvcNAQELBQAw
RjELMAkGA1UEBhMCVVMxIjAgBgNVBAoTGUdvb2dsZSBUcnVzdCBTZXJ2aWNlcyBM
TEMxEzARBgNVBAMTCkdUUyBDQSAxQzMwHhcNMjMwMzI4MTY1NDMwWhcNMjMwNjIw
MTY1NDI5WjAYMRYwFAYDVQQDEw13d3cuZ29vZ2xlLmRlMFkwEwYHKoZIzj0CAQYI
KoZIzj0DAQcDQgAEZKedP4Y2ToNbBEWdSO58hrDZ/1koQ6BiRciDoK+exh/DrHa/
J0Cqmu6NR8b5wsIuBwspTDGT4GMf7pGdmgk6YqOCAmUwggJhMA4GA1UdDwEB/wQE
AwIHgDATBgNVHSUEDDAKBggrBgEFBQcDATAMBgNVHRMBAf8EAjAAMB0GA1UdDgQW
BBSytm0ZL0I9iGV2JRr81XwGijETPzAfBgNVHSMEGDAWgBSKdH+vhc3ulc09nNDi
RhTzcTUdJzBqBggrBgEFBQcBAQReMFwwJwYIKwYBBQUHMAGGG2h0dHA6Ly9vY3Nw
LnBraS5nb29nL2d0czFjMzAxBggrBgEFBQcwAoYlaHR0cDovL3BraS5nb29nL3Jl
cG8vY2VydHMvZ3RzMWMzLmRlcjAYBgNVHREEETAPgg13d3cuZ29vZ2xlLmRlMCEG
A1UdIAQaMBgwCAYGZ4EMAQIBMAwGCisGAQQB1nkCBQMwPAYDVR0fBDUwMzAxoC+g
LYYraHR0cDovL2NybHMucGtpLmdvb2cvZ3RzMWMzL3pkQVR0MEV4X0ZrLmNybDCC
AQMGCisGAQQB1nkCBAIEgfQEgfEA7wB2AK33vvp8/xDIi509nB4+GGq0Zyldz7EM
JMqFhjTr3IKKAAABhylbcmgAAAQDAEcwRQIgEsP7rZrJh/4X9uymxCyls+cLlq/m
WoG2K8i6qU/5F0ECIQCG6TiACSK06fyl9vdKJ1KkoyKIQu6Ca9gncpemT/ou7QB1
ALNzdwfhhFD4Y4bWBancEQlKeS2xZwwLh9zwAw55NqWaAAABhylbcyUAAAQDAEYw
RAIgBhXpY3IRHWsU0BVEtm8fgqYsSGwZK/5sxWnnxJUp56UCIBbra//rG7Im33Mj
QD2hzZ5dj48z5S47v6USlG2s1qXaMA0GCSqGSIb3DQEBCwUAA4IBAQDfmn44xaxh
7kn3mWgxLtXt8RADmf5GamsYUgoFG3dagRKzawwwuwYxBMsu0fQrEupK0fCH+w7B
a+c8/XJyIp8mo9iwgbswQf4atAwY6qp5C8sdYHH8e00vHvhEpRD7ak2DHJMuyPA6
Cm7+g6snDJUrVbO4VoCsQOmc8G74vq3zcQ1Eex00/MHERqSS6CbGnSKfmQtn7FlI
z0AM7ES1A58EHWz1WcovUq4J0EcccRxFIgA4Z65u5CspHPwaQceUXM1d+ljO+qkE
Pgi6Ixx0UGjJI6MQt2s5LTL51w3fVMKtta0kDf1DKWSMxdkaSWY9jpln1FQ9jaOy
apK7ZBQWO48R
-----END CERTIFICATE-----
subject=CN = www.google.de
issuer=C = US, O = Google Trust Services LLC, CN = GTS CA 1C3
---
No client certificate CA names sent
Peer signing digest: SHA256
Peer signature type: ECDSA
Server Temp Key: X25519, 253 bits
---
SSL handshake has read 4293 bytes and written 395 bytes
Verification: OK
---
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
Server public key is 256 bit
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
Early data was not sent
Verify return code: 0 (ok)
---

````
You can extract the public key information if you pipe the above output through `openssl` again.
It produces a human-readable version of the public key.

````shell
➜ openssl s_client -connect www.google.de:443 | openssl x509 -text
depth=2 C = US, O = Google Trust Services LLC, CN = GTS Root R1
verify return:1
depth=1 C = US, O = Google Trust Services LLC, CN = GTS CA 1C3
verify return:1
depth=0 CN = www.google.de
verify return:1
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            a4:8d:ed:68:bd:24:5d:bf:0a:c3:b0:12:1f:9b:8c:b5
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: C = US, O = Google Trust Services LLC, CN = GTS CA 1C3
        Validity
            Not Before: Mar 28 16:54:30 2023 GMT
            Not After : Jun 20 16:54:29 2023 GMT
        Subject: CN = www.google.de
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    04:64:a7:9d:3f:86:36:4e:83:5b:04:45:9d:48:ee:
                    7c:86:b0:d9:ff:59:28:43:a0:62:45:c8:83:a0:af:
                    9e:c6:1f:c3:ac:76:bf:27:40:aa:9a:ee:8d:47:c6:
                    f9:c2:c2:2e:07:0b:29:4c:31:93:e0:63:1f:ee:91:
                    9d:9a:09:3a:62
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature
            X509v3 Extended Key Usage: 
                TLS Web Server Authentication
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Subject Key Identifier: 
                B2:B6:6D:19:2F:42:3D:88:65:76:25:1A:FC:D5:7C:06:8A:31:13:3F
            X509v3 Authority Key Identifier: 
                8A:74:7F:AF:85:CD:EE:95:CD:3D:9C:D0:E2:46:14:F3:71:35:1D:27
            Authority Information Access: 
                OCSP - URI:http://ocsp.pki.goog/gts1c3
                CA Issuers - URI:http://pki.goog/repo/certs/gts1c3.der
            X509v3 Subject Alternative Name: 
                DNS:www.google.de
            X509v3 Certificate Policies: 
                Policy: 2.23.140.1.2.1
                Policy: 1.3.6.1.4.1.11129.2.5.3
            X509v3 CRL Distribution Points: 
                Full Name:
                  URI:http://crls.pki.goog/gts1c3/zdATt0Ex_Fk.crl
            CT Precertificate SCTs: 
                Signed Certificate Timestamp:
                    Version   : v1 (0x0)
                    Log ID    : AD:F7:BE:FA:7C:FF:10:C8:8B:9D:3D:9C:1E:3E:18:6A:
                                B4:67:29:5D:CF:B1:0C:24:CA:85:86:34:EB:DC:82:8A
                    Timestamp : Mar 28 17:54:31.656 2023 GMT
                    Extensions: none
                    Signature : ecdsa-with-SHA256
                                30:45:02:20:12:C3:FB:AD:9A:C9:87:FE:17:F6:EC:A6:
                                C4:2C:A5:B3:E7:0B:96:AF:E6:5A:81:B6:2B:C8:BA:A9:
                                4F:F9:17:41:02:21:00:86:E9:38:80:09:22:B4:E9:FC:
                                A5:F6:F7:4A:27:52:A4:A3:22:88:42:EE:82:6B:D8:27:
                                72:97:A6:4F:FA:2E:ED
                Signed Certificate Timestamp:
                    Version   : v1 (0x0)
                    Log ID    : B3:73:77:07:E1:84:50:F8:63:86:D6:05:A9:DC:11:09:
                                4A:79:2D:B1:67:0C:0B:87:DC:F0:03:0E:79:36:A5:9A
                    Timestamp : Mar 28 17:54:31.845 2023 GMT
                    Extensions: none
                    Signature : ecdsa-with-SHA256
                                30:44:02:20:06:15:E9:63:72:11:1D:6B:14:D0:15:44:
                                B6:6F:1F:82:A6:2C:48:6C:19:2B:FE:6C:C5:69:E7:C4:
                                95:29:E7:A5:02:20:16:EB:6B:FF:EB:1B:B2:26:DF:73:
                                23:40:3D:A1:CD:9E:5D:8F:8F:33:E5:2E:3B:BF:A5:12:
                                94:6D:AC:D6:A5:DA
    Signature Algorithm: sha256WithRSAEncryption
    Signature Value:
        df:9a:7e:38:c5:ac:61:ee:49:f7:99:68:31:2e:d5:ed:f1:10:
        03:99:fe:46:6a:6b:18:52:0a:05:1b:77:5a:81:12:b3:6b:0c:
        30:bb:06:31:04:cb:2e:d1:f4:2b:12:ea:4a:d1:f0:87:fb:0e:
        c1:6b:e7:3c:fd:72:72:22:9f:26:a3:d8:b0:81:bb:30:41:fe:
        1a:b4:0c:18:ea:aa:79:0b:cb:1d:60:71:fc:7b:4d:2f:1e:f8:
        44:a5:10:fb:6a:4d:83:1c:93:2e:c8:f0:3a:0a:6e:fe:83:ab:
        27:0c:95:2b:55:b3:b8:56:80:ac:40:e9:9c:f0:6e:f8:be:ad:
        f3:71:0d:44:7b:1d:34:fc:c1:c4:46:a4:92:e8:26:c6:9d:22:
        9f:99:0b:67:ec:59:48:cf:40:0c:ec:44:b5:03:9f:04:1d:6c:
        f5:59:ca:2f:52:ae:09:d0:47:1c:71:1c:45:22:00:38:67:ae:
        6e:e4:2b:29:1c:fc:1a:41:c7:94:5c:cd:5d:fa:58:ce:fa:a9:
        04:3e:08:ba:23:1c:74:50:68:c9:23:a3:10:b7:6b:39:2d:32:
        f9:d7:0d:df:54:c2:ad:b5:ad:24:0d:fd:43:29:64:8c:c5:d9:
        1a:49:66:3d:8e:99:67:d4:54:3d:8d:a3:b2:6a:92:bb:64:14:
        16:3b:8f:11
-----BEGIN CERTIFICATE-----
MIIEhTCCA22gAwIBAgIRAKSN7Wi9JF2/CsOwEh+bjLUwDQYJKoZIhvcNAQELBQAw
RjELMAkGA1UEBhMCVVMxIjAgBgNVBAoTGUdvb2dsZSBUcnVzdCBTZXJ2aWNlcyBM
TEMxEzARBgNVBAMTCkdUUyBDQSAxQzMwHhcNMjMwMzI4MTY1NDMwWhcNMjMwNjIw
MTY1NDI5WjAYMRYwFAYDVQQDEw13d3cuZ29vZ2xlLmRlMFkwEwYHKoZIzj0CAQYI
KoZIzj0DAQcDQgAEZKedP4Y2ToNbBEWdSO58hrDZ/1koQ6BiRciDoK+exh/DrHa/
J0Cqmu6NR8b5wsIuBwspTDGT4GMf7pGdmgk6YqOCAmUwggJhMA4GA1UdDwEB/wQE
AwIHgDATBgNVHSUEDDAKBggrBgEFBQcDATAMBgNVHRMBAf8EAjAAMB0GA1UdDgQW
BBSytm0ZL0I9iGV2JRr81XwGijETPzAfBgNVHSMEGDAWgBSKdH+vhc3ulc09nNDi
RhTzcTUdJzBqBggrBgEFBQcBAQReMFwwJwYIKwYBBQUHMAGGG2h0dHA6Ly9vY3Nw
LnBraS5nb29nL2d0czFjMzAxBggrBgEFBQcwAoYlaHR0cDovL3BraS5nb29nL3Jl
cG8vY2VydHMvZ3RzMWMzLmRlcjAYBgNVHREEETAPgg13d3cuZ29vZ2xlLmRlMCEG
A1UdIAQaMBgwCAYGZ4EMAQIBMAwGCisGAQQB1nkCBQMwPAYDVR0fBDUwMzAxoC+g
LYYraHR0cDovL2NybHMucGtpLmdvb2cvZ3RzMWMzL3pkQVR0MEV4X0ZrLmNybDCC
AQMGCisGAQQB1nkCBAIEgfQEgfEA7wB2AK33vvp8/xDIi509nB4+GGq0Zyldz7EM
JMqFhjTr3IKKAAABhylbcmgAAAQDAEcwRQIgEsP7rZrJh/4X9uymxCyls+cLlq/m
WoG2K8i6qU/5F0ECIQCG6TiACSK06fyl9vdKJ1KkoyKIQu6Ca9gncpemT/ou7QB1
ALNzdwfhhFD4Y4bWBancEQlKeS2xZwwLh9zwAw55NqWaAAABhylbcyUAAAQDAEYw
RAIgBhXpY3IRHWsU0BVEtm8fgqYsSGwZK/5sxWnnxJUp56UCIBbra//rG7Im33Mj
QD2hzZ5dj48z5S47v6USlG2s1qXaMA0GCSqGSIb3DQEBCwUAA4IBAQDfmn44xaxh
7kn3mWgxLtXt8RADmf5GamsYUgoFG3dagRKzawwwuwYxBMsu0fQrEupK0fCH+w7B
a+c8/XJyIp8mo9iwgbswQf4atAwY6qp5C8sdYHH8e00vHvhEpRD7ak2DHJMuyPA6
Cm7+g6snDJUrVbO4VoCsQOmc8G74vq3zcQ1Eex00/MHERqSS6CbGnSKfmQtn7FlI
z0AM7ES1A58EHWz1WcovUq4J0EcccRxFIgA4Z65u5CspHPwaQceUXM1d+ljO+qkE
Pgi6Ixx0UGjJI6MQt2s5LTL51w3fVMKtta0kDf1DKWSMxdkaSWY9jpln1FQ9jaOy
apK7ZBQWO48R
-----END CERTIFICATE-----
````

The above output can be stored in a PEM file (`> server-public-key.pem`) that you will need the next step.

Alternative to using OpenSSL, you can use the CLI tool [certificate-ripper](https://github.com/Hakky54/certificate-ripper) for this use case. 


Alternative: Download the public key of the server with Certificate-ripper
-------------------
Alternative, you can use the CLI tool [certificate-ripper](https://github.com/Hakky54/certificate-ripper) for downloading the public key.

````shell
crip export pem -u https://yourserver.example
````

With `-u`, you set the URL to your server and the tool downloads the public key in the current directory.



Import the public key in the JVM truststore
-------------------
The next step is to import the public key with the JDK tool `keytool` to Java's default truststore. The location of the default truststore depends on the Java version that you are using.

For example, in Java 17, it is `$JAVA_HOME/lib/security/cacerts` and in Java 8, it is `$JAVA_HOME/jre/lib/security/cacerts`. 


````shell
➜ keytool -import -file server-public-key.pem -keystore $JAVA_HOME/lib/security/cacerts
Enter keystore password: changeit
Owner: CN=www.google.de
Issuer: CN=GTS CA 1C3, O=Google Trust Services LLC, C=US
Serial number: a48ded68bd245dbf0ac3b0121f9b8cb5
Valid from: Tue Mar 28 18:54:30 CEST 2023 until: Tue Jun 20 18:54:29 CEST 2023
Certificate fingerprints:
         SHA1: 82:E8:B2:13:E3:9A:14:0D:53:BB:CF:9B:4B:25:BB:5B:6B:54:3B:7F
         SHA256: BC:4A:DE:E9:D9:8D:E0:7A:41:02:35:8A:7E:5C:33:15:8E:B2:50:C3:A8:3D:94:4D:36:E3:AA:56:CE:8B:61:0D
Signature algorithm name: SHA256withRSA
Subject Public Key Algorithm: 256-bit EC (secp256r1) key
Version: 3

Extensions: 

#1: ObjectId: 1.3.6.1.4.1.11129.2.4.2 Criticality=false
0000: 04 81 F1 00 EF 00 76 00   AD F7 BE FA 7C FF 10 C8  ......v.........
0010: 8B 9D 3D 9C 1E 3E 18 6A   B4 67 29 5D CF B1 0C 24  ..=..>.j.g)]...$
0020: CA 85 86 34 EB DC 82 8A   00 00 01 87 29 5B 72 68  ...4........)[rh
0030: 00 00 04 03 00 47 30 45   02 20 12 C3 FB AD 9A C9  .....G0E. ......
0040: 87 FE 17 F6 EC A6 C4 2C   A5 B3 E7 0B 96 AF E6 5A  .......,.......Z
0050: 81 B6 2B C8 BA A9 4F F9   17 41 02 21 00 86 E9 38  ..+...O..A.!...8
0060: 80 09 22 B4 E9 FC A5 F6   F7 4A 27 52 A4 A3 22 88  .."......J'R..".
0070: 42 EE 82 6B D8 27 72 97   A6 4F FA 2E ED 00 75 00  B..k.'r..O....u.
0080: B3 73 77 07 E1 84 50 F8   63 86 D6 05 A9 DC 11 09  .sw...P.c.......
0090: 4A 79 2D B1 67 0C 0B 87   DC F0 03 0E 79 36 A5 9A  Jy-.g.......y6..
00A0: 00 00 01 87 29 5B 73 25   00 00 04 03 00 46 30 44  ....)[s%.....F0D
00B0: 02 20 06 15 E9 63 72 11   1D 6B 14 D0 15 44 B6 6F  . ...cr..k...D.o
00C0: 1F 82 A6 2C 48 6C 19 2B   FE 6C C5 69 E7 C4 95 29  ...,Hl.+.l.i...)
00D0: E7 A5 02 20 16 EB 6B FF   EB 1B B2 26 DF 73 23 40  ... ..k....&.s#@
00E0: 3D A1 CD 9E 5D 8F 8F 33   E5 2E 3B BF A5 12 94 6D  =...]..3..;....m
00F0: AC D6 A5 DA                                        ....

...

Trust this certificate? [no]:  yes
Certificate was added to keystore

````

Restart your app and you are ready.

Alternative: Import the public key in a self-created truststore
-------------------

For some reason, you don't want to modify Java's default truststore. 
Therefore, you can use own truststore in your application. 
Nevertheless, I recommend to use Java's default truststore as a base, because it has all RootCAs and so you don't have to think about them.

````shell
cp $JAVA_HOME/lib/security/cacerts truststore.jks
````

Maybe you want to change the password of this new truststore.

````shell
➜ keytool -storepasswd -keystore truststore.jks
Enter keystore password:  changeit
New keystore password:  newPassword
Re-enter new keystore password:  newPassword
````

Then you have to import the server public key in this truststore like you already see above.

````shell
keytool -import -file server-public-key.pem -keystore $JAVA_HOME/lib/security/cacerts
````

Now, you have to configure your Java application to use this customize truststore.
This is possible via Java properties:

````properties
javax.net.ssl.trustStore=$PATH_CUSTOMIZE_STORE/truststore.jks
javax.net.ssl.trustStorePassword=newPassword
```` 


Further Information
-------------------
1. Wikipedia about [Man in the Middle Attack](https://en.wikipedia.org/wiki/Man-in-the-middle_attack)
2. Blog post ["How to solve javax.net.ssl.SSLHandshakeException"](http://wsigrid.blogspot.com/2008/12/how-to-solve-javaxnetsslsslhandshakeexc.html)
3. Stack Overflow post [Using OpenSSL to get the certificate from a server](https://stackoverflow.com/questions/7885785/using-openssl-to-get-the-certificate-from-a-server)
4. CLI Tool [Certificate Ripper](https://github.com/Hakky54/certificate-ripper)