# Implement Popover

For your reference, these are the sections included in this tutorial:

1. [Section 1: Create a simple web application](/flask_popover.md#create-a-simple-application)
2. [Section 2: Web Forms](web_forms.md)
3. [Sectin 3: Working with Databases](database.md)
4. [Section 4: User Login](user_login.md)
5. [Section 5: User Posts](user_posts.md)
6. [Section 6: Implement Popover](popover.md)

Popovers are a common feature in social web applications where a quick summary of a user's profile is displayed when the user hovers over a link or image. In this section, we will implement a popover that displays a user's profile.

![Popover](/images/flask_popover/popover.png)

The popover is actually a minified template based off a user's profile. We can render it just as we would render any other template.

`app/routes.py`: User popover function
```python
# ...


@app.route('/user/<username>/popup')
@login_required
def user_popup(username):
    user = User.query.filter_by(username=username).first_or_404()
    return render_template('user_popup.html', user=user)
```

A shorter version of a user's profile template would be:

`app/templates/user_popup.html`: User popover template
```html
<table class="table table-hover">
    <tr valign='top'>
        <td width='70px'>
            <img src=" {{ user.avatar(36) }} ">
        </td>
        <td>
            <p>
                <a href="{{ url_for('user', username=user.username) }}">
                    {{ user.username }}
                </a>
            </p>
            {% if user.about_me %}
            <p>
                    {{ user.about_me }}
            </p>
            {% endif %}
            <p>
                    Last seen: {{ moment(user.last_seen).format('LLL') }}
            </p>
        </td>        
    </tr>
</table>
```
I am only displaying the user's avatar, their username, bio and last seen date in the popup. Make user that you create this template:

```python
(venv) $ cd templates
(venv) $ touch user_popup.html
```
We will use JavaScript to invoke this route when the user hovers the mouse pointer over a username. In response, the server will return the HTML content for the popup. To check how the popup looks like, you can navigate to the URL _/user/username/popup_ in your browser. I have simply appended _/popup_ to a user's profile URL.

Since we are using Boostrap to style our application, we can as well refer to its documentation to learn how it implement popovers. [Bootstrap popovers](https://getbootstrap.com/docs/3.3/javascript/#popovers), as it is said, add small overlays of content to any element for housing secondary information.

It is actually very simple to implement popovers to a HTML portion. Identify what element you want to attach the popover then invoke the `popover()` function in JavaScript to initialize the popover. In our case, this will be the username link:

`app/templates/_post.html`: Identify the element to attach the popover
```python
<a href="{{ url_for('user', username=post.author.username) }}">
    {{ post.author.username }}
</a>
```

## DOM Elements for the Popover

We can take advantage of JQuery, a JavaScript library, to register a function which will be invoked on every page load. This is done by wrapping the function inside a `$(...)` block. Within our flask application, we can add this function in our `scripts` block in `base.html`.

`app/templates/base.html`: Register the popover function
```html
{% block scripts %}
    {{ super() }}
    {{ moment.include_moment() }}
    <script>
        $(function() {
            // code will go here
        });
    </script>
{% endblock %}
```
To select an element, we can use either the `id` or the `class` attribute. `id` is applied to an element when it is unique to a page. `class` is applied to an element when it is shared by multiple elements. Since our application will have multiple posts from users, the `class` attribute is more appropriate. If you try to use the `id` attribute, you will notice that only the first post in your page will have a popover.

Bootstrap creates the popover component as a sibling of the target element in the DOM. This means that to add a popover to a link (in our case the username link), the popover will acquire the behaviour of the link, and the end result would be something like:

```html
<a class="user_popup" href="{{ url_for('user', username=post.author.username) }}">
    {{ post.author.username }}
    <div> <!--popover element will go here--> </div>
</a>
```
This is not desirable. To avoid it, we can wrap the `<a>` element inside a `<span>` element then associate the hover event to the `<span>` element.

```html
<span class="user_popup">
    <a class="user_popup" href="{{ url_for('user', username=post.author.username) }}">
        {{ post.author.username }}
    </a>
    <div> <!--popover element will go here--> </div>
</span>
```
The `<div>` and `<span>` elements are invisible, and therefore, great for our use case. Let us restructure our `_post.html` template to use the `<span>` element.

`app/templates/_post.html`: Use the `<span>` element
```html
<!-- Previous code -->

<span class="user_popup">
    <a href="{{ url_for('user', username=post.author.username) }}">
        {{ post.author.username }}
    </a>
</span>

<!-- Previous code -->
```

## Hover Event

Using JQuery, the hover event can be attached to any HTML element by calling the `element.hover(handlerIn, handlerOut)` function. The two arguments are functions that will be invoked when the mouse pointer enters and leaves the element.

`app/templates/base.html`: Hover event
```html
{% block scripts %}
    {{ super() }}
    {{ moment.include_moment() }}
    <script>
        $(function() {
            $('.user_popup').hover(
                function(event) {
                    // mouse enter
                    var elem = $(event.currentTarget);
                },
                function(event) {
                    // mouse leave
                    var elem = $(event.currentTarget);
                }
            )
        });
    </script>
{% endblock %}
```
The `event` argument is the event object containing information about the event. The `event.currentTarget` is the element that triggered the event. So, our event here would be the profile popup.

A browser will _immediately_ dispatch the hover event by invoking the `handlerIn` function as soon as the mouse pointer enters the affected area. What we would like to see is a slight delay when the mouse pointer briefly passes over the element but does not stay on it, meaning there will be no need to flash a popup.

`app/templates/base.html`: Delay the popup
```js
$(function() {
    var timer = null;
    $('.user_popup').hover(
        function(event){
            // mouse enter
            var elem = $(event.currentTarget);
            timer = setTimeout(function() {
                timer = null;
                // popup logic will go here
            }, 500);
        },
        function(event){
            // mouse leave
            var elem = $(event.currentTarget);
            if (timer) {
                clearTimeout(timer);
                timer = null;
            }
        }
    )
});
```

`setTimeout()` takes two arguments, a function and time in milliseconds. This function will be invoked after the specified time has elapsed, which is half a second. In a subsequent section below, we will use this function to display the popover.

We have stored the `timer` object in a `timer` variable defined outside the `hover()` function. This is to make it available and accessible to the `handlerOut` function as well. Why should a timer be added to the `handlerOut` function? The reason is that we do not want the popover component to be displayed when the mouse pointer is over the element for a shorter period of time, which is half a second for now. Once the timer has elapsed, the `handlerIn` function will be invoked again.

## Ajax Requests

When using JQuery, the `$.ajax()` function is used to make an asynchronous HTTP request to the server. The request we are going to make is to fetch the user profile information as seen in the URL _/user/username/popup_.

The big challenge here would be to include a username in the request URL. The `elem` variable contains the target element from the hover event. To extract the username, we can parse through the target DOM element's `href` attribute.

```js
elem.first().text().trim()
```
* The `first()` function applied to the DOM node returns its first child.
* The `text()` function applied to the DOM node returns the text content of the node.
* The `trim()` function removes leading and trailing whitespace from the string.

Typically, you would have the `<a>` element as:

```html
<a href="#"><!--- text ---></a>
```
However, since we have broken down our `<a>` element to span three lines, we will need to trim the whitespaces around the text. This is how our username link looks like in the `_post.html` template:

```html
<a href="#">
    <!--- text --->
</a>
```

Now, we can issue a request to the server.

`app/templates/base.html`: Ajax request
```js
$(function() {
    var timer = null;
    var xhr = null;
    $('.user_popup').hover(
        function(event) {
            // mouse enter
            var elem = $(event.currentTarget);
            timer = setTimeout(function() {
                timer = null;
                xhr = $.ajax(
                    '/user/' + elem.first().text().trim() + '/popup').done(
                        function(data) {
                            xhr = null
                            // create and display popover here
                        }
                    );
            }, 500);
        };
        function(event) {
            // mouse leave
            var elem = $(event.currentTarget);
            if (timer) {
                clearTimeout(timer);
                timer = null;
            }
            else if (xhr) {
                xhr.abort();
                xhr = null;
            }
            else {
                // destroy the xhr request
            }
        }
    )
});
```
The `xhr` variable defined in the outer scope is used to store the asynchronous request object from the call to `$.ajax()`. It would have been great to use `url_for()` to generate the URL, but we cannot use it here because we are not in the context of a Flask request.

This call returns a promise. We have added a callback function `done(function)` which will be invoked once the request is completed. The callback function will receive the response data as its argument. If the `xhr` request object already exists, we can use the `abort()` function to abort the request.

## Create Popover Component

Using the `data` argument, we can now create a popover component. 

`app/templates/base.html`: Create popover
```js
function(data) {
    xhr = null;
    elem.popover({
        trigger: 'manual',
        html: true,
        animation: false,
        container: elem,
        content: data
    }).popover('show');
    flask_moment_render_all();
}
```
As you can see, the actual creation of the popover is simple. All that needs to be done is call the `popover()` function on the target element. 

* The `trigger` argument is set to `manual`, meaning that we have to call the `popover('show')` function to display the popover.
* The `html` argument is set to `true` to allow the popover to contain HTML content. 
* The `animation` argument is set to `false` so that the popover appears and disappears more quickly.
* The `container` argument is set to the target element to ensure that the popover is displayed inside the element. 
* The `content` argument is set to the response data.

Strangely enough, we have had to call another `popover('show')` function to make the popoup appear on the page. When `flask-moment` is added to Ajax, the function `flask_moment_render_all()` is called to appropriately render timestamp elements.

## Destroy Popover Component

The `handlerOut` function has the logic to abort the request if it is interrupted by the user moving the mouse pointer away from the element. If none of the above conditions are met, that means that the popover is currently displayed and the user is leaving the target element. A call to `popover('destroy')` will remove and clean up the popover from the DOM.

`app/templates/base.html`: Destroy popover
```js
function(event) {
    // mouse out
    var elem = $(event.currentTarget);
    if (timer) {
        clearTimeout(timer);
        timer = null;
    }
    else if(xhr) {
        xhr.abort();
        xhr = null;
    }
    else {
        elem.popover('destroy');
    }
}
```

![Another popover](/images/flask_popover/another_popover.gif)