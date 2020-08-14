This page depicts the available implementations of the OWL to OAS mapping

## Ontology-Based APIs (OBA)
The [Ontology-Based APIs (OBA) framework](https://oba.readthedocs.io/en/latest/) takes as input an ontology or ontology network (specified in OWL) and generates an OpenAPI Specification (OAS). Using this definition, OBA creates a REST API server automatically that can validate the requests from users; deliver JSON objects following the structure described in the ontology; accept custom queries needed by users; and support clients for easing the interaction with the API.

OBA supports most of the mapping proposed in this specification (see [supported mapping](https://oba.readthedocs.io/en/latest/mapping/)).