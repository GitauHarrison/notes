# Unit Testing in Flask

In this article, I will be using the sample project [Configure Flask To Use PostgreSQL](https://github.com/GitauHarrison/configure-flask-to-use-postgresql) to discus how to write unit tests in a Flask web applications. The concepts, though, are universal and can be applied to other frameworks as well.

![Unit testing in flask](/unit_testing/images/unit_testing_in_flask.gif)

Before we can go any further, these are the other sections in the unit testing reviews. You can click on any of the following links to access additional content:

1. [Unit Testing Overview](/unit_testing/unit_testing_overview.md)
2. [Unit Testing In Python](/unit_testing/unit_testing_in_python.md) (this article)
3. [Unit Testing In Flask](/unit_testing/unit_testing_in_flask.md)

If you would like to follow along using this application, follow [these guidelines](https://github.com/GitauHarrison/configure-flask-to-use-postgresql#testing-the-application-locally) to get started. At this point in the setup process, let us also install the following packages which we shall use to create the testing infrastructure.

```python
(venv)$ pip3 install pytest pytest-cov
```

If you haven't already checked out the other reviews on unit testing, I recommend you go back to them to learn more, then continue with this. As I write this review, there is no `test` module in the application's root directory. I will be adding one and updating it as we go along. By the time you clone the repository, you may find that it has a test module, but that is because I added it at the end.

The application features the following:

- [x] A user can register for an account
- [x] A registered user's data is stored in a database
- [x] The user's data is retrieved to allow them to log in
- [x] Once logged in, the go to the profile page
- [x] Userful feedback is provided to the user along the way
- [x] The profile page can only be accessed by a logged in user
- [x] The profile page displays the logged in user's username
- [x] The profile page features a _post_ form
- [x] The user can make a post, which would be displayed below the form
- [x] The logged in user can log themselves out of their accounts
- [x] Once logged out, they are redirected to the login page

It is important to know what parts the application you want to build will have so that you can adequately plan for them by creating tests that need to be passed before accepting any changes. Python includes a very useful `unittest` package that makes it easy to wite and execute unit tests. In this example, I will be using an "enhanced" test runner called `pytest`.

## Writing A Test Case

Let us begin by creating a `test` module in the top-level directory.

```python
(venv)$ touch tests.py
```

