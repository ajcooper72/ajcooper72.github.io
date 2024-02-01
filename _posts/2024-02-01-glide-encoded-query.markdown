---
layout: single
title:  "Glide(Encoded)Query"
date:   2024-02-01 15:26:00 +0000
categories: javascript glidequery
excerpt: "Using GlideQuery.parse() to process encoded queries"
---

## Glide(Encoded)Query
One of the early shortfalls with GlideQuery was that you couldn't do encoded queries.  This changed a few releases ago which I'll admit I didn't notice.  There are however some limitations to watch out for.
 
Given the following encoded query 

```
caller_id=77ad8176731313005754660c4cf6a7de^priority=4
```

 We can now use the GlideQuery.parse() method where in the past we would have to build the equivalent .query() calls.  This is called and handled slightly differently to a normal GlideQuery call in that we don't need the 'new' keyword.

 ```javascript
 var gqIncidents = global.GlideQuery.parse('incident', 'caller_id=77ad8176731313005754660c4cf6a7de^priority=4')
    .select([ 'number', 'short_description' ])
    // the reduce call turns the stream object returned by GlideQuery into an array
    .reduce(function (arr, e) { arr.push(e); return arr; }, []);

gs.info(JSON.stringify(gqIncidents, '', 3))
```
There are limitations as to what operators can be used and also it can only handle one query at a time if it gets too complex.  For example, this query (which is not of much use, but is being used just to show the issue) has a '^NQ' operator and also an '^OR' operator, which GlideQuery is still not happy with :(

```
caller_id=77ad8176731313005754660c4cf6a7de^NQstate=7^ORstate=8^assignment_group=NULL
```

To handle this, I've come up with the following.  This is a first stab and so has limited error handling (i.e. none), it also only works in scoped applications due to it's reliance on some EcmaScript 2021 features.

```javascript
function runEncodedQuery(tableName, encodedQuery, fields) {
    var allRecords = [];

    // split the encoded query into multiple queries if ^NQ is present
    var queries = encodedQuery.split('^NQ');

    queries.forEach(query => {
        var gqRecords = global.GlideQuery.parse(tableName, query)
            .select(fields)
            .reduce(function (arr, e) { arr.push(e); return arr; }, [])

        // use the spread operator to concat the new set of records with those discovered so far
        allRecords = [ ...allRecords, ...gqRecords];
    });

    // using the spread operator again along with a Map allows us to quickly remove duplciate values.
    allRecords = [ ...new Map(allRecords.map(v => [v.sys_id, v])).values() ];

    return allRecords;
}
```

We can now call something similar to the following to get a list of incidents using a more complex encoded query.

```javascript
var gqIncidents = runEncodedQuery('incident', 'caller_id=77ad8176731313005754660c4cf6a7de^NQstate=7^ORstate=8^assignment_group=NULL', ['number', 'short_description']);

gs.info(JSON.stringify(gqIncidents))
```

To summarise, the parse method has some way to go to be a good replacement for GlideRecord.addEncodedQuery() so my preference would be to still use GlideRecord.  