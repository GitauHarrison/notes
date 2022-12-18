# Django Overview

Django was initially developed between 2003 and 2005 by a web team who were responsible for creating and maintaining newspaper websites. After creating a number of sites, the team began to factor out and reuse lots of common code and design patterns. This common code evolved into a generic web development framework, which was open-sourced as the "Django" project in July 2005.

> [Django](https://www.djangoproject.com/) is a high-level Python web framework that encourages rapid development and clean, pragmatic design. Built by experienced developers, it takes care of much of the hassle of web development, so you can focus on writing your app without needing to reinvent the wheel. It’s free and open source.

Django follows the "Batteries included" philosophy and provides almost everything developers might want to do "out of the box". Because everything you need is part of the one "product", it all works seamlessly together, follows consistent design principles, and has extensive and [up-to-date documentation](https://docs.djangoproject.com/en/4.1/).

Django helps developers avoid many common security mistakes by providing a framework that has been engineered to "do the right things" to protect the website automatically. For example, Django provides a secure way to manage user accounts and passwords, avoiding common mistakes like putting session information in cookies where it is vulnerable (instead cookies just contain a key, and the actual data is stored in the database) or directly storing passwords rather than a password hash.

Django code is written using design principles and patterns that encourage the creation of maintainable and reusable code. In particular, it makes use of the Don't Repeat Yourself (DRY) principle so there is no unnecessary duplication, reducing the amount of code. Django also promotes the grouping of related functionality into reusable "applications" and, at a lower level, groups related code into modules (along the lines of the Model View Controller (MVC) pattern).

## Is Django Opinionated?

Opinionated frameworks encourage the use of the 'right way' to handle any particular task, because that right-way is well-understood and well-documented. However, outside the 'right-way', opinionated frameworks offer less flexibility and fewer approaches that you can use.

Django is partly opinionated and partly unopinionated. It provides a set of components and preferred ways to handle a task right out of the box. On the other hand, it leaves room for a developer to use different options if desired.

## How Django Compares to Flask

Flask, on contrast, does not provide components or preferred ways of handling a task. It is intentionally referred to as a micro-framework because it only provides the core functionality to run an application. Anything beyond that is achieved by the help of extensions that support the framework.

## Table of Contents

Welcome to the Gently Introduction to Django. Below are the sections in the series whose aim is to help you get started with Django. Intentionally, we are going to learn how to build a Django project together so that you can get a hands-on experience on how to navigate this new Python framework.

- [Django Overview](01_getting_started.md) (this article)
- [Getting Started With Django](01_getting_started.md)
- [Applications and routes](02_applications_and_routes.md)
- [Templates in Django](03_templates.md)
- [Admin Page](04_admin_page.md)
- [Databases and Migrations](05_database_and_migrations.md) 
- [User Registration](06_user_registration.md)
- [User Login and Logout](07_user_login_and_logout.md)

The project we are going to work on together will be a `blog` app as seen below.

![Blog App Demo Project](/02_django/images/demo_blog_app.png)
