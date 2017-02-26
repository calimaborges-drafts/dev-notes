# Mongo DB



## Indexes


### Explain

#### No query

```javascript
db.students.explain().find({student_id:5});
```

#### With query

```javascript
db.students.explain(true).find({student_id:5});
```


### Create indexes

#### One field

```javascript
db.students.createIndex({ student_id:1});
```

#### Two fields

```javascript
db.students.createIndex({ student_id:1, class_id: -1 });
```


#### Discover indexes

```javascript
db.students.getIndexes();
```

#### Delete index

```javascript
db.students.dropIndex({ student_id: 1});
```


### Create indexes on array (multikey indexes)

```javascript
db.foo.insert({ a: 1, b: 2});
db.foo.createIndex({a:1, b:1});
db.foo.insert({a:3, b:[3,5,7]}); // index became multikey
db.foo.insert({b:3, a:[3,5,7]}); // index became multikey
db.foo.insert({a: [3,4,6], b:[7,8,9]}); // error! multikey index with 2 arrays
```

### Create index with dot notation

```javascript
db.students.createIndex({'scores.score':1});
```

### Create unique index

```javascript
db.stuff.createIndex({thing:1}, {unique: true});
```

### Create unique index in null fields

*Obs.: sort does not use the index in this case*

```javascript
db.employees.createIndex({cell:1}, {unique: true, sparse: true});
```

db.stores.find({location:{ $near:[50,50]}});


### Full text search indexes

#### Create index

```javascript
db.senteces.ensureIndex({'words': 'text'});
```

#### Find indexes field

```javascript
db.sentences.find({ { $text: { $search: 'dog' } } });
```

#### Find indexes with metadata

```javascript
db.sentences.find(
    { $text: { $search: 'rat tree obsidian' } },
    { score: { $meta: 'textScore' } }
).sort({ score: { $meta: 'textScore' } });
```


### Profiling

#### Query profile

```javascript
db.system.profile.find()
db.system.profile.find({millis:{$gt:1}}).sort({ts:1}).pretty()
```

#### Profiling level

*Obs.: 0 - none; 1 - slow queries; 2 - all queries;*

```javascript
db.getProfilingLevel()
db.getProfilingStatus()
db.setProfilingLevel(1,4) // profile slow queries of about 4 ms
```


### Aggregation

Help: Aggregation Pipeline Quickreference

#### Aggregation Query

***Obs.: The order of the operators matters***

```javascript
db.companies.aggregate([
    { $match: { founded_year: 2004 } },
    { $sort: { name: 1 } },
    { $skip: 10 },
    { $limit: 5 },
    { $project: {
        _id: 0,
        name: 1,
        founded_year: 1
    } }
]);
```

#### Reshapping

#### Example 1

```javascript
db.companies.aggregate([
    { $match: { "funding_rounds.investments.financial_org.permalink": "greylock" }}.
    { $project: {
        _id: 0,
        name: 1,
        ipo: "$ipo.pub_year",
        valuation: "$ipo.valuation_amount",
        funders: "$funding_rounds.investments.financial_org.permalink"
    }}
]);
```

#### Example 2

```javascript
db.companies.aggregate([
    { $match: { "funding_rounds.investments.financial_org.permalink": "greylock" }},
    { $project: {
        _id: 0,
        name: 1,
        founded: {
            year: "$founded_year",
            month: "$founded_month",
            day: "$founded_day"
        }
    }}
]);
```

#### Unwind

##### Example 1

```javascript
db.companies.aggregate([
    { $match: { "funding_rounds.investments.financial_org.permalink": "greylock"} }, // Reduce amount of results
    { $unwind: "$funding_rounds" },
    { $unwind: "$funding_rounds.investments" },
    { $match: { "funding_rounds.investments.financial_org.permalink": "greylock"} }, // Show only permalink with greylock
    { $project: {
        _id: 0,
        name: 1,
        fundingOrganization: "$funding_rounds.investments.financial_org.permalink",
        amount: "$funding_rounds.raised_amount",
        year: "$funding_rounds.funded_year"
    } },
])
```

#### Array Expressions

##### Example 1 - Filter

```javascript
db.companies.aggregate([
    { $match: {"funding_rounds.investments.financial_org.permalink": "greylock" } },
    { $project: {
        _id: 0,
        name: 1,
        founded_year: 1,
        rounds: { $filter: {
            input: "$funding_rounds",
            as: "round",
            cond: { $gte: ["$$round.raised_amount", 10000000 ] }
        } },
    } },
    { $match: {"rounds.investments.financial_org.permalink": "greylock" } },
]);
```

##### Example 2 - Array Element At

```javascript
db.companies.aggregate([
    { $match: {"funding_rounds.investments.financial_org.permalink": "greylock" } },
    { $project: {
        _id: 0,
        name: 1,
        founded_year: 1,
        first_round: { $arrayElemAt: [ "$funding_rounds", 0 ] },
        last_round: { $arrayElemAt: [ "$funding_rounds", -1 ] },
    } }
]);
```

##### Example 3 - Slice

```javascript
db.companies.aggregate([
    { $match: {"funding_rounds.investments.financial_org.permalink": "greylock" } },
    { $project: {
        _id: 0,
        name: 1,
        founded_year: 1,
        early_rounds: { $slice: [ "$funding_rounds", 1, 3 ] },
        first_round: { $slice: [ "$funding_rounds", 1 ] },
        last_round: { $slice: [ "$funding_rounds", -1 ] }
    } }
]);
```

##### Example 4 - Size

```javascript
db.companies.aggregate([
    { $match: {"funding_rounds.investments.financial_org.permalink": "greylock" } },
    { $project: {
        _id: 0,
        name: 1,
        founded_year: 1,
        total_rounds: { $size: "$funding_rounds" }
    } }
]);
```

#### Accumulators in $project stages

##### Example 1

```javascript
db.companies.aggregate([
    { $match: { "funding_rounds": { $exists: true, $ne: [ ] } } },
    { $project: {
        _id: 0,
        name: 1,
        largest_round: { $max: "$funding_rounds.raised_amount" }
    } }
]);
```

#### Using $group

##### Example 1

```javascript
db.companies.aggregate([
    { $group: {
        _id: { founded_year: "$founded_year" },
        average_number_of_employees: { $avg: "$number_of_employees" }
    } },
    { $sort: { average_number_of_employees: -1 } }
]);
```

##### Example 2

```javascript
db.companies.aggregate( [
    { $match: { "relationships.person": { $ne: null } } },
    { $project: { relationships: 1, _id: 0 } },
    { $unwind: "$relationships" },
    { $group: {
        _id: "$relationships.person",
        count: { $sum: 1 }
    } },
    { $sort: { count: -1 } }
] );
```

##### Example 3

```javascript
db.companies.aggregate([
    { $group: {
        _id: { ipo_year: "$ipo.pub_year" },
        companies: { $push: "$name" }
    } },
    { $sort: { "_id.ipo_year": 1 } }
]);
```
