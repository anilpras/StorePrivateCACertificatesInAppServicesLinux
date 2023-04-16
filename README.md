## Issue

Linux App services app may take additional ~10ms - 1Sec during the outbound call if using private CA certificates + having client certificate auth enbabled, along if using DownloadCertificate method. 

As of now by default app services does not cache the private CA's root/intermediate certificate(s), and with each http(s) call app services makes a call to the private certificate endpoints to download the .CRT call. These calls appears just before the “Client Hello” and essentially these .CRT calls causes delay initiating the "Client Hello". Ideally the client hello should have been sent almost instantaneously.

Note - SNAT ports are limited in azure app services, so this will also bring down your SNAT calls.

## Scenario
The latecny can be seen when the dotnet core application hosted on linux app service, and during the outbound call having the mutual server auth and the app is communicating with private certificate authoritiy servers.

Basically, if the root/itermediate certificates are not stored locally app would approach to the private certificate server to download the certificates. This behaviour does not manifest for the public CA certificates as by default app service downloads and store the certificates locally.

What all certificate store you need to know about:

For "Debian GNU/Linux 10 (buster)"
 1. Uploaded public certificates from the portal(this is readonly, you can write directly to this folder)
    root@b73852890d8e:~# ls /var/ssl/certs
                         '*2C**20***************************6DB81F.der'
                         '*1F**3C**************************8D77165.der'
    
  2. Public certificate 
    root@b73852890d8e:~# ls /etc/ssl/certs

  3. /usr/local/share/ca-certificates - The certificates stored in /usr/local/share/ca-certificates are typically provided by third-party organizations or individuals,      and are not included in the operating system's default set of certificates. This directory is included in the default search path for trusted certificates.
     so any  certificates stored here will be automatically trusted by the system.
  
[I haven't tested this for other application stack]

## How to identify if your app is impacted due to the issue

1. Collect the network trace and look for the 'time since previous frame in this TCP stream' and find out if there are delay. 

   ![image](https://user-images.githubusercontent.com/14159197/219201164-0331b391-2be9-44a0-b10f-e78b13638115.png)

2. Check the packets between tcp handshake and 'client hello', if you find the .CRT calls related to the Https outbound call then your application may be having the issue as discussed as above.  

## How to fix it (the script has targetted for the dotnet core application running on linux)

1. Upload the certificates root and intermediate .crt file as discussed here https://azure.github.io/AppService/2017/11/01/App-Service-Certificates-now-supports-public-certificates-(.cer).html

2. Created a customStartupScript.sh file, inside the /home directory, with following content, please note you need root and all the intermediate certificates, in my case I just had to add root and intermediate.
   ```
   openssl x509 -inform der -in /var/ssl/certs/Public-CA-cert-GUID-for-root.der -outform pem -out /usr/local/share/ca-certificates/root.crt
   openssl x509 -inform der -in /var/ssl/certs/Public-CA-cert-GUID-for-intermediate.der -outform pem -out /usr/local/share/ca-certificates/intermediate.crt
   update-ca-certificates
   cd "/home/site/wwwroot"
   dotnet dotnetstartupdllname.dll
   
   #If your dll is resting inside the web.config file then this can be generalized by the couple of lines as below
   #-----------------------------------------------------------------------------
   #var=$(grep '"*.dll"' web.config | grep -o -P '(?<=arguments=".\\).*(?=.dll)') 
   #dotnet $var".dll"
   #-----------------------------------------------------------------------------
   ```
3. Set WEBSITE_LOAD_CERTIFICATES with value of * or the cert thumbprints.
4. Add the /home/customStartupScript.sh in the startup command.
5. If your script has any issues the process won't start, so ensure your script is working and executable. 
6. Collect the traces again and validate on your findings.


