# ![Django CRUD App - Cat Collector - One-to-Many Relationship](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be able to tktk

# A cat’s got to eat!

In this lesson, we'll explore how to add another model in Django to demonstrate working with **one-to-many** relationships.

We'll use an Entity-Relationship Diagram (ERD) to illustrate this relationship:

![tktk - Hunter, can we get a new erd here](./assets/one-to-many-erd.png)

In this relationship, each cat will have many feedings, and each feeding will belong to a specific cat.

## Adding a new `Feeding` Model

Using the ERD above as a guide, let’s add a new `Feeding` Model (below the current `Cat` Model) in our `models.py`:

```python
# Add new Feeding model below Cat model

class Feeding(models.Model):
  date = models.DateField()
  meal = models.CharField(max_length=1)
```

> Note that we’re going to use just a single-character to represent what meal the feeding is for: ***B***reakfast, ***L***unch or ***D***inner.

## Field.choices

Django has a feature, **[Field.choices](https://docs.djangoproject.com/en/5.0/ref/models/fields/#choices)**, that will make the single-characters more user-friendly by automatically generating a select dropdown in the form using descriptions that we define.

The first step is to define a tuple of 2-tuples. Because we might need to access this tuple within the `Cat` class also, let’s define it above both of the Model classes:

```python
# A tuple of 2-tuples added above our models
MEALS = (
  ('B', 'Breakfast'),
  ('L', 'Lunch'),
  ('D', 'Dinner')
)


class Cat(models.Model):
```

As you can see, the first item in each 2-tuple represents the value that will be stored in the database, e.g. `B`.

The second item represents the human-friendly “display” value, e.g., `Breakfast`.

Now let’s enhance the `meal` field as follows:

```python
class Feeding(models.Model):
  date = models.DateField()
  meal = models.CharField(
    max_length=1,
    # add the 'choices' field option
    choices=MEALS,
    # set the default value for meal to be 'B'
    default=MEALS[0][0]
  )
```

## Add the `__str__` Method

As recommended, we should override the `__str__` method to Models so that they provide more meaningful output when they are printed:

```python
class Feeding(models.Model):
  ...

  def __str__(self):
    # Nice method for obtaining the friendly value of a Field.choice
    return f"{self.get_meal_display()} on {self.date}"
```

Check out the convenient `get_meal_display()` method Django automagically creates to access the human-friendly value of a Field.choice like we have on `meal`.

## Add the Foreign Key

Since a `Feeding` **belongs to** a `Cat`, it must hold the `id` of the cat object it belongs to - it needs a **foreign key**!

Here’s how it’s done - Django style:

```python
class Feeding(models.Model):
  date = models.DateField()
  meal = models.CharField(
		max_length=1,
		choices=MEALS,
		default=MEALS[0][0]
  )
  # Create a cat_id column for each feeding in the database
  cat = models.ForeignKey(Cat, on_delete=models.CASCADE)

  def __str__(self):
    return f"{self.get_meal_display()} on {self.date}"
```

As you can see, the `ForeignKey` field-type is used to create a one-to-many relationship.

- The first argument provides the parent Model, `Cat`.

- In a one-to-many relationship, the `on_delete=models.CASCADE` is required. It ensures that if a `Cat` record is deleted, all of the child `Feedings` will be deleted automatically as well - thus avoiding *orphan* records.

> 🧠 In the database, the column in the feedings table for the FK will actually be called `cat_id` because Django by default appends `_id` to the name of the attribute used in the Model.

## Make and Run the Migration

We changed our `models.py` file, so it’s that time again! Open your terminal and run:

```bash
python3 manage.py makemigrations
```

Then run:

```bash
python3 manage.py migrate
```

## Use the Admin Portal to create some new feedings

Before we start building the user interface (UI) for our new feedings model, let's practice adding a few feedings using the Django admin portal. This will give us some data to display and allow us to create our feedings' index UI first.

### Registering the Model

To use the built-in Django admin portal for CRUD operations, we must first register the model. Update `main_app/admin.py` to include the `Feeding` model:

```python
from django.contrib import admin
# Add Feeding to the import
from .models import Cat, Feeding

admin.site.register(Cat)
# Register the new Feeding model
admin.site.register(Feeding)
```

Now, navigate to `http://127.0.0.1:8000/admin` and click on the **Feeding +Add** link.


![Django Admin UI Add Feeding](./assets/django-admin-add.png)

Notice the select drop-downs for assigning both the Meal and the Cat!

## Custom field labels

Let’s say though, that you would like a less vague label than **Date**.

If you want to use a more descriptive label than **Date**, you can customize field labels directly in the model. Since ***Django models are the single source of truth about your data***, this is where you should add customizations.

In `main_app/models.py`, update the `Feeding` model to include a user-friendly label for the `date` field:

```python
class Feeding(models.Model):
  # The first optional positional argument overrides the label
  date = models.DateField('Feeding date')
  ...
```

After making this change, refresh the admin portal page to see the updated label.

![Feeding Label Update](./assets/update-date-label.png)

These custom labels will also be used in all of Django’s `ModelForms`.

### Add a few feedings

Now, go ahead and add a few feedings for each of your cats using the admin portal. This will populate your database with initial data, which will be useful for building and testing your UI.

## Displaying a Cat’s Feedings

The cat's detail page is the perfect place to display a cat's feedings because it provides a comprehensive view of all the important information about that specific cat in one place. This is similar to how you would see a list of all tasks on a project detail page. By showing feedings on the cat's detail page, users can easily see the feeding history along with other details about the cat, making the information more accessible and organized.

No additional views or templates are necessary. We only need to update the `detail.html` template.

### Update the `detail.html` template

Here's the new content for `detail.html`. Copy and paste this markup, and then we'll review it:

```html
<!-- Existing cat-container above -->
<div class="feedings-toy-container">
  <section class="feedings">
    <div class="subsection-title">
      <h2>Feedings</h2>
      <img 
        src="{% static 'images/cat-cone.svg' %}" 
        alt="An ice cream cone cat"
      />
      <img 
        src="{% static 'images/cat-onigiri.svg' %}"
        alt="A cat as onigiri"
      />
      <img 
        src="{% static 'images/kitty-kabob.svg' %}" 
        alt="A kabob of kittens"
      />
    </div>
    <table>
      <thead>
        <tr>
          <th>Date</th>
          <th>Meal</th>
        </tr>
      </thead>
      <tbody>
        {% for feeding in cat.feeding_set.all %}
        <tr>
          <td>{{feeding.date}}</td>
          <td>{{feeding.get_meal_display}}</td>
        </tr>
        {% endfor %}
      </tbody>
    </table>
  </section>
</div>

{% endblock %}
```

- Here we use a table structure to present the feeding data in an organized way.
- The table body (`<tbody>`) dynamically lists each feeding associated with the cat. This is done through a loop (`{% for feeding in cat.feeding_set.all %}`) that iterates over each feeding related to the cat.
- For each feeding, the date is displayed, and the meal type is shown using Django's `get_meal_display` method, which translates a database value into a more user-friendly format (e.g., converting a single character like 'B' into "Breakfast").

Refresh your browser and select a cat to see their feedings! 

![Feedings UI](./assets/feedings-ui.png)

## Adding new feedings from the Detail page

Next we will improve the user experience by allowing users to add feedings directly from a cat's detail page. This requires integrating a form on the page that can handle the creation of new feedings linked to that specific cat.

In previous sections, we've utilized Django's class-based views to manage create and update operations. These CBVs automatically handle form generation and data submission by using a `ModelForm` tied to a specific model behind the scenes. 

For adding feedings through the cat's detail page, we'll need to add a custom `ModelForm`. This form will allow us to embed feeding-related inputs within a custom `<form>` tag, manage form validation, and save new feeding entries to the database.

## Create the ModelForm for the `Feeding` Model

We could define the `ModelForm` inside of the `models.py` module, but we’re going to follow a best practice of defining it inside of a `forms.py` module instead. Run the following in the terminal:

```bash
touch main_app/forms.py
```

Let’s open it and add this code:

```python
from django import forms
from .models import Feeding

class FeedingForm(forms.ModelForm):
    class Meta:
        model = Feeding
        fields = ['date', 'meal']
```

Note that our custom form inherits from `ModelForm`.

> Many of the attributes in the `Meta` class are in common with CBVs because the CBV was using them behind the scenes to create a ModelForm as previously mentioned.

For more options, check out the **[Django ModelForms documentation](https://docs.djangoproject.com/en/5.0/topics/forms/modelforms/#modelform)**.

## Passing an instance of `FeedingForm`

`FeedingForm` is a class that needs to be instantiated in the `cat_detail` view function so that it can be rendered inside of `detail.html`.

Here’s the updated `cat_detail` view function in `main_app/views.py`:

```python
from .models import Cat
# Import the FeedingForm
from .forms import FeedingForm

...

# update this view function
def cat_detail(request, cat_id):
  cat = Cat.objects.get(id=cat_id)
  # instantiate FeedingForm to be rendered in the template
  feeding_form = FeedingForm()
  return render(request, 'cats/detail.html', {
    # include the cat and feeding_form in the context
    'cat': cat, 'feeding_form': feeding_form
  })
```

`feeding_form` is set to an instance of `FeedingForm` and then it’s passed to `detail.html` just like `cat`.

## Displaying `FeedingForm` inside of **`detail.html`**

Okay, so we’re going to need a form used to submit a new feeding.

We’re going to display a `<form>` at the top of the feedings column in **`detail.html`**.

This is how we can "render" the `ModelForm`’s inputs within `<form>` tags. Add this section in `templates/cats/detail.html` just above the `<table>` for displaying feedings:

```html
  <!-- Add just above the feedings table -->
  <h3>Add a Feeding</h3>
  <form
    method="post" 
    class="subsection-content"
    autocomplete="off"
  >
    {% csrf_token %}
    {{ feeding_form.as_p }}
    <button type="submit" class="btn submit">Add Feeding</button>
  </form>
  ...
```

As before, we need to include the `{% csrf_token %}` for security purposes.

A form’s `action` attribute determines the URL that a form is submitted to. For now, we’ll leave it out and come back to it in a bit.

The `{{ feeding_form.as_p }}` will generate the `<input>` tags wrapped in `<p>` tags for each field we specified in `FeedingForm`.

Let’s see what it looks like - not perfect but it’s a start:

tktk - Hunter this is the image from the notion content
![](./assets/detail-feeding-wireframe.png)

Unfortunately, the *Feeding Date* field is just a basic text input. This is what Django uses by default for `DateField`s. Let's make it a date picker instead.

## Add MCDatepicker

We're going to use [**MCDatepicker**](https://mcdatepicker.netlify.app/) to allow the user to pick a date using a fancy interface. We can use it just by adding the CDN to our head block on the cats detail page! Make sure it's ***above*** your existing CSS file.

```html
{% block head %}
<!--new content-->
<link 
  href="https://cdn.jsdelivr.net/npm/mc-datepicker/dist/mc-calendar.min.css" 
  rel="stylesheet" 
/>
<link rel="stylesheet" href="{% static 'css/mcdp.css' %}">
<script 
  src="https://cdn.jsdelivr.net/npm/mc-datepicker/dist/mc-calendar.min.js"
>
</script>
<!--existing css link-->
<link 
  rel="stylesheet" 
  type="text/css" 
  href="{% static 'css/cats/cat-detail.css' %}"
/>
{% endblock %}
```

Note the stylesheet to target some of the MCDatepicker elements as well. First create it. Run the following in the terminal:

```bash
touch main_app/static/css/mcdp.css
```

Then add in the styling we need (this just resizes things around our root element size)

```css
.mc-display__day {
  font-size:clamp(1.6rem, 8vw, 2rem);
}

.mc-display__date {
  font-size:clamp(8rem, 40vw, 11.2rem);
}

.mc-display__month {
  font-size: clamp(1.9rem, 9vw, 2.4rem);
}

.mc-display__year {
  font-size: clamp(3.8rem,18vw,4.8rem);
}

.mc-select__data--month, .mc-select__data--month span {
  width: 40vw;
  min-width: 8rem;
  max-width: 10rem;
}

.mc-select__data--year, .mc-select__data--year span {
  width: 22vw;
  min-width: 4.8rem;
  max-width: 5.6rem;
}

.mc-select__data {
  font-size: clamp(1.6rem, 8vw, 2rem);
}

.mc-btn--danger,
.mc-btn--success,
.mc-date,
.mc-table__weekday {
  font-size: 1.6rem;
}

.mc-table__weekday {
  padding: .5rem;
}

.mc-picker__footer {
  padding: 1.6rem 0;
}
```

### Using a Date-picker for *Feeding Date*

We need another directory in our `main_app/static` directory to hold some JavaScript. Run the following in the terminal:

```bash
mkdir main_app/static/js
```

And now a file. Run the following in the terminal:

```bash
touch main_app/static/js/cat-detail.js
```

Go ahead and throw a quick `console.log()` in this file as a sanity check and make use of it on your cats detail page.

```html
<link 
  rel="stylesheet" 
  type="text/css" 
  href="{% static 'css/cats/cat-detail.css' %}"
/>
{% comment %} Add this code below {% endcomment %}
<script defer src="{% static 'js/cat-detail.js' %}"></script>
{% comment %} Add this code above {% endcomment %}
{% endblock %}
```

Okay now let’s make the `<input>` for *Feeding Date* a date-picker! MCDatepicker requires us to initialize certain inputs using a bit of JavaScript. We want this JS to run after the elements are in the DOM, so we’ll just defer the loading of the script.

It takes three steps to get the input the way we want it:

- Select the element in the DOM
- Create the element using MCDatepicker library
- When the user clicks on the input open the calendar so that the user can pick a date.

That's right, we're doing a Unit 1 throwback! The good news is though is that we're only 12 lines away from a functioning date picker!

In your `main_app/static/js/cat-detail`:

```jsx
const dateInput = document.getElementById('id_date')

const picker = MCDatepicker.create({
  el: '#id_date',
  dateFormat: 'yyyy-mm-dd',
  closeOnBlur: true,
  selectedDate: new Date(),
})

dateInput.addEventListener("click", (evt) => {
  picker.open()
})
```

As you can see, the `ModelForm` automatically generated an `id` attribute for each `<input>`. Always be sure to use Devtools to figure out how to select elements for styling and functionality.

There are plenty of additional options in addition to the four used above. Learn more [**here**](https://mcdatepicker.netlify.app/docs/configuration).

Refresh and test it out by clicking inside of Feeding date input - you should see a sweet date-picker pop up like this:

tktk - Hunter this is the image from the notion content
![](./assets/calendar-wireframe.png)

There's even a mobile view!

tktk - Hunter this is the image from the notion content
![](./assets/mobile-calendar-wireframe.png)

Now let’s make this form work!

## Add Another `path(...)` to `main_app/urls.py`

Every `Feeding` object needs a `cat_id` that holds the primary key of the cat object that it belongs to.

Therefore, we need to ensure that the route includes a *URL parameter* for capturing the cat’s `id` like we’ve done in other routes.

Okay, let’s add a route to the `urlpatterns` list in **`urls.py`** like this:

```python
urlpatterns = [
	...
  path('cats/<int:cat_id>/add-feeding/', views.add_feeding, name='add-feeding'),
]
```

The above route is basically saying that the `<form>`’s **action** attribute will need to look something like `/cats/2/add-feeding`. Let’s go there now…

## Add the `action` Attribute to the `<form>`

Now that we have a *named URL*, let’s add the `action` attribute to the `<form>` in `detail.html`:

```html
    <h3>Add a Feeding</h3>
    {% comment %} Add the action below! {% endcomment %}
    <form
      action="{% url 'add-feeding' cat.id %}" 
      method="post" 
      class="subsection-content"
			autocomplete="off"
    >
      {% csrf_token %}
      {{ feeding_form.as_p }}
      <button type="submit" class="btn submit">Add Feeding</button>
    </form>
```

Once again, we’re using the better practice of using the `url` template tag to write out the correct the URL for a route.

If the URL requires values for named parameters such as `<int:cat_id>`, the `url` template tag accepts them after the name of the route.

> Note that arguments provided to template tags are always separated using a space character, not a comma.

We won’t be able to refresh and check it out until we add the `views.add_feeding` function…

## Add the `views.add_feeding` View Function to `views.py`

Let’s start by stubbing up an `add_feeding` view function in `main_app/views.py` so that the server will start back up and we can inspect our `<form>`:

```python
...

# add this new function below cats_detail
def add_feeding(request, cat_id):
  pass
```

Using `pass` is a way to define an “empty” Python function.

Refreshing and inspecting the elements using DevTools shows that our `<form>` is looking good:

tktk - Hunter this is the image from the notion content
![](./assets/form-action.png)

Now let’s turn our attention back to the `add_feeding` function.

Here it is:

```python
def add_feeding(request, cat_id):
  # create a ModelForm instance using the data in request.POST
  form = FeedingForm(request.POST)
  # validate the form
  if form.is_valid():
    # don't save the form to the db until it
    # has the cat_id assigned
    new_feeding = form.save(commit=False)
    new_feeding.cat_id = cat_id
    new_feeding.save()
  return redirect('cat-detail', cat_id=cat_id)
```

After ensuring that the form contains valid data, we save the form with the `commit=False` option, which returns an in-memory model object so that we can assign the `cat_id` before actually saving to the database.

Always be sure to `redirect` instead of `render` if data has been changed in the database.

We also need to import `redirect` by adding after `render`:

```python
from django.shortcuts import render, redirect
```

Add a feeding and rejoice!

tktk - Hunter this is the image from the notion content
![](./assets/feeding-added.png)

Well, almost…

Two quick details. First, it would be nice to have the most recent dates displayed at the top.

Yup, *a Model is the single, definitive source of truth…*

The answer lies in adding a `class Meta` with the `ordering` *Model Meta option* within the `Feeding` Model. Let's update it:

```python
class Feeding(models.Model):
  ...
  def __str__(self):
    # Nice method for obtaining the friendly value of a Field.choice
    return f"{self.get_meal_display()} on {self.date}"

  # change the default sort
  class Meta:
    ordering = ['-date']
```

Second, let's show that a cat hasn't been fed when there are no feedings associated with it. It's easier than you think! Let's update the cat detail template:

```html
    <h3>Past Feedings</h3>
    {% if cat.feeding_set.all.count %}
      <table>
        <thead>
          <tr><th>Date</th><th>Meal</th></tr>
        </thead>
        <tbody>
          {% for feeding in cat.feeding_set.all %}
            <tr>
              <td>{{feeding.date}}</td>
              <td>{{feeding.get_meal_display}}</td>
            </tr>
          {% endfor %}
        </tbody>
      </table>
    {% else %}
			<div class="subsection-content">
        <p>{{cat.name}} has never been fed 😔</p>
      </div>
    {% endif %}
```

Feed your cats!
