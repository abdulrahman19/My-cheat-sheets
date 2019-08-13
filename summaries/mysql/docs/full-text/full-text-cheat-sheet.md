# MySQL Full-Text Search Cheat Sheet

* [Introduction](#introduction)
* [Defining FULLTEXT Index](#defining-fulltext-index)
* [Removing FULLTEXT Index](#removing-fulltext-index)
* [Natural Language FTS](#natural-language-fts)
* [Boolean FTS](#boolean-fts)
* [FTS Query Expansion](#fts-query-expansion)
* [Ngram FT Parser (not important)](#ngram-ft-parser)


MySQL Full-Text search provides a simple way to implement various advanced search techniques such as `natural language search`, `Boolean text search` and `query expansion`.

### Introduction

**> Why you need to use Full-Text index?**

When you search by using the `LIKE` operator and regular expression you will face some limitations like:
* **Bad Performance**: MySQL has to scan the whole table to find the search result.
* **Not a flexible search**: it is difficult to have a complex search queries.
* **Can't specify relevance ranking**.

The following are some important features of MySQL full-text search:
* **Fully dynamic index**: MySQL automatically updates the index.
* **Moderate index size**: it doesn’t take much memory to store the index.
* **Flexible search**.
* **Can specify relevance ranking**.

**> How Full-Text search index works?**

Technically, MySQL creates an index from the words of the enabled full-text search columns and performs searches on this index. MySQL uses a sophisticated algorithm to determine the rows matched against the search query.

**> There are some important points you should remember when using the full-text search:**
* The minimum length of the search term defined in MySQL full-text search engine is 4.
* Stop words are ignored. like `a`, `an` and `about`. Note that you must define your own stopword list if the language is other than English.

### Defining FULLTEXT Index
```sql
CREATE TABLE table_name(
    column1 data_type,
    column2 data_type,
    …
    PRIMARY_KEY(key_column),
    FULLTEXT KEY index_name (column1,column2,..)
);
```

**Defining FULLTEXT Index For Existing Tables**

Using `ALTER TABLE` statement:
```sql
ALTER TABLE table_name
ADD FULLTEXT(column_name1, column_name2,…)
```
Using `CREATE INDEX` statement:
```sql
CREATE FULLTEXT INDEX index_name
ON table_name(idx_column_name,...)
```

### Removing FULLTEXT Index
```sql
ALTER TABLE table_name
DROP INDEX index_name;
```

### Natural Language FTS
In natural language full-text searches, MySQL looks for rows or documents that are relevant to the free-text natural human language query, for example, “How to use MySQL natural language full-text searches”.

MySQL computes the relevance based on various factors including:
* The number of words in the document.
* The number of unique words in the document.
* The total number of words in the collection.
* The number of documents (rows) that contain a particular word.

```sql
SELECT productName, productline
FROM products
WHERE MATCH(productName) AGAINST('1932 Ford' IN NATURAL LANGUAGE MODE);
```
The `AGAINST()` function uses `IN NATURAL LANGUAGE MODE` search modifier by default therefore you can omit it in the query.
```sql
SELECT productName, productline
FROM products
WHERE MATCH(productline) AGAINST('1932 Ford');
```

### Boolean FTS
In the Boolean mode, MySQL searches for words instead of the concept like in the natural language search.

To perform a full-text search in the Boolean mode, you use the `IN BOOLEAN MODE` modifier in the `AGAINST` expression.
```sql
SELECT productName, productline
FROM products
WHERE MATCH(productName)
      AGAINST('Truck' IN BOOLEAN MODE )
```

**Boolean FTS have these characteristics:**
* MySQL does not automatically sort rows in the order of decreasing relevance.
* InnoDB full-text search does not support trailing plus (`+`) or minus (`-`) sign.
* The 50% threshold is not applied. By the way, 50% threshold means if a word appears in more than 50% of the rows, MySQL will ignore it in the search result.

Feature | InnoDB | MyISAM | Note
---|---|---|---|
Require Full-Text Index For All Columns | Yes | No | `MyISAM`: Search executing will be quite slow.
Support Multiple Boolean Operators | No | Yes | `MyISAM`: For example, `+-mysql` will become `-mysql`.

**Boolean FTS Operators**

<br>Operator <img width=35/>| Description
---|---|
no operator | By default (when neither `+` nor `-` is specified), the word is optional, but the rows that contain it are rated higher.
`+` | Include, the word must be present.
`–` | Exclude, the word must not be present.
`>` | Include, and increase ranking value.
`<` | Include, and decrease the ranking value.
`()` | Group words into subexpressions (allowing them to be included, excluded, ranked, and so forth as a group).
`~` | Negate a word’s ranking value.
`*` | Wildcard at the end of the word.
`"` | A phrase that is enclosed within double quote (`"`) characters matches only rows that contain the phrase literally, as it was typed.

### FTS Query Expansion
The query expansion is used to widen the search result of the full-text searches based on `Automatic Relevance Feedback` (or `blind query expansion`).

Technically, MySQL full-text search engine performs the following steps when the query expansion is used:
* First, MySQL full-text search engine looks for all rows that match the search query.
* Second, it checks all rows in the search result and finds the relevant words.
* Third, it performs a search again based on the relevant words instead of the original keywords provided by the users.

The following illustrates the syntax of the query using the `WITH QUERY EXPANSION` search modifier.
```sql
SELECT column1, column2
FROM table1
WHERE MATCH(column1,column2)
      AGAINST('keyword',WITH QUERY EXPANSION);
```

### Ngram FT Parser
MySQL use ngram full-text parser to support full-text searches for ideographic languages such as Chinese, Japanese, Korean, etc.

MySQL included ngram full-text parser as a built-in server plugin. The main function of ngram full-text parser is tokenizing a sequence of text into a contiguous sequence of `n` characters.

The Ngram parser excludes tokens that contain the stopword in the stopword list. Note that you must define your own stopword list if the language is other than English. In addition, the stopwords with lengths that are greater than `ngram_token_size` are ignored.

The token size (`n`) in the Ngram by default is `2`. To change the token size, you use the `ngram_token_size` configuration option, which has a value between `1` and `10`.

Note that a smaller token size makes smaller full-text search index and allows you to search faster.

**Create FT with Ngram Parser**

You add the `WITH PARSER ngram` in the `CREATE TABLE`, `ALTER TABLE`, or `CREATE INDEX` statements.
```sql
CREATE TABLE posts (
    id INT PRIMARY KEY AUTO_INCREMENT,
    title VARCHAR(255),
    body TEXT,
    FULLTEXT ( title , body ) WITH PARSER NGRAM
)  ENGINE=INNODB CHARACTER SET UTF8MB4;
```
