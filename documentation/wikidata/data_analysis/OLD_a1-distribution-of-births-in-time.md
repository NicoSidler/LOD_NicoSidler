# Distribution of births and genders in time

## Explore available data

### Count available triples
77180

```sparql
### Number of triples in the graph
SELECT (COUNT(*) as ?n)
WHERE {
    GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata>
        {?s ?p ?o}
}
```

### Inspect the labels

Number of persons with more than one label : 504

```sparql
### 
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT (COUNT(*) AS ?n)
WHERE {
  {
    SELECT ?s
    WHERE {
      GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
        ?s a wd:Q5 ;
           rdfs:label ?label .
      }
    }
    GROUP BY ?s
    HAVING (COUNT(*) > 1)
  }
}
```

### Explore the gender

#### Group by and count genders

```
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?genLabel (COUNT(*) as ?n)
WHERE {
    GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata>
  {
    ?s a wd:Q5.   
    OPTIONAL {
      ?s wdt:P21 ?gen.
      ?gen rdfs:label ?genLabel
    }
  }
}
GROUP BY ?genLabel
ORDER BY DESC(?n)
```
| genLabel           | n     |
|--------------------|-------|
| male               | 13395 |
| female             | 5457  |
|                    | 162   |
| trans woman        | 9     |
| trans man          | 6     |
| non-binary         | 3     |
| undisclosed gender | 3     |
| cisgender woman    | 1     |
| genderqueer        | 1     |
| transmasculine     | 1     |



#### Persons with more than one gender

```
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?s (GROUP_CONCAT(?genLabel; separator=" | ") AS ?labels)
WHERE {
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
    ?s wdt:P21 ?gen .
    ?gen rdfs:label ?genLabel .
  }
}
GROUP BY ?s
HAVING (COUNT(*) > 1)

```
| s                                         | labels               |
|-------------------------------------------|----------------------|
| http://www.wikidata.org/entity/Q25394676  | male | female        |
| http://www.wikidata.org/entity/Q105840219 | male | female        |
| http://www.wikidata.org/entity/Q20968749  | male | female        |
| http://www.wikidata.org/entity/Q23946881  | male | female        |
| http://www.wikidata.org/entity/Q52016574  | male | female        |
| http://www.wikidata.org/entity/Q25460025  | female | trans woman |
| http://www.wikidata.org/entity/Q12358460  | male | female        |
| http://www.wikidata.org/entity/Q7383082   | male | female        |
| http://www.wikidata.org/entity/Q56849343  | male | trans man     |
| http://www.wikidata.org/entity/Q30694160  | male | female        |
| http://www.wikidata.org/entity/Q105646335 | male | female        |

## Prepare data to analyse

```sparql
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>


SELECT ?s ?label ?birthDate ?genLabel
WHERE {
    GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata>
        {
            ## A property path passes through 
            # two or more properties
            ?s wdt:P21 / rdfs:label ?genLabel;
            rdfs:label ?label;
            wdt:P569 ?birthDate.
          }
}
ORDER BY ?birthDate
LIMIT 10
```

### Number of persons (with double labels, genders, etc.)

Nuber: 19027

```sparql

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT (COUNT(*) as ?n)
WHERE {
    GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata>
        {
          # ?s wdt:P31 wd:Q5 
          ?s a wd:Q5
          }
}
```

### Take one person only once (FINAL QUERY)

This is the query to prepare the data for the analysis

```
### If there are two values for a property we take just one

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>


SELECT  ?s (MIN(?llabel) as ?label) (xsd:integer(MIN(?bbirthDate)) as ?birthDate) (MIN(?ggenLabel) AS ?genLabel)
WHERE {
    GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata>
        {?s a wd:Q5;
            wdt:P21 ?gen;
            rdfs:label ?llabel;
            wdt:P569 ?bbirthDate.
        ?gen rdfs:label ?ggenLabel  
          }
}
GROUP BY ?s
LIMIT 10

```

#### Number of real persons, without double labels, etc.

Number: 18865

```sparql

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT (COUNT(*) as ?n)
WHERE {
    SELECT  ?s (MIN(?llabel) as ?label) (xsd:integer(MIN(?bbirthDate)) as ?birthDate) (MIN(?ggenLabel) AS ?genLabel)
    WHERE {
        GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata>
            {?s a wd:Q5;
                wdt:P21 ?gen;
                rdfs:label ?llabel;
                wdt:P569 ?bbirthDate.
            ?gen rdfs:label ?ggenLabel  
            }
    }
    GROUP BY ?s
}
```

## Export the data for analysis

Execute the query, export the data in CSV format and store the *birth-dates-gender.csv* file in the *notebooks_jupyter/wikidata_exploration/da1_data/* directory

```
### If there are two values for a property we take just one

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>


SELECT  ?s (MIN(?llabel) as ?label) (xsd:integer(MIN(?bbirthDate)) as ?birthDate) (MIN(?ggenLabel) AS ?genLabel)
WHERE {
    GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata>
        {?s a wd:Q5;
            wdt:P21 ?gen;
            rdfs:label ?llabel;
            wdt:P569 ?bbirthDate.
        ?gen rdfs:label ?ggenLabel  
          }
}
GROUP BY ?s #getting rid of double values

```
