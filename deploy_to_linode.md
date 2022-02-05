# Deploy Your Flask Application on Linode

When you start your flask server by running the command `flask run` in your terminal, you will get a message similar to this:

```python
(venv)$ flask run

 * Serving Flask app 'blog.py' (lazy loading)
 * Environment: development
 * Debug mode: on
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
 * Restarting with stat
 * Debugger is active!
 * Debugger PIN: 195-448-500
```

Of interest here is the environment used by the server. You can see that I have explicitly  set flask to use a "development" server. Development servers are intended for use only during local development because they are designed not to be efficient, stable or secure. 

Let us say you have completed building your application and you have hosted it in a remote version control system such as GitHub. Flask does not recommend the use of the built-in server in production because it does not scale. It recommends a few deployment options that you can take advantage of to properly run your flask app in production:


- [Deploying Flask on Heroku](https://devcenter.heroku.com/articles/getting-started-with-python)
- [Deploying Flask on Google App Engine](https://cloud.google.com/appengine/docs/standard/python3/runtime)
- [Deploying Flask on Google Cloud Run](https://cloud.google.com/run/docs/quickstarts/build-and-deploy/python)
- [Deploying Flask on AWS Elastic Beanstalk](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create-deploy-python-flask.html)
- [Deploying on Azure (IIS)](https://docs.microsoft.com/en-us/azure/app-service/containers/how-to-configure-python)
- [Deploying on PythonAnywhere](https://help.pythonanywhere.com/pages/Flask/)

The services above are all hosted on the cloud and are designed to scale, be secure and stable. They are characterized by lifting the burden of setting up, configuring and maintaining the server from the developer.

Alternatively, you can choose to host a flask application yourself. This approach will allow you to learn the technical details that go into web hosting and server configuration and maintainance. During this tutorial, I will show how you can deploy your flask application on [Linode](https://www.linode.com/). In particular, I will show you three things:

1. [How to deploy your flask app on Linode](/linode/deploy_on_linode.md)
2. [How to buy a domain name for your deployed application](/linode/buy_domain.md)
3. [How to secure your domain with SSL](/linode/secure_domain_with_ssl.md)

I won't lie to you, deploying an application can be overwhelming because there are a lot of different ways to do it. Knowing what is best for your specific application can also be difficult. Here, you will learn how to deploy your flask application to a Linux server using NGINX and GUNICORN.

