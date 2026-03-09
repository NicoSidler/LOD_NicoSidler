## Get the most frequent employers

* The property _P108 employer_ is quite frequent and adds an interesting relation to organisations that we will use for graph analysis



```
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd:  <http://www.w3.org/2001/XMLSchema#>
PREFIX wd:   <http://www.wikidata.org/entity/>
PREFIX wdt:  <http://www.wikidata.org/prop/direct/>

SELECT ?employer ?employerLabel (COUNT(DISTINCT ?item) AS ?eff)
WHERE {
  # --- population (same logic as your first query)
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

  # --- employer (outgoing property)
  ?item wdt:P108 ?employer .

  # --- English label
  ?employer rdfs:label ?employerLabel .
  FILTER(LANG(?employerLabel) = "en")
}
GROUP BY ?employer ?employerLabel
ORDER BY DESC(?eff)
LIMIT 30
```

## The 30 most frequent employers

* We observe that these are universities and research institutions.
* Are there also private companies ? 

| employer                                | employerLabel                                      | eff |
| --------------------------------------- | -------------------------------------------------- | --- |
| http://www.wikidata.org/entity/Q280413  | National Center for Scientific Research            | 151 |
| http://www.wikidata.org/entity/Q144488  | University of Warsaw                               | 147 |
| http://www.wikidata.org/entity/Q13371   | Harvard University                                 | 144 |
| http://www.wikidata.org/entity/Q49088   | Columbia University                                | 123 |
| http://www.wikidata.org/entity/Q131252  | University of Chicago                              | 120 |
| http://www.wikidata.org/entity/Q174710  | University of California, Los Angeles              | 101 |
| http://www.wikidata.org/entity/Q174570  | London School of Economics and Political Science   | 98  |
| http://www.wikidata.org/entity/Q168756  | University of California, Berkeley                 | 95  |
| http://www.wikidata.org/entity/Q273518  | School for Advanced Studies in the Social Sciences | 89  |
| http://www.wikidata.org/entity/Q1067935 | Laval University                                   | 85  |
| http://www.wikidata.org/entity/Q230492  | University of Michigan                             | 82  |
| http://www.wikidata.org/entity/Q153006  | Freie Universität Berlin                           | 78  |
| http://www.wikidata.org/entity/Q50662   | Goethe University Frankfurt                        | 72  |
| http://www.wikidata.org/entity/Q152087  | Humboldt-Universität zu Berlin                     | 71  |
| http://www.wikidata.org/entity/Q7842    | University of Tokyo                                | 69  |
| http://www.wikidata.org/entity/Q41506   | Stanford University                                | 66  |
| http://www.wikidata.org/entity/Q49210   | New York University                                | 63  |
| http://www.wikidata.org/entity/Q49112   | Yale University                                    | 61  |
| http://www.wikidata.org/entity/Q165980  | University of Vienna                               | 55  |
| http://www.wikidata.org/entity/Q392189  | Université de Montréal                             | 55  |
| http://www.wikidata.org/entity/Q214341  | University of Amsterdam                            | 54  |
| http://www.wikidata.org/entity/Q309350  | Northwestern University                            | 54  |
| http://www.wikidata.org/entity/Q49115   | Cornell University                                 | 53  |
| http://www.wikidata.org/entity/Q34433   | University of Oxford                               | 52  |
| http://www.wikidata.org/entity/Q156725  | University of Hamburg                              | 51  |
| http://www.wikidata.org/entity/Q189441  | Jagiellonian University                            | 50  |
| http://www.wikidata.org/entity/Q49117   | University of Pennsylvania                         | 49  |
| http://www.wikidata.org/entity/Q859363  | Sciences Po                                        | 49  |
| http://www.wikidata.org/entity/Q838330  | University of Wisconsin–Madison                    | 49  |
| http://www.wikidata.org/entity/Q230899  | University of Manchester                           | 48  |
|                                         |                                                    |     |






## Inspect the employer classes

```
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd:  <http://www.w3.org/2001/XMLSchema#>
PREFIX wd:   <http://www.wikidata.org/entity/>
PREFIX wdt:  <http://www.wikidata.org/prop/direct/>

PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
SELECT ?class ?classLabel (COUNT(DISTINCT ?item) AS ?eff)
WHERE {
  # --- subquery adding DISTINCT people (same population as your first query)
  {
    SELECT DISTINCT ?item
    WHERE {
      ?item wdt:P31 wd:Q5 ;
            wdt:P569 ?birthDate .

      BIND(REPLACE(STR(?birthDate), "(.*)([0-9]{4})(.*)", "$2") AS ?year)
      FILTER(xsd:integer(?year) >= 1801 && xsd:integer(?year) <= 1991)

      { ?item wdt:P106 wd:Q2306091 }
      UNION
      { ?item wdt:P101 wd:Q21201 }
    }
  }

  # --- employer and its class (instance of)
  ?item wdt:P108 ?employer .
  ?employer wdt:P31 ?class .

  # --- English labels
  ?class rdfs:label ?classLabel .
  FILTER(LANG(?classLabel) = "en")
}
GROUP BY ?class ?classLabel
ORDER BY DESC(?eff)
LIMIT 30
```

<br>

### The first 30 most frequent employer classes

| class                                     | classLabel                                          | eff  |
| ----------------------------------------- | --------------------------------------------------- | ---- |
| http://www.wikidata.org/entity/Q875538    | public university                                   | 3226 |
| http://www.wikidata.org/entity/Q45400320  | open-access publisher                               | 2837 |
| http://www.wikidata.org/entity/Q3918      | university                                          | 2795 |
| http://www.wikidata.org/entity/Q62078547  | public research university                          | 1953 |
| http://www.wikidata.org/entity/Q902104    | private university                                  | 1295 |
| http://www.wikidata.org/entity/Q43229     | organization                                        | 1256 |
| http://www.wikidata.org/entity/Q23002054  | private not-for-profit educational institution      | 1111 |
| http://www.wikidata.org/entity/Q23002039  | public educational institution of the United States | 1043 |
| http://www.wikidata.org/entity/Q15936437  | research university                                 | 993  |
| http://www.wikidata.org/entity/Q31855     | research institute                                  | 789  |
| http://www.wikidata.org/entity/Q1767829   | comprehensive university                            | 783  |
| http://www.wikidata.org/entity/Q96888669  | academic publisher                                  | 733  |
| http://www.wikidata.org/entity/Q38723     | higher education institution                        | 661  |
| http://www.wikidata.org/entity/Q5341295   | educational organization                            | 623  |
| http://www.wikidata.org/entity/Q615150    | land-grant university                               | 527  |
| http://www.wikidata.org/entity/Q1188663   | Colonial Colleges                                   | 421  |
| http://www.wikidata.org/entity/Q163740    | nonprofit organization                              | 413  |
| http://www.wikidata.org/entity/Q708676    | charitable organization                             | 359  |
| http://www.wikidata.org/entity/Q115427560 | University of Excellence                            | 357  |
| http://www.wikidata.org/entity/Q2385804   | educational institution                             | 356  |
| http://www.wikidata.org/entity/Q3551775   | university in France                                | 345  |
| http://www.wikidata.org/entity/Q2085381   | publishing house                                    | 332  |
| http://www.wikidata.org/entity/Q641347    | local Internet registry                             | 257  |
| http://www.wikidata.org/entity/Q5028741   | campus university                                   | 245  |
| http://www.wikidata.org/entity/Q265662    | national university                                 | 238  |
| http://www.wikidata.org/entity/Q209465    | campus                                              | 236  |
| http://www.wikidata.org/entity/Q557206    | Catholic university                                 | 189  |
| http://www.wikidata.org/entity/Q3551519   | university in Quebec                                | 185  |
| http://www.wikidata.org/entity/Q3914      | school                                              | 170  |
| http://www.wikidata.org/entity/Q1371037   | institute of technology                             | 165  |
|                                           |                                                     |      |




<br/>


### Examples of private companies

```
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd:  <http://www.w3.org/2001/XMLSchema#>
PREFIX wd:   <http://www.wikidata.org/entity/>
PREFIX wdt:  <http://www.wikidata.org/prop/direct/>

PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
SELECT ?employer ?employerLabel ?classLabel (COUNT(DISTINCT ?item) AS ?eff)
WHERE {
  # --- subquery: distinct people (same population as your first query)
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

  # --- employer and its class (instance of)
  ?item wdt:P108 ?employer .
  ?employer wdt:P31 ?class .

  # --- labels
  ?employer rdfs:label ?employerLabel .
  FILTER(LANG(?employerLabel) = "en")

  ?class rdfs:label ?classLabel .
  FILTER(LANG(?classLabel) = "en")

  # --- keep only employer classes whose English label suggests a company-like entity
  FILTER(REGEX(?classLabel, "(business|enterprise|company)", "i"))
}
GROUP BY ?employer ?employerLabel ?classLabel
ORDER BY DESC(?eff) ?employerLabel
LIMIT 20
```


| employer                                  | employerLabel                          | classLabel         | eff |
| ----------------------------------------- | -------------------------------------- | ------------------ | --- |
| http://www.wikidata.org/entity/Q49112     | Yale University                        | production company | 61  |
| http://www.wikidata.org/entity/Q238101    | University of Minnesota                | production company | 24  |
| http://www.wikidata.org/entity/Q127990    | Australian National University         | production company | 19  |
| http://www.wikidata.org/entity/Q703620    | Copenhagen Business School             | business school    | 16  |
| http://www.wikidata.org/entity/Q598841    | Monash University                      | production company | 15  |
| http://www.wikidata.org/entity/Q734764    | University of New South Wales          | production company | 13  |
| http://www.wikidata.org/entity/Q332498    | Brigham Young University               | production company | 10  |
| http://www.wikidata.org/entity/Q1057890   | RMIT University                        | production company | 8   |
| http://www.wikidata.org/entity/Q1394594   | SGH Warsaw School of Economics         | business school    | 8   |
| http://www.wikidata.org/entity/Q273535    | HEC Paris                              | business school    | 6   |
| http://www.wikidata.org/entity/Q1574858   | Handelshochschule Berlin               | business school    | 5   |
| http://www.wikidata.org/entity/Q658192    | Vilnius University                     | business           | 5   |
| http://www.wikidata.org/entity/Q2822255   | Kozminski University                   | business school    | 4   |
| http://www.wikidata.org/entity/Q142740    | MIT Sloan School of Management         | business school    | 4   |
| http://www.wikidata.org/entity/Q1329269   | The Wharton School                     | business school    | 4   |
| http://www.wikidata.org/entity/Q1756541   | Vytautas Magnus University             | business           | 4   |
| http://www.wikidata.org/entity/Q114878562 | Analysis & Numbers                     | consulting company | 3   |
| http://www.wikidata.org/entity/Q49126     | Harvard Business School                | business school    | 3   |
| http://www.wikidata.org/entity/Q273527    | HEC Montréal                           | business school    | 3   |
| http://www.wikidata.org/entity/Q3546588   | Institut Mines-Telecom Business School | business school    | 3   |
|                                           |                                        |                    |     |
