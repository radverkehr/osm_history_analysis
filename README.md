# osm_history_analysis

In Deutschland existiert bisher keine einheitliche, öffentlich zugängliche Datenbank, die verkehrliche  Maßnahmen und Änderungen räumlich und zeitlich darstellt. Eine solche Datenbasis könnte jedoch wertvolle Erkenntnisse zur Bewertung verkehrlicher Maßnahmen liefern, indem sie deren Auswirkungen –  beispielsweise auf Verkehrsnachfrage oder Unfallhäufigkeit – transparenter macht.
Eine mögliche Lösung wäre die Entwicklung eines Prozesses, der mithilfe der OpenStreetMap (OSM)-Datenbank und des ohsome-Frameworks [1] spezifische Maßnahmen wie bspw. Geschwindigkeitsbeschränkungen identifiziert und deren räumliche sowie zeitliche Veränderungen erfasst.

**Vorgehen im Projekt:**

1. Grundlage schaffen: 
   * Im ersten Schritt wurde ein grundlegender Verarbeitungsprozess in  Python entwickelt, um Daten aus ohsome zu extrahieren und zu  verarbeiten.

2. Erstellung von Prototypen:

   * Prototyp 1: Die Entwicklung eines ersten Prototyps wurde in einem kleinen Testgebiet durchgeführt, bei dem Änderungen der zulässigen Höchstgeschwindigkeit (`maxspeed`) zum 1. Januar eines Jahres ausgewertet wurden.

     * [Visualisierung öffnen](https://radverkehr.github.io/digitalisierungsmodul/SoSe2024/viz/historic_osm-maxspeed_hermannstr.html)

     * Diagramm:

       <details>
       <summary>Click to expand mermaid diagram</summary>

       ```mermaid
       ---
       title: Prototype1 (as mermaid flowchart)
       ---
       flowchart  TD
           subgraph SG1["choose area of interest"]
       		direction RL
           		A2@{ shape: cyl, label: "custom shape"}--> A1("get geometry")
       	end

           subgraph SG2["processing historic osm-data"]
           	direction TB
       		subgraph SG2-1["get data"]
       			direction RL
           			B2@{ shape: cyl, label: "OpenStreetMap"}--> |ohsome API| B1("get highways with _maxspeed_")
               end   

               subgraph SG2-2["clean up/process data"]
               	direction TB
               	C(retrieve tags to dataframe columns) --> D(get state of each 1st january)
               end 
               SG2-1 --> SG2-2
           end

       subgraph SG3["vizualize"]
           direction BT
       	G(select year) --> E(vizualize as interactive map via altair) 
       end

       SG1 --> SG2 --> SG3

       %% Comments after double percent signs
       ```
       </details>
    * Prototyp 2: Ein weiterer Prototyp stellt Geschwindigkeitsänderungen (`maxspeed`) auf Hauptverkehrsstraßen im Berliner Bezirk Neukölln dar. Die Visualisierung enthält einen Zeitstrahl zur Darstellung der Änderungen über einen längeren Zeitraum sowie eine Verlinkung der Way-Historie.

      * [Visualisierung öffnen](https://radverkehr.github.io/osm_history_analysis/viz/maps_nk_no-basemap_PrimSecTert_wayChart_24-11-04.html)

      * Diagramm:

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


3. Ein Benchmarking soll zeigen, wie gut die Methode funktioniert und ob bzw. welche Probleme auftreten.  
   * Beispielsweise wird untersucht, wie schnell bzw. ob neue **Fahrradstraßen** in OSM getaggt wurden. Dazu werden als Vergleich die amtlichen Daten zu Fahrradstraßen der Berliner Senatsverwaltung [2] herangezogen und mit dem Zeitpunkt der Änderung in OSM (`bicycle_road=yes`) verglichen.

     * [Visualisierung 1: Zeitlicher Vergleich OSM vs. Senat (Line Chart)](https://radverkehr.github.io/osm_history_analysis/viz/fahrradstrassen/lineplot_fahrradstr_berlin_osm2senat.html)  
     * [Visualisierung 2: Kartendarstellung der Fahrradstraßen in Berlin](https://radverkehr.github.io/osm_history_analysis/viz/fahrradstrassen/maps_fahrradstr_berlin.html)

   * Zusätzlich soll untersucht werden, ob eine Reduktion der **zulässigen Geschwindigkeit** oder die Einrichtung separater Radinfrastrukturen zu  einem Rückgang der Fahrradunfälle in den jeweiligen Straßenabschnitten führt. Grundlage hierfür sind Daten aus dem **Unfallatlas** [3].
        * [Visualisierung 1: Zusammenhang zwischen Unfallentwicklung und Veränderung zul. Geschwindigkeit](https://radverkehr.github.io/osm_history_analysis/viz/germany_raster2022_net_acc.html) 



**Herausforderungen bei der Umsetzung:**

* Unterscheidung zwischen tatsächlichen Maßnahmen und reinen Tagging-Anpassungen:
  * Nicht jede Änderung in OSM spiegelt eine reale verkehrliche Maßnahme wider. Manche Anpassungen resultieren aus Korrekturen, Verfeinerungen bestehender Daten oder Missverständnissen. Eine zentrale Herausforderung besteht darin, echte infrastrukturelle Veränderungen von bloßen redaktionellen Änderungen zu unterscheiden.
  * Erst wenn eine gewisse Vollständigkeit der Information vorliegt, kann eine Änderung als Hinweis auf eine tatsächliche Maßnahme gewertet werden.
  Handelt es sich lediglich um das Ergänzen bisher fehlender Informationen, lässt sich höchstens spekulieren, ob dies mit einer Maßnahme im realen Raum zusammenhängt oder nur eine verspätete Erfassung darstellt.

* Umgang mit Geometrieänderungen und deren Auswirkungen:
  * Geometrische Änderungen – etwa durch das Aufteilen, Zusammenführen oder Ergänzen von Straßenabschnitten – können zu neuen oder veränderten Way-IDs führen. Das erschwert die eindeutige Rückverfolgung historischer Entwicklungen und macht eine sorgfältige Datenverknüpfung erforderlich.

* Verzögerung (Time Lag) zwischen Maßnahme und OSM-Tagging:
  * Die zeitliche Verzögerung zwischen einer tatsächlichen Maßnahme vor Ort und ihrer Erfassung in OSM variiert stark. Sie hängt unter anderem vom spezifischen Tagging ab sowie davon, wie aktiv und aufmerksam die lokale OSM-Community ist.


Quellen:

* [1] https://docs.ohsome.org/ohsome-api/v1/ 
* [2] https://gdi.berlin.de/services/wfs/fahrradstrassen 
* [3] https://unfallatlas.statistikportal.de/
