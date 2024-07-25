This appendix contains the SPARQL query used to collect triples from Wikidata:

SELECT ?player ?playerLabel ?birthDate ?birthPlaceLabel ?nationalityLabel 
       ?memberOf2017Label ?memberOf2020Label ?memberOf2023Label ?memberOf2024Label
       ?spouseLabel ?homeLocationLabel ?genderLabel ?deathDate ?deathPlaceLabel ?childrenLabel WHERE {
  ?player wdt:P31 wd:Q5;                      (*@\#@*) Instance of human
          wdt:P106 wd:Q937857;               (*@\#@*) Occupation football player
          wdt:P569 ?birthDate.               (*@\#@*) Birth date

  OPTIONAL { ?player wdt:P19 ?birthPlace. }  (*@\#@*) Birth place
  OPTIONAL { ?player wdt:P27 ?nationality. } (*@\#@*) Nationality
  
  OPTIONAL { 
    ?player p:P54 ?teamMembership2017.
    ?teamMembership2017 ps:P54 ?memberOf2017.
    ?teamMembership2017 pq:P580 ?start_date2017.
    FILTER(YEAR(?start_date2017) = 2017)
  }                                          (*@\#@*) Member of sports team with start date in 2017
  
  OPTIONAL { 
    ?player p:P54 ?teamMembership2020.
    ?teamMembership2020 ps:P54 ?memberOf2020.
    ?teamMembership2020 pq:P580 ?start_date2020.
    FILTER(YEAR(?start_date2020) = 2020)
  }                                          (*@\#@*) Member of sports team with start date in 2020

  OPTIONAL { 
    ?player p:P54 ?teamMembership2023.
    ?teamMembership2023 ps:P54 ?memberOf2023.
    ?teamMembership2023 pq:P580 ?start_date2023.
    FILTER(YEAR(?start_date2023) = 2023)
  }                                          (*@\#@*) Member of sports team with start date in 2023

  OPTIONAL { 
    ?player p:P54 ?teamMembership2024.
    ?teamMembership2024 ps:P54 ?memberOf2024.
    ?teamMembership2024 pq:P580 ?start_date2024.
    FILTER(YEAR(?start_date2024) = 2024)
  }                                          (*@\#@*) Member of sports team with start date in 2024

  OPTIONAL { ?player wdt:P26 ?spouse. }      (*@\#@*) Spouse
  OPTIONAL { ?player wdt:P551 ?homeLocation.}(*@\#@*) Home location
  OPTIONAL { ?player wdt:P21 ?gender. }      (*@\#@*) Gender
  OPTIONAL { ?player wdt:P570 ?deathDate. }  (*@\#@*) Death date
  OPTIONAL { ?player wdt:P20 ?deathPlace. }  (*@\#@*) Death place
  OPTIONAL { ?player wdt:P40 ?children. }    (*@\#@*) Children

  FILTER(BOUND(?memberOf2017) || BOUND(?memberOf2020) || BOUND(?memberOf2023) || BOUND(?memberOf2024))

  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
LIMIT 1000  (*@\#@*) Adjust this to manage return size
OFFSET 0     (*@\#@*) Adjust this in subsequent queries to get new batches of data
