---
title: "Formulare auf statischen Webseiten einrichten"
description: ""
date: 2017-08-26
draft: false
tags: ["hugo", "static site generator", "aws", "amazon", "aws lambda", "hosting", "serverless"]
---

Statische Webseiten haben Vor- und Nachteile. Sie sind auf der einen Seite unfassbar schnell in der Auslieferung, da der DOM nicht erst serverseitig zusammengestellt werden muss. Auf der anderen Seite ist der wohl größte Nachteil die fehlende Dynamik. Bei einem Hosting auf einer **Serverless Infrastructure** kommt zudem dazu, dass man keinen Zugriff auf den Server hat und somit andere Wege gehen muss, um serverseitigen Code auszuführen.

In diesem Artikel erkläre ich, wie ich dieses Problem am Beispiel eines Kontaktformulares gelöst habe. 

## Statische Webseiten und Formulare

Wie schon erwähnt ist es nicht möglich auf statischen Webseiten ohne Weiteres dynamische Seiteninhalte wie Formulare zu hinterlegen. Um das zu erreichen, wird entweder ein Drittanbieter benötigt, oder man schreibt sich die entsprechenden Scripte auf einem eigenen Server selbst.

Die folgenden Drittanbieter bieten einfache Formularlösungen für statische Webseiten an:

- {{<link href="http://formspree.io/" target="_blank">}}Formspree{{</link>}}
- {{<link href="https://www.elformo.com/" target="_blank">}}elFormo{{</link>}}
- {{<link href="http://flipmail.co/" target="_blank">}}Flipmail{{</link>}}
- {{<link href="http://mailthis.to/" target="_blank">}}MailThis{{</link>}} - optionale Anhänge.
- {{<link href="https://getsimpleform.com/" target="_blank">}}Simple Form{{</link>}} - optionale Anhänge .
- {{<link href="https://github.com/stevensona/briskforms" target="_blank">}}Brisk Forms{{</link>}} - E-Mail Adresse bleibt privat.

Da ich datenschutzrechtlich allen angesprochenen Webseiten nicht vertraue, habe ich mich gegen diese einfache Variante entschieden.

Es gibt jedoch noch weitere **Drittanbieter von Web-Formularen**. Diese bieten aber meistens kein einfaches Formular an, welches frei gestaltet werden kann. Entweder ist es schon vorgestyled und muss über Iframes eingebunden werden, oder es handelt sich eher um Umfragen die z. B. in Google Sheets gepushed werden und keine E-Mail auslösen. All das war für mich auch keine Alternative. Nachfolgend trotzdem der Vollständigkeit halber die Links zu den Anbietern:

- {{<link href="https://www.google.com/forms/about/" target="_blank">}}Google Forms{{</link>}} - Speichert die Ergebnisse der Umfrage in Google Sheets. Es gibt eine E-Mail Benachrichtigung wenn jemand eine Umfrage ausgefüllt hat.
- {{<link href="https://formkeep.com/" target="_blank">}}FormKeep{{</link>}} - Formulare mit Spam Filter, Webhooks für z. B. Gmail, Trello und Basecamp.
- {{<link href="http://www.123contactform.com/" target="_blank">}}123 Contact Form{{</link>}} - Formulare mit Verbindung zu z. B. MailChimp, Salesforce und Google Drive.
- {{<link href="http://www.formassembly.com/" target="_blank">}}FormAssembly{{</link>}}
- {{<link href="https://www.formsite.com/" target="_blank">}}FormSite{{</link>}}
- {{<link href="https://www.formstack.com/" target="_blank">}}FormStack{{</link>}} - Formulare mit A/B Testing.
- {{<link href="https://sheetsu.com/" target="_blank">}}Sheetsu{{</link>}}
- {{<link href="http://www.typeform.com/" target="_blank">}}Typeform{{</link>}}
- {{<link href="http://www.wufoo.com/" target="_blank">}}Wufoo{{</link>}}
- {{<link href="https://www.zoho.com/crm/help/web-forms/set-up-web-forms.html" target="_blank">}}Zoho{{</link>}} - Formulare mit File Upload und Captcha.
- {{<link href="https://help.github.com/articles/about-issues/" target="_blank">}}Github Issues{{</link>}}
- {{<link href="https://www.formbackend.com" target="_blank">}}FormBackend{{</link>}}

## AWS Lambda

Da Drittanbieter von Formularen für mich wegfielen musste eine eigene Lösung her. Theoretisch muss ein HTML-Formular auf der statischen Website an einen Endpunkt **posten**, der den **post-Request** verarbeiten kann. Da ich Fan von Serverless Infrastructure bin, habe ich das entsprechende Script nicht auf einem eigenen Server gehostet, sondern als **AWS Lambda** Funktion aufgesetzt.

**Setup für das Kontaktformular**

- Eine AWS Lambda Funktion hinter dem Amazon API Gateway
- {{<link href="https://www.mailgun.com/" target="_blank">}}mailgun{{</link>}} zum versenden der E-Mails getriggered von AWS Lambda

### AWS Lambda einrichten

Um die Amazon Services nutzen zu können benötigt man lediglich ein Amazon Account, welcher dann auch für die Services freigeschaltet wird. Um loszulegen muss ebenfalls eine Kreditkarte hinterlegt und die Identität durch einen automatischen Telefonanruf (für die Pineingabe) verifiziert werden. Von der Kreditkarte wird nichts abgebucht, solange man sich im freien Kontingent der Services bewegt. {{<link href="https://aws.amazon.com/de/lambda/pricing/" target="_blank">}}Bei Amazon Lambda sind die ersten 1.000.000 Anfragen pro Monat und 400.000 GB/s Datenverarbeitungszeit kostenlos.{{</link>}} Danach kostet es pro weitere Millionen Anfragen 0.20 USD.

Einmal angemeldet kann nun die erste Lambda Funktion unter **Services -> Lambda** angelegt werden. Die folgenden Sprachen sind für Lambda Funktionen möglich:

- C#
- Java 8
- Node.js 4.3
- Node.js 6.10
- Python 2.7
- Python 3.6

Die Sprache einer Funktion wird in der Konfigurationsansicht der Lambda Funktion eingestellt. Hier werden ebenfalls **Umgebungsvariablen** definiert, sowie der **Einsprungpunkt der Funktion** angegeben.

### Funktion für E-Mail Versand

Für das Script – zum Anstoßen des E-Mail Versandes – habe ich Python gewählt. Der nachfolgende Code zeigt einen API Aufruf der **mailgun API**. Die entsprechenden Werte wie die **mailgun API Url** oder den **mailgun API Key** habe ich in den bereits angesprochenen **Umgebungsvariablen** der Lambda Funktion hinterlegt. Mit `os.environ['name_der_umgebungsvariable']` können diese in der Funktion abgerufen werden.

Als Einsprungpunkt habe ich unter **Handler** in der Funktionskonfiguration `index.lambda_handler` eingetragen. Bedeutet: Der Einstiegspunkt für die Routine ist die Funktion `lambda_handler` in der `index.py`.

{{<highlight python>}}
# AWS Lambda Function
# index.py

import requests
import os 

def lambda_handler(event, context):
    send_simple_message(event)
    return 'message sent'
    
def send_simple_message(event):
    text = []
    text.append('name: ' + event['name'])
    text.append('\n')
    text.append('email: ' + event['email'])
    text.append('\n')
    text.append('message: ' + event['message'])
    
    return requests.post(
        os.environ['api_base_url'],
        auth=("api", os.environ['mailgun_api_key']),
        data={"from": os.environ['from'],
              "to": os.environ['to'],
              "subject": os.environ['subject'],
              "text": ''.join(text)})
{{</highlight>}}

Da die Funktion eine externe Library verwendet, muss diese ebenfalls mit der eigentlichen Lambda Funktion gepackt und hochgeladen werden. Hierfür einfach ein entsprechendes kleines Projektverzeichnis anlegen und die externe Python Library direkt in dieses Verzeichnis installieren. In meinem Fall war das die Library `requests`:

`pip install requests -t /path/to/project-dir`

Auf der root-Ebene des Projektverzeichnisses befinden sich nun also die Ordner für die externe Library `requests` und die `index.py` welche meine eigentliche Lambda Funktion beschreibt.

Um die Lambda Funktion zu deployen, einfach den Inhalt des Projekt Ordners zippen und in der AWS Management Console für die entsprechende Lambda Funktion hochladen.

### API Gateway einrichten

Um die Funktion von außen erreichbar zu machen muss nun noch ein API Endpunkt im AWS API Gateway eingerichtet werden. Am besten legt man diesen direkt im Konfigurationsmenü der Funktion unter dem Menüpunkt **Triggers -> Add trigger -> API Gateway** an.

Einen **API Namen** wählen, den Wert unter **Development stage** auf **prod** belassen und unter **Security** die Option **open** auswählen. In der **Services -> API Gateway** Ansicht wird nun die neue API unter **Resources** angezeigt.

Voreingestellt ist hier die Methode **ANY**. Für die Lambda Funktion muss man die Methoden **OPTIONS** und **POST** unter **Actions -> Create Method** einrichten. Hier als **Integration type -> Lambda Function** und die entsprechende **Lambda Region** auswählen. Danach erscheint ein Feld **Lambda Funktion**. Ein Klick auf die Leertaste lässt ein Dropdown Menü erscheinen in welcher schon der Name der Lambda Funktion steht. Diesen Namen dann auswählen und auf **Save** klicken.

Am Schluss ist es noch wichtig CORS (Cross-Origin Resource Sharing) für die Methoden **OPTIONS** und **POST** zu aktivieren. Diese Einstellung kann unter **Actions -> Enable CORS** vorgenommen werden. Unter **Access-Control-Allow-Origin** sollte bestenfalls kein '*', sondern die entsprechende Domain stehen von der die Requests kommen.

Zum Schluss muss die API dann noch unter **Actions -> Deploy API** deployed werden.

### Client Integration

Der Ajax Request um die API anzusprechen sieht wie folgt aus:

{{<highlight js>}}
var url = 'https://api-endpoint-url.amazon.com';
$.ajax({
  url: url,
  type: 'POST',
  crossDomain: true,
  contentType: 'application/json',
  dataType: 'json',
  data: JSON.stringify({
    name: name,
    email: email,
    message: message,
  }),
})
{{</highlight>}}

Das fertige Resultat könnt ihr auf meiner {{<link href="/kontakt">}}Kontakt Seite{{</link>}} sehen.

## Risiken von AWS Services

Amazon AWS ist eine tolle Sache. Man kann sich in kurzer Zeit ziemlich viel einfach zusammen klicken. Das tolle aber auch gefährliche an allen Services ist, dass sie ziemlich schnell hoch skalieren wenn die Anfragen steigen. Deshalb habe ich vorsichtshalber das **Event throttling** in der API ziemlich niedrig eingestellt, sodass nur wenige Anfragen auf einmal gestellt werden können und ich in der kostenlosen Zohne für die Nutzung der Services bleibe.

Außerdem können im **Cloud Watch Service** Alarme eingestellt werden die E-Mails verschicken, sobald die Nutzung einer Lambda Funktion die durchschnittlichen Grenzwerte überschreitet.  