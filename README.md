## Issue

Linux App services app may take *additional* ~40 ms - 1 second during the outbound call if using private CA certificates.

As of now by default app services does not cache the private CA's root/intermediate certificate(s), and with each http(s) call app services makes a call to the private certificate endpoints to download the .CRT call. These calls appears just before the “Client Hello” and essentially these .CRT calls causes delay initiating the "Client Hello". Ideally the client hello should have been sent almost instantaneously.

## Scenario
Linux Based App Service
Using Private Certificate Servers
Using .Net Core
<I haven't tested this for other application stack>

## How to identify the slowness

1. Collect the network trace and look for the Time since previous frame in this TCP stream and find out if you find the delay there

   ![image](https://user-images.githubusercontent.com/14159197/219192616-5ca33ad8-8af9-478b-822d-44a1c5566da3.png)




## How to fix it

Upload the certificates root and intermediate .crt file
Created a customStartupScript.sh file, inside the /home directory.

Used the following content:
```
openssl x509 -inform der -in /var/ssl/certs/Public-CA-cert-GUID-for-root.der -outform pem -out /usr/local/share/ca-certificates/root.crt
openssl x509 -inform der -in /var/ssl/certs/Public-CA-cert-GUID-for-intermediate.der -outform pem -out /usr/local/share/ca-certificates/intermediate.crt
update-ca-certificates
cd "/home/site/wwwroot"
var=$(grep '"*.dll"' web.config | grep -o -P '(?<=arguments=".\\).*(?=.dll)') 
dotnet $var".dll"
```
------------------------------------------------------------------

Set WEBSITE_LOAD_CERTIFICATES with value of * or the cert thumbprints.
Add the /home/customStartupScript.sh in the startup command.


