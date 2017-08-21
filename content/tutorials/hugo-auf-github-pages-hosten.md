---
title: "Static Site Generator Hugo auf Github Pages hosten"
date: 2017-08-21
draft: false
tags: ["hugo", "static site generator", "github", "jekyll", "travis", "hosting"]
---

## Website Hosting auf Github Pages

**Serverless Infrastructure** ist eines der Buzzwords, welches in den letzten Jahren immer mehr im Kommen ist. So könnte man auch das **Hosting einer Website** auf {{< link href="https://pages.github.com/" target="_blank">}}Github Pages{{</ link >}} als **serverless** bezeichnen. 

Github erlaubt dem Standard User eine Website **kostenlos** auf deren Infrastruktur zu hosten. Voraussetzung hierfür ist, dass es sich um eine statische Website handelt. Serverseitige Code-Ausführung ist nicht möglich.

### Usage Limits

Wer bei Github seinen Blog oder seine Website hosten möchte, muss sich an einige Rahmenbedingungen halten. Ansonsten wird man früher oder später freundlich von Github aufgefordert den Traffic auf den Github Servern zu reduzieren:

- Maximalgröße des Repositories: 1 GB
- Maximale Bandbreite pro Monat: 100 GB
- Publikationen (Seiten Builds) pro Stunde: 10

Falls man mit seiner statischen Website an die Grenzen der Brandbreite stößt empfieht sich ein CDN wie z. B. {{< link href="https://www.cloudflare.com/de/" target="_blank">}}Cloudflare{{</ link >}}.

### Einrichtung von Github Pages

Um eine Website auf Github Pages zu hosten muss als erstes ein entsprechendes Repository auf Github angelegt werden. Der Repository Name muss in dem Fall immer mit dem Kürzel des Users oder der Organisation anfangen und endet mit **github.io**. In meinem Fall war das **meinto.github.io**.

{{% img src="images/tutorials/hugo-auf-github-pages-hosten/pages-repo-name.jpg" %}}
Der Name des Github Pages Repositories muss mit dem Kürzel des Users anfangen und mit **github.io** enden.  
*Achtung: Github Pages Repositories sind immer public - deshalb sollten sich keine sensiblen Daten in dem Repository befinden*  
{{%/ img %}}

Nach der Anlage kann das Repository unter dem Reiter **Settings** für Github Pages freigeschaltet werden:

{{% img src="images/tutorials/hugo-auf-github-pages-hosten/repository-settings.jpg" %}}
Der default-Branch bei Pages ist der master-Branch. Dieser kann auch nicht geändert werden. Alles was sich auf dem master-Branch befindet wird von Github Pages als Grundlage genommen und veröffentlicht.
{{%/ img %}}

Die Standard-**Domain** lautet wird er Repository Name. In meinem Fall *meinto.github.io*. Es kann zwar eine eigene Domain hinterlegt werden, jedoch gehe ich hier nicht weiter auf diese Feature ein...

## Static Site Generator Jekyll oder Hugo

Nachdem das Repository auf Github angelegt wurde muss es natürlich befüllt werden. Github Pages arbeitet hier von Haus aus sehr eng mit {{< link href="https://jekyllrb.com/" target="_blank">}}Jekyll{{</ link >}} zusammen, einem Static-Site-Generator auf Ruby Basis. Wer Jekyll verwendet hat den Vorteil, dass er sich um fast nichts mehr Gedanken machen muss. Einfach das Jekyll-Projekt in Github Pages auf den master-Branch legen und Github Pages buildet die Seite automatisch und veröffenlicht sie.

Für alle die Ruby schon auf ihrem Rechner installiert und eingerichtet haben ist das also die einfachste Lösung. Ich dagegen hatte Ruby auf meinem Windows-PC noch nicht installiert und bin auf {{< link href="https://gohugo.io/" target="_blank">}}Hugo{{</ link >}} gestoßen. Da Hugo in Go geschrieben ist funktionierte alles Plug and Play mäßig out of the Box. 

**Doch wie verknüpft man nun ein Hugo Projekt am besten mit Github Pages?**

Da ich mich um so wenig wie möglich selbst kümmern möchte habe ich folgendes Setup gewählt:

1. Das Hugo Projekt liegt in einem eigenen Repository: meinto.github.io.hugo
2. Mit **Travis** wird das Hugo Projekt abgeholt und automatisch gebuilded und deployed
3. Deployment Ziel ist das eigentliche Github Pages Repository: meinto.github.io

## Auto Deployment auf Github Pages mit Travis

{{< link href="https://travis-ci.org/" target="_blank">}}Travis{{</ link >}} ist ein Couninuous Integration Tool. Man kann es z. B. nutzen um seinen auf Github gehosteten Code zu testen und zu deployen. In meinem Fall habe ich es ausschließlich fürs Deployment verwendet.

Ist Travis mit dem eigenen Github Account verbunden sieht man auf dem Dashboard eine Auflistung seiner Repositories. Um Travis für ein bestimmtes Repository zu aktivieren muss einfach nur der Schalter umgelegt werden (siehe Abbildung).

{{% img src="images/tutorials/hugo-auf-github-pages-hosten/travis-ci.jpg" %}}
Werden nicht alle Projekte auf dem Dashboard angezeigt hilft meist ein beherzter Klick auf den **Sync Account** Button...
{{%/ img %}}

Über das Zahnrädchen kommt man dann auf die Einstellungen für das gewünschte Repository. Standardmäßig buildet Travis die Projekte automatisch wenn es auf dem jeweiligen Branch Updates gibt. Damit Travis weiß was es zu tun hat, muss dem Hugo Projekt nun noch eine `.travis.yml` hinzugefügt werden.

{{< highlight yaml>}}
language: none

sudo: true
dist: trusty

install:
  - sudo apt-get --yes install snapd
  - sudo snap install hugo
  - sudo apt-get install python3-pygments
  - git clone --recursive -j8 https://github.com/meinto/meinto.github.io.hugo.git
  - cd meinto.github.io.hugo

script:
  - /snap/bin/hugo
  - cd public
  - ls -la

deploy:
  repo: meinto/meinto.github.io
  target_branch: master
  provider: pages
  skip_cleanup: true
  github_token: $GITHUB_TOKEN
  on:
    branch: master
{{</ highlight >}}

In meinem Fall veranlasst die `.travis.yml` dass eine **Ubuntu trusty Umgebung** zum builden des Projektes hochgefahren wird. Danach wird **hugo über snapd** sowie **pygments** für das Code Highlighting installiert. Im Anschluss wird das **Hugo Projekt** von meinem Github Account gecloned und gebuildet. Der `deploy` Block der `.travis.yml` ist für die Veröffentlichung auf dem Github Pages Repository zuständig. Im Grunde wird der von Hugo erstellte `public` Ordner auf dieses Repository gepushed.

Github Pages macht dann den Rest. 

Voilà.
