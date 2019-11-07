---
name: Search Extension
---

### Implementing Multi-Table Full Text Search
Easily searching across an application’s data is a pervasive need. If you are lucky, you can get away with simple sorting or searching on a single column, but it is more likely that you need full text search across multiple models, all from a single search field

Search has been a problem for content related systems. But nowadays, we have many solutions for this.
As Sociozat, for search we support full text search for `postgres`, `mysql` and `sqlite`. Also, `elasticsearch` and `solr` adapters will be released in future.


## SQL Changes

We need different where statements for different kind of databases, and also we will look for different types of documents (eg `topic`, `user`, `channel`).
To do that, we need to construct a database view which presents a polymorphic relationship to the individual result and the text column being searched.

### Postgres

Create a [view table](https://www.postgresql.org/docs/9.4/sql-createview.html)

```sql
CREATE VIEW searches AS
  SELECT
    'topic' AS type,
    topics.name as title,
    topics.slug
  FROM topics

  UNION

  SELECT
    'user' as type,
    users.username as title,
    users.slug
  FROM users

  UNION

  SELECT
    'channel' as type,
    channels.name as title,
    channels.slug
  FROM channels
;
```

From here, we add [gin](https://www.postgresql.org/docs/9.4/textsearch-indexes.html) indices to the columns on which we are searching. In our case similar indices to these made the difference between a 3-5 second lookup and ~100ms.

```sql
CREATE INDEX index_channels_on_name ON channels USING gin(to_tsvector('simple', name));
CREATE INDEX index_topics_on_name ON topics USING gin(to_tsvector('simple', name));
CREATE INDEX index_users_on_name ON users USING gin(to_tsvector('simple', username));
```

### Mysql

Create a [view table](https://dev.mysql.com/doc/refman/8.0/en/create-view.html).
You can use the query above for this. 

And our Full Text indexes.
```sql
ALTER TABLE `channels` ADD FULLTEXT INDEX `index_channels_on_name` (`name`);
ALTER TABLE `topics` ADD FULLTEXT INDEX `index_topics_on_name` (`name`);
ALTER TABLE `users` ADD FULLTEXT INDEX `index_users_on_name` (`username`);
```

### Sqlite

Docs will be updated ...