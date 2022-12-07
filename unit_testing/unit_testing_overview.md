# Unit Testing Overview

To help you understand what unit testing is, I think we can start by answering the question, "Why would you want to test?" Well, there are a couple of reasons why you would want to test your Python scripts or applications. If you have gotten to a point in your software development journey where you are looking into the concept of testing, you can appreciate that it seems to be so much easier to simply go ahead and create a script or build a project without having to worry about whether something will work or not. As a matter of fact, when creating, we manually test each part of the script or module along the way to fix all or at least the most bugs. It therefore, makes sense to argue that making automated testing infrastructure prior to building your application or writing your script is burdensome.

On one hand, it is true that if you are building something from scratch, there is a good chance that there will be few bugs, especially if you have previous coding background. On the other hand, it becomes incredibly challenging to know the impact your code will have on pre-existing code you are contributing to. Your effort is going to be focused more on the work you are doing and less on its impact on old code. Bugs that are indirectly caused by your new contributions (what's referred to as _regressions_) become incredibly time-consuming to test as the application grows. 

The reason why you would want to write tests is to insure your newly written code against regressions in the future. Running such tests is easy, and therefore, we can be certain that an application is not affected by the additions. Additionally, automated tests help us to ascertain conditions that are difficult to test manually such as when  APIs return errors due to a third party being down. How can you test that manually? It is very difficult. With automated tests, though, we are able to achieve this.

## Different Types Of Tests

There are three kinds of tests:

- **Unit test**: They focus on individual modules of a project in isolation to ascertain that everything works as expected. This will be the focus of our testing efforts.
- **Integration test**: They test how two or more modules work together.
- **Functional test**: This is an end-to-end testing of an application to ensure that everything works as expected. [Selenium](https://www.selenium.dev/) is commonly used here. 

The difference among the mentioned tests is their scope. Notice that unit tests focus on specific part of a system while functional tests narrow down on all components of an application or system, making it the hardest to write. Integration tests fall somewhere in between the two.

## Table of Contents

As mentioned above, I will focus mainly on unit tests. I will walk you through how to perform tests on the following:

- [Unit Testing In Python](/unit_testing/unit_testing_in_python.md)
- [Unit Testing In Flask](/unit_testing/unit_testing_in_flask.md)

