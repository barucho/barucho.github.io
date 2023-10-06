---
title: 'test blog page'
date: 2012-08-14
permalink: /posts/test/
tags:
  - test
---

This is a sample blog post. Lorem ipsum I can't remember the rest of lorem ipsum and don't have an internet connection right now. Testing testing testing this blog post. Blog posts are cool.

Headings are cool
======

You can have many headings
======

<pre>
select * from table
</pre>

```sql
select * from table;
SELECT * from TABLE;
```

```python
def lambda_handler(event, context):
    print("START...")
    count = 0
    for record in event['Records']:
        #id = record['eventID']
        timestamp = record['kinesis']['approximateArrivalTimestamp']
        # Kinesis data is base64-encoded, so decode here
        message = base64.b64decode(record['kinesis']['data'])
        message_json = json.loads(message)
        print("message",message_json)
        # get ops type
        op_type = message_json['type']
        # build data 
        if op_type == ('WriteRowsEvent'):
            if message_json['table'] == ('user'):
                _id =  str(message_json['row']['values']['user_id'])
                _name =   str(message_json['row']['values']['u_name'])
                _text = None
                label = 'user'
            elif message_json['table'] == ('post'):
                _id =  str(message_json['row']['values']['post_id'])
                _name = str(message_json['row']['values']['p_name'])
                _text = str(message_json['row']['values']['p_text'])
                label = 'post'
            elif message_json['table'] == ('route'):
               _post_id =  str(message_json['row']['values']['post_id'])
               _user_id = str(message_json['row']['values']['user_id'])
               _status = str(message_json['row']['values']['status_id'])
               label = 'route'
            ## create the relent object in neptune 
            lbl = T.label
            id = T.id
            single = Cardinality.single
            # sending data to neptune
            if label == 'route':
                #gremlin_status = g.addV().property(id, _post_id).property('user_name', _name).property(lbl, label).next()
                print("route")
            else:
                _id = label+"_" +_id
                gremlin_status = g.addV().property(id, _id).property('user_name', _name).property(lbl, label).next()
            
            print("debug: data",_id,_name,label)
            print("gremlin_status:",gremlin_status)
            print("END")
    return 'Processed ' + str(count) + ' items.'

```

Aren't headings cool?
------