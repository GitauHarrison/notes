# Applications in Django

The thinking behind Django is that we have a website project, which we have already created in [the previous tutorial](01_getting_started.md), and have multiple apps each doing its own thing. For example, we can have a blog section of the website, which is an app on its own, then we can have a store section which is an app as well. So, a single project can contain multiple apps.

If you would like to skip to a particular section in the entire Django tutorial, you can do so by clicking on any of the links below:

- [Django Overview](00_overview.md)
- [Getting Started With Django](01_getting_started.md)
- [Applications and routes](02_applications_and_routes.md) (this article)


### Table of Contents

In this tutorial, we are going to add a blog app within our project.

- [Create an app](#create-an-app)




## Create An App

Ensure you are currently at the top-level directory of our project, where `manage.py` is located. From the terminal, run the following command:

```python
(venv)$ python3 manage.py startapp blog
```

