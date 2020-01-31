---
title: "AWS Lambda Best Practices 🙌"
date: 2020-01-28T09:45:49-03:00
tags: ["AWS", "Lambda", "Blog"]
---
![A rocket lambda](/images/articles/rocket-lambda.png)
## Code optimizations that you should be aware 
```python
from database import connect 

database_uri = "uri://database/"
connection = connection(database_uri)

def extract_name():
    return event['key']

def handler(event, context):
    """A handler that just does what must be done."""
    connection.call(extract_name(event))
```
There are a few things that you could do with your code that might improve lambda's performance, and some of them are:
- Keep your declarations and instantiations outside the handler function. Lambda will declare/instantiate them once and recycle then in the subsequent invocations.
- Use interpreted languages like Python or NodeJs instead of compiled languages like C or Java. It might be counter-intuitive, but because of the cold start phenomena, the total amount of time spent to start and execute a function is
bigger on compiled languages. If you properly warm your functions, then F# and C# have shown the fastest execution time.
- Remove all unnecessary dependencies! There is a size limit for the package you can send to lambda and even if your code
is performing simple tasks, it's libraries might contain dozens of packages that are not needed and it could be taking a 
lot of disk space.
- Use NodeJS when concerned about space. The JavaScript ecosystem has optimization tools that allow you to use strategies like minification, uglification and tree shaking. For functions smaller than 600 characteres, the V8 runtime will inline the code, making the files even smaller.
- Keep your declarations and instantiations outside the handler function. When the container goes down everything in the
global scope will be retained and will be used in the subsequent invocations making the execution faster. Also, notice that
this is usually a source of unexpected problems.
 

## Optimize performance and cost by balancing the resources
Increase the memory will automatically increase the CPU allocation so, if you have a CPU heavy task, then it might worth
to increase the memory amount to something way higher than needed. Having more computation power available can make your
function execute faster, optimizing for speed and reducing the cost as a side effect.

Notice that this is just a rule of thumb, in some cases increase the resources doesn't help to improve performance or 
reduce costs, so there is no better way to optimize than experimenting and monitoring logs.

## Keep an 👀 on security 
AWS recommends you to follow the principle of least privilege by giving to your resources permission to access
just what they need to perform a task. For lambda, it is helpful for this principle to map one role per lambda even
if they are giving the same permissions. In case a function receives any enhancement, a change in the role's policy
wouldn't affect each other functions.

Also, be aware that lambda run in a shared VPC, so it is not safe to kept credentials in code. In case the role is not
enough you, like if you need to access the resource in another account, you can retrieve temporary credentials by assuming
a role. If the function needs long-lived credentials, then you can use encrypted environment variables.

## Unlink temporary files 
```python
from subprocess import call

def handler(event, context):
    ...
    call('rm -rf /tmp/*', shell=True)
```
If you create some temporary files under `/tmp`, make sure they are unlinked before leaving the handler. Otherwise, they will persist in future executions. You can take advantage of that persistence but keep in mind that it is not the intention behind the temporary folder.

## Use privates VPCs only if necessary
Invoke permissions are decoupled from execution permissions, so there is no security benefit of using a private VPC, unless your function depends on resources available there. 

The connection between lambdas' managed and private VPCs was recently improved, but you still can face longer cold start latency due to the need to create and invoke the network interface to link the resources.

## Understand how reserved concurrency works
You might have seen ~~everywhere~~ somewhere that lambdas scale infinitely but the truth is, in a world of limited resources, nothing is really infinite and what they mean is that it scales to a size so big that you will probably not need to care about it.

Concurrency on lambdas is limited to 1000 parallel execution per account and, by default, 
each function has unrestricted access to the account's concurrency limits. It is not a good thing and one function could potentially use all resources available, throttling the execution of the other ones. 

To avoid that, we should always limit the concurrency limit per function so we can control the impact of
a heavy called task over the other tasks on our account. Also, don't forget that if your function is using other services like dynamoDB, you should be aware of their execution limits so your functions don't flood them with a high amount of calls.

## Keep your lambdas warm
![Warm lambdas](/images/articles/warm-lambdas.png)
A cold start happens when you call an inactive function and it needs to initialize before executes, introducing latency to whole lambda call. To prevent that what you can do is to keep your lambdas warm by,
for example, making dumb calls on it to prevent AWS to deactivate them so, once a real request is made, the initialization
doesn't need to be done and the latency is really low.

## One task per Lambda
Although you can add a whole web server inside of a lambda function, they are made to execute only one task and you should do as intended. By isolating one functionality per function you will be able to best optimize your functions and it will be easier to debug.

## Use frameworks like The Serverless Application Model (SAM)
There are many advantages of using a framework to develop your serverless application but the one I like the most is
the possibility of testing and debug your functions locally. Having to test your app on AWS every time you make a change will turn the enjoyable task of developing serverless applications into a nightmare, due to the overhead of
the deployment process. With AWS SAM you can invoke functions, run API and there's even a docker container to run 
dynamoDB locally.
