---
id: 59
title: 'Deploying/Copying SSIS Package to another SQL Server'
date: '2023-09-06T02:26:09+00:00'
author: solutionadmin
layout: post
guid: 'https://blog.sanchitwadhwa.com/?p=59'
permalink: /2023/09/06/deploying-copying-ssis-package-to-another-sql-server/
categories:
    - SQL
tags:
    - dba
    - deployment
    - sql
    - ssis
---

There are times you may want to deploy or copy SSIS package from one server to another. Here is a quick and easy way of copying it over using command line.

```
<pre class="wp-block-code">```
Call "C:\Program Files\Microsoft SQL Server\110\DTS\Binn\dtutil.EXE" /SourceServer <SourceServerName> /SQL "<SourcePackageName>" /DestServer <TargetServerName> /COPY "SQL;<PackageNameToReplace>" /Q
```
```