# osm_history_analysis

In Deutschland existiert bisher keine einheitliche, öffentlich zugängliche Datenbank, die verkehrliche  Maßnahmen und Änderungen räumlich und zeitlich darstellt. Eine solche Datenbasis könnte jedoch wertvolle Erkenntnisse zur Bewertung verkehrlicher Maßnahmen liefern, indem sie deren Auswirkungen –  beispielsweise auf Verkehrsnachfrage oder Unfallhäufigkeit – transparenter macht.
Eine mögliche Lösung wäre die Entwicklung eines Prozesses, der mithilfe der OpenStreetMap (OSM)-Datenbank und des ohsome-Frameworks [1] spezifische Maßnahmen wie bspw. Geschwindigkeitsbeschränkungen  identifiziert und deren räumliche sowie zeitliche Veränderungen erfasst.

**Vorgehen im Projekt:**

1. Grundlage schaffen: 
   * Im ersten Schritt wurde ein grundlegender Verarbeitungsprozess in  Python entwickelt, um Daten aus ohsome zu extrahieren und zu  verarbeiten.

2. Erstellung von Prototypen:

   * Prototyp 1: Die Entwicklung eines ersten Prototyps wurde in einem  kleinen Testgebiet durchgeführt, bei dem Änderungen der zulässigen Höchstgeschwindigkeit (`maxspeed`) zum 1. Januar eines Jahres ausgewertet wurden.

     * https://radverkehr.github.io/digitalisierungsmodul/SoSe2024/viz/historic_osm-maxspeed_hermannstr.html

       

     * Final
           
       
       ```mermaid
       flowchart TD
           subgraph Z["Choose Area of interest"]
       		direction RL
          		A2@{ shape: cyl, label: "Overpass / CustomShape"}--> A1("get geometry")
       	end
           
           
           subgraph ZA["Processing historic data"]
       		direction RL
          		B2@{ shape: cyl, label: "ohsome API"}--> B1("get geometry")
       		
               B1 --> C(adding tags to df)
               C --> D(adding year to linestrings, if current for 1st jan)
           end
           
           subgraph ZB["Vizualize"]
       		E(Vizualize as interactive map via altair)
           end
           
           
       Z --> ZA
       ZA --> ZB
       ```
       
     * Test1
       
       ```mermaid
       flowchart TD
           A(choose\get geometry:\n custom shape )--> B(get historic Data for geometry via ohsome)
           
           
           subgraph Processing histoic data 
               B --> C(adding tags to df)
               C --> D(adding year to linestrings, if current for 1st jan)
           end
           
           D --> E(Vizualize as interactive map via altair)
       
       ```

   * Test 2
     
     ```mermaid
     
     flowchart TD
     
     subgraph Z[" "]
     direction LR
       A1(choose) --> A2[sfs]
     
     end
     
     subgraph ZA[" "]
     direction RL
         D-->E
         E-->F
     end
     
     Z --> ZA
     ```
     
     Test 3
     
     ```mermaid
     
     flowchart TD
     subgraph Z[" "]
     direction RL
        A2@{ shape: cyl, label: "Database"}--> A1(choose\get geometry:\n custom shape)
     
     end
     
     subgraph ZA[" "]
     direction RL
         D-->E
         E-->F
     end
     
     Z --> ZA
     ```
     
     Test 
     
       ```mermaid
     ---
     title: Prototype1 as mermaid flowchart
     ---
     flowchart  TD
         subgraph SG1["Choose Area of interest"]
     		direction RL
        		A2@{ shape: cyl, label: "CustomShape"}--> A1("get geometry")
     	end
         
         subgraph SG2["Processing historic osm-data"]
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
         
     
         subgraph SG3["Vizualize"]
         	direction BT
     		G(select year) --> E(Vizualize as interactive map via altair) 
         end
         
         
     
     SG1 --> SG2 --> SG3
     
     %% Comments after double percent signs
     %% Z7 --> ZA7 --> ZB7
     
     
   * Prototyp 2: Ein weiterer Prototyp stellt Geschwindigkeitsänderungen (`maxspeed`) auf Hauptverkehrsstraßen im Berliner Bezirk Neukölln dar. Die  Visualisierung enthält einen Zeitstrahl zur Darstellung der Änderungen  über einen längeren Zeitraum sowie eine Verlinkung der Way-Historie. 

     * https://radverkehr.github.io/osm_history_analysis/viz/maps_nk_no-basemap_PrimSecTert_wayChart_24-11-04.html   
     
     *   
     
       ```mermaid
       ---
       title: Prototype2 as mermaid flowchart
       ---
       flowchart  TD
           subgraph SG1["Choose Area of interest"]
       		direction RL
          		A2@{ shape: cyl, label: "OpenStreetMap"}-->|Overpass API| A1("get geometry")
       	end
           
           subgraph SG2["Processing historic osm-data"]
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
           
       
           subgraph SG3["Vizualize"]
           	direction BT
       		F(Include way's timeline as chart) --> E(Vizualize as interactive map via altair) 
       		G(select from continuous time) --> E(Vizualize as interactive map via altair) 
           end
           
           
       
       SG1 --> SG2 --> SG3
       
       %% Comments after double percent signs
       %% Z7 --> ZA7 --> ZB7
       ```
     

3. Ein Benchmarking soll zeigen, wie gut die Methode funktioniert und ob bzw. welche Probleme auftreten. 
   * Beispielsweise wird untersucht, wie schnell bzw. ob neue  Fahrradstraßen in OSM getaggt wurden. Dazu werden als Vergleich die  amtlichen Daten zu Fahrradstraßen der Berliner Senatsverwaltung [2]  herangezogen und mit dem Zeitpunkt der Änderung in OSM (`bicycle_road=yes`) verglichen.
   * Zusätzlich soll untersucht werden, ob eine Reduktion der zulässigen Geschwindigkeit oder die Einrichtung separater Radinfrastrukturen zu  einem Rückgang der Fahrradunfälle in den jeweiligen Straßenabschnitten  führt. Grundlage hierfür sind Daten aus dem Unfallatlas [3].

**Herausforderungen bei der Umsetzung:**

* Unterscheidung von Änderungen und Maßnahmen: Wann handelt es sich  bei einer Änderung in OSM um eine tatsächliche verkehrliche Maßnahme,  und wann nur um eine Korrektur, Verfeinerung oder ein Missverständnis?
* Umgang mit Geometrieänderungen: Änderungen können zu veränderten  Geometrien führen (z.B. durch Aufteilung oder Ergänzung von  Straßenabschnitten), was Auswirkungen auf die entsprechenden Way-IDs  haben kann und die eindeutige Nachverfolgbarkeit erschwert.

[1] https://docs.ohsome.org/ohsome-api/v1/ 
[2] https://gdi.berlin.de/services/wfs/fahrradstrassen 
[3] https://unfallatlas.statistikportal.de/
