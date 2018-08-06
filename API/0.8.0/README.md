# NCATS Translator Reasoners API
This API specification represents a version 0.8.0 draft of the NCATS Translator Reasoners API.
It is intended that the various reasoner tools will support this API so that remote calls to
any of the reasoners may be made using the same API with the same result format, which will
facilitate comparison among reasoners and chaining of queries to different reasoners to
achieve an aggregated result.

## Previous versions
- Previous versions of the draft standard may be found at https://github.com/NCATS-Tangerine/NCATS-ReasonerStdAPI/tree/master/API
- Early Google-doc based discussion may be found at https://drive.google.com/drive/folders/1kTIW6W7sLdSAhH9qBBeSAEZ4ouAViI4x
- Emerging KG Standard: https://docs.google.com/document/d/1TrvqJPe_HwmRJ5HCwZ7fsi9_mwGcwLOZ53Fnjo8Sh4E/edit#

## Notes
- The target output is intended to be JSON-LD

# Specification
Below is a description of the elements of the JSON format.

## Top level (Response)
- **context** - URI - URL of the JSON-LD context file of this document. An actual context file remains yet been developed.
- **type** - string - Type definition of this response object. Should always be "medical_translator_query_result"
- **id** - URI - URI of this Reasoning Tools response if it is persisted somewhere.
- **reasoner_id** - string - String identifier of the reasoner providing the response ("RTX", "Robokop", "Indigo", "Integrator", etc.)
- **tool_version** - string - The version string of the reasoning tool that provided this response.
- **schema_version** - string - The API standard will likely evolve over time. This encodes the schema version used in this response.
- **datetime** - datetime - The datetime stamp when this response was provided to a user.
- **original_question_text** - string - The exact string that the original user provided to the reasoning tool
- **restated_question_text** - string - A restatement of the question that the reasoning tool understood and is answering with this response. This may not match the intent of the original_question_text.
- **query_type_id** - string - The query type id if one is known for the query/response (as defined in https://docs.google.com/spreadsheets/d/1Gna_yCbHj14Brp-8GBY50Mq36nwKGl5T5z4REUQQsfw/edit)
- **term** - object - A dict/hash/object of the terms needed to convert the referenced **query_type_id** into a specific instance of a question. For example: "{ 'disease': 'malaria' }"
- **n_results** - int - Total number of results in the response (which may be less than what is returned if limits were placed on the results to return)
- **response_code** - string - A terse code indicating success or error message for the query overall. OK is normal completion. Available error codes are not yet defined. These probably should be mapped to HTTP error codes in the YAML or entirely replaced by YAML-defined error codes.
- **message** - string - A detailed message from the Reasoning Tool to the user about degree of success of answering the query. If there are no results returned, then this message should detail why there are no results. If there are results returned, the Reasoning Tool may still provide some commentary to the user about how act of addressing the query result went. This is NOT intended to describe and answer/result, but rather just for the Reasoning Tool to provide information external to any specific result to the user.
- **table_column_names** - array - List of column names that corresponds to the row_data for each result. Example: '[ "chemical_substance.name", "chemical_substance.id" ]'
- **result_list** - array - A response from a tool may contain multiple results, where a result is an independent potential answer to the query.
- **query_template** - object - Some reasoners may not work from an English text question, but may begin with a series of notes or node types. This section is intended to encode such a beginning. It is still not completely specified. There is some initial work on this in the QuerySpecification doc in the NCATS Hackathon 44/51 folder. To be fleshed out later with appropriate input from groups who want this functionality.
- **known_query_type_id** - string - DEPRECATED. Older, deprecated name for what is now **query_type_id**
- **result_code** - string - DEPRECATED. Older, deprecated name for what is now properly named **response_code**


## result (each object within result_list)
- **id** - URI - URI of this specific result if it is persisted somewhere.
- **description** - string - A free text field describing this result (answer to the query).
- **essence** - string - A single string that is the terse essence of the result (useful for simple answers)
- **row_data** - array - An arbitrary list of values that captures the essence of the result that can be turned into a tabular result across all answers (each result is a row) for a user that wants tabular output
- **score** - number - Any type of score associated with this result (highest confidence)
- **score_name** - string - Name for the score (e.g. "Jaccard distance")
- **score_direction** - string - Sorting indicator for the score: one of higher_is_better or lower_is_better
- **confidence** - float - A numerical confidence score for this result, where 1.0 denotes the highest confidence and 0.0 denote no confidence.
- **result_type** - string - One of several possible result types: 'individual query answer', 'neighborhood graph', 'type summary graph'
- **result_group** - integer - An integer group number for results for use in cases where several results should be grouped together. Also useful to control sorting ascending. The intended use for this is when multiple reasoner outputs are merged with similar answers grouped together.
- **result_group_similarity_score** - number - A score that denotes the similarity of this result to other members of the result_group
- **reasoner_id** - string - String identifier of the reasoner providing the response ("RTX", "Robokop", "Indigo", "Integrator", etc.)
- **result_graph** - object - A serialization of the thought pattern or graph path for this result (answer to the query).
- **text** - string - DEPRECATED. Older, deprecated name for what is now better named **description**

##result_graph (a container for nodes and edges)
- **node_list** - array - An array container for multiple node objects in arbitrary order
- **edge_list** - array - an array container for multiple edge objects in arbitrary order

## node (each object within node_list)
- **id** - string - CURIE corresponding to the bioentity
- **uri** - URI - Full URI corresponding to the bioentity
- **name** - string - bioentity name of the node
- **type** - string - bioentity type of the node, as defined by the KG standard
- **description** - Full 1+ sentence description/definition of the bioentity
- **symbol** - string - Equivalent symbol for this bioentity. This is most common with the protein or gene bioentity types, but other types may also have symbols or abbreviations
- **node_attributes** - array - container for a series of node_property objects

## node_attribute (each object within node_attributes)
- **type** - string - controlled type of the property
- **name** - string - name of the node property
- **value** - any - value associate with the name and type
- **url** - URI - URL associated with this node property

## edge (each object within edge_list)
- **type** - string - controlled edge type / predicate from the KG standard minimum list
- **relation** - string - controlled edge type / predicate from the KG standard maximal list
- **source** - string - id of the source node
- **target** - string - id of the target node
- **is_defined_by** - string - A CURIE/URI for the translator group that made the KG
- **provided_by** - string - A CURIE/URI for the knowledge source that defined this edge
- **confidence** - float - Confidence metric for this relationship/assertion/edge. 1.0 indicates the highest confidence. 0.0 indicates no confidence. The confidence may come directly from a knowledge source, or may come from come from the KG builder or even Reasoning Tool based on other contextual information. (NOTE: This is not in the KG standard, but is being proposed as an addition there)
- **publications** - string - A CURIE/URI for publications associated with this edge
- **evidence_type** - string - A CURIE/URI for class of evidence supporting the statement made in an edge - typically a class from the ECO ontology (e.g. ECO:0000220)
- **qualifiers** - string - Terms representing qualifiers that modify or qualify the meaning of the statement made in an edge
- **negated** - boolean - Boolean that if set to true, indicates the edge statement is negated i.e. is not true
- **attribute_list** - array - A list of additional attributes for this edge

## edge_attribute (each object within an edge attribute_list)
- **type** - string - controlled type of the property
- **name** - string - name of the edge property
- **value** - any - value associate with the name and type
- **url** - URI - URL associated with this edge property
