# Install and use Gollum Wiki

Gollum ist ein Offline-Wiki powered by git. Das bedeutet, ihr könnt das Wiki direkt auf eurem Rechner auch ohne Internet-Access benutzen und die bestehende Doku lesen.

## GitHub

Wenn ihr eigene Änderungen am Wiki für alle zugänglich machen wollt, benötigt ihr einen [GitHub](https://gitthub.com/) Account. Außerdem müsst ihr im Terminal21 sein. Stefan ansprechen.

## Vorbereitung

`sudo aptitude install rubygems git libxml2-dev libxslt1-dev ruby-dev`

`sudo gem install gollum --no-ri --no-rdoc`

## Wiki-Content initial auschecken
`git clone git@github.com:Terminal21/Fusion2014.git`

`cd Fusion2014`

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
