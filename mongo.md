# Mongo DB



## Indexes


### Explain

#### No query

```
db.students.explain().find({student_id:5});
```

#### With query

```
db.students.explain(true).find({student_id:5});
```


### Create indexes

#### One field

```
db.students.createIndex({ student_id:1});
```

#### Two fields

```
db.students.createIndex({ student_id:1, class_id: -1 });
```


#### Discover indexes

```
db.students.getIndexes();
```

#### Delete index

```
db.students.dropIndex({ student_id: 1});
```


### Create indexes on array (multikey indexes)

```
db.foo.insert({ a: 1, b: 2});
db.foo.createIndex({a:1, b:1});
db.foo.insert({a:3, b:[3,5,7]}); // index became multikey
db.foo.insert({b:3, a:[3,5,7]}); // index became multikey
db.foo.insert({a: [3,4,6], b:[7,8,9]}); // error! multikey index with 2 arrays
```

### Create index with dot notation

```
db.students.createIndex({'scores.score':1});
```

### Create unique index

```
db.stuff.createIndex({thing:1}, {unique: true});
```

### Create unique index in null fields

*Obs.: sort does not use the index in this case*

```
db.employees.createIndex({cell:1}, {unique: true, sparse: true});
```

db.stores.find({location:{ $near:[50,50]}});


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


### Profiling

#### Query profile

```
db.system.profile.find()
db.system.profile.find({millis:{$gt:1}}).sort({ts:1}).pretty()
```

#### Profiling level

*Obs.: 0 - none; 1 - slow queries; 2 - all queries;*

```
db.getProfilingLevel()
db.getProfilingStatus()
db.setProfilingLevel(1,4) // profile slow queries of about 4 ms
```
