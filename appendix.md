## Appendix: SPARQL Query for Collecting Triples from Wikidata

```sparql
SELECT ?player ?playerLabel ?birthDate ?birthPlaceLabel ?nationalityLabel ?memberOf2017Label ?memberOf2020Label ?memberOf2023Label ?memberOf2024Label ?spouseLabel ?homeLocationLabel ?genderLabel ?deathDate ?deathPlaceLabel ?childrenLabel WHERE {
  ?player wdt:P31 wd:Q5;
          wdt:P106 wd:Q937857;
          wdt:P569 ?birthDate.

  OPTIONAL { ?player wdt:P19 ?birthPlace. }
  OPTIONAL { ?player wdt:P27 ?nationality. }

  OPTIONAL { 
    ?player p:P54 ?teamMembership2017. 
    ?teamMembership2017 ps:P54 ?memberOf2017. 
    ?teamMembership2017 pq:P580 ?start_date2017. 
    FILTER(YEAR(?start_date2017) = 2017) 
  }

  OPTIONAL { 
    ?player p:P54 ?teamMembership2020. 
    ?teamMembership2020 ps:P54 ?memberOf2020. 
    ?teamMembership2020 pq:P580 ?start_date2020. 
    FILTER(YEAR(?start_date2020) = 2020) 
  }

  OPTIONAL { 
    ?player p:P54 ?teamMembership2023. 
    ?teamMembership2023 ps:P54 ?memberOf2023. 
    ?teamMembership2023 pq:P580 ?start_date2023. 
    FILTER(YEAR(?start_date2023) = 2023) 
  }

  OPTIONAL { 
    ?player p:P54 ?teamMembership2024. 
    ?teamMembership2024 ps:P54 ?memberOf2024. 
    ?teamMembership2024 pq:P580 ?start_date2024. 
    FILTER(YEAR(?start_date2024) = 2024) 
  }

  OPTIONAL { ?player wdt:P26 ?spouse. }
  OPTIONAL { ?player wdt:P551 ?homeLocation. }
  OPTIONAL { ?player wdt:P21 ?gender. }
  OPTIONAL { ?player wdt:P570 ?deathDate. }
  OPTIONAL { ?player wdt:P20 ?deathPlace. }
  OPTIONAL { ?player wdt:P40 ?children. }

  FILTER(BOUND(?memberOf2017) || BOUND(?memberOf2020) || BOUND(?memberOf2023) || BOUND(?memberOf2024))

  SERVICE wikibase:label { bd:serviceParam wikibase:language "[AUTO_LANGUAGE],en". }
}
LIMIT 1000
```

Adjust the `LIMIT` to manage the return size and the `OFFSET` in subsequent queries to get new batches of data.

You can copy this Markdown content and include it in your appendix file before uploading it to GitHub.
