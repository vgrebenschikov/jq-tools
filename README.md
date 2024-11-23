# Various hints and tricks about jq tool usage 

## Scripts

### csv2json

csv2json - convert CSV file to json

```bash
$ cat file.csv
name,age,city
Alice,30,New York
Bob,25,San Francisco
Charlie,35,Los Angeles
```

How to run

```json
$ ./csv2json < file.csv
[
  {
    "name": "Alice",
    "age": "30",
    "city": "New York"
  },
  {
    "name": "Bob",
    "age": "25",
    "city": "San Francisco"
  },
  {
    "name": "Charlie",
    "age": "35",
    "city": "Los Angeles"
  }
]
```

### csv2json-stream

csv2json-stream - convert CSV file to json in streaming mode - fits will for huge files

How to run

```json
$ cat huge-file.csv | ./csv2json 
[
{"name":"Alice","age":30,"city":"New York"}
,{"name":"Bob","age":25,"city":"San Francisco"}
,{"name":"Charlie","age":35,"city":"Los Angeles"}
...
```

### csv2json-flow

csv2json-flow - process csv file, generating JSON per each row

How to run

```json
$ cat huge-file.csv | ./cvs2json-flow -c
{"name":"Alice","age":30,"city":"New York"}
{"name":"Bob","age":25,"city":"San Francisco"}
{"name":"Charlie","age":35,"city":"Los Angeles"}
```

Can be used with more filters to process small JSONs

```json
$ cat huge-file.csv | ./cvs2json-flow -c | jq -r .name
Alice
Bob
Charlie
```

### json-flow-uniq

Process flow of JSONs in streaming way, finding uniq keys

how to run:

```json
$ echo '{"name":"Alice","age":30,"city":"New York"}
{"name":"Bob","age":25,"city":"San Francisco"}
{"name":"Charlie","age":35,"city":"Los Angeles"}' | ./json-flow-uniq -r --arg key id
Alice
Bob
Charlie
```

## Hints

### Process huge JSON in streaming mode

Read JSON which is huge list of object, parsing object by object - fits well for streaming processing

or converting JSON list of object into flow of JSON objects

```json
$ cat huge.json 
[
    {"name":"Alice","age":30,"city":"New York"},
    {"name":"Bob","age":25,"city":"San Francisco"},
    {"name":"Charlie","age":35,"city":"Los Angeles"},
...
```

```json
$ cat huge.json | jq -n -c --stream 'fromstream(1|truncate_stream(inputs))'
{"name":"Alice","age":30,"city":"New York"}
{"name":"Bob","age":25,"city":"San Francisco"}
{"name":"Charlie","age":35,"city":"Los Angeles"}
```

### Process huge JSON in streaming mode calculating unique values by the key

```jq
reduce fromstream(1|truncate_stream(inputs)) as $item (
    {}; 
    $item.value as $val | if has($val) | not then .[$val] = true else . end
) | keys[]
```

### Process huge flow of JSONs calculating unique values

Below example makes uniq by .name

```jq
reduce inputs as $item (
    {};
    $item.name as $val | if has($val) | not then .[$val] = true else . end
) | keys[]
```
