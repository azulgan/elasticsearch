# Goal of this tutorial

Two days ago I needed one simple data migration from one index in elastic search into the same index, but into a different target cluster. The goal was to replicate in a non-production system the same data as in the production.
I was looking for simple ways to do it, without using too specific software. 

## setup environment

Here I will do everything in a local elaticsearch installation (version 7.5.2, on ubuntu), totally empty at first.

````sh
export TYPE=doc
export PREV=http://localhost:9200/orig
export NEW_INDEX=the-target
export NEW=http://localhost:9200/$NEW_INDEX
````
## create some initial data
````sh
cat > input1.json <<EOF
{
"name":"John",
"age":30,
"tags":[ "Ford", "BMW", "Fiat" ]
}
EOF
cat > input2.json <<EOF
{
"name":"Carrie",
"age":70,
"tags":[ "Speeder", "Vader", "Blaster" ]
}
EOF

curl -XPOST $PREV/$TYPE -H "Content-Type: application/json" -d @input1.json

curl -XPOST $PREV/$TYPE -H "Content-Type: application/json" -d @input2.json

````
Here we have now an 'orig' index containing... 2 documents of fairly simple json.
I use POST to create the records and not PUT, I did not want to create the ids myself

## Export all data from previous location
````sh
curl -XGET $PREV/_search?pretty=true > export.json
````
This export can also be merged with the next block, but using export.json file is clearer

## reformat into bulk format
````sh
id=0; cat export.json | jq -cr ' .hits.hits[]._source ' | while read line;
 do echo '{ "index": {"_index":"'$NEW_INDEX'","_type":"'$TYPE'","_id":"'$id'" }}'; echo $line; id=`expr $id + 1`; done > bulk.json
 echo >> bulk.json
````

## bulk insert
````sh
curl -XPOST $NEW/$TYPE/_bulk -H "Content-Type: application/x-ndjson" -d @./bulk.json 
````
But here I have a problem. The elastic search engine keeps complaining about the absence of a CRLF in the end of the file. No matter what, I just can't add it.

So I change slightly the solution, to input the lines one by one into the main loop.

## No bulk insert possible
````sh
cat export.json | jq -cr ' .hits.hits[]._source ' | while read line;
 do 
   echo "Inserting $line"
   curl -XPOST $NEW/$TYPE -H "Content-Type: application/json" -d "$line" 
 done
````
using 'jq -c' to keep each part of the extracted json in its own line of the output

## verify that everything went well in the new location
````sh
nb=`curl -XGET $NEW/_search 2> /dev/null | jq -r '.hits.total.value'`
if [ $nb -eq 2 ]; then echo 'OK'; else echo 'NOK'; fi
````
You should see 'OK', right ?

## Performance
This chapter should be covered... Let's say procrastination has merits

## Limitations
The same

## Conclusion
The goal of this howto was to permit to transmit data from one elasticsearch index into one another. 
I have used only basic shell commands like curl and of course the elastic search API.
The only tool that might be non installed by default on your linux distribution is jq. It permits to query and reformat the input json.
But it should be far from efficient when talking about performance. For a real life index, if it is not possible to avoid the migration of data, I would recommand to use )[elasticsearch-dump](elasticsearch-dump.md)

