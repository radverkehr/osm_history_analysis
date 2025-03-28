# osm_history_analysis

In Deutschland fehlt bislang eine einheitliche, öffentlich zugängliche Datenbank, die verkehrliche Maßnahmen und deren zeitliche sowie räumliche Veränderungen systematisch darstellt. Eine solche Datenbasis könnte jedoch wertvolle Einblicke ermöglichen, um die Auswirkungen von Maßnahmen – beispielsweise auf die Verkehrsnachfrage oder die Unfallhäufigkeit – transparenter zu analysieren und zu bewerten.

Ein vielversprechender Ansatz wäre die Entwicklung eines Prozesses, der mithilfe der OpenStreetMap (OSM)-Datenbank und des ohsome-Frameworks [1] gezielt Maßnahmen wie bspw. Änderungen von Geschwindigkeitsbeschränkungen identifiziert und deren zeitliche sowie räumliche Veränderungen nachvollziehbar macht.

## **Vorgehen im Projekt:**

### 1. Grundlage schaffen

* Im ersten Schritt wurde ein grundlegender Verarbeitungsprozess in Python entwickelt, um Daten aus ohsome zu extrahieren und zu verarbeiten.

### 2. Erstellung von Prototypen

* **Prototyp 1:** Die Entwicklung eines ersten Prototyps wurde in einem kleinen Testgebiet durchgeführt. Dabei wurden Änderungen der zulässigen Höchstgeschwindigkeit (`maxspeed`) zum 1. Januar eines Jahres ausgewertet.

  * [Visualisierung öffnen](https://radverkehr.github.io/digitalisierungsmodul/SoSe2024/viz/historic_osm-maxspeed_hermannstr.html)

  * Mermaid-Diagramm:

    <details>
    <summary>Click to expand mermaid diagram</summary>

    ```mermaid
    ---
    title: Prototype1 (as mermaid flowchart)
    ---
    flowchart TD
        subgraph SG1["Choose Area of Interest"]
            direction RL
            A2@{ shape: cyl, label: "Custom Shape"} --> A1("Get Geometry")
        end

        subgraph SG2["Processing Historic OSM Data"]
            direction TB
            subgraph SG2-1["Get Data"]
                direction RL
                B2@{ shape: cyl, label: "OpenStreetMap"} --> |ohsome API| B1("Get Highways with _maxspeed_")
            end   

            subgraph SG2-2["Clean Up/Process Data"]
                direction TB
                C("Retrieve Tags to DataFrame Columns") --> D("Get State of Each 1st January")
            end 
            SG2-1 --> SG2-2
        end

        subgraph SG3["Visualize"]
            direction BT
            G("Select Year") --> E("Visualize as Interactive Map via Altair") 
        end

        SG1 --> SG2 --> SG3

        %% Comments after double percent signs
    ```
    </details>
* **Prototyp 2:**  Ein weiterer Prototyp stellt Geschwindigkeitsänderungen (`maxspeed`) auf Hauptverkehrsstraßen im Berliner Bezirk Neukölln dar. Die Visualisierung enthält einen Zeitstrahl zur Darstellung der Änderungen über einen längeren Zeitraum sowie eine Verlinkung der Way-Historie.

  * [Visualisierung öffnen](https://radverkehr.github.io/osm_history_analysis/viz/maps_nk_no-basemap_PrimSecTert_wayChart_24-11-04.html)

  * Mermaid-Diagramm:

      <details>
      <summary>Click to expand mermaid diagram</summary>

      ```mermaid
      ---
      title: Prototype2 (as mermaid flowchart)
      ---
      flowchart  TD
          subgraph SG1["choose area of interest"]
            direction RL
              A2@{ shape: cyl, label: "OpenStreetMap"}-->|Overpass API| A1("get geometry")
          end
          
          subgraph SG2["processing historic osm-data"]
            direction TB
            subgraph SG2-1["get data"]
              direction RL
                B2@{ shape: cyl, label: "OpenStreetMap"}--> |ohsome API| B1("get highways with _maxspeed_")
              end   
              
              subgraph SG2-2["clean up/process data"]
                direction TB
                C(retrieve tags to dataframe columns) --> D(filter only Hauptverkehrsstraßen)
              end 
              SG2-1 --> SG2-2
          end
          
          subgraph SG3["vizualize"]
            direction BT
            F(Include way's timeline as chart) --> E(vizualize as interactive map via altair) 
            G(select from continuous time) --> E
          end
          
      SG1 --> SG2 --> SG3

      %% Comments after double percent signs
      ```
      </details>

### 3. Analysen: Verknüpfung von historischen OSM-Daten mit anderen Datenquellen

Die Verknüpfung historischer OSM-Daten mit externen Datenquellen ermöglicht eine tiefere Analyse verkehrlicher Maßnahmen und deren Auswirkungen. Zwei zentrale Fragestellungen stehen dabei im Fokus:

#### a) Zeitliche Erfassung von Fahrradstraßen in OSM

Durch den Vergleich amtlicher Daten zu Fahrradstraßen der Berliner Senatsverwaltung [2] mit entsprechenden Änderungen in OSM (`bicycle_road=yes`) wird untersucht, wie schnell neue Fahrradstraßen in OSM erfasst werden. Diese Analyse bietet Einblicke in die Verzögerung (Time Lag) zwischen der Umsetzung einer Maßnahme und ihrer Erfassung in OSM.

* **Visualisierungen:**
  * [Zeitlicher Vergleich OSM vs. Senat (Line Chart)](https://radverkehr.github.io/osm_history_analysis/viz/fahrradstrassen/lineplot_fahrradstr_berlin_osm2senat.html)
  * [Kartendarstellung der OSM-Fahrradstraßen in Berlin](https://radverkehr.github.io/osm_history_analysis/viz/fahrradstrassen/maps_fahrradstr_berlin.html)

#### b) Auswirkungen von Maßnahmen auf Unfälle

Ein weiterer Untersuchungsaspekt ist, ob nachgewiesen werden kann, dass eine Reduzierung der **zulässigen Geschwindigkeit** zu einem Rückgang der Unfallzahlen in den betroffenen Bereichen führt. Hierfür wurden Daten aus dem **Unfallatlas** [3] herangezogen. Aufgrund der umfangreichen Datenmengen kam anstelle des ohsome-Frameworks eine alternative Datenverarbeitung zum Einsatz, bei der pbf-Dateien [4] mit Osmium [5] verarbeitet wurden. Als Grundlage der Analyse dienen die OSM-Netze vom 1. Januar 2018 und 1. Januar 2024.

<img src="https://raw.githubusercontent.com/radverkehr/osm_history_analysis/main/img/timeline.png" alt="timeline" width="350">

* **Visualisierungen:**
  * **Karten:**  
    * [Übergeordnete Kartendarstellung: Unfallentwicklung und Veränderung zulässiger Geschwindigkeit](https://radverkehr.github.io/osm_history_analysis/viz/germany_raster2022_net_acc.html)
      * [Beispiel: Detailsansicht je Kachel](https://radverkehr.github.io/osm_history_analysis/viz/raster/1kmN3269E4554_2022_dash_net1824_acc1819-2223.html)
        - Aktuell nur verfügbar in Berlin. Klickbar über die Kartendarstellung.
  * **Plots:**  
    * **Scatterplot:**  
      <img src="https://raw.githubusercontent.com/radverkehr/osm_history_analysis/main/plots/scatterplot_tempo30_unfallanalyse.png" alt="Scatterplot" width="500">  
    * **Boxplot:**  
      <img src="https://raw.githubusercontent.com/radverkehr/osm_history_analysis/main/plots/boxplot_tempo30_unfallanalyse.png" alt="Boxplot" width="500">

* **Tabelle: Übersicht der analysierten Kacheln** Wieso nur 2.801?

  | Beschreibung                                                                                              | Anzahl Kacheln | Begründung                                                                                                     |
  |----------------------------------------------------------------------------------------------------------|---------------:|---------------------------------------------------------------------------------------------------------------|
  | Gesamtanzahl der 1km-Kacheln (Zensus 2022 [6]) in Deutschland                                                 |        384.181 |                                                                                                               |
  | Anzahl der Kacheln in Deutschland ohne TH, NRW, MV                                                        |        287.784 | Keine Unfalldaten für diese Bundesländer in 2018/2019                                                         |
  | Anzahl der Kacheln mit Straßen, die 2018 oder 2024 `maxspeed` 50 oder 30 aufweisen                        |         70.506 | Ohne Straßen wäre die Analyse nicht sinnvoll                                                                  |
  | Anzahl der Kacheln mit Straßen, die 2024 eine beliebige Länge `maxspeed` 30 aufweisen                     |         12.706 | Fokus auf Unfallveränderungen bei Tempo-30-Straßen                                                            |
  | Anzahl der Kacheln, bei denen sich die Gesamtlänge der Straßen mit `maxspeed` 30 oder 50 um max. 10% änderte |          7.399 | Ausschluss von Kacheln mit größeren Änderungen wie neuen Straßen                                              |
  | Anzahl der Kacheln mit einer Gesamtlänge von Straßen (`maxspeed` 30 oder 50) von mindestens 300m in 2024  |          6.358 | Mindestlänge für statistische Verlässlichkeit                                                                 |
  | Anzahl der Kacheln mit mehr als 2 Unfällen in 2018 & 2019 entlang einer Straße mit `maxspeed` 30 oder 50  |          2.801 | Mindestanzahl an Unfällen für statistische Verlässlichkeit                                                    |

* **Fazit:**

  Die Analyse zeigt, dass die Unfallzahlen in Bereichen, in denen eine Geschwindigkeitsreduzierung von Tempo 50 auf Tempo 30 erfolgte, zurückgegangen sind. Obwohl dieses Ergebnis erwartbar ist und keine neue Erkenntnis darstellt, verdeutlicht es, dass historische OSM-Daten grundsätzlich in der Lage sind, verkehrliche Maßnahmen zeitlich und räumlich abzubilden und deren Auswirkungen zu analysieren.

### 4. Herausforderungen bei der Umsetzung

* Unterscheidung zwischen tatsächlichen Maßnahmen und reinen Tagging-Anpassungen:
  * Nicht jede Änderung in OSM spiegelt eine reale verkehrliche Maßnahme wider. Manche Anpassungen resultieren bspw. aus Korrekturen, Verfeinerungen bestehender Daten oder Missverständnissen. Eine zentrale Herausforderung besteht darin, echte infrastrukturelle Veränderungen von Änderungen zu unterscheiden.
  * Erst wenn eine gewisse Vollständigkeit der Information vorliegt, kann eine Änderung als Hinweis auf eine tatsächliche Maßnahme gewertet werden.
  Handelt es sich lediglich um das Ergänzen bisher fehlender Informationen, lässt sich höchstens spekulieren, ob dies mit einer Maßnahme im realen Raum zusammenhängt oder nur eine verspätete Erfassung darstellt.

* Umgang mit Geometrieänderungen und deren Auswirkungen:
  * Geometrische Änderungen – etwa durch das Aufteilen, Zusammenführen oder Ergänzen von Straßenabschnitten – können zu neuen oder veränderten Way-IDs führen. Das erschwert die eindeutige Rückverfolgung historischer Entwicklungen und macht eine komplexe Datenverknüpfung erforderlich.

* Verzögerung (Time Lag) zwischen Maßnahme und OSM-Tagging:
  * Die zeitliche Verzögerung zwischen einer tatsächlichen Maßnahme vor Ort und ihrer Erfassung in OSM variiert stark. Sie hängt unter anderem vom spezifischen Tagging ab sowie davon, wie aktiv und aufmerksam die lokale OSM-Community ist.

### 5. Mitmachen

Du kannst dazu beitragen, die Qualität des Prozesses – insbesondere im Hinblick auf die Analyse unter Punkt 3b – zu verbessern, indem du Straßenabschnitte ohne Angaben zur zulässigen Geschwindigkeit, aber mit einer hohen Unfallhäufigkeit, vervollständigst.

#### Beispiele relevante Straßenabschnitte ohne Angabe der zulässigen Geschwindigkeit

* Straßenabschnitte in Deutschland mit mehr als 20 Unfällen in den letzten Jahren, aber ohne Angabe der zulässigen Geschwindigkeit:
  * [Query aufrufen](https://overpass-turbo.eu/s/21dx)

* Straßenabschnitte in Brandenburg, vermutlich innerorts, mit mehr als 5 Unfällen in den letzten Jahren, aber ohne Angabe der zulässigen Geschwindigkeit:
  * [Query aufrufen](https://overpass-turbo.eu/s/21dR)

Quellen:

* [1] https://docs.ohsome.org/ohsome-api/v1/ 
* [2] https://gdi.berlin.de/services/wfs/fahrradstrassen 
* [3] https://unfallatlas.statistikportal.de/
* [4] https://download.geofabrik.de/europe/germany.html#
* [5] https://osmcode.org/osmium-tool/
* [6] https://www.zensus2022.de/DE/Home/_inhalt.html
