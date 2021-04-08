# Build a Video Conference Application Using Flask and Twilio

![Ona Ana Full Screen Share](images/video_app/full_screen.gif)

The Covid-19 pandemic has forced many businesses to close shop and ask their employees to work from home. Almost everyone was forced into remote work. The use of video calling applications (such as Zoom and Google Meet among others) rose as more and more people began embracing this new norm. In this article, I will show you how to build similar video calling applications that offer satisfactory levels and quality of features.

## Project Requirements

There are a number of things we need in order to build our project:
* A Twilio account. Create a [free Twilio account](https://www.twilio.com/try-twilio?promo=WNPWrR) now.
* A web browser compatible with the Twilio Programmable Video JavaScript library. Check your browser among [this list](https://www.twilio.com/docs/video/javascript).
* Python 3.6 and above.
* This project makes use of `Ngrok`. Ngrok provides public URLs that redirect to the application. If you do not know what it is or how to use it, refer to the reference section at the end of this article.

Once you have an account with Twilio:
* Click [Console Dashboard](https://www.twilio.com/console), 
* Click [Settings](https://www.twilio.com/console/project/settings) then,
* Click [API Keys](https://www.twilio.com/console/project/api-keys)
* Create your project API Key by clicking on the Red Plus(+) button. You will be provided with API Key SID and API Key Secret. 
* Click 'Create API Key' button to save.

![New API Key](images/video_app/new_api_key.png)

Note that when you save your keys, the API Secret Key will never be shown again. Make sure to save it somewhere else safe because you will need to use it. This is what you will get:

![Actual Video API Key](images/video_app/actual_video_api_keys.png)

Typically, these keys should be kept secret. I not worried showing you these keys because soon I will generate another pair. 

* Additionally, you will need your _Account SID_ from the [Twilio console](https://twilio.com/console)

![Twilio Account SID](images/video_app/twilio_account_SID.png)


Make sure you save these keys somewhere safe. We will need them later in the application.

## Project Dependencies

Before we can begin working with any python package for this project, it is recommended that you install them within an activated virtual environment. Run the command below in your terminal to create and activate one:

```python
$ mkvirtualenv video_app # I am using a virtualenvwrapper

# Output
(video_app)$
```

Virtual environments help us isolate our machine's Operating System from those needed by the many projects we may build. Install the following dependencies in your virtual environment:

* flask
* twilio
* python-dotenv
* pyngrok
* flask-bootstrap
* flask_wtf

Run:

```python
(video_app)$ pip3 install flask pyngrok twilio python-dotenv flask-bootstrap flask-wtf 
```

## Project Structure

Use the terminal commands `mkdir` and `touch` to create the project structure below. For example:

```python
(video_app)$ mkdir video_app # this create an empty directory
(video_app)$ touch video_app/config.py # this creates an empty config file in video_app directory
```

![Project Struture](images/video_app/video_structure.png)

To begin with, we will save all the application's dependencies in `requirements.txt` as follows:

```python
(video_app)$ pip3 freeze > requirements.txt
```

## Page Layout

The `home.html` page will inherit base styles and layout from `base.html` as seen below:

```html
<!--base.html-->

{% extends 'bootstrap/base.html' %}

<!-- Head image goes here -->
{% block head %}
    {{ super() }}
    
    <!-- Google Fonts -->
    <link rel="preconnect" href="https://fonts.gstatic.com">
    <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@300&display=swap" rel="stylesheet">
    
    <!-- Head image -->
    <link rel="icon" type="image/png" href="{{ url_for('static', filename = 'img/video-call.png') }}">
{% endblock %}

<!-- Title Section -->
{% block title %}
    {% if title %}
        Ona Ana | {{ title }}
    {% else %}
        Sample Video Call App
    {% endif %}
{% endblock %}

<!-- Import styles -->
{% block styles %}
    {{ super() }}
    <link rel="stylesheet" type="text/css" href="{{ url_for('static', filename = 'css/styles.css') }}">
{% endblock %}


<!-- Navbar Section -->
{% block navbar %}
<nav class="navbar navbar-default">
    <div class="container">
        <div class="navbar-header">
            <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#bs-example-navbar-collapse-1" aria-expanded="false">
                <span class="sr-only">Toggle navigation</span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
                <span class="icon-bar"></span>
            </button>
            <a class="navbar-brand" href="#">Ona Ana</a>
        </div>
        <div class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">                                   
        </div>
    </div>
</nav>
{% endblock %}

<!-- Blog Content Goes Here -->
{% block content %}
    <div class="container">
        {% block app_content %}
        
        {% endblock %}
    </div>
{% endblock %}

<!-- Scripts Section -->
{% block scripts %}
    {{ super() }}
    
    <!-- Twilio JS -->
    <script src="//media.twiliocdn.com/sdk/js/video/releases/2.3.0/twilio-video.min.js"></script>
    
    <!-- Custom JS -->
    <script type="text/javascript" src=" {{ url_for('static', filename='js/app.js') }} "></script>
{% endblock %}




<!-- home.html -->

{% extends 'base.html' %}

{% block app_content %}
    <div class="row">
        <div class="col-md-12 text-center">
            <h1>Ona Ana Video Conferencing App</h1>
        </div>
    </div>
    <div class="row">
        <div class="col-md-12 my_form">
            <!-- Join/Leave Actions -->
            <form class="form-inline">
                <div class="form-group">
                    <label class="mb-2 mr-sm-2" for="username">Username: </label>
                    <input class="form-control mb2 mr-sm-2" type="text" name="username" id="username">                    
                </div>
                <button class="btn" id="join_leave">Join Call</button>
            </form>
        </div>
    </div>
    <div class="row">
        <div class="col-md-12">
            <!-- Participants Count -->
            <p id="count">You need to join the call</p>
        </div>
    </div>
    <div class="row">
        <div class="col-md-12">
            <!-- Video Feed -->
            <div id="video_container" class="video_container">
                <div id="local" class="participant"><div></div><div>Me</div></div>
                <!-- More participants will be added dynamicly -->
            </div>
        </div>
    </div>
     
{% endblock %}
```

Flask's `url_for` has been used to help generate the correct URL for `styles.css` and `app.js` files. We also need the official release of the _twilio-video.js_ library. All these files are added to the base template and are inherited by the home template using the keyword `extends`.

We will update the style of our application in `styles.css` as seen below:

```css
.video_container {
    margin-top: 20px;
    width: 100%;
    display: flex;
    flex-wrap: wrap;
    text-align: center;
    
}
.participant {
    margin-bottom: 5px;
    margin-right: 5px;
}
.participant div:first-child {
    width: 320px;
    height: 250px;
    background-color: #ddd;
    border: 1px solid black;
}
.participant video {
    width: 100%;
    height: 100%;
}


/* Optional General Styling */
.title, .my_form, #count{
    text-align: center;
    padding-bottom: 20px;
    padding-top: 20px;
}
.btn{
    border: 1px solid #F2F2F2;
    color: black;
}
h1{
    font-size: 50px;
}
body{
    font-family: 'Roboto', sans-serif;

}
```

Flask-bootstrap makes use of the class `container`. So that we do not overide the default styles associated with this class, I have created a new class called `video_container`. This way, there is no conflict. If you have noted, this is the same class I have passed to the video feed `div` in the home template.

`.participant div:first-child` class applies to all first child elements of `div`s that have the class `participant`. We are limiting the size of the video to 320 by 250 pixels. To make this video noticable, we pass a background color to it and also define a border around it.

## Starting the Flask Server

With the templates in place, we need to complete the application so that we can started the flask server. Let us create an application instance and create object of the packages we have imported.

`app/__init__.py: Create application instance`
```python
from flask import Flask
from flask_bootstrap import Bootstrap
from config import Config

app = Flask(__name__)
app.config.from_object(Config)

bootstrap = Bootstrap(app)


def start_ngrok():
    from pyngrok import ngrok

    url = ngrok.connect(5000)
    print('*Tunnel: ', url)


if app.config['START_NGROK']:
    start_ngrok()

from app import routes

```
We have created an instance of `flask-bootstrap` which we installed earlier. We are also setting our application to download and run `ngrok`, a service that provides free public URLs to help access our application, which is currently running on [localhost](http://127.0.0.1:5000/), from another device. Everytime we start th flask server, `ngrok` will also start and generate useful URLs for us.

We have imported configurations to our flask instance but it does not exist yet. Let us define our application's configurations. 

`config.py: Load environment variables`
```python
import os
from dotenv import load_dotenv

load_dotenv()


class Config(object):
    START_NGROK = os.environ.get('START_NGROK') is not None
    SECRET_KEY = os.environ.get("SECRET_KEY") or 'you-will-never-guess'

    TWILIO_ACCOUNT_SID = os.environ.get("TWILIO_ACCOUNT_SID")
    TWILIO_API_KEY_SID = os.environ.get("TWILIO_API_KEY_SID")
    TWILIO_API_KEY_SECRET = os.environ.get("TWILIO_API_KEY_SECRET")

```

Remember at the beginning of this article we saved some keys that we promised to use. `python-dotenv` package will be used to load these keys from environent variables. 

Let us update `.env` file we created ealier to hold these secret keys.

`.env: Secret application configurations`
```python
TWILIO_ACCOUNT_SID='add-your-account-sid'
TWILIO_API_KEY_SID='add-your-api-key-sid'
TWILIO_API_KEY_SECRET='add-your-api-secret'
```

It is good practice to use `os.environ.get()` method to load these keys rather than passing them directly to variables in the `config.py` file. These keys should not be committed to version control. To guide users who might be interested in our project from a version control site like GitHub, we will update the `.env-template` file to show what keys they will need before running this application.


`.env-template: Show needed configurations`
```python
TWILIO_ACCOUNT_SID=
TWILIO_API_KEY_SID=
TWILIO_API_KEY_SECRET=
```

Our application instance imports the routes module. We need to update this module to render the home page.

`app/routes.py`
```python
from app import app
from flask import render_template


@app.route('/')
@app.route('/home')
def home():
    return render_template('home.html', title='Home')

```

Flask will need to know the application's entry point. So, in `app.py`, we will add this code:

`app.py: Application entry point`
```python
from app import app
```

Just before we start the flask server, let us pass Flask's environment variables:

`.flaskenv: Pass Flask's environment variables`
```python
FLASK_APP=app.py
FLASK_ENV=development
FLASK_DEBUG=True
START_NGROK=1
```

We are on a development server, and it is okay to enable Flask's auto-reload feature that is quite useful when debugging. Now, we can start the flask server from the terminal"

```python
(video_app)$ flask run
```

This is what we have at the moment:

![Initial Look of Video App](images/video_app/initial_app_look.png)

## Display Video Feed

First, let us display our own video feed. We will add this to the first child of the div that has class `local`.

`js/app.js: Display own video feed`

```js
function addLocalVideo() {
    Twilio.Video.createLocalVideoTrack().then(track => {
        let video = document.getElementById('local').firstChild;
        video.appendChild(track.attach());
    });
};

addLocalVideo();
```

You should be able to see your own video feed after calling the `addLocalVideo` function.

![Own Video Feed](images/video_app/own_video_feed.png)

The video gets attached to the first `div` whose parent has the ID `local`. We have set this `div` to be empty (if you look carefully at the home template). This is because the video feed will be attached here.


## Generate Access Token

The fact that you have built an application which can access a user's camera and microphone raises the issue of security. Thankfully, Twilio takes security seriously. Before any user can join the call, we need our application to verify that the user is allowed and the application must generate an access token for them. This is done in the Python server. The `.env` secrets will be required.

`app/routes.py: Generate access token`

```python
from twilio.jwt.access_token import AccessToken
from twilio.jwt.access_token.grants import VideoGrant
from flask import request, abort

@app.route('/login', methods=['POST'])
def login():
    username = request.get_json(force=True).get('username')
    if not username:
        abort(401)
    token = AccessToken(
        app.config['TWILIO_ACCOUNT_SID'],
        app.config['TWILIO_API_KEY_SID'],
        app.config['TWILIO_API_KEY_SECRET'],
        identity=username
    )
    token.add_grant(VideoGrant(room='My Room'))
    return {'token': token.to_jwt().decode()}

```

Our current form of authentication will only involve the use of `username`, but more robust applications will require better user authentication. 

Basically, we are getting the user's identity in a JSON payload. We check that the `username` is not empty, otherwise we abort and return a [`401 Unauthorized`](https://httpstatuses.com/401) error. The `AccessToken` helper class is used to generate the token before attaching a video grant to a room called `My Room`. Read more from the [Twilio Programmable Video JavaScript SDK documentation](https://www.twilio.com/docs/video/javascript-getting-started). The token is returned as a JSON payload in this format:

```python
{
    'token': 'token-will-go-here'
}
```

An application can work with more than one video room and decide which video room a user can enter.

## Connect/Disconnect Using the Form Button

What does the 'Join Call' button really do?
* It can be used to connect a user to the video call
* It can also be used to disconnect a user from the video call

Let us see how such an event can be handled in JavaScript;

`app/js/app.js: Button Event Handler`
```js
let connected = false;
let room;
const usernameInput = document.getElementById('username');
const button = document.getElementById('join_leave');
const container = document.getElementById('video_container')
const count = document.getElementById('count');

function addLocalVideo() { /* no changes */ }

function connectButtonHandler(event) {
    event.preventDefault();
    if (!connected) {
        let username = usernameInput.value;
        if (!username) {
            alert('Enter your name before connecting');
            return;
        }
        button.disabled = true;
        button.innerHTML = 'Connecting...';
        connect(username).then(() => {
            button.innerHTML = 'Leave call';
            button.disabled = false;
        }).catch(() => {
            alert('Connection failed. Is the backend running?');
            button.innerHTML = 'Join call';
            button.disabled = false;    
        });
    }
    else {
        disconnect();
        button.innerHTML = 'Join call';
        connected = false;
    }
};

addLocalVideo();
button.addEventListener('click', connectButtonHandler);
```

We begin by defining a few global varibles which are used to access elements in our home page. For the button, we will work with `button`. If a user is not connected, we get their useranme and store it in the variable `username`. We will pass this variable to the `connect` function. We make sure that the `username` variable is not empty before initiating a connection. 

Just before the conncetion is successful, we rename the button to 'Connecting' and effectively disable it during this process. Once connected, we enable the button to make it active one more time. We also rename it to 'Leave call'. If the connection fails, we provide a fallback using the `catch` function.

In the case where a user is already connected, we disconnect him by calling the function `disconnect` and renaming the button to 'Join call'.

## Connect to A Video Room

Here is where we define the `connect` function is seen the click event `connectButtonHandler`. Two sequential operations have to take place before a successful connection:

* Request an access token by calling the we server
* Calling the Twilio Video Library with the token received to make the connection

`app/js/app.js: Connecting to a video room`

```js
function connect(username) {
    let promise = new Promise((resolve, reject) => {
        // get token from the back end
        fetch('/login', {
            method: 'POST',
            body: JSON.stringify({'username': username})
        }).then(res => res.json()).then(data => {
            // join the video call
            return Twilio.Video.connect(data.token);
        }).then(_room => {
            room = _room;
            room.participants.forEach(participantConnected);
            room.on('participantConnected', participantConnected);
            room.on('participantDisconnected', participantDisconnected);
            connected = true;
            updateParticipantCount();
            resolve();
        }).catch(() => {
            reject();
        });
    });
    return promise;
};
```

A [promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) constructor is returned when the `connect` function is called, which is controlled by the `resolve()` and the `reject()` functions. A call to `resolve()` triggers a success callback whereas `reject()` does the same for the error callback.

This is how the `connect()` function is structured:

```js
connect() { promise = { fetch(/*token*/).then(/*join video call*/).then(/*enter room*/).catch(/*error*/); }; return promise; };
```

Two steps create a connection logic. 

* First, the `fetch()` function is used to send a request to the `/login` route and return a promise. After a successful call, we decode the JSON payload into the `data` variable and then call `connect()` from the Twilio video library by passing the newly acquired token.
* Secondly, if the `fetch()` fails, we call `reject()`on our promise.

Once we are connected and in a video call room, we need to arrange the `video_container` part of the page to reflect this. We have passed `_room` argument to represent the the global `room` variable, therefore making it available to the rest of the application.

We find out and list the participants already connected to the room, encapsulated in `participantConnected` function. There are two events that are likely to happen inside a video call room:

* Another participant can join the call
* A participant can disconnect from the room

We, therefore make calls to the `participantConnected` and `participantDisconnected` functions to handle these events. The final action would be to count the number of participants and update the `<p>` element based on the length of `room.participants` array. 

```js
function updateParticipantCount() {
    if (!connected)
        count.innerHTML = 'Disconnected';
    else
        count.innerHTML = (room.participants.size + 1) + 'participants online';
};
```

The total length of `room.participants` includes ourselves. 

## Connecting and Disconnecting Participants

The function `participantConnect` has been used to connect a user to a video call room. So what exactly happens?

Technically, this function creates a new `div` inside the `video_container` element just like the `local` element.

```js
function participantConnected(participant) {
    let participantDiv = document.createElement('div');
    participantDiv.setAttribute('id', participant.sid);
    participantDiv.setAttribute('class', 'participant');

    let tracksDiv = document.createElement('div');
    participantDiv.appendChild(tracksDiv);

    let labelDiv = document.createElement('div');
    labelDiv.innerHTML = participant.identity;
    participantDiv.appendChild(labelDiv);

    container.appendChild(participantDiv);

    participant.tracks.forEach(publication => {
        if (publication.isSubscribed)
            trackSubscribed(tracksDiv, publication.track);
    });
    participant.on('trackSubscribed', track => trackSubscribed(tracksDiv, track));
    participant.on('trackUnsubscribed', trackUnsubscribed);

    updateParticipantCount();
};

function participantDisconnected(participant) {
    document.getElementById(participant.sid).remove();
    updateParticipantCount();
};

function trackSubscribed(div, track) {
    div.appendChild(track.attach());
};

function trackUnsubscribed(track) {
    track.detach().forEach(element => element.remove());
};

```

This will create a dynamic HTML structure such as this:

```html
<div id="{{ participant.sid }}" class="participant">
    <div></div>  <!-- video and audio tracks will be attached to this div -->
    <div>{{ participant.name }}</div>
</div>
```

`participantConnected()` function gets a [`participant`](https://media.twiliocdn.com/sdk/js/video/releases/2.3.0/docs/Participant.html#toc0__anchor) object from the Twilio video library. As a result, we are able to obtain `participant.id` and `participant.identity` objects which are unique session identifiers and name. The `identity` was used in the Python token generation function in our routes module.

After creating the HTML structure, we now attach the video and audio tracks tot he `tracksDiv` element. We follow the guidance shown in the [library's documentation](https://media.twiliocdn.com/sdk/js/video/releases/2.3.0/docs/) to attach looped tracks to which we are subscribed. The auxillary `trackSubscribed()` function handles the actual track attachment. 

## Disconnect From the Chat Room

If a user can `connect()` to a video call room, then they can possible `disconnect()`. We can achieve this by removing the child elements of `video_container` except our own local video stream.

```js
function disconnect() {
    room.disconnect();
    while (container.lastChild.id != 'local')
        container.removeChild(container.lastChild);
    button.innerHTML = 'Join call';
    connected = false;
    updateParticipantCount();
};
```

We remove all the children of the `video_container` element starting from the last up until we come to `local` div, which was non-dynamically created.

## Start Your Flask Server

Run the command `flask run` in your terminal and you should be able to access your localhost. You can connect to your application from another device using `ngrok`. You have probably noticed something such as `* Tunnel URL: NgrokTunnel: "http://4209c9af6d43.ngrok.io" -> "http://localhost:5000"` in your terminal as soon as your flask server stated. This is a free public URL that `ngrok` provides to allow for access to your video service. 

Most web browsers do not allow `http//:` when working with video calls. Therefore we need to generate a secure public URL that redirects to the application.

Open a new terminal window (consider using [byobu](https://www.byobu.org/)) and run `ngrok`:

```python
(video_app)$ ngrok http 5000

# Output

ngrok by @inconshreveable                                                  (Ctrl+C to quit)
                                                                                           
Session Status                online                                                       
Session Expires               1 hour, 59 minutes                                           
Version                       2.3.35                                                       
Region                        United States (us)                                           
Web Interface                 http://127.0.0.1:4041                                        
Forwarding                    http://13c58a13e6c2.ngrok.io -> http://localhost:5000        
Forwarding                    https://13c58a13e6c2.ngrok.io -> http://localhost:5000       
                                                                                           
Connections                   ttl     opn     rt1     rt5     p50     p90                  
                              0       0       0.00    0.00    0.00    0.00
```

Note the lines beginning with 'Forwarding'. These show the public URLs that ngrok uses to redirect requests into our service. We will make use of the `https//:` URL.

Copy the `https//:` url to another device, say your smartphone or another computer. You should be able to access the application. Click the 'Join Call' button to test it out.