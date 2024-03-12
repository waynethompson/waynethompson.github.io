---
layout: post
title:  "How to remove x-powered-by header in .net core"
date:   2020-11-30 15:34:00 +1000
categories: [dotnet, swagger, development]
permalink: /blog/nswag-swashbuckle-enum-from-swagger/
---


Any asp dotnet developer who has had to pass security or OWASP top 10 audits knows that probably the most annoying thing about dotnet is that IIS, and now Azure app service, is the custom headers which give away what server and the .net version you are running on.

Most can be removed with code or using NWebsec middleware, but we still are left with that pesky "X-Powered-By" which you can easily get rid of in web.config. 
Problem is you don't have a web.config file in an aspdotnet core website project, since that gets generated when you publish the project.

The solution is to create a new ```web.Release.config``` file with the following transform to add it when it is being generate.




    <?xml version="1.0" encoding="utf-8"?>
    <configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">
        <location>
            <system.webServer>
                <httpProtocol xdt:Transform="InsertIfMissing">
                    <customHeaders>
                        <remove name="x-powered-by" />
                    </customHeaders>
                </httpProtocol>
                <security xdt:Transform="InsertIfMissing">
                    <requestFiltering removeServerHeader="true" />
                    <!-- Removes Server header in IIS10 or later and also in Azure Web Apps -->
                </security>
            </system.webServer>
        </location>
    </configuration>

What this will do is drop in the web.config settings under ```<system.webServer>``` to tell the server to remove the headers.

Next if someone can tell me how to remove the ```server: CloudFlare``` header that would be great.