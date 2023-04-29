# Getting Started With Tailwind CSS In Flask

![Tailwindcss](/images/tailwindcss/getting_started/tailwindcss.png)

[Tailwind CSS](https://tailwindcss.com/) is a utility-first CSS framework. To better understand it, we can contrast it with other popular frameworks such as [Bootstrap](https://getbootstrap.com/) or [Materialize CSS](https://materializecss.com/). These other frameworks come with predefined components, but Tailwind CSS does not. Instead, it operates on a lower level and provides us with a set of CSS helper classes. Using these classes, we can rapidly create a custom (unopinionated) design with ease.

> Tailwind CSS is used to rapidly build modern websites without ever leaving your HTML.


### Table of Contents

In this article, I am going to show you how to quickly set up Tailwind CSS to give your Flask website a distinctive look and feel. There are two approaches we shall see:

- [Adding Tailwind CSS Link In The `head` Element](#quickly-mess-around-with-tailwind-css)
- [Recommended Way To Set Up Tailwind CSS](#recommended-way-to-set-up-tailwind-css)
- [Sample Flask Project Using Tailwind CSS](#sample-flask-project-using-tailwind-css) (bonus)


## Quickly Mess Around With Tailwind CSS

The easiest way to start messing around with Tailwind CSS is to include the link below in your HTML file:

```html
<head>
    <link href="https://unpkg.com/tailwindcss@^2/dist/tailwind.min.css" rel="stylesheet">
</head>
```

Let us begin by creating a simple HTML file called `index.html` in our Flask app following this simple project structure:

```
project_folder
    | --- main.py
    | --- .flaskenv
    | --- requirements.txt
    | --- app/
           | --- routes.py
           | --- __init__.py
           | --- templates/
                    | --- index.html
           | --- static/
                    | --- css/
                            | --- main.css
```

Do you have to go this far to get started with Tailwind CSS, creating a full Flask project? No, not really! You can easily create a single HTML file (`index.html`) and go live with it. However, this is an attempt to show how you can do the same in Flask. This file can be created in the terminal using:

```python
$ app/templates/touch index.html
```

We can then update it as below:

```html
<!-- index.html -->


<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Demo Tailwind CSS</title>
    <link href="https://unpkg.com/tailwindcss@^2/dist/tailwind.min.css" rel="stylesheet">
</head>
<body>
    <h1>Tailwind CSS Demo</h1>
    <p>Learn how to use Tailwind CSS</p>
    <button>Learn More</button>
</body>
</html>
```

You will have this displayed.

![Getting started](/images/tailwindcss/getting_started/getting_started.png)

Nothing special, right? Not really. Notice that the font size of the `<h1>` and the `<p>` tags are all the same! This is not a mistake but by design. Tailwind CSS resets all the styles to their most basic level so that all elements begin from a well-known state. Typically, you would need to use [Eric Meyer's Reset CSS](https://meyerweb.com/eric/tools/css/reset/) to achieve the same. We don't have to do the same in Tailwind CSS.

Let us start applying some styles the Tailwind CSS way:

```html
<body>
    <div class="container border border-red-500">
        <h1>Tailwind CSS Demo</h1>
        <p>Learn how to use Tailwind CSS</p>
        <button>Learn More</button>
    </div>
</body>
```

I begin by wrapping everything in a `div` with a class `container`. This class is similar to what you'd use in other frameworks. Besides the `container`, I have added a red border on the `div` element whose weight is 500.

![Container](/images/tailwindcss/getting_started/container.png)

The page's content is misaligned. To fix this, we can use the `mx-auto` to horizontally add a margin to our `container`.

```html
<div class="container mx-auto border border-red-500">
    ...
</div>
```

![Center horizontally](/images/tailwindcss/getting_started/center_horizontally.png)

This is not a Tailwind CSS style, but a standard CSS technique relative to its parent. Below, let us try to add some margin to the top and bottom of our `div` element.

```html
<div class="container mx-auto my-4 border border-red-500">
    ...
</div>
```

![Vertical margins](/images/tailwindcss/getting_started/margin_top_bottom.png)


The size of the margin is 1/4 rem units. A class of `m-1` sets the margin to `0.25rem` while `m-4` sets it to `1rem`. If you are coming across the `rem` unit for the first time, it is important to note that this measurement is based on the font size of the `<html>` element, which is the root element. For most browsers, the default value is `16px.` There is also the `em` unit of measurement. `em` values are relative to the closest parent element. 

To better understand the `rem` unit, consider this:

```css
/* Root font-size on the document level */
html {
  font-size: 20px;
}

@media (max-width: 900px) {
  html { font-size: 16px; }
}

@media (max-width: 400px) {
  html { font-size: 12px; }
}

/* Type will scale with document */
h1 {
  font-size: 2.6rem;
}
```

In the code above, we have set a different root size depending on the viewport of a device. The `h1` will scale as follows:

- 20px * 2.6
- 16px * 2.6
- 12px * 2.6

Note that it is always a bad idea to set the font size to `px` primarily because it overrides a user's browser setting. Ideally, you'd want to use `%` or avoid setting the root size explicitly. 

Back to Tailwind CSS, we can optionally provide more customization using the following classes:

```css
m-{size} - margin around all borders

mx-{size} - horizontal margin

my-{size} - vertical margin

mt-{size} - top margin

mb-{size} - bottom margin

ml-{size} - left margin

mr-{size} - right margin
```

### Style A Button

Now that we have a basic understanding of Tailwind CSS, let us see how we can achieve a more interesting custom style for a button.

```html
<div class="container mx-auto my-4 p-4 border border-red-500">
    ...
    <button class="my-2 px-4 py-2 border-2 border-yellow-500 rounded-md bg-gradient-to-b from-yellow-600 to-yellow-400 text-white shadow-lg">Learn More</button>
</div>
```

![Interesting btn](/images/tailwindcss/getting_started/interesting_btn.png)

The `bg-gradient-to-b` class makes the background of the element a gradient from top to bottom (the `to-b` part determines its direction), and the `from-` and `to-` classes set what colors to use.

If we'd want to add some hover effects, then we can do so as follows:

```html
<div class="container mx-auto my-4 p-4 border border-red-500">
    ...
    <button class="my-2 px-4 py-2
                    border-2 border-yellow-500 rounded-md
                    bg-gradient-to-b from-yellow-600 to-yellow-400
                    hover:from-yellow-500 hover:to-yellow-300 
                    text-white shadow-lg">Learn More</button>
</div>
```

The `hover:` prefix is one of the state modifiers used to apply conditional styles. Now, if you hover your mouse on top of the button, you will notice a slight change in style.

What you will notice is that the `html` file quickly gets long. To help with visibility, I have broken the classes as seen above to show all the styles in one glance of the eye. It is one of the criticisms of using Tailwind CSS. Arguably, it is no different than using inline styles. 

```html
<!-- Inline HTML style -->
<p style="color:blue; font-size:46px;">
    I'm a big, blue, <strong>strong</strong> paragraph
</p>
```

To manage this growing list of classes, you may choose to import or include a button-specific HTML file that defines a button. This way, you do not need to write the styles multiple times.

```html
<!-- Create a file called _button.html -->

<button class="my-2 px-4 py-2
               border-2 border-yellow-500 rounded-md
               bg-gradient-to-b from-yellow-600 to-yellow-400
               hover:from-yellow-500 hover:to-yellow-300 
               text-white shadow-lg">{{ label }}</button>
```

In your `index.html` file, include the file as follows:

```html
<!-- Include this file in your index.html file -->

<div class="container mx-auto my-4 p-4 border border-red-500">
    ...
    {% include '_button.html` %}
</div
```


## Recommended Way To Set Up Tailwind CSS

The approach we have used above can quickly see that the file becomes big. In production, this is not recommended. You do not want to have an unusually large file in your production deployment of a website. Tailwind CSS can be installed as a plugin for the [PostCSS](https://tailwindcss.com/docs/installation/using-postcss) CSS transformation tool used to optimize the CSS file to include only classes we'd want to use. These are the things we will do to install Tailwind CSS in the recommended way:

- [Install Node.js and `npm` (using APT)](#install-nodejs-and-npm-using-apt)
- [Install Node.js and `npm` (from source)](#install-nodejs-and-npm-from-source)
- [Update Flask Project Structure](#update-flask-project-structure)
- [Install Tailwind CSS in Flask](#install-tailwind-css-in-flask)
- [Configure PostCSS](#configure-postcss)
- [Configure Tailwind CSS](#configure-tailwind-css)
- [Start the Build Process](#start-the-build-process)
- [Simplied Build Process](#simplied-build-process)


### Install Node.js and npm (using APT)

To install Tailwind CSS in your Ubuntu machine, you will need to use `npm` or `yarn`, depending on what you have in your system. NPM comes in conjunction with Node.js, so ensure you have that installed too if you don't have it.

```python
$ sudo apt update
$ sudo apt install nodejs npm
```

Once done, you can check the versions available in your system by running the commands below:

```python
$ npm -v && nodejs -v

# Output
9.3.1
v19.5.0
```

### Install Node.js and npm (from source)

Alternatively, you may want to install them from the source. Run the following command as a user with `sudo` privileges to download and execute the NodeSource installation script :

```python
$ curl -sL https://deb.nodesource.com/setup_19.x | sudo -E bash -
```

Feel free to change the version to one you want (`setup_19.x` to `setup_18.x`). The script will add the NodeSource signing key to your system, create an apt repository file, install all necessary packages, and refresh the apt cache. Once the NodeSource repository is enabled, install Node.js and `npm`:

```python
$ sudo apt install nodejs
```

Want to compile native addons from `npm`? Install the development tools by running:

```python
$ sudo apt install build-essential
```

### Update Flask Project Structure

Given our [earlier structure](#quickly-mess-around-with-tailwind-css), we are going to make a few changes to incorporate Tailwind CSS.


```
project_folder
    | --- main.py
    | --- .flaskenv
    | requirements.txt
    | --- app/
           | --- routes.py
           | --- __init__.py
           | --- templates/
                    | --- index.html
           | --- static/
                    | --- css/
                            | --- main.css
                    | --- src/
                            | --- styles.css
```

The only change we have made here is to add a `src/` sub-folder to the `static` directory. We then need to update `src/styles.css` with the following:

```css
/* styles.css */

@tailwind base;
@tailwind components;
@tailwind utilities;
```

The `styles.css` file contains preprocessor directives which essentially pass a lot of utility CSS classes during compile time. The resulting CSS from this process is what we shall use to link to our flask templates for styling. The `css/main.css` is the dumpsite of the process. This is where Tailwind CSS utility classes will go.


### Install Tailwind CSS in Flask

In the terminal, let us run:

```python
$ npm install -d tailwindcss postcss autoprefixer
```

This installs `tailwindcss` and its peer dependencies using `npm`. The project structure will be modified as below:

```python
project_folder
    | --- node_modules/      # < --- new
    | --- package-lock.json  # < --- new
    | --- package.json       # < --- new
    | --- main.py
    | --- .flaskenv
    | --- requirements.txt
    | --- app/    
```

There is a new folder called **node_modules** that has been added. `package-lock.json` and `package.json` files have also been added to the root folder.


### Configure PostCSS

To ensure that PostCSS plugin works, we will need to create a new file in the root directory called `postcss.config.js`:

```python
$ touch postcss.config.js
```

Then, update this new file with the following:

```js
// postcss.config.js

module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  }
}
```

Your final project structure should be as follows:

```python
project_folder
    | --- node_modules/     
    | --- package-lock.json 
    | --- package.json      
    | --- postcss.config.js  # < --- new
    | --- main.py
    | --- .flaskenv
    | --- requirements.txt
    | --- app/    
```

Remember, [PostCSS](https://postcss.org/) is used to build optimized CSS file for a website that only includes the classes that we want to use.


### Configure Tailwind CSS

Next, we need to create a `tailwindcss` configuration file. Run the command below in your root terminal:

```python
$ npx tailwindcss init
```

`tailwind.config.js` file will be added to the root folder of the project. We need to configure the template files here:

```js
// tailwind.config.js

/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "app/templates/**/*.html"
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}

```

We have added the path to our templates within the `content` list following the [glob](https://en.wikipedia.org/wiki/Glob_(programming)) pattern.

- `**` has been used to match zero or more directories found in `templates`
- `*` has been used to match any file with a `.html` extension.


### Start the Build Process

In the project's root terminal, let us run this:

```python
$ npx tailwindcss -i app/static/src/styles.css -o app/static/css/main.css --watch

# Output

Rebuilding...

Done in 258ms.
```

This generates the preprocessors directives. In the event you have an error in your tailwind configuration files, you will see a soft warning shown in the terminal. Follow it to rectify the error. Above, the command runs without any errors.


### Simplified Build Process

Now, we will need to run the `npx tailwindcss - i ...` command (shown above) everytime we add CSS to our HTML files to see the changes. The best practice here would be to make the above command a script that will rebuild the CSS without the need to run the CSS build process code every time.

```json
// package.json

{
  "devDependencies": {
    "autoprefixer": "^10.4.13",
    "postcss": "^8.4.21",
    "tailwindcss": "^3.2.4"
  },
  "scripts": {
    "create-css": "npx tailwindcss - app/static/src/styles.css -o app/static/css/main.css --watch"
  }
}

```

Now, in the terminal, all I need to do is to run:

```python
$ npm run create-css
```

This command eliminates the need to run the CSS build process every time after changing the code in an HTML file.


## Sample Flask Project Using Tailwind CSS

I will begin by creating a base template with styles I want to be applied across all templates:

```html
<!-- templates/base.html -->

<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">

        <!-- The title of our application is defined here -->
        {% block title %}
            {% if title %}
                <title>{{ title }} - Tailwind</title>
            {% else %}
                <title>Tailwind Demo</title>
            {% endif %}
        {% endblock %}

        <!-- Link all style files here -->
        {% block head %}
        <link rel="stylesheet" type="text/css" href="{{ url_for('static', filename='css/main.css') }}">
        {% endblock %}

    </head>
    <body>
        <!-- Contents of all our pages will go here -->
        {% block content %}
        <div class="container mx-auto">
            {% block app_content %}{% endblock %}
        </div>
        {% endblock %}

        <!-- All scripts will go here -->
        {% block scripts %}

        {% endblock %}
    </body>
</html>

```

A departure from what we did earlier, by adding a `<link>` element linked to Tailwind CSS's style file, here we are using our custom `main.css` file containing dumped styles from the preprocessor directives. This file (`index.html`) will act as a base file, a parent, whose children shall inherit from.

```html
<!-- templates/index.html -->

{% extends 'base.html' %}

{% block app_content %}
<section class="text-gray-600 body-font">
    <div class="container px-5 py-24 mx-auto">
      <h1 class="text-3xl font-medium title-font text-gray-900 mb-12 text-center">Testimonials</h1>
      <div class="flex flex-wrap -m-4">
        <div class="p-4 md:w-1/2 w-full">
          <div class="h-full bg-gray-100 p-8 rounded">
            <p class="leading-relaxed mb-6">Synth chartreuse iPhone lomo cray raw denim brunch everyday carry neutra before they sold out fixie 90's microdosing. Tacos pinterest fanny pack venmo, post-ironic heirloom try-hard pabst authentic iceland.</p>
            <a class="inline-flex items-center">
              <img alt="testimonial" src="https://dummyimage.com/106x106" class="w-12 h-12 rounded-full flex-shrink-0 object-cover object-center">
              <span class="flex-grow flex flex-col pl-4">
                <span class="title-font font-medium text-gray-900">Muthoni Gitau</span>
                <span class="text-gray-500 text-sm text-transform: uppercase">Python Engineer</span>
              </span>
            </a>
          </div>
        </div>
        <div class="p-4 md:w-1/2 w-full">
          <div class="h-full bg-gray-100 p-8 rounded">
            <p class="leading-relaxed mb-6">Synth chartreuse iPhone lomo cray raw denim brunch everyday carry neutra before they sold out fixie 90's microdosing. Tacos pinterest fanny pack venmo, post-ironic heirloom try-hard pabst authentic iceland.</p>
            <a class="inline-flex items-center">
              <img alt="testimonial" src="https://dummyimage.com/107x107" class="w-12 h-12 rounded-full flex-shrink-0 object-cover object-center">
              <span class="flex-grow flex flex-col pl-4">
                <span class="title-font font-medium text-gray-900">Wangari Njeri</span>
                <span class="text-gray-500 text-sm text-transform: uppercase">Designer</span>
              </span>
            </a>
          </div>
        </div>
      </div>
    </div>
  </section>
{% endblock %}
```

`index.html` inherits the `base.html` file using the keyword `extends`. It then goes ahead to define its template. Below, see a very responsive page built using Tailwind CSS in Flask.

![Tailwind flask](/images/tailwindcss/getting_started/tailwind_flask.gif)