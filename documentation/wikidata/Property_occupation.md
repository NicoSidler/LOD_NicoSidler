## Get the most frequent occupations

* If your population is in great number, use the [QLever SPARQL Endpoint](https://qlever.dev/wikidata)
* In the case of this population, we observe that the main occupations are the same that the ones used to define the population. The information appears to be less relevant to answer research questions.

```
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd:  <http://www.w3.org/2001/XMLSchema#>
PREFIX wd:   <http://www.wikidata.org/entity/>
PREFIX wdt:  <http://www.wikidata.org/prop/direct/>

SELECT ?object ?objectLabel (COUNT(DISTINCT ?item) AS ?eff)
WHERE {
  {
    SELECT DISTINCT ?item
    WHERE {
      ?item wdt:P31 wd:Q5 ;
            wdt:P569 ?birthDate .

      BIND(REPLACE(STR(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
      FILTER(xsd:integer(?year) >= 1801 && xsd:integer(?year) <= 1991)

      { ?item wdt:P106 wd:Q2306091 }   # occupation: sociologist
      UNION
      { ?item wdt:P101 wd:Q21201 }     # field of work: sociology
    }
  }

  ?item wdt:P106 ?object .
  ?object rdfs:label ?objectLabel .
  FILTER(LANG(?objectLabel) = "en")
}
GROUP BY ?object ?objectLabel
ORDER BY DESC(?eff)
LIMIT 10
```

### Most frequent occupations

|object                                 |objectLabel        |eff  |
|---------------------------------------|-------------------|-----|
|http://www.wikidata.org/entity/Q2306091|sociologist        |18324|
|http://www.wikidata.org/entity/Q1622272|university teacher |6277 |
|http://www.wikidata.org/entity/Q36180  |writer             |1600 |
|http://www.wikidata.org/entity/Q82955  |politician         |1196 |
|http://www.wikidata.org/entity/Q1238570|political scientist|1021 |
|http://www.wikidata.org/entity/Q201788 |historian          |1010 |
|http://www.wikidata.org/entity/Q4964182|philosopher        |924  |
|http://www.wikidata.org/entity/Q1930187|journalist         |766  |
|http://www.wikidata.org/entity/Q188094 |economist          |671  |
|http://www.wikidata.org/entity/Q4773904|anthropologist     |599  |

