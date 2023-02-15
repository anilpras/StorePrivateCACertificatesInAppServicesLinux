##Issue 
Linux App services app may take *additional* ~40 - 500 milliseconds during the outbound call if using Private CA certificates.
In this case you would find the .CRT calls to the private certificate endpoints just before the “Client Hello” and essentially these .CRT calls causes delay initiating the "Client Hello". Ideally the client hello should have been sent almost instantaneously.

## Scenario
Linux Based App Service
Using Private Certificate Servers
Using .Net Core
<I haven't tested this for other application stack>

##How to identify the slowness


##How to fix it

Upload the certificates root and intermediate .crt file
Created a customStartupScript.sh file, inside the /home directory.

Used the following content:

openssl x509 -inform der -in /var/ssl/certs/Public-CA-cert-GUID-for-root.der -outform pem -out /usr/local/share/ca-certificates/root.crt
openssl x509 -inform der -in /var/ssl/certs/Public-CA-cert-GUID-for-intermediate.der -outform pem -out /usr/local/share/ca-certificates/intermediate.crt
update-ca-certificates
cd "/home/site/wwwroot"
var=$(grep '"*.dll"' web.config | grep -o -P '(?<=arguments=".\\).*(?=.dll)') 
dotnet $var".dll"
------------------------------------------------------------------

Set WEBSITE_LOAD_CERTIFICATES with value of * or the cert thumbprints.
Add the /home/customStartupScript.sh in the startup command.


