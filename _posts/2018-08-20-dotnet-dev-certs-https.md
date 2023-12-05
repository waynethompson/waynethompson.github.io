---
layout: post
title:  "Unable to configure HTTPS endpoint. No server certificate was specified, and the default developer certificate could not be found."
date:   2018-08-20 19:00:00 +1000
categories: dotnet
permalink: /blog/dotnet-dev-certs-https/
---



Today we noticed something strange happen with our dotnet core web apps. They started to throw an error when we ran them on the commandline with ```dotnet run```: 


    System.InvalidOperationException: Unable to configure HTTPS endpoint. No server certificate was specified, and the default developer certificate could not be found.
    To generate a developer certificate run 'dotnet dev-certs https'. To trust the certificate (Windows and macOS only) run 'dotnet dev-certs https --trust'.
    For more information on configuring HTTPS see https://go.microsoft.com/fwlink/?linkid=848054.


It also happened that in iisexpress we were getting an error in the browser that the request was rejected.

We hypothesised (partly by reading the error) that the developer certs had expired. 

We tried to run ```dotnet dev-certs https``` to generate a new certificate but it seemed that dotnet still could  not find the certificate.
We went on to run ```'dotnet dev-certs https --trust'``` and got the popup to trust the certificate but still no luck.


### But wait there is options...
Looking at the options for the command it can be seen that there is the ability to clean out the old certificates.

    Options:
      -ep|--export-path  Full path to the exported certificate
      -p|--password      Password to use when exporting the certificate with the private key into a pfx file
      -c|--check         Check for the existence of the certificate but do not perform any action
      --clean            Cleans all HTTPS development certificates from the machine.
      -t|--trust         Trust the certificate on the current platform
      -v|--verbose       Display more debug information.
      -q|--quiet         Display warnings and errors only.
      -h|--help          Show help information


# Steps to fix the problem

1. Close your browsers so that they do not cache the certificate because that will cause other issues.

2. On the commandline run 

    dotnet dev-certs https --clean

3. run 

    dotnet dev-certs https -t


You can then check the certificate with ```dotnet dev-certs https --check``` but this returned nothing which I assume means everything is ok.

when I then ran ```dotnet run``` the project ran as expected.
