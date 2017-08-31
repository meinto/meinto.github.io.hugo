---
title: "Vorstellung: Facebook Graph Api"
description: "Vorstellung der Facebook Graph API. ⌨ Binde Daten des sozialen Netzwerkes in deine eigene Website oder App ein! ☕"
date: 2017-08-31
draft: false
tags: ["api", "facebook", "sdk", "json", "social"]
---

Facebook - das derzeit größte soziale Netzwerk der Welt - bietet eine API zum Auslesen von Informationen und sogar zum Verwalten von Facebookseiten, Gruppen etc. an. Die Rede ist in diesem Artikel vom {{<link href="https://developers.facebook.com/docs/graph-api" target="_blank">}}Facebook Graph{{</link>}}. Neben Facebook Graph gibt es noch weitere APIs von Facebook auf die hier aber nicht weiter eingegangen wird:

- Atlas API
- Instagram API
- Marketing API

Der Facebook Graph ist dafür gedacht, die Daten des sozialen Netzwerkes einfach in seine eigene App oder seine Website zu integrieren. Dabei bietet Facebook allerlei {{<link href="https://developers.facebook.com/docs/apis-and-sdks" target="_blank">}}SDKs um den Entwicklern die Anbindung{{</link>}} zur erleichtern. Auch die Facebook Community ist hier sehr aktiv und hat eine ganze Reihe an Community-SDKs entwickelt.

## Graph API Explorer

Doch ganz von vorne. Die Graph API ist eine REST-API: Um schnell mal schauen zu können was man vom Facebook Graph alles zurück bekommt, gibt es den {{<link href="https://developers.facebook.com/tools/explorer/" target="_blank">}}Graph API Explorer{{</link>}}. Voreingestellt ist die folgende GET Abfrage:

`me?fields=id,name`

Drückt man den Senden Button wird die Anfrage mit einem temporären Token abgesendet und man sieht das Ergebnis JSON mit einem `id` und einem `name` Feld:

{{<highlight json>}}
{
  "id": "<hier steht die ID meine Accounts>",
  "name": "Tobias Meinhardt"
}
{{</highlight>}}

Ohne weiteren großen Aufwand kann man nun links in der Graph Navigation weitere Felder der Suche hinzufügen - z. B. das Feld `context` das einem alle Freunde mit Name und ID auflistet.

Viele Felder sind jedoch mit dem Hinweis gesperrt das der User - in dem Fall ich - keine Berechtigung gegeben hat das Feld mittels öffentlicher API auszulesen. Jetzt wird es interessant: **Wie komme ich trotzdem an diese Felder?**

## Facebook Apps

Um als Entwickler Daten von Facebook Users auszulesen muss zuerst über das Facebook Profil des Entwicklers eine **Facebook App** angelegt werden. Der Menüpunkt hierfür ist im Facebook Profil links in der Navigationsleiste unter **Entdecken -> Apps verwalten** zu finden.

### Abruf von User Daten

Mithilfe dieser Facebook App ist es später möglich, über z. B. eines der Facebook SDKs, User nach der Berechtigung zu fragen, private Daten zum Betrieb der eigenen App aus deren Facebook Profil abzufragen. 

*Ein Beispiel:*

Die vom Entwickler erstellte Facebook App ist z. B. auf einer Website eingebunden, die einen Service anbietet, für den private Daten von dem Facebook Profil des Users benötigt werden. Für den User, der den Service nutzen will, öffnet sich ein Popup der Facebook App, welches ihn um die entsprechenden Berechtigungen fragt, diese Daten abrufen zu dürfen. Akzeptiert der User diese Anfrage, sind die entsprechend angeforderten Daten über die Graph API abrufbar.

### Abruf von Page Daten

Der Abruf von Daten einer z. B. Facebook Seite ist wesentlich einfacher. Diese Daten sind meistens öffentlich. Wenn man jedoch schreibenden Zugriff auf die Facebook Seite benötigt, muss man selbst der Owner der Facebook Seite sein und mit dem entsprechend eigenen Facebook Zugriffstoken die API anfragen.

## Access Tokens

Wenn ein SDK für den Abruf der Daten eingesetzt wird brauch man grundsätzlich drei Dinge:

- Facebook App ID
- Facebook App Secret
- Access Token

Dabei gibt es 2 Sorten von **Access Tokens**. Einen Temporären und einen **Long Lived Access Token**. Es empfiehlt sich letzteren für den live Betrieb einer eigenen App einzusetzen, da er 60 Tage lang gültig bleibt.
