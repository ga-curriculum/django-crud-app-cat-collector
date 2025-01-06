# ![Django CRUD App - Cat Collector - Using Django’s ORM in the Terminal](./assets/hero.png)

**Learning objective:** By the end of this lesson, learners will be able to perform CRUD operations using Django's ORM within the Django shell environment, and apply these techniques to manage data effectively in their Django applications.

## Performing CRUD using Django’s ORM

### What’s an ORM?

ORM stands for Object-Relational Mapper. It allows developers to work with databases using Python objects instead of writing SQL directly. ORMs create a bridge between the relational database tables and the Python objects.

Benefits of using Django's ORM:

- Developers can write object-oriented code to create, read, update, and delete (CRUD) data without needing to write complex SQL queries.
- The ORM abstracts away the differences between various SQL databases, allowing you to use the same Python code regardless of the underlying database.
- The ORM can generate optimized SQL queries that even experienced developers might find challenging to write manually.

## Django’s ORM

Django's ORM automatically provides a variety of methods for each Model, making it easy to interact with the database:

- Filtering (querying based on criteria)
- Ordering
- Even accessing the data from related Models!

Django refers to the ORM functions available as its **[Database API](https://docs.djangoproject.com/en/5.0/topics/db/queries/)**. Additional information can also be found in **[Django's Model documentation](https://docs.djangoproject.com/en/5.0/ref/models/)**.

## Performing CRUD in a Python interactive shell

After creating a new Model, you can take it for a test drive using a Python shell that loads the Django environment.

Let's experiment with our new model in the terminal:

```bash
python3 manage.py shell
```

You’re now in a Python shell with your Django environment loaded in your terminal!
Your prompt should look like this:

```bash
>>>
```

Any model you want to work with must be imported just like you would do in the application:

```python
from main_app.models import Cat
```

> 💡 Key Point: The code we use in the Django shell to perform CRUD operations will be very similar to the code we use in our application's views. We'll practice in the shell to get comfortable with the commands, as this will make it easier to apply them in your applications.

To retrieve all the Cat objects, enter this command:

```python
Cat.objects.all()
```

This will return a `<QuerySet []>` containing all the Cat objects in the database. With no cats in our database this QuerySet is currently empty.

### Django model manager

Any time you want to perform query operations on a **Model** to retrieve model objects (rows) from a database table, it is done via a **Manager** object.

By default, Django adds a Manager to every Model class. This Manager is available through the `objects` attribute. For example, `Cat.objects` is the Manager for the `Cat` model.

### **The `<QuerySet>`**

The `<QuerySet []>` returned from a query represents a database query that can be refined by chaining additional methods to it.

When the app needs the data, for example, to iterate over cats, the query will be executed, and the result will be a list-like object that represents a collection of model instances (rows) from the database.

Besides `Cat.objects.all()`, there are many other common ORM operations you can perform. Let's explore!

## Give me a "C"

Here’s how we can `CREATE` an in-memory model (an instance) and then save it to the database.

In the terminal:

```python
c = Cat(name='Biscuit', breed='Sphinx', description='Cuddle monster. Hairless.', age=2)
```

As you can see, we pass the data for the model’s attributes as kwargs.

> 💡 This model does not currently have an `id` because it is not yet saved to the database. Let’s do that next!

```python
c.save()
```

Now we can reference `c` and all of its properties, including `id`:

```python
c.id
```

We’ve created a cat!

## Give me a "R"

If you call `Cat.objects.all()` again you’ll be able to `READ` all `Cat` objects that exist in the database now:

```python
Cat.objects.all()
```

This action should return:

```plaintext
<QuerySet [<Cat: Biscuit>]>
```

We’re reading cats! You should also now be able to see Biscuit in the `psql` interface.

## 🎓 You Do: Create another cat

Create a new `Cat` object with attribute values of your choice. You can use the same terminal variable, like `c`, for convenience unless you need to keep the current object stored in `c`.

Check that your cat was added by using `Cat.objects.all()`.

## Give me a "U"

To `UPDATE` a single attribute value, assign the new value and call `save()`. Start by getting the cat you want to update—in this case, the first one:

```bash
c = Cat.objects.first()
```

Confirm it’s the cat you want:

```python
c
```

This will output something like `<Cat: Biscuit>`. Let’s change Biscuit’s name:

```python
c.name = 'Rubber Biscuit'
```

And save this cat:

```python
c.save()
```

We’ve updated a cat!

## Give me a "D"

Finally we can `DELETE` records easily using the ORM's build in `.delete()` method.

First we need a spare cat:

```bash
c = Cat(name='Pebbles', breed='alley cat', description='smells like old socks', age=7)
c.save()
```

Confirm that a new cat has been added to your collection:

```bash
Cat.objects.all()
```

Now that we have a cat to spare, let's set him free again:

```bash
c.delete()
```

## Read one

We've seen how to use `Cat.objects.all()` to retrieve lists of objects. However, retrieving a **_single_** specific model object from the database, typically by its `id`, is a common operation.

### Using the `get()` method

For fetching a single object, use the `get()` method. For example:

```python
Cat.objects.get(id=1)
```

You can specify multiple conditions by using multiple `field=value` pairs with the `get()` method.

### Handling errors with `get()`

The `get()` method raises an error if no object is found. It's important to handle this error appropriately to prevent your application from crashing. This will come in handy when we start querying for cats in our Views.

_example:_

```python
try:
    cat = Cat.objects.get(id=1)
except Cat.DoesNotExist:
    # Handle the case where the object does not exist
    print("This cat does not exist!)
```

This error handling strategy ensures that your application manages the scenario gracefully when an object isn't found in the database. This syntax might feel familiar, as it's quite similar to using `try-catch` blocks in JavaScript to handle exceptions.

## Filtering (querying) for records

We can use **[objects.filter()](https://docs.djangoproject.com/en/5.0/ref/models/querysets/#filter)** to query a Model’s table for data that matches specific criteria, similar to the `find` method in Mongoose.

For example, this query returns all cats with the name "Rubber Biscuit":

```python
Cat.objects.filter(name='Rubber Biscuit')
```

Using `objects.filter()` and `objects.exclude()` is like writing a `WHERE` clause in SQL.

The Django ORM provides several helpful **[Field lookups](https://docs.djangoproject.com/en/5.0/topics/db/queries/#field-lookups)**.

For example, to query for all cats whose names _contain_ a specific string:

```python
Cat.objects.filter(name__contains='Bis')
```

The SQL equivalent of the above query would be:

```sql
SELECT * FROM main_app_cat WHERE name LIKE '%Bis%';
```

Another example: to find cats that have an age _equal to or less than_ 3:

```python
Cat.objects.filter(age__lte=3)
```

> 🧠 For basic lookups, the format is: `field__lookuptype=value` (using a double underscore).

The SQL equivalent of the above filter operation would be:

```sql
SELECT * FROM main_app_cat WHERE age <= 3;
```

Filters can even be chained!

## Ordering (sorting) querysets

Django's **[order_by](https://docs.djangoproject.com/en/5.0/ref/models/querysets/#order-by)** method allows you to sort query results, similar to SQL's `ORDER BY` clause.

### Sorting in ascending order

To sort cats by name in ascending order:

```python
Cat.objects.order_by('name')
```

### Sorting in descending order

To sort cats by age in descending order:

```python
Cat.objects.order_by('-age')
```

### Accessing specific records

The resulting `<QuerySet>` can be treated like any sequence in Python, allowing you to access specific items or slices:

```python
# Retrieves the oldest cat
oldest_cat = Cat.objects.order_by('-age')[0]
```

### Exiting the shell

When you're done experimenting, you can exit the Django shell by calling the `quit()` method:

```python
quit()
```

Let's return to our Views!
