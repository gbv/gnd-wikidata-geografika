# Abgleich von Geografika aus der GND mit Geonames und Wikidata

Eine größere Menge von GND-IDs soll mit Geonames und Wikidata abgeglichen
werden. Zu beachten ist dass die Daten auch Umlenkungen von veralteten oder
zusammengeführten GND-Datensätzen enthalten.

## Direkten Mappings in der GND

Ein aktueller Gesamtabzug der GND im Entity-Facts Format (ein JSON-LD Profil) ist unter der URL
<https://data.dnb.de/opendata/authorities_entityfacts.jsonld.gz> verfügbar. Der Abzug umfass gepackt
etwas weniger als 1GB. Die Datei enthält ein einziges JSON-Array. Die Geografika lassen sich mit jq
folgendermaßen herausfiltern:

~~~sh
curl https://data.dnb.de/opendata/authorities_entityfacts.jsonld.gz \
 | zcat \
 | jq -cn --stream 'fromstream(1|truncate_stream(inputs))|select(.["@type"]=="place")' \
 > entityfacts_places.ndjson
~~~

Die Filterung dauert einige Minuten und ließe sich vermutlich etwas beschleunigen da der
Dump derzeit anscheinend nach Entitätstyp sortiert ist.

Die GND-Daten lassen sich nun folgendermaßen auf die darin enthaltenen direkten Mappings beschränken
und in das JSKOS-Format überführen:

~~~sh
cat entityfacts_places.ndjson \
  | jq -rc '{id:.["@id"], identifier:[ .sameAs[]["@id"]?|select(endswith("/about")|not) ]}' \
  > gnd-geografika.ndjson
~~~

## Statistik

Der Datenbestand umfasst momentan etwas über 310 Tausend Datensätze und 560
Tausend Identifier:

~~~sh
$ wc -l gnd-geografika.ndjson
~~~

`310793 gnd-geografika.ndjson`

~~~sh
$ jq .identifier[] gnd-geografika.ndjson | wc -l
~~~

`562504`

Welche Identifier-Systeme kommen wie häufig in den GND-Daten vor? Eine
Statistik der URI-Präfike mit mindestens 100 Vorkommen:

~~~sh
cat gnd-geografika.ndjson \
  | jq -r '.identifier[] | split("/")[:-1]|join("/")' \
  | sort | uniq -c | sort -rnk1 | awk '$1>100'
~~~

~~~
 282243 http://viaf.org/viaf
  68367 https://de.wikipedia.org/wiki
  60214 https://en.wikipedia.org/wiki
  56799 http://sws.geonames.org
  55295 http://www.wikidata.org/entity
  29060 http://id.loc.gov/rwo/agents
   8191 http://catalogue.bnf.fr/ark:/12148
   1072 http://isni.org/isni
    715 https://de.wikisource.org/wiki
    441 http://kalliope-verbund.info/gnd
~~~

Es fällt auf, dass alternative GND-Ids (Umlenkungen) nicht auftauchen, hier
müssten die Daten nochmal kontrolliert werden!

Grundsätzlich sind etwa je 18% der Einträge auf GeoNames bzw. auf Wikidata gemappt.

## Zusätzliche Mappings in Wikidata

...

## Ermittlung fehlender Mappings

Mit der Webanwendung [Cocoda](https://coli-conc.gbv.de/cocoda/) können Mappings zwischen Wikidata
auf der einen Seite und GND auf der anderen Seite in Wikidata eingetragen werden. Unterstützung von
GeoNames ist noch nicht direkt umgesetzt.

Zum Mapping von gepgraphischen Entitäten könnten zudem weitere Heuristiken und Hilfsmittel herangezogen werden
(siehe u.A. [dieses Issue](https://github.com/gbv/jskos-server/issues/88)).

...


