---
Category: CTF
LAB: https://www.vulnhub.com/entry/csharp-vulnsoap,135/
Difficulty: Medium
tags:
Featured: yes
---
---

# Information / Description


The CsharpVulnSoap virtual appliance is a purposefully vulnerable SOAP service, focusing on using XML, which is a core feature of APIs implemented using SOAP. The web application, listening on port 80, allows you to list, create, and delete users in the PostgreSQL database. The web application is written in the C# programming language and uses apache+mod_mono to run. The main focus of intentional vulnerabilities was SQL injections.

The vulnerable SOAP service is available on `http://<ip>/Vulnerable.asmx`, and by appending ?WSDL to the URL, you can get an XML document detailing the functions exposed by the service. Using this document, you can automatically fuzz the endpoint for any vulnerabilities by parsing the document and creating the HTTP requests expected programmatically.

The SQL injections yield a variety of potential exploit techniques since different SQL verbs are used to perform actions against the server. For instance, a SQL injection in an INSERT statement may not be exploitable in the same ways the DELETE or SELECT statements will be. Using a tool like sqlmap will help you learn how to exploit each SQL injection vulnerability using a variety of techniques.

If you are curious how sqlmap is performing the checks for, and ultimately exploiting, the vulnerabilities in the web application, you can use the --proxy option for sqlmap and pass the HTTP requests through Burp suite. You can then see in the HTTP history tab the raw HTTP requests made by sqlmap.

---

# Disclaimer

The intention of this lab is to get comfortable with `web serivices` , `enumerating` then trying to `exploit` them, but the last is not the end goal.

```
Exposed WSDL
→ list operations and parameters
→ generate request with SoapUI
→ intercept/replay it with Burp
→ manually test for SQLi in SOAP parameters
```

---

# Walkthrough

For setting up the lab I downloaded the machine and set both machines (`victim` and my `kali`) in `Only host` for the network configuration, this is the IP the DHCP gave the victim:

| `Victim IP` | 192.168.133.128 |
| ----------- | --------------- |

## Enumeration

### nmap

```
nmap -sS -sV -sC -Pn -n -p- 192.168.133.128 -T5
```

```
┌──(marc㉿martin)-[~]
└─$ nmap -sS -sV -sC -Pn -n -p- 192.168.133.128 -T5
Starting Nmap 7.95 ( https://nmap.org ) at 2026-07-21 16:50 CEST
Nmap scan report for 192.168.133.128
Host is up (0.00049s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.7 (Ubuntu)
MAC Address: 00:0C:29:D1:95:58 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.01 seconds

```

After scanning all `65535 TCP ports` we only encounter `port 80`, cool let's open it and see what we encounter.

### Port 80

When we open the website in a browser we encounter this directly

![](../0.%20Assets/CsharpVulnSoap-1784645547819.webp)

Basically an invitation for listing the `SOAP` service

![](../0.%20Assets/CsharpVulnSoap-1784645990985.webp)

When we append the `?wsdl` query parameter to the SOAP endpoint, the server returns the WSDL document. In this case, the developers exposed the service’s full documentation publicly.

![](../0.%20Assets/CsharpVulnSoap-1784647058267.webp)

This is huge because we can see all (or at least the majority) of the endpoints in the website, even the supposedly hidden ones.

From the WSDL, find:

|Need|Look for|
|---|---|
|Where to send the POST|`soap:address location="..."`|
|Operation name|`operation name="..."`|
|Required parameters and types|`element name="..." type="..."`|
|Optional parameters|`minOccurs="0"`|
|SOAP action|`soapAction="..."`|
An easy way we van get all this information is:

### Golden commands

```
curl -s http://192.168.133.128/Vulnerable.asmx?wsdl -o service.wsdl # Download wsdl file

grep -iE "address|operation|action|element|minOccurs|type=" service.wsdl # List important parameters

grep -ni -m1 -A 15 '<xs:element name="<operation name>"' service.wsdl # Get needed variables 
``` 

![](../0.%20Assets/CsharpVulnSoap-1784647580941.webp)

### Filtering
And then filter for whatever operation interests you:

```
grep -iE "address|operation|action|element|minOccurs|type=" service.wsdl | grep "ListUsers"
```

And even filter out specific values like `SOAPaction`

```
grep -iE "action" service.wsdl | grep "ListUsers"
```


![](../0.%20Assets/CsharpVulnSoap-1784647926946.webp)

With this information we can craft our requests to the server.

The should look something similar to this:
- change the values for the ones found in the `wsdl file`

```
POST /Vulnerable.asmx HTTP/1.1
Host: TARGET
Content-Type: text/xml; charset=utf-8
SOAPAction: "ACTION-FROM-WSDL"

<?xml version="1.0"?>
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:ser="NAMESPACE-FROM-WSDL">
  <soapenv:Header/>
  <soapenv:Body>
    <ser:OperationName>
      <ser:parameterName>test-value</ser:parameterName>
    </ser:OperationName>
  </soapenv:Body>
</soapenv:Envelope>
```


### Request

Let's take a look at some interesting ones and then let's play with them

```
<xs:element name="ListUsers">
```

Let's craft a request to test what we just learned

![](../0.%20Assets/CsharpVulnSoap-1784647791277.webp)

Set your `Proxy` to `Burp Suite`

![](../0.%20Assets/CsharpVulnSoap-1784648013407.webp)

Intercept the request to the `URL` that points to the `SOAP` service (not the `WSLD file`)

And send it to the repeater with `Control + r`

Once in the repeater we are going to modify it so it looks like this:

```
POST /Vulnerable.asmx HTTP/1.1
Host: 192.168.133.128
Content-Type: text/xml; charset=utf-8
SOAPAction: "http://tempuri.org/ListUsers"
Content-Length: 231

<?xml version="1.0" encoding="utf-8"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"
               xmlns:ser="http://tempuri.org/">
  <soap:Body>
    <ser:ListUsers/>
  </soap:Body>
</soap:Envelope>
```

![](../0.%20Assets/CsharpVulnSoap-1784648877016.webp)


I made a little explanation to make it easier to visualize

![](../0.%20Assets/CsharpVulnSoap-1784648896690.webp)

---

Let's make it a bit harder by `deleting a user`

I used the command:

```
grep -iE "operation" service.wsdl
```

To only filter through operations and found this one

![](../0.%20Assets/CsharpVulnSoap-1784649012509.webp)

Let's get all the information from the endpoint with:

```
grep -iE "address|operation|action|element|minOccurs|type=" service.wsdl | grep "DeleteUser"
```

![](../0.%20Assets/CsharpVulnSoap-1784649070988.webp)

To see extra information like the elements we need to pass like the username:

```
grep -ni -m1 -A 15 '<xs:element name="DeleteUser"' service.wsdl
```

![](../0.%20Assets/CsharpVulnSoap-1784649429577.webp)

And this is how the query must look now

![](../0.%20Assets/CsharpVulnSoap-1784649716651.webp)

We open the `DeleteUser` and close it like an `HTML tag` and add the `variable` we found before 