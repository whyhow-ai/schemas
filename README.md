# WhyHow Knowledge Graph Schemas

Welcome to the repository for WhyHow Knowledge Graph Schemas. This repository contains a collection of predefined schemas that can be used with the WhyHow [automated knowledge graph creation SDK](https://github.com/whyhow-ai/whyhow) to facilitate the rapid development and deployment of knowledge graphs tailored to specific domains or applications.

WhyHow builds developer tools that simplify and streamline the process of building, managing, and integrating graphs into your RAG pipelines. We believe that graph-based RAG systems are most performant when using graphs that are small and narrowly-scoped to your unique view of the world. By including only the data that you need for your use case, you reduce the likelihood of poisoning your context window with potentially irrelevant information and causing hallucinations. Smaller graphs are also easier to maintain and modify, giving developers much more control to build accurate, reliable RAG systems.

Some of these schemas were created by the WhyHow team, with certain aspects of the schemas removed to anonymize the use-case. Others were created by our design partners that were open to sharing their schemas. Some include the underlying data used to generate the schema, while others do not due to confidentiality reasons.

## How does it work?

These schemas work best with the WhyHow.AI SDK given the very specific multi-agentic approach that is on the backend to use natural language schemas with descriptions as the basis for graph creation.

We aren't just throwing a schema at an LLM and telling it to build us a graph. While that approach is a relevant (and admittedly fun) way to explore your data through LLM-reasoned patterns, it lacks the precise control and domain specificity needed to reliably perform entity and relationship extraction and build graphs in a repeatable way. Entity resolution and coreference resolution also remain difficult, especially when building with a corpus of larger documents.

Our schema-based extraction solution take a multi agent approach and employs task-specific language models throughout multiple stages (such as entity definition and extraction, relevancy checks, relationship detection, pattern alignment, etc.) to build more reliable and comprehensive extraction. Our modular approach enables users to inject meaningful context and swap in domain-specific models at the right stage of extraaction to enable more customized, reliable extraction that maps to your unique use case.

## Schemas

Schemas are how how WhyHow enables developers to express and inject their unique view of the world. The automated knowledge graph creation SDK leverages schemas to enable customized entity extraction and graph creation. Schemas are made up of three parts:

### 1. Entities

Entities are the elements represented as nodes in the graph. For example, in a knowledge graph about movies, entities could be individual movies, actors, directors, genres, etc.

### 2. Relations

Relations are the graph edges, they denote the connections or associations between entities in the graph. For example, relations in a movie graph could indicate that an actor "acted in" a movie, or that a director "directed" a movie.

### 3. Patterns

Patterns define recurring arrangements or sequences of entities and relations that are allowed to occur in the graph. They constrain the types of triples that are created.

For each type, you must provide specific metadata, such as a name, description, or a reference to an entity or a relation. In the example below, we built a schema for how the characters and objects interact with each other in Harry Potter and the Philosopher's Stone. In this example, we have two entities “character” and “magical_object”, two relations “friends_with” and “interacts_with”, and two patterns, one which says that a character can be friends with another character, and another which says that a character can interact with a magical objects. This simple schema demonstrates how straightforward it is to inject a high level of contextual control into your graph creation workflow.

```json
{
  "entities": [
    {
      "name": "character",
      "description": "A wizard mentioned in the text, such as Harry, Ron, or Hermione."
    },
    {
      "name": "magical_object",
      "description": "Inanimate items that characters use or interact with, such as invisibility cloak, or snitch."
    }
  ],
  "relations": [
    {
      "name": "friends_with",
      "description": "Denotes a friendly relationship between characters."
    },
    {
      "name": "interacts_with",
      "description": "Describes a scenario in which a character engages with a magical object."
    }
  ],
  "patterns": [
    {
      "head": "character",
      "relation": "friends_with",
      "tail": "character",
      "description": "A character is friends with another."
    },
    {
      "head": "character",
      "relation": "interacts_with",
      "tail": "magical_object",
      "description": "A character interacts with a magical object."
    }
  ]
}
```

## Example

Let's say you're a pharmaceutical company building a RAG solution for a chatbot that allows users to more easily and naturally explore the treatments, side effects, and other critical information about medications you provide provide. You have a large database of drug fact documents in PDF format, and they you to use them to construct a medication knowledge graph to answer questions such as:

- Which drugs cause drowsiness?
- Which drugs contain a certain active ingredient?
- What does a particular medication treat?

Using the schema-based graph approach provided by the WhyHow automated knowledge graph creation solution, we can programatically build a graph using raw PDFs, and a simple JSON schema:

### Step 1: Import and initialize WhyHow client

```
from whyhow import WhyHow

client = WhyHow()
```

### Step 2: Create dedicated namespace and drug fact PDFs

```
# Define namespace name
namespace = "drug-facts"

# Specify relative path of the documents you’d like to upload
documents = [
    "assets/advil_drug_facts.pdf",
    "assets/tylenol_drug_facts.pdf"
]

# Add documents to your namespace
documents_response = client.graph.add_documents(
  namespace = namespace, documents = documents
)
```

### Step 3: Define the schema

```json
{
  "entities": [
    {
      "name": "medication",
      "description": "The brand name for a medication. Examples are Advil, Benadryl, Tylenol, etc."
    },
    {
      "name": "side_effect",
      "description": "Harmful bodily effects that can be caused by using the medication. Examples of side effects are nausea, swelling, asthma, shock, blisters, rash, liver damage"
    },
    {
      "name": "active_ingredient",
      "description": "Chemical compounds and ingredients used to make a medication. Examples of active ingredients are Acetaminophen, Ibuprofen, NSAID, Diphenhydramine HCl"
    },
    {
      "name": "symptom",
      "description": "An ailment or issue that is treated or relieved by using the medication. Examples of symptoms are headache, soreness, pain, toothache, backache, arthritis"
    },
    {
      "name": "instruction",
      "description": "A direction for how you should consume a medication."
    }
  ],
  "relations": [
    {
      "name": "causes",
      "description": "Indicates that the medication has an undesirable bodily side effect."
    },
    {
      "name": "contains",
      "description": "Medications contain an active ingredient."
    },
    {
      "name": "treats",
      "description": "Medications are used to treat bodily ailments or sicknesses."
    },
    {
      "name": "has_instruction",
      "description": "Medications have an instruction for how to use it."
    }
  ],
  "patterns": [
    {
      "head": "active_ingredient",
      "relation": "causes",
      "tail": "side_effect",
      "description": "A medication causes a side effect."
    },
    {
      "head": "medication",
      "relation": "contains",
      "tail": "active_ingredient",
      "description": "A medication contains an active ingredient."
    },
    {
      "head": "active_ingredient",
      "relation": "treats",
      "tail": "symptom",
      "description": "A medication treats a symptom."
    },
    {
      "head": "medication",
      "relation": "has_instruction",
      "tail": "instruction",
      "description": "A medication has an instruction."
    }
  ]
}
```

### Step 4: Create the graph using schema

```
# Create graph from schema

schema = "./schema.json"

extracted_graph = client.graph.create_graph_from_schema(
  namespace = namespace, schema_file = schema_file
)
```

## Schema Development Process

The WhyHow schema library helps you get started faster with schema-defined graph creation feature.

This is the typical engagement process we have with our design partners:

- They read our [Medium articles](https://medium.com/enterprise-rag), case studies, and schemas and they come to us with an understanding of the specific use-case and data they want to build graphs for.
- They use our question-based graph creation feature to generate a list of potential schemas that would be a fit for their use case.
- They use an LLM to generate the schema based on a few high-level example schemas, raw text samples, and project objectives.
- They then edit the auto-generated schema, and use it to automatically create a graph using the WhyHow SDK.

Your process may look a little different depending on your use-case, data, and how specific you'd like your graph to be. At the end of the day, granular tooling for graph experimentation, iteration, and management is needed for incorporating more deterministic graph structures, and it will be exciting to see more systems incorporate knowledge graphs.
