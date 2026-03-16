## Import the data into a triplestore

&nbsp;

In this notebook we describe the steps needed to import the data into your own triplestore. As a triplestore you can use Fuseki, or Allegrograph, or any other technology.

The triplestore can be local or online.

First we check the basic properties of the population: name, gender, year of birth.

Copy-paste this query to the SPARQL-editor of your choice (Fuseki, Allegrograph, etc.)

```sparql
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?item ?gender ?year ?itemLabel
WHERE {

  ## note the service address
  SERVICE <https://query.wikidata.org/sparql>
    {
      {?item wdt:P106 wd:Q2306091}  # sociologist
      UNION
      {?item wdt:P101 wd:Q21201}    # sociology

      ?item wdt:P31 wd:Q5;          # Any instance of a human.
            wdt:P569 ?birthDate.    # It must necessarily have a birth date property

      OPTIONAL {
        ?item wdt:P21 ?gender.      # gender is optional
      }

      BIND(year(?birthDate) as ?year)
      FILTER(xsd:integer(?year) > 1801 && xsd:integer(?year) < 1990)

      OPTIONAL {
        ?item rdfs:label ?itemLabel.
        FILTER(LANG(?itemLabel) = 'en')
      }
    }
}
LIMIT 10
  
```

### Count number of persons to import

23,208 people as of 12 March 2026

```sparql
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT (count(*) as ?effectif)
WHERE {

  ## note the service address
  SERVICE <https://query.wikidata.org/sparql>
    {
      {?item wdt:P106 wd:Q2306091}  # sociologist
      UNION
      {?item wdt:P101 wd:Q21201}    # sociology

      ?item wdt:P31 wd:Q5;          # Any instance of a human.
            wdt:P569 ?birthDate.    # It must necessarily have a birth date property

      OPTIONAL {
        ?item wdt:P21 ?gender.      # gender is optional
      }

      BIND(year(?birthDate) as ?year)
      FILTER(xsd:integer(?year) >= 1801 && xsd:integer(?year) <= 1990)

      OPTIONAL {
        ?item rdfs:label ?itemLabel.
        FILTER(LANG(?itemLabel) = 'en')
      }
    }
}
```

### Preparing to import data

* Here we use the CONSTRUCT query to prepare the triples for import into a graph database.
* We limit the test to a few rows to avoid displaying thousands of them.
* Inspect and check the triplets that are generated.
* Reuse if possible the Wikidata properties

```sparql
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

CONSTRUCT 
{
  ?item wdt:P21 ?gender.
  ?item wdt:P569 ?year.
  ?item rdfs:label ?itemLabel.
  ?item rdf:type wd:Q5.
}
WHERE {

  ## note the service address
  SERVICE <https://query.wikidata.org/sparql>
    {
      {?item wdt:P106 wd:Q2306091}  # sociologist
      UNION
      {?item wdt:P101 wd:Q21201}    # sociology

      ?item wdt:P31 wd:Q5;          # Any instance of a human.
            wdt:P569 ?birthDate.    # It must necessarily have a birth date property

      OPTIONAL {
        ?item wdt:P21 ?gender.      # gender is optional
      }

      BIND(year(?birthDate) as ?year)
      FILTER(xsd:integer(?year) >= 1801 && xsd:integer(?year) <= 1990)

      OPTIONAL {
        ?item rdfs:label ?itemLabel.
        FILTER(LANG(?itemLabel) = 'en')
      }
    }
}
LIMIT 5
  

```

### Import the triples into a dedicated graph

Two import strategies are possible:

* [to be preferred] directly in your triplestore using a federated query (clause SERVICE <service.url>)
  * the query can be executed on a sparql-book
  * or directly in the query page of your triplestore
* directly in Wikidata with export of the data and then import to your triplestore
  * execute a CONTRUCT query with the complete data (without the SERVICE and LIMIT clause) and export it to the Turtle format (suffix: .ttl)
  * then import the data into your triplestore using the import graphical interface

This is the recommended format for a graph URI in this context:

```
<https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata>

```


The graph URI is in fact a URL pointing to a page with the description of the [imported data](https://NicoSidler.github.io/sociologists/graphs-defs.html).

#### Base request

```sparql
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

INSERT {

  ### Note that the data is imported into a named graph and not the DEFAULT one
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata>
  {
    ?item wdt:P21 ?gender.
    ?item wdt:P569 ?year.
    ?item rdfs:label ?itemLabel.
    ?item rdf:type wd:Q5.
  }
}

WHERE {

  SELECT DISTINCT ?item ?year ?gender ?itemLabel

  WHERE {

    ## note the service address
    SERVICE <https://query.wikidata.org/sparql>
      {
        {?item wdt:P106 wd:Q2306091}  # sociologist
        UNION
        {?item wdt:P101 wd:Q21201}    # sociology

        ?item wdt:P31 wd:Q5;
              wdt:P569 ?birthDate.

        OPTIONAL {
          ?item wdt:P21 ?gender.
        }

        BIND(year(?birthDate) as ?year)
        FILTER(xsd:integer(?year) >= 1801 && xsd:integer(?year) <= 1990)

        OPTIONAL {
          ?item rdfs:label ?itemLabel.
          FILTER(LANG(?itemLabel) = 'en')
        }
      }
  }
  ORDER BY ?item
  OFFSET 0
  LIMIT 10000
}

```

### Inspect imported data

```
PREFIX wd: <http://www.wikidata.org/entity/>

SELECT *
WHERE {
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
    ?item a wd:Q5;
          ?p ?o.
  }
}
ORDER BY ?item ?p
LIMIT 20
      
```

### Count imported data

Imported: 9972

La différence d'effectif peut s'expliquer par des propriétés doubles.

```
PREFIX wd: <http://www.wikidata.org/entity/>

SELECT (COUNT(*) as ?number)
WHERE {
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
    ## deux expressions équivalentes
    # ?item rdf:type wd:Q5
    ?item a wd:Q5
  }
} 
```

### Multiple dates

```
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT (COUNT(*) as ?number) ?item
WHERE {
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
    ## deux expressions équivalentes
    # ?item rdf:type wd:Q5
    ?item a wd:Q5;
          wdt:P569 ?birthDate.
  }
}
GROUP BY ?item
HAVING (COUNT(*) > 1)

```

### Multiple genders

```
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT (COUNT(*) as ?number) ?item
WHERE {
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
    ## deux expressions équivalentes
    # ?item rdf:type wd:Q5
    ?item a wd:Q5;
          wdt:P21 ?gender.
  }
}
GROUP BY ?item
HAVING (COUNT(*) > 1)
```

### Multiple labels

```
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT (COUNT(*) as ?number) ?item
WHERE {
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
    ## deux expressions équivalentes
    # ?item rdf:type wd:Q5
    ?item a wd:Q5;
          rdfs:label ?label.
  }
}
GROUP BY ?item
HAVING (COUNT(*) > 1)
```






## New request to correct multiple dates

On vide d'abord le graphe préalablement rempli (ATTENTION: tous les triplets sont éliminés)

```
CLEAR GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata>

```

On le remplite ensuite de nouveau.

Noter la clause qui permet de traiter progressivement les données afin d'éviter les blocages du triplestore.

D'abord on prend les 10000 premiers triplets:

```
OFFSET 0
LIMIT 10000
```

puis, après succès de l'update, on prend les dix-mille suivants:

```
OFFSET 10000
LIMIT 20000
```
et ainsi de suite.


La requête perfectionnée:

```sparql
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

INSERT {

  ### Note that the data is imported into a named graph and not the DEFAULT one
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata>
  {
    ?item wdt:P21 ?min_gender.
    ?item wdt:P569 ?min_year.
    ?item rdfs:label ?min_label.
    ?item rdf:type wd:Q5.
  }
}

WHERE {

  SELECT ?item (MIN(?year) as ?min_year) (MIN(?gender) as ?min_gender) (MIN(?itemLabel) as ?min_label)

  WHERE {

    SERVICE <https://query.wikidata.org/sparql>
      {
        {?item wdt:P106 wd:Q2306091}  # sociologist
        UNION
        {?item wdt:P101 wd:Q21201}    # sociology

        ?item wdt:P31 wd:Q5;
              wdt:P569 ?birthDate.

        OPTIONAL {
          ?item wdt:P21 ?gender.
        }

        BIND(year(?birthDate) as ?year)
        FILTER(xsd:integer(?year) >= 1801 && xsd:integer(?year) <= 1990)

        OPTIONAL {
          ?item rdfs:label ?itemLabel.
          FILTER(LANG(?itemLabel) = 'en')
        }
      }
  }
  GROUP BY ?item
  ORDER BY ?item
  OFFSET 0
  LIMIT 10000
}
```










### Find persons without labels

707 persons

```
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT (COUNT(*) AS ?number)
WHERE {
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
    ?item a wd:Q5.
    MINUS { ?item rdfs:label ?label }
  }
}
```


```
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

SELECT ?item ?itemLabel
WHERE {
  {
    SELECT ?item
    WHERE {
      GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
        ?item a wd:Q5 .
        MINUS { ?item rdfs:label ?label }
      }
    }
    LIMIT 10
  }

  SERVICE <https://query.wikidata.org/sparql> {
    SERVICE wikibase:label { bd:serviceParam wikibase:language "ru". }
  }
}
```




#### Add a label to the class "Person"

```sparql
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

INSERT DATA {
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata>
  {
    wd:Q5 rdfs:label "Person".
  }
}

```

### Add the gender class

```sparql
### Inspect the genders:9

PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT (COUNT(*) as ?n)
WHERE {
  SELECT DISTINCT ?gender
  WHERE {
    GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
      ?s wdt:P21 ?gender.
    }
  }
}
```

```sparql
### Insert the class 'gender' for all types of gender
# Please note that strictly speaking Wikidata has no ontology,
# therefore no classes. We add this for our convenience

PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

WITH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata>
INSERT {
  ?gender rdf:type wd:Q48264.
}
WHERE {
  SELECT DISTINCT ?gender
  WHERE {
    {
      ?s wdt:P21 ?gender.
    }
  }
}
```

```sparql
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

INSERT DATA {
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata>
  {
    wd:Q48264 rdfs:label "Gender Identity".
  }
}
```

### Verify imported triples and add labels to genders
75207 triples

```sparql
### Number of triples in the graph
SELECT (COUNT(*) as ?n)
WHERE {
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
    ?s ?p ?o
  }
}
```
None
```sparql
### Number of persons with more than one label : no person
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT (COUNT(*) as ?n)
WHERE {
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
    ?s rdfs:label ?o
  }
}
GROUP BY ?s
HAVING (?n > 1)
```

### Explore the gender


None
```sparql
### Number of persons having more than one gender
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT ?s (COUNT(*) as ?n)
WHERE {
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
    ?s wdt:P21 ?gen
  }
}
GROUP BY ?s
HAVING (?n > 1)
```

```sparql
### Number of persons per gender
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT ?gen (COUNT(*) as ?n)
WHERE {
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
    ?s wdt:P21 ?gen
  }
}
GROUP BY ?gen
#HAVING (?n > 1)
```

|gen                                      |n    |
|-----------------------------------------|-----|
|http://www.wikidata.org/entity/Q15145779 |1    |
|http://www.wikidata.org/entity/Q1052281  |9    |
|http://www.wikidata.org/entity/Q6581072  |5452 |
|http://www.wikidata.org/entity/Q2449503  |6    |
|http://www.wikidata.org/entity/Q12964198 |1    |
|http://www.wikidata.org/entity/Q48270    |3    |
|http://www.wikidata.org/entity/Q27679766 |1    |
|http://www.wikidata.org/entity/Q6581097  |13379|
|http://www.wikidata.org/entity/Q113124952|3    |


```sparql
### Number of persons per gender in relation to a period
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT ?gen (COUNT(*) as ?n)
WHERE {
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
    ?s wdt:P21 ?gen;
       wdt:P569 ?birthDate.
    FILTER(?birthDate < 1900)
  }
}
GROUP BY ?gen
#HAVING (?n > 1)
```
|gen                                      |n    |
|-----------------------------------------|-----|
|http://www.wikidata.org/entity/Q6581072  |104  |
|http://www.wikidata.org/entity/Q6581097  |1075 |


```sparql
### Add the label to the gender

# This query will first retrieve all the genders,
# then fetch in Wikidata the gender's labels

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>

SELECT ?gen ?genLabel
WHERE {

  {
    SELECT DISTINCT ?gen
    WHERE {
      GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
        ?s wdt:P21 ?gen
      }
    }
  }

  SERVICE <https://query.wikidata.org/sparql> {
    ## Add this clause in order to fill the variable
    BIND(?gen as ?gen)
    BIND(?genLabel as ?genLabel)
    SERVICE wikibase:label { bd:serviceParam wikibase:language "en" }
  }
}
```

```sparql
### Add the label to the gender

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

CONSTRUCT {
  ?gen rdfs:label ?genLabel
}
WHERE {
  {
    SELECT DISTINCT ?gen
    WHERE {
      GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
        ?s wdt:P21 ?gen
      }
    }
  }

  SERVICE <https://query.wikidata.org/sparql> {
    BIND(?gen AS ?gen)
    BIND(?genLabel AS ?genLabel)
    SERVICE wikibase:label { bd:serviceParam wikibase:language "en" }
  }
}
```

```sparql
### Add the label to the gender: INSERT

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

WITH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata>
INSERT {
  ?gen rdfs:label ?genLabel
}
WHERE {
  {
    SELECT DISTINCT ?gen
    WHERE {
      GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
        ?s wdt:P21 ?gen
      }
    }
  }

  SERVICE <https://query.wikidata.org/sparql> {
    ## Add this clause in order to fill the variable
    BIND(?gen as ?gen)
    BIND(?genLabel as ?genLabel)
    SERVICE wikibase:label { bd:serviceParam wikibase:language "en" }
  }
}
```

```sparql
### Verify data insertion - using only Allegrograph data

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?gen ?genLabel ?n
WHERE {
  {
    SELECT ?gen (COUNT(*) as ?n)
    WHERE {
      GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
        ?s wdt:P21 ?gen.
      }
    }
    GROUP BY ?gen
  }
  ?gen rdfs:label ?genLabel
}

```

### Prepare data to analyse

```sparql
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT ?s ?label ?birthDate ?genLabel
WHERE {
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
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

```sparql
### Number of persons: 19016

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT (COUNT(*) as ?n)
WHERE {
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
    # ?s wdt:P31 wd:Q5
    ?s a wd:Q5
  }
}
```

```sparql
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT ?s
       (MAX(?label) AS ?oneLabel)
       (xsd:integer(MAX(?birthDate)) AS ?oneBirthDate)
       (MAX(?gen) AS ?oneGen)
       (MAX(?genLabel) AS ?oneGenLabel)
WHERE {
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
    ?s wdt:P21 ?gen ;
       rdfs:label ?label ;
       wdt:P569 ?birthDate .
    ?gen rdfs:label ?genLabel .
  }
}
GROUP BY ?s
LIMIT 10
```

```sparql
### Nombre de personnes avec propriétés de base sans doublons (choix aléatoire par MAX)

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

SELECT (COUNT(*) as ?n)
WHERE {
  SELECT ?s (MAX(?label) as ?label) (xsd:integer(MAX(?birthDate)) as ?birthDate)
         (MAX(?gen) as ?gen) (MAX(?genLabel) AS ?genLabel)
  WHERE {
    GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
      ?s wdt:P21 ?gen;
         rdfs:label ?label;
         wdt:P569 ?birthDate.
    }
  }
  GROUP BY ?s
}
```

```sparql
### Ajouter le label pour la propriété "date of birth"

PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

INSERT DATA {
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
    wdt:P569 rdfs:label "date of birth"@en
  }
}


```

```sparql
### Nombre de personnes avec propriétés de base sans doublons (choix aléatoire)

PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

INSERT DATA {
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
    wdt:P21 rdfs:label "sex or gender"@en
  }
}

```
