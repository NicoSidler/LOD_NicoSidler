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

23,231 people as of 16 March 2026

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

Imported: 19027

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
61 entries

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
11 entries

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
0 entries

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


### Find persons without labels

708 persons

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


### Find labels for persons without English label


```
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?item ?item_non_en_label
WHERE {
  SERVICE <https://query.wikidata.org/sparql> {
    
    {
      { ?item wdt:P106 wd:Q2306091 }  # sociologist
      UNION
      { ?item wdt:P101 wd:Q21201 }    # sociology

      ?item wdt:P31 wd:Q5;            # human
            wdt:P569 ?birthDate.      # must have date of birth

      BIND(year(?birthDate) AS ?year)
      FILTER(xsd:integer(?year) >= 1801 && xsd:integer(?year) <= 1990)

      MINUS {
        ?item rdfs:label ?itemLabel.
        FILTER(LANG(?itemLabel) = "en")
      }
    }

    ?item rdfs:label ?item_non_en_label
  }
}
LIMIT 10
```


### Insert non English labels 

The persons already exist in our graph, we only add their labels


```
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX xsd: <http://www.w3.org/2001/XMLSchema#>

INSERT {
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
    ?item rdfs:label ?item_non_en_label.
  }
}
WHERE {
  SERVICE <https://query.wikidata.org/sparql> {
    {
      { ?item wdt:P106 wd:Q2306091 }  # sociologist
      UNION
      { ?item wdt:P101 wd:Q21201 }    # sociology

      ?item wdt:P31 wd:Q5;
            wdt:P569 ?birthDate.

      BIND(year(?birthDate) AS ?year)
      FILTER(xsd:integer(?year) >= 1801 && xsd:integer(?year) <= 1990)

      MINUS {
        ?item rdfs:label ?itemLabel.
        FILTER(LANG(?itemLabel) = "en")
      }
    }

    ?item rdfs:label ?item_non_en_label
  }
}
```

If you execute again the query to find persons without labels, the COUNT result will be zero.

But be aware you'll have a lot of extra names:

```
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT *
# SELECT (COUNT(*) AS ?number)
WHERE {
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
    ?item a wd:Q5.
    ?item rdfs:label ?label.
    FILTER(LANG(?label) != "en")
  }
}
LIMIT 20
```




### Add a label to the class "Person"

```sparql
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

INSERT DATA {
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
    wd:Q5 rdfs:label "Person".
  }
}

```

### Add the "Gender" class

Inspect the genders: the number of different genders is 9

```

PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

SELECT (COUNT(*) AS ?n)
WHERE {
  SELECT DISTINCT ?gender
  WHERE {
    GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
      ?s wdt:P21 ?gender.
    }
  }
}
```



#### Insert the class 'gender' for all types of gender

Please note that strictly speaking Wikidata has no ontology, therefore no classes. We add this for our convenience


``` 

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


#### Add the gender class label
```
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

INSERT DATA {
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
    wd:Q48264 rdfs:label "Gender Identity".
  }
}

```





### Verify the available classes

The result should be Person and Gender.

```
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT ?class ?classLabel
WHERE {
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
    ?s a ?class.
    ?class rdfs:label ?classLabel.
  }
}
```


## Add gender labels


### Find gender labels

```
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

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
    BIND(?gen AS ?gen)
    ?gen rdfs:label ?genLabel
    FILTER(LANG(?genLabel) = "en")
  }
}
```

### Add gender labels

```
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>

INSERT {
  GRAPH <https://NicoSidler.github.io/sociologists/graphs-defs.html#wikidata> {
    ?gen rdfs:label ?genLabel
  }
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
    ?gen rdfs:label ?genLabel
    FILTER(LANG(?genLabel) = "en")
  }
}
```
