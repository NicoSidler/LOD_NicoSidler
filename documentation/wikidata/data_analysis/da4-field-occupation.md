## Inspect the most frequent fields of work

* In the case of this population, we observe that although the main fields of work are physics and astronomy, there are many other fiels and it would be interesting to inspect specificities related to them.

```
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT ?object ?objectLabel (COUNT(*) AS ?eff)
WHERE {
  {
    SELECT DISTINCT ?person_uri
    WHERE {
      ?person_uri wdt:P31 wd:Q5;
                  wdt:P569 ?birthDate.
      BIND(REPLACE(STR(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
      FILTER(xsd:integer(?year) >= 1801 && xsd:integer(?year) <= 1990)

      { ?person_uri wdt:P106 wd:Q2306091 }   # sociologist
      UNION
      { ?person_uri wdt:P101 wd:Q21201 }     # sociology
    }
  }

  ?person_uri wdt:P101 ?object.
  ?object rdfs:label ?objectLabel.
  FILTER(LANG(?objectLabel) = "en")
}
GROUP BY ?object ?objectLabel
ORDER BY DESC(?eff)
LIMIT 100
```




### Most frequent fields of work

|object                                   |objectLabel                      |eff |
|-----------------------------------------|---------------------------------|----|
|http://www.wikidata.org/entity/Q21201    |sociology                        |4645|
|http://www.wikidata.org/entity/Q36442    |political science                |528 |
|http://www.wikidata.org/entity/Q5891     |philosophy                       |415 |
|http://www.wikidata.org/entity/Q461659   |sociology of religion            |300 |
|http://www.wikidata.org/entity/Q309      |history                          |295 |
|http://www.wikidata.org/entity/Q1662673  |gender studies                   |233 |
|http://www.wikidata.org/entity/Q8134     |economics                        |233 |
|http://www.wikidata.org/entity/Q1570681  |sociology of culture             |213 |
|http://www.wikidata.org/entity/Q23404    |anthropology                     |200 |
|http://www.wikidata.org/entity/Q9418     |psychology                       |173 |
|http://www.wikidata.org/entity/Q11030    |journalism                       |169 |
|http://www.wikidata.org/entity/Q828395   |social policy                    |168 |
|http://www.wikidata.org/entity/Q745692   |political sociology              |163 |
|http://www.wikidata.org/entity/Q7163     |politics                         |157 |
|http://www.wikidata.org/entity/Q12336277 |social research                  |155 |
|http://www.wikidata.org/entity/Q48277    |gender                           |149 |
|http://www.wikidata.org/entity/Q161733   |criminology                      |139 |
|http://www.wikidata.org/entity/Q156035   |opinion journalism               |133 |
|http://www.wikidata.org/entity/Q161272   |social psychology                |132 |
|http://www.wikidata.org/entity/Q7922     |pedagogy                         |131 |
|http://www.wikidata.org/entity/Q34749    |social science                   |129 |
|http://www.wikidata.org/entity/Q8434     |education                        |127 |
|http://www.wikidata.org/entity/Q7748     |law                              |119 |
|http://www.wikidata.org/entity/Q205398   |social work                      |109 |
|http://www.wikidata.org/entity/Q5431887  |social inequality                |106 |
|http://www.wikidata.org/entity/Q597680   |urban sociology                  |105 |
|http://www.wikidata.org/entity/Q7252     |feminism                         |103 |

&nbsp;

### Query to get the data and import them into the database

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT DISTINCT ?person_uri ?field_uri ?field_label
WHERE {
  {
    SELECT DISTINCT ?person_uri
    WHERE {
      ?person_uri wdt:P31 wd:Q5;
                  wdt:P569 ?birthDate.
      BIND(REPLACE(STR(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
      FILTER(xsd:integer(?year) >= 1801 && xsd:integer(?year) <= 1990)

      { ?person_uri wdt:P106 wd:Q2306091 }   # sociologist
      UNION
      { ?person_uri wdt:P101 wd:Q21201 }     # sociology
    }
  }

  ?person_uri wdt:P101 ?field_uri.
  ?field_uri rdfs:label ?field_label.
  FILTER(LANG(?field_label) = "en")
}
LIMIT 100

```



#### Experimental : DO NOT USE
```sparql

PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
SELECT DISTINCT ?person_uri ?field ?fieldLabel ?parentField ?parentFieldLabel ?parentClass ?parentClassLabel
WHERE
    {
    ### subquery adding the distinct clause
        {
        SELECT DISTINCT ?person_uri
        WHERE {
        ?person_uri wdt:P31 wd:Q5; 
              wdt:P569 ?birthDate.
        BIND(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
        FILTER(xsd:integer(?year) > 1780 && xsd:integer(?year) < 1981)# Any instance of a human.
            {?person_uri wdt:P106 wd:Q11063}
            UNION
            {?person_uri wdt:P101 wd:Q333} 
            UNION
            {?person_uri wdt:P106 wd:Q169470}
            UNION
            {?person_uri wdt:P101 wd:Q413} 
            }
        } 
        ### The property P101 associates fields of work to persons
        ?person_uri wdt:P101 ?field.
        ?field rdfs:label ?fieldLabel.
        FILTER(LANG(?fieldLabel) = 'en')
        OPTIONAL {
                # is instance of
                ?field wdt:P31 ?parentClass.
                        ?parentClass rdfs:label ?parentClassLabel.
                FILTER(LANG(?parentClassLabel) = 'en')
            }
        OPTIONAL {
                # is subclass of
                ?field wdt:P279 ?parentField.
                        ?parentField rdfs:label ?parentFieldLabel.
                FILTER(LANG(?parentFieldLabel) = 'en')
            }  
}  
# LIMIT 100

```




### Create a new table

* Download the result of this query as a CSV file
* Import it as a new table into the SQLite database
  * rename the column names during the import process 
* Inspect the imported data using the SQL scripts in the file [] 




## Occupations

```sparql
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT ?person_uri ?occupation_label ?occupation_uri
WHERE
    {
        {
        SELECT DISTINCT ?person_uri
        WHERE {
        ?person_uri wdt:P31 wd:Q5; 
              wdt:P569 ?birthDate.
        BIND(REPLACE(str(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
        FILTER(xsd:integer(?year) >= 1801 && xsd:integer(?year) <= 1990)
            {?person_uri wdt:P106 wd:Q2306091}   # sociologist
            UNION
            {?person_uri wdt:P101 wd:Q21201}     # sociology
            }
        } 
	
        ### The property P106 associates occupations to persons
        ?person_uri wdt:P106 ?occupation_uri.
        ?occupation_uri rdfs:label ?occupation_label.
        FILTER(LANG(?occupation_label) = 'en')
}  
ORDER BY ?person_uri ?occupation_uri
LIMIT 10
  
```
