# Mineralogy Knowledge Graph

An OWL ontology modeling minerals, their physical/chemical properties, and the deposits and localities where they occur — built in [Protégé](https://protege.stanford.edu/), validated with the HermiT reasoner, and queried with SPARQL in GraphDB.

## What it models

Minerals are classified first by chemical composition: silicates (split further into tectosilicates, nesosilicates, and phyllosilicates), oxides, sulfides, sulfates, carbonates, halides, phosphates, and native elements. Physical description follows a separate logic. Hardness and density are numeric, while color, luster, cleavage, and crystal system are each tied to a fixed set of categories instead of free text, so a mineral can't end up described inconsistently across instances.

Composition links a mineral to its constituent chemical elements. Geography chains further: a mineral occurs in a deposit, a deposit sits at a locality (a mine, a quarry, an outcrop), and localities are placed inside regions, which are placed inside countries. That chain is transitive, so a query can find everything within a given country without every locality pointing straight at it. Deposits also carry their own geological context: how they formed (hydrothermal, magmatic, metamorphic, sedimentary) and what type of rock hosts them.

## Ontology stats

| Classes | Object properties | Data properties | Individuals |
|---|---|---|---|
| 33 | 17 | 14 | 65 (incl. 15 mineral species) |

## Reasoning features

A handful of object properties carry formal characteristics that the reasoner uses during inference, not just for organizing data.

| Property | Characteristic | Effect |
|---|---|---|
| `hasCrystalSystem`, `hasTypeLocality` | Functional | A mineral has exactly one value; asserting a second is a reasoner-detected error. |
| `coOccursWith`, `polymorphOf` | Symmetric | Stating quartz `coOccursWith` pyrite also gives pyrite `coOccursWith` quartz, no second triple needed. |
| `locatedIn`, `varietyOf` | Transitive | A locality's membership in a region propagates up to the country, without a direct locality-to-country triple. |
| `hasMineral` / `occursInDeposit` | Inverse pair | Deposit-to-mineral and mineral-to-deposit views stay consistent automatically. |
| `hasOreMineral`, `hasGangueMineral` | Sub-properties of `hasMineral` | Ore and gangue minerals are distinguished but both still retrievable through the general property. |

## Validation

Consistency was checked in Protégé with the HermiT reasoner, and it didn't pass on the first attempt. An earlier version declared `Mineral` as a disjoint union that included several sibling top-level classes, which collapsed the whole ontology (`owl:Thing` became a subclass of `owl:Nothing`). Protégé's explanation feature traced the failure back to that one axiom. Once removed, the reasoner ran clean, with no unsatisfiable classes.

## SPARQL queries

Seven queries were run against the populated ontology in GraphDB, across three levels of complexity: basic triple-pattern lookups (mineral name/formula/hardness, minerals containing iron), intermediate queries using `OPTIONAL`, `FILTER`, `ORDER BY`, and `LIMIT` (discovery year with missing values handled, top-5 hardest minerals), and advanced queries using `GROUP BY`/aggregates and a property path over the transitive `locatedIn` property.

That last query is the one worth looking at, since it recovers a fact that was never stated directly:

```sparql
PREFIX : <http://www.semanticweb.org/reotu/ontologies/2026/6/mineralv1/>

SELECT ?area WHERE {
  ?area :locatedIn+ :Brazil .
}
```

A mine and a locality both show up here, not because either is linked to Brazil directly, but because each is linked to a region that is.

## Files

| File | What it holds |
|------|---------------|
| `mineralv6.ttl` | The ontology in Turtle format: classes, properties, individuals |
| `mineraliv1seminar.pdf` | Full seminar report (Croatian): ontology design, validation walkthrough, all seven SPARQL queries with results, coverage/accuracy discussion |

## Running it

```bash
# Browse the class hierarchy and run the HermiT reasoner
# Protégé -> File -> Open -> mineralv6.ttl

# Or import into a triple store and run the SPARQL queries
# GraphDB -> create repository -> import mineralv6.ttl
```

## References

- Model built in [Protégé](https://protege.stanford.edu/); queries tested in [GraphDB](https://www.ontotext.com/products/graphdb/).
- An LLM (Claude Sonnet 4.6) was used during development as a sounding board for mineralogy terminology and as a starting point for the SPARQL queries, and for light grammar passes on the report. Classes, properties, individuals, validation, and final query logic were built and checked by hand.
