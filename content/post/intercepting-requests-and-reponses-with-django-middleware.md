---
title: "Intercepting Requests and Reponses With Django's Middleware 👀"
date: 2020-04-11T17:51:21-03:00
tags: ["Middleware", "Django", "Python", "Blog"]
---

I have been working with a project where a user can have multiple projects and all features should work against the currently selected project. I want to make the project available in the request object so all views can easily apply its business rules to it without caring about if the project is selected and which project is that. One solution for that is to intercept all requests before they hit the views and add the currently selected project to the requests object so it will be available for later usage. How to do that in Django? Creating a custom middleware.
![Middleware cycle](/images/articles/middleware-cycle.png)

From the docs

>Middleware is a framework of hooks into Django’s request/response processing. It’s a light, low-level “plugin” system for globally altering Django’s input or output.

This is exactly what we need. Django provides an elegant way to plug a custom middleware that will be executed in the request/response lifecycle. You just need to implements its interface and tell Django that it should look for your middleware when processing client calls. In my case, I need to add two of them, one to enforce that at least one project is required to use the system and another to inject the project into the requests so we can use it within the views.

```python
class NoProjectsMiddleware:
    """
    This should redirect the authenticated user to the project list
    if he doesn't have any project created. (The system needs a project
    selected to behaves correctly)
    """

    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        # anything that should be done before processing the
        # request needs to be added here
        if self.need_to_redirect_to_project_list(request):
            return redirect('projects:list')

        response = self.get_response(request)
        
        # anything that should be done before returning the
        # reponse needs to be added here
        return response

    @staticmethod
    def need_to_redirect_to_project_list(request):
        full_path = request.get_full_path()
        return (
                not '__debug__' in full_path
                and not full_path == reverse('projects:list')
                and request.user.is_authenticated
                and not request.user.has_projects()
        )


class ProjectInjectorMiddleware:
    """
    A user should always have a project selected so if the user hasn’t done that
    this will select the first one automatically
    """

    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        # anything that should be done before processing the
        # request needs to be added here
        if request.user.is_authenticated and request.user.has_projects():
            if not self.project_set(request):
                request.session['selected_project_id'] = str(request.user.first_project_sorted_by_name().id)

            if request.session['selected_project_id']:
                request.selected_project = Project.objects.get(id=request.session['selected_project_id'])

            request.user_projects = request.user.project_list()

        response = self.get_response(request)

        # anything that should be done before returning the
        # reponse needs to be added here
        return response

    @staticmethod
    def project_set(request):
        selected_project_id = request.session.get('selected_project_id')
        return selected_project_id and request.user.has_project(selected_project_id)
```

The `__init__` method will be called one time and this is where you should initialize and configure anything you need. The `__call__` method is where you should add the business rule you have to process, on NoProjectsMiddleware I am forcing the user to go to the project lists (to create one project) until he creates one and on ProjectInjectorMiddleware I am adding the first project to the session if he doesn’t have one selected yet, I am adding the project to the `selected_project` variable and finally adding all existent projects to the `user_projects` list so he can select the project he wants to work with from a UI component globally available in the system.

We need to tell Django that it should be executing them and for that we just need to add their path to the `MIDDLEWARE` variable. 

```python
MIDDLEWARE = [
    .
    .
    .

    # custom middleware
    'myproject.projects.middleware.NoProjectsMiddleware',
    'myproject.projects.middleware.ProjectInjectorMiddleware',

]
```

Notice that the order they are added is the order they will be executed, although it’s not required,  worth checking if there are no projects in the system before trying to work with them.

