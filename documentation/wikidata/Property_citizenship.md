## Get the most frequent citizenships

* If your population is in great number, use the [QLever SPARQL Endpoint](https://qlever.dev/wikidata)
* In the case of this population, we observe that the main occupations are the same that the ones used to define the population. The information appears to be less relevant to answer research questions.

```
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd:  <http://www.w3.org/2001/XMLSchema#>
PREFIX wd:   <http://www.wikidata.org/entity/>
PREFIX wdt:  <http://www.wikidata.org/prop/direct/>

SELECT ?citizenship ?citizenshipLabel (COUNT(DISTINCT ?item) AS ?eff)
WHERE {
  {
    SELECT DISTINCT ?item
    WHERE {
      ?item wdt:P31 wd:Q5 ;
            wdt:P569 ?birthDate .

      FILTER("1801-00-00T00:00:00Z"^^xsd:dateTime <= ?birthDate &&
             ?birthDate < "1992-00-00T00:00:00Z"^^xsd:dateTime)

      { ?item wdt:P106 wd:Q2306091 }   # occupation: sociologist
      UNION
      { ?item wdt:P101 wd:Q21201 }     # field of work: sociology
    }
  }

  ?item wdt:P27 ?citizenship .
  ?citizenship rdfs:label ?citizenshipLabel .
  FILTER(LANG(?citizenshipLabel) = "en")
}
GROUP BY ?citizenship ?citizenshipLabel
ORDER BY DESC(?eff)
LIMIT 30
```

### Most frequent citizenships

|citizenship                           |citizenshipLabel          |eff |
|--------------------------------------|--------------------------|----|
|http://www.wikidata.org/entity/Q183   |Germany                   |1759|
|http://www.wikidata.org/entity/Q30    |United States             |1676|
|http://www.wikidata.org/entity/Q142   |France                    |1236|
|http://www.wikidata.org/entity/Q36    |Poland                    |735 |
|http://www.wikidata.org/entity/Q17    |Japan                     |709 |
|http://www.wikidata.org/entity/Q38    |Italy                     |640 |
|http://www.wikidata.org/entity/Q145   |United Kingdom            |476 |
|http://www.wikidata.org/entity/Q29    |Spain                     |420 |
|http://www.wikidata.org/entity/Q16    |Canada                    |389 |
|http://www.wikidata.org/entity/Q15180 |Soviet Union              |360 |
|http://www.wikidata.org/entity/Q28    |Hungary                   |340 |
|http://www.wikidata.org/entity/Q159   |Russia                    |284 |
|http://www.wikidata.org/entity/Q155   |Brazil                    |261 |
|http://www.wikidata.org/entity/Q29999 |Kingdom of the Netherlands|242 |
|http://www.wikidata.org/entity/Q215   |Slovenia                  |236 |
|http://www.wikidata.org/entity/Q20    |Norway                    |215 |
|http://www.wikidata.org/entity/Q40    |Austria                   |199 |
|http://www.wikidata.org/entity/Q172579|Kingdom of Italy          |180 |
|http://www.wikidata.org/entity/Q33946 |Czechoslovakia            |176 |
|http://www.wikidata.org/entity/Q39    |Switzerland               |173 |
|http://www.wikidata.org/entity/Q801   |Israel                    |160 |
|http://www.wikidata.org/entity/Q33    |Finland                   |146 |
|http://www.wikidata.org/entity/Q414   |Argentina                 |145 |
|http://www.wikidata.org/entity/Q34    |Sweden                    |134 |
|http://www.wikidata.org/entity/Q43    |Turkey                    |130 |
|http://www.wikidata.org/entity/Q188712|Empire of Japan           |127 |
|http://www.wikidata.org/entity/Q31    |Belgium                   |125 |
|http://www.wikidata.org/entity/Q218   |Romania                   |117 |
|http://www.wikidata.org/entity/Q419   |Peru                      |104 |
|http://www.wikidata.org/entity/Q191   |Estonia                   |104 |

