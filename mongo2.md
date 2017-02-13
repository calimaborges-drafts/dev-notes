### Full text search indexes

#### Create index

```
db.senteces.ensureIndex({'words': 'text'});
```

#### Find indexes field

```
db.sentences.find({ { $text: { $search: 'dog' } } });
```

#### Find indexes with metadata

```
db.sentences.find(
    { $text: { $search: 'rat tree obsidian' } },
    { score: { $meta: 'textScore' } }
).sort({ score: { $meta: 'textScore' } });
```
