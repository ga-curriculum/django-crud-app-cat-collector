<!-- {% raw %} -->
# ![Django CRUD App - Cat Collector - Django Templates](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be able to use Django's Template Language (DTL) to create, manage, and apply dynamic HTML templates.

## Using Django templates

In the previous section, we learned how to respond to HTTP requests using Django's `HttpResponse()` to send back HTML directly as strings, similar to how you might use `res.send()` in Express. Now, we're going to step up our approach by using templates, which allow for more dynamic HTML content.

### Understanding Django's templating engines

Django supports multiple templating engines for rendering HTML:

- **Django Template Language (DTL)**: This is Django's own templating engine, designed to be easy to use and secure. DTL is highly integrated with Django and will be our primary tool in this course.
- **Jinja2**: This is a popular templating engine for Python, known for its speed and flexibility. Jinja2 is inspired by DTL and can be used in future Django projects if you feel like exploring!

For our purposes, we'll stick with DTL, which is pre-configured in all Django projects and offers a robust set of features for typical web development tasks.

### Setting up templates

Django looks for HTML templates in a specific directory structure within each app. To use templates, we need to first set up this structure in our project.

1. **Create a Templates Directory**: Each Django app should have its own `templates` directory where all its templates will be stored. This keeps your project organized and makes templates easy to find.

   Open your terminal and create a `templates` directory inside your `main_app` directory:

   ```bash
   mkdir main_app/templates
   ```

This setup allows Django to automatically find and use templates stored in this directory when rendering responses. By organizing templates this way, you ensure that each component of your application can manage its own presentation layer independently, enhancing modularity and maintainability.

### Create an `about.html` Template

Let’s start with a simple template for the About page.

1. In the terminal, create the template file:

```bash
touch main_app/templates/about.html
```

When creating templates in Django, ensure that your template files have a `.html` file extension. This is standard for HTML files, but when used in Django, these templates can include specific Django Template Language (DTL) syntax that Django will process.

> To ensure that your development environment recognizes and properly highlights DTL syntax within these `.html` files, it is recommended to install the Django extension for Visual Studio Code. This extension enhances syntax highlighting and provides IntelliSense for template tags and filters, improving your coding efficiency. Install the [**Django extension**](https://marketplace.visualstudio.com/items?itemName=batisteo.vscode-django) by following the link and clicking "Install".

1. Open `about.html` and add the boilerplate (`! + tab`) and update the `<title>`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Cat Collector</title>
  </head>
  <body></body>
</html>
```

2. Add a bit of custom markup within the `<body>`:

```html
<h1>About the Cat Collector</h1>
```

3. Now update the `about` view in `views.py` to `render` the `about.html` template instead of sending a string response:

```python
# main_app/views.py

def about(request):
    return render(request, 'about.html')
```

The `render()` shortcut in Django is similar to Express’ `res.render()`, except for the positional `request` arg. Also, the `.html` extension is required. For more on `render()`, check out more in the **[Django docs](https://docs.djangoproject.com/en/5.0/topics/http/shortcuts/#render)**.

Browse to [http://127.0.0.1:8000/about](http://127.0.0.1:8000/about) to see the newly rendered `about.html` template!

# Template inheritance (Partials)

Template inheritance is a powerful feature in Django, allowing you to extend base templates and override specific sections with content specific to individual pages. This approach helps maintain the DRY (Don't Repeat Yourself) principle by avoiding repetitive boilerplate across multiple templates.

### How Template Inheritance Works

- **Extending Templates**: A child template extends a base template using the `{% extends %}` tag. This setup enables a child template to inherit all the html content from a base template.
- **Overriding Blocks**: Within these templates, you can define blocks with `{% block %}` tags. A child template can override these blocks with its own content, replacing the original content from the base template.

This method is similar to using partials in other templating languages like EJS for Express, but offers more flexibility and integration with Django's overall framework. For more details, refer to the official **[Django documentation on template inheritance](https://docs.djangoproject.com/en/5.0/ref/templates/language/#template-inheritance)**.

Let's take a look at an example:

Base : `base.html`

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>
      {% block title %} 
      {% endblock %}
    </title>
  </head>
  <body>
    {% block body %} 
    {% endblock %}
  </body>
</html>
```

Child : `index.html`

```html
{% extends "base.html" %} 
{% block title %} 
  Main Page 
{% endblock %} 
{% block body %}
<h1>Content!</h1>
{% endblock %}
```

This produces the following final code that will be delivered to the client when rendering `index.html`:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Main Page</title>
  </head>
  <body>
    <h1>Content!</h1>
  </body>
</html>
```

## Creating a `base.html` Template

The `base.html` file acts as the foundational layout for your Django application. It includes common elements like the `<head>`, navigation bar, and optional footer that are shared across multiple pages.

1. **Create the Template File**: Begin by creating the `base.html` file in your project's template directory:

   ```bash
   touch main_app/templates/base.html
   ```

2. **Define the Template Structure**: Populate `base.html` with a basic HTML boilerplate and Django template blocks. This structure should include meta tags for responsiveness, a placeholder for the favicon, a title, and designated blocks for adding custom head elements and main content in child templates.

   Initial setup for `base.html`:

   ```html
   <!DOCTYPE html>
   <html lang="en">
     <head>
       <meta charset="UTF-8" />
       <meta http-equiv="X-UA-Compatible" content="IE=edge" />
       <meta name="viewport" content="width=device-width, initial-scale=1.0" />
       <link rel="shortcut icon" type="image/png" href="" />
       <title>Cat Collector</title>
       {% block head %} 
       {% endblock %}
     </head>
     <body>
       <header>
         <div class="header-logo-container">
           <a href="/">
             <img src="" alt="The Cat Collector Logo" />
           </a>
         </div>
         <nav>
           <ul>
             <li><a href="/about">About</a></li>
           </ul>
         </nav>
       </header>
       <main>
        {% block content %} 
        {% endblock %}
        </main>
     </body>
   </html>
   ```

This template will serve as the base for all other pages, allowing you to maintain a consistent look and feel throughout your application while adhering to DRY principles.

> Currently, the image reference in the template may lead to a 404 error because the image file is not yet added. We will address this and add the correct image in the next section.

## Understanding template inheritance and blocks in Django

Template inheritance is a key feature in Django that allows you to build a base "skeleton" template containing elements that are common across multiple pages, such as headers, footers, and common scripts.

The `base.html` template typically includes placeholder blocks that can be overridden by child templates:

```html
{% block head %} {% endblock %}
```

This block is used to inject custom `<head>` content such as CSS links or scripts specific to a child template.

and

```html
{% block content %} {% endblock %}
```

This block is where the main content of each child template is defined.

### Using template tags

Django’s template system utilizes **template tags**, enclosed within `{% %}` delimiters, to inject logic and control flow into templates. The `block` and `endblock` tags define areas in a template that can be overridden by child templates. Learn more about **[template tags in Django's documentation](https://docs.djangoproject.com/en/5.0/ref/templates/builtins/#ref-templates-builtins-tags)**.

### Template inheritance

When a template extends `base.html`, it inherits its entire structure. The child template can then define or override content within the blocks established in the base template.

Replace the entire contents of `about.html` with the following code:

```html
{% extends 'base.html' %} 
{% block content %}
<h1>About the Cat Collector</h1>
<hr />
<p class="page-content">
  I like long romantic walks down the produce aisle. Also, I am currently job
  seeking!
</p>
{% endblock %}
```

If you refresh your browser you will see the new content displayed within the `content` block of `base.html`.
<!-- {% endraw %} -->