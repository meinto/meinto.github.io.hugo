---
title: "Static Site Generator Hugo auf Github-Pages hosten"
date: 2017-08-21
draft: false
tags: ["hugo", "static site generator", "github", "jekyll", "travis", "hosting"]
---

## Website Hosting auf Github-Pages

**Serverless Infrastructure** ist eines der Buzzwords, welches in den letzten Jahren immer mehr im Kommen ist. So könnte man auch das **Hosting einer Website** auf {{< link href="https://pages.github.com/" target="_blank">}}Github-Pages{{</ link >}} als **serverless** bezeichnen. 

Github erlaubt dem Standard User eine Website **kostenlos** auf deren Infrastruktur zu hosten. Voraussetzung hierfür ist, dass es sich um eine statische Website handelt. Serverseitige Code-Ausführung ist nicht möglich.

### Einrichtung von Github-Pages

{{% img src="images/tutorials/hugo-auf-github-pages-hosten/pages-repo-name.jpg" %}}Der Name des Github-Pages Repositories muss mit dem Kürzel des Users anfangen und mit **github.io** enden{{%/ img %}}

### Usage Limits

Wer bei Github seinen Blog oder seine Website hosten möchte, muss sich an einige Rahmenbedingungen halten. Ansonsten wird man füher oder später freundlich von Github aufgefordert den Traffic auf den Github Servern zu reduzieren:

- Maximalgröße des Repositories: 1 GB
- Maximale Bandbreite pro Monat: 100 GB
- Publikationen (Seiten Builds) pro Stunde: 10

Falls man mit seiner statischen Website an die Grenzen der Brandbreite stößt empfieht sich ein CDN wie z. B. {{< link href="https://www.cloudflare.com/de/" target="_blank">}}Cloudflare{{</ link >}}.

## Static Site Generator Jekyll vs. Hugo



## Auto Deployment auf Github-Pages mit Travis