---
title: "Intercepting Requests and Reponses With Django's Middleware"
date: 2020-04-11T17:51:21-03:00
tags: ["Middleware", "Django", "Python", "Blog"]
---

I have been working on an app where a user can have multiple projects, and all features should work against the currently selected project. I want to make the project automagically available to the views (all of them) without any hassle and one way to do that is by injecting the project into the request before they hit the view layer. In Django, we can easily accomplish this task by creating a custom middleware.

![Middleware cycle](/images/articles/middleware-cycle.png)

From the docs

>Middleware is a framework of hooks into Django’s request/response processing. It’s a light, low-level “plugin” system for globally altering Django’s input or output.

This is exactly what we need. Django provides an elegant way to plug a custom middleware that will be executed in the request/response lifecycle. We just need to implement its interface and tell Django to look into it when processing client's calls. In my case, I need to add two of them, one to enforce that at least one project is required to use the system and another to inject the project into the requests so we can use it within the views.

```python
class CustomMiddleware:

    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        # anything that should be done before hitting the view.
        # In my case, here is where I inject the selected project

        response = self.get_response(request)

        # anything that should be done before returning
        # the reponse to the client
        return response

```

Notice that the `__init__` method will be called one time since the object is instantiated once. The `__call__` method is where you have to add the logic you want to apply. Also, we need to tell Django that it should be executing them by adding their path to the `MIDDLEWARE` variable on the project's setting.

```python
MIDDLEWARE = [
    .
    .
    .

    # custom middleware, they are executed in the order of this list
    'myproject.projects.middleware.CustomMiddleware'

]
```
