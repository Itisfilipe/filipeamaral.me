---
title: "Serverless Workflows With AWS Step Functions"
date: 2020-03-21T09:23:18-03:00
draft: true
---

If you are from the IT universe you surely know that there is a new trend almost every week. There
is aways a new javascript framework that someone is making to avoid the painful that of writting
vanilla javascript or a new tool that does exactly the same thing that the older tool was doing but for whatever
reason the new tool is better (Slack?). There are also trends on the infrastructure side of the force,
like when they decided to go from servers to cloud (maybe because they became more fragile 🤔) or like
now that we are seeing the term serverless poping up everywhere. Well, despite the name, there are
still servers on serverless so what is the real benefit here? Let's find out.

The most common product on the serverless world are functions. Functions as we have on our apps,
the ones that receive something and return something else. They are sold as the holly grail of
computing, they are fast, cheap and although serverless functions aren't meant for heavy tasks they
scale indefinetly, what is ideal of the majority of web requests. But what about complex workflows,
can we orchestrate them to execute tasks that aren't necessesarily heavy but requires many steps to
complete? Well, apparently we can. AWS has created a product called Step Functions that
allow us to create complex state machines where each node is a function that will do what was programmed
to do without knowing about the other functions in the machine.
