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

Notice that this is just a rule of thumb, in some cases increase the resources doesn't help to improve performance or 
reduce costs so there is no better way to optmize than experimenting and monitoring logs.

## Keep an 👀 on security 
AWS recommends you to follow the principle of least privilege by giving to your resources permissions to access
just what they need to performa a task. For lambda, it is helpful for this principle to map one role per lambda even
if they are giving the same permissions. In case a function receives any enhancement a change the role's policy
wouldn't affect every other functions.

Also, be aware that lambda run in a shared VPC so its not safe to kept credentials in code. In case the role is not
enough you, like if you need to access resource in another account, you can retrieve temporary credentials by assuming
a role. If the function needs long-lived credentials then you can use encrypted enviroment variables.

## Unlink temporary files 
If you create some temporary files under `/tmp`, make sure they are unlinked before leaving the handler otherwise
they will be persisted for future executions. If you need them to be available so be aware of that but also be
aware that the `/tmp` folder is not intendeed for that.

## Use privates VPCs only if necessary
Invoke permissions are decoupled from execution permissions so there is no security benefit of using a private VPC,
unless your function depends on resources available there. Although the connection between lambdas' managed and 
private VPCs were improved you still can face longer cold start latency due to the need of create and invoke
the network interface to link the resources.

## Understand how reserved concurrency works


## Use error handlers and dead letter queues

## Keep containers warm

## One task per Lambda

## Use frameworks like The Serverless Application Model (SAM)

## Use continuous delivery and continuous integration tools
