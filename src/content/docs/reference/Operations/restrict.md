---
title: restrict
---

```ruby
rel.restrict(Predicate.eq(:a, "foo"))
rel.restrict(a: "foo", b: "bar", ...)
rel.restrict(->(t) { t[:a] == "foo" })
```

Alias: `where`

### Problem

Remove tuples from a relation according to a given criterion. 

Example: *Which are all the products with price less than 500?*

### Description

Creates a new relation by selecting some tuples of the original using a predicate, so that the tuples of the output are a subset of the tuples of the original.

Predicates are instances of the [`predicate` gem](https://github.com/enspirit/predicate), which also handles conversion of predicates to SQL. This means predicates can be created and composed in a flexible and composable way. Read more about creating and composing predicates on the [Predicates](/usage/predicates) page.

Apart from using the predicate factory methods `Predicate.eq`, `Predicate.neq` etc, one can pass a hash specifying a predicate using equality. For example, in:

`rel.restrict(a: "foo", b: "bar")`

the argument is equivalent to `Predicate.eq(:a, "foo") & Predicate.eq(:b, "bar")`

It is also possible to define the predicate in terms of a lambda (or proc). The lambda will be passed one tuple at a time, and returns `true`/`false` to determine which tuples are kept and which are discarded. Note that `restrict` using a lambda cannot be compiled to SQL (but can be arbitrarily chained with operations that can). See the documentation on [using a RDBMS backend](/usage/with-rdbms) for more information.

In some contexts it can also be convenient to create a predicate which selects everything (`Predicate.tautology`) or one that selects nothing (`Predicate.contradiction`).

### Requirements

The specified attributes referenced by the predicate must be part of the input relation's heading.

### Examples

*Consult the [Overview page](/reference/overview) for the data model used in these examples.*

#### With regular predicate

```ruby
suppliers.restrict(Predicate.gt(:status, 20) | Predicate.eq(:sid, "S1")).to_a

=>
[{:sid=>"S1", :name=>"Smith", :status=>20, :city=>"London"},
 {:sid=>"S3", :name=>"Blake", :status=>30, :city=>"Paris"},
 {:sid=>"S5", :name=>"Adams", :status=>30, :city=>"Athens"}]
```

##### Generated SQL

```sql
SELECT `t1`.`sid`, `t1`.`name`, `t1`.`status`, `t1`.`city`
FROM `suppliers` AS 't1'
WHERE ((`t1`.`status` > 20) OR (`t1`.`sid` = 'S1'))
```

#### With lambda predicate

```ruby
suppliers.restrict(->(t) { t[:city].length < 6 }).to_a

=>
[{:sid=>"S2", :name=>"Jones", :status=>10, :city=>"Paris"},
 {:sid=>"S3", :name=>"Blake", :status=>30, :city=>"Paris"}]
```

##### Generated SQL

Predicates using lambdas are not compiled to SQL.
