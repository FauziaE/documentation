---
description: >-
  This page introduces you to the meaning of the symbols that constitute
  relation algebra. The time you spend getting acquainted with this will pay off
  in designing correct software...
---

# Terms

## Purpose

The purpose of an term is to compute pairs that constitute a relation. We use relation operators to assemble terms from smaller ones, to let the result correspond closely to the natural language of the business.

## Description

An term is a combination of operators and relations. Its meaning is a set of pairs.

## Examples

`owner`

`r;s~`

`I /\ goalkeeper;goalkeeper~`

`destination;"Algarve" |- spoken;"Portugese"`

## Syntax

Every term is built out of relations, which are combined by operators. An term has one of the following 8 syntactic structures

```text
<Term> <BinaryOperator> <Term>
<UnaryOpPre> <Term>
<Term> <UnaryOpPost>
<RelationRef> <signature>?
I <signature>?
V <signature>?
<atom>
( <Term> )
```

## `Operators`

The operators are categorized. We advise novices to study only the categories rules, boolean and relational, because there is a wealth of things you can express in only these categories.

| Category | set |
| :--- | :--- |
| rules | $$=$$ and $$\subseteq\$$ |
| boolean | $$\cup$$, $$\cap$$, and $$-$$ |
| relational | $$;$$ and $$\smallsmile\$$ |
| product | $$\times$$ and $$\dagger$$ |
| residual | $$\backslash$$, $$/$$, and $$♢$$ |
| Kleene | $$∗$$ and $$+$$ |



When coding in Ampersand, these operators are typed with characters on the keyboard. The following correspondence exists between the operators in code and in math:

| operator | code | math | position | category |
| :--- | :---: | :---: | :--- | :--- |
| equivalence \(equal\) | `=` | $$=$$ | Binary | rule forming |
| inclusion | `|-` | $$\subseteq$$ | Binary | rule forming |
| intersect | `/\` | $$∩$$ | Binary | boolean |
| union | `\/` | $$∪$$ | Binary | boolean |
| difference \(minus\) | `-` | $$-$$ | Binary | boolean |
| complement | `-` | $$-$$ | Overline | boolean |
| compose | `;` | $$;$$ | Binary | relational |
| converse \(flip\) | `~` | $$\smallsmile$$ | Postfix | relational |
| left residual | `/` | $$/$$ | Binary | residual |
| right residual | `\` | $$\backslash$$ | Binary | residual |
| diamond | `<>` | $$♢$$ | Binary | residual |
| relational product | `!` | $$†$$ | Binary | product |
| cartesian product | `#` | $$\times$$ | Binary | product |
| reflexive transitive closure | `*` | $$∗$$ | Postfix | Kleene \(not yet implemented\) |
| transitive closure | `+` | $$+$$ | Postfix | Kleene \(not yet implemented\) |

## Semantics

The semantics are discussed in terms of sets. The following table shows the way. :
