---
title: "AWS Lambda Best Practices 🙌"
date: 2020-01-28T09:45:49-03:00
draft: true
tags: ["AWS", "Lambda", "Blog"]
---
![A rocket lambda](/images/articles/rocket-lambda.png)
## Code optimizations that you should be aware 
```python
import connect from database

database_uri = "uri://database/"
connection = connection(database_uri)

def extract_name():
    return event['key']

def handler(event, context):
    """A handler that just does what must be done."""
    connection.call(extract_name(event))
    
```
- Keep your declarations and instantiations outside the handler function. Lambda will declare/instantiate them once
and recycle then in the subsequent invocations.
- Use intepreted languages like Python or NodeJs instead of compiled languages like C or Java. This might be counter
intuitive but because of the cold start REFENCE phenomena the total amount of time spent to start and execute a function is
bigger REFERENCE on compiled languages. If you properly warm your functions then F# and c# REFERENCE has shown the fastest execution time.
- Remove all unecessary dependencies! There is a size limit for the package you can send to lambda and even if your code
is performing simple tasks it's libraries might contain dozens of packages that are not needed and it could be taking a 
lot of disk space.
- Use NodeJS when concerned about space. With that you can use strategies like minification, uglification, tree shaking
and for functions smallers than 600 characteres the V8 runtime will inline the code, making the files even smaller.
- Keep your declarations and instantiations outside the handler function. When the container goes down everything in the
global scope will be retained and will be used in the subsequent invocations making the execution faster. Also notice that
this is usually a source of unexpected problems.
 

## Optimize performance and cost by balancing the resources
Increase the memory will automatically increase the CPU allocation so, if you have a CPU heavy task then it might worth
to increase the memory amount to something way higher than needed. Having more computation power available can make your
function execute faster, optimizing for speed and reducing the cost as a side effect.

## Grant only the necessary permissions


## User VPC only if necessary

## Understand how reserved concurrencyt works


## Use error handlers and dead letter queues

## Keep containers warm

## One task per Lambda

## Use frameworks like The Serverless Application Model (SAM)

## Use continuous delivery and continuous integration tools
