# Install and use Gollum Wiki

Gollum ist ein Offline-Wiki powered by git. Das bedeutet, ihr könnt das Wiki direkt auf eurem Rechner auch ohne Internet-Access benutzen und die bestehende Doku lesen.

## Bitbucket

Wenn ihr eigene Änderungen am Wiki für alle zugänglich machen wollt, benötigt ihr einen [Bitbucket](https://bitbucket.org/) Account. Außerdem müsst ihr im Terminal.21-Team sein. Stefan ansprechen.

## Vorbereitung

`sudo aptitude install rubygems git libxml2-dev libxslt1-dev`

`sudo gem install gollum --no-ri --no-rdoc`

## Wiki-Content initial auschecken
`git clone https://<BITBUCKET_USERNAME>@bitbucket.org/terminal21/fusion2013.git`

`cd fusion2013`

## Wiki lokal starten

`gollum`

warten bis da steht: INFO  WEBrick::HTTPServer#start: pid=5757 port=4567

Browser öffnen und los: [http://localhost:4567](http://localhost:4567)

## Änderungen einpflegen und veröffentlichen

Das Wiki versteht [Markdown Syntax](http://daringfireball.net/projects/markdown/)

Anschließend müsst ihr eure Änderungen pushen, damit sie im zentralen Repo landen und für alle anderen zugänglich sind.

`git push origin master`

## Änderungen der anderen abholen

In regelmäßigen Abständen müsst ihr euer lokales Repo auf den aktuellen Stand bringen, damit ihr auch alle Infos habt.

`git pull`

## Fragen?

-> Stefan