---
status: experimental
author: joseph gardner
version: draft-2
---

# **Tripl** – A Simple Domain-Specific Language (DSL) for Mapping Ontologies Using RDF Triples

This RFC defines **Tripl**, a Domain-Specific Language (DSL) for expressing RDF triples with a simplified syntax for ontology mapping. The language prioritizes human-readable syntax, short URIs, and automatic resolution of predicates via a well-known path for mappings. **Tripl** is designed to work with RDF-based data and provides a mechanism for linking short names to full URIs through a centralized mapping system. This approach improves flexibility by allowing custom property mappings to be stored in external files.

- [1. Introduction](#1-introduction)
- [2. Syntax Overview](#2-syntax-overview)
  - [2.1 Base URI](#21-base-uri)
  - [2.3 Triples](#23-triples)
  - [2.4 Literals](#24-literals)
- [3. Mapping Mechanism](#3-mapping-mechanism)
  - [3.1 Well-Known Path](#31-well-known-path)
  - [3.2 tripl.json Configuration File](#32-tripljson-configuration-file)
  - [3.3 Mapping File Format](#33-mapping-file-format)
- [4. Example Usage](#4-example-usage)
  - [Example 1: Basic Document](#example-1-basic-document)
  - [Example 2: tripl.json Configuration](#example-2-tripljson-configuration)
  - [Example 3: Mapping File](#example-3-mapping-file)
- [5. Error Handling and Validation](#5-error-handling-and-validation)
- [6. Security Considerations](#6-security-considerations)
- [7. IANA Considerations](#7-iana-considerations)
- [8. References](#8-references)
  - [Appendix A: Example System Workflow](#appendix-a-example-system-workflow)
  - [Appendix B: Visualize a Tripl document with Graphviz](#appendix-b-visualize-a-tripl-document-with-graphviz)

## 1. Introduction

This document specifies **Tripl**, a DSL for expressing RDF triples with an emphasis on simplicity and readability. In RDF, data is represented as triples consisting of a **subject**, **predicate**, and **object**, which together define relationships between resources. Tripl aims to make defining these triples easier, without the need to focus on full URI paths. The language allows short names for predicates to be mapped to their full URIs, keeping it concise and accessible while ensuring flexibility and extensibility. This approach makes working with RDF data more intuitive and user-friendly.

**Tripl** is ideal for use cases such as knowledge representation and linked data where a human-readable abstraction of RDF data is desirable. 

---

## 2. Syntax Overview

### 2.1 Base URI

The `@base` directive is used to specify the base URI for the document. All identifiers MUST be resolved relative to this base URI unless explicitly overridden by a full URI.

Example:
```plaintext
@base http://example.com/
```
This implies that any identifier like `book123` would be resolved as `http://example.com/book123`.

Mappings link short names (e.g., `title`, `creator`) to full URIs, allowing human-readable predicates in RDF triples. They are retrieved from a `.well-known` path relative to the base URI:

```
<base URI>/.well-known/tripl.json
```

### 2.3 Triples

Triples in **Tripl** are expressed in the form:
```plaintext
subject predicate object
```
- **Subject** and **object** MUST be either URIs or literals (enclosed in quotes).
- **Predicate** SHOULD be a short name that will be resolved based on the mappings defined in the `propertyMapping` file.

Example Triple:
```plaintext
book123 title "The Great Gatsby"
```
Here, `book123` will resolve to `http://example.com/book123`, and `title` will be resolved using the mappings in the `mapping.json` file.

### 2.4 Literals

Literals are enclosed in double quotes and represent data values. These are distinct from URIs.

Example:
```plaintext
book123 date "1925-04-10"
```
Here, `"1925-04-10"` is a literal value.

---

## 3. Mapping Mechanism

### 3.1 Well-Known Path

The system MUST look for a configuration file at the `.well-known` path relative to the `@base` URI. The configuration file MUST be named `tripl.json`.

The `tripl.json` file MUST contain a `propertyMapping` property, which points to an external file where predicate mappings are defined. This file SHOULD be a JSON file.

Example configuration:
```json
{
  "propertyMapping": "http://example.com/.well-known/mapping.json"
}
```

### 3.2 tripl.json Configuration File

The `tripl.json` configuration file is the primary entry point for the system. It contains directives such as `propertyMapping`, which refers to the location of the predicate mappings.

- The **`propertyMapping`** key MUST contain a URI that points to the location of the external mapping file.

### 3.3 Mapping File Format

The external mapping file referred to by `propertyMapping` SHOULD be a **JSON** file containing a list of key-value pairs, where the key is the short name of the predicate and the value is the full URI.

Example `mapping.json`:
```json
{
  "title": "http://purl.org/dc/elements/1.1/title",
  "creator": "http://purl.org/dc/elements/1.1/creator",
  "name": "http://purl.org/dc/elements/1.1/name",
  "date": "http://purl.org/dc/elements/1.1/date",
  "identifier": "http://purl.org/dc/elements/1.1/identifier"
}
```

The mapping file is OPTIONAL, but if it exists, the system MUST automatically apply these mappings during processing.

---

## 4. Example Usage

### Example 1: Basic Document

```plaintext
@base http://example.com/

# The system will automatically load mappings from .well-known/tripl.json

book123 title "The Great Gatsby"
book123 creator author1
book123 date "1925-04-10"
book123 identifier "9780743273565"

book124 title "To Kill a Mockingbird"
book124 creator author2
book124 date "1960-07-11"
book124 identifier "9780061120084"

author1 name "F. Scott Fitzgerald"
author2 name "Harper Lee"
```

### Example 2: tripl.json Configuration

```json
{
  "propertyMapping": "http://example.com/mapping.json"
}
```

### Example 3: Mapping File

```json
{
  "title": "http://purl.org/dc/elements/1.1/title",
  "creator": "http://purl.org/dc/elements/1.1/creator",
  "name": "http://purl.org/dc/elements/1.1/name",
  "date": "http://purl.org/dc/elements/1.1/date",
  "identifier": "http://purl.org/dc/elements/1.1/identifier"
}
```

In this example, the `tripl.json` configuration points to the external `mapping.json` file, which contains the mappings for predicates. The triples will resolve the short names (e.g., `title`, `creator`, `name`) to the corresponding full URIs.

---

## 5. Error Handling and Validation

The system MUST handle errors gracefully:
- If the `@base` URI is not specified, an error MUST be raised.
- If the `tripl.json` configuration file is missing or malformed, the system SHOULD issue a warning and fall back to default configurations or throw an error depending on the configuration.
- If the `propertyMapping` URI points to a non-existent or malformed file, the system SHOULD raise an error indicating the problem.

---

## 6. Security Considerations

- **Data Integrity**: The `tripl.json` and `mapping.json` files SHOULD be served from trusted sources (e.g., HTTPS) to ensure the integrity of the mappings.
- **URI Validation**: Full URIs MUST be validated to ensure they conform to proper URI structure to prevent injection attacks.

---

## 7. IANA Considerations

This RFC does not require any IANA (Internet Assigned Numbers Authority) actions. The `.well-known` path is already a recognized convention for various metadata files on the web.

---

## 8. References

- **RDF 1.1 Concepts and Abstract Syntax** – [W3C Recommendation](https://www.w3.org/TR/rdf11-concepts/)
- **RDF Vocabulary Description Language (RDF Schema)** – [W3C Recommendation](https://www.w3.org/TR/rdf-schema/)
- **Uniform Resource Identifier (URI): Generic Syntax** – [RFC 3986](https://tools.ietf.org/html/rfc3986)

---

### Appendix A: Example System Workflow

1. The document is processed starting with the `@base` URI.
2. The system checks for the existence of the `.well-known/tripl.json` configuration file.
3. The system loads the `propertyMapping` URI from the `tripl.json` file and retrieves the predicate mappings.

### Appendix B: [Visualize a Tripl document with Graphviz](./graphviz.md)

