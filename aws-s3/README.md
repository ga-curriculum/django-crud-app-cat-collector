# ![Django CRUD App - Cat Collector - tktk Microlesson Name](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will be able to tktk

tktk - Hunter, there are so many screenshots on this page so I'm not going to highlight them all. They are all from the notion content and are every couple of lines.

# Intro

Many applications we develop benefit from the ability of its users being able to upload and display images.

For example, an application that tracks a user’s hikes would allow the user to upload photos of those hikes. Or better yet, our Cat Collector app can allow images to be uploaded for a cat!

This lesson will cover how to add this ability to your Django project should you choose to.

## Ready the Starter Code

This lesson’s starter code picks up from the [Authentication](../authentication/README.md) lesson.

# Set up Amazon S3

AWS (Amazon Web Services) is a cloud platform offering a slew of web services such as compute power, database storage, content delivery and more.

You can see all of the services available by clicking here [**Amazon AWS**](https://aws.amazon.com/).

The specific web service we are going to use to host our uploaded images is the _Simple Storage Service_, better known as **Amazon S3**.

## Set up an Amazon AWS Account

Before we can use S3, or any Amazon Web Service, we’ll need an Amazon AWS account.

> Even though AWS has a "free tier" and we will do nothing in SEI will cost money, AWS requires a credit card to open an account. If you don’t wish to provide a credit card, you will not be able to complete this activity and you won’t be able to upload images using Amazon Web Services.

Click the orange **Create an AWS Account** button, then click **Create a new AWS account** (or browse **[here](https://portal.aws.amazon.com/billing/signup#/start)**):

![](./assets/aws-create-account.png)

Unfortunately, the signup process is a bit lengthy…

You’ll need to set up a super-secure password that meets strict requirements, confirm your email address, add a credit card, and verify a code via SMS.

### Sign in to the AWS Console

After you have signed up, click **Go to the AWS Management Console**:

![](./assets/aws-go-to-console.png)

Then sign in as the Root user using the account you just created:

![](./assets/aws-sign-in.png)
![](./assets/aws-console-home.png)

> What you should see when logging in

## Create Access ID & Secret Keys

We need to do create an **Access Key ID** and a **Secret Access Key** to access S3 with.

Click the search bar at the top of the page, search for **iam** then click on the **IAM** option:

![](./assets/aws-iam.png)

Locate the **Services** section and click **IAM** (Identity and Access Management):

Click **Users** in the sidebar (also take note that MFA is not enabled for the root user - you may want to do this yourself later for added security):

![](./assets/aws-iam-users.png)

Click the blue Add users button:

![](./assets/aws-iam-add-user.png)

Enter a user name for the credentials being created (your web app’s name if perfect) and click Next.

![](./assets/aws-iam-add-user-2.png)

Now we need to create a group that will have S3 permissions that the user can be assigned to.

Click the **Create group** button:

![](./assets/aws-iam-create-group.png)

Enter a **Group name** - `django-s3-assets` is fine.

Then search for `S3Full` and select the **AmazonS3FullAccess** checkbox and click the blue **Create group** button:

![](./assets/aws-iam-create-group-2.png)

You'll be taken back to the **Add user** step. On this page you **_must_** ensure that:

- The group you just created is displayed
- The group you just created has the `AmazonS3FullAccess` policy attached to it
- The group you just created has a selected checkbox next to it

After checking these items, click the **Next: Tags** button.

On the following screen, ensure that the `django-s3-assets` group is selected, then click the blue **Next** button:

![](./assets/aws-iam-add-user-3.png)

Now click the Create user button:

![](./assets/aws-iam-add-user-4.png)

Click on the User name:

![](./assets/aws-iam-user-name.png)

On this page, click on the Security credentials tab:

![](./assets/aws-iam-user-security-credentials.png)

Scroll down and click on the Create access key button:

![](./assets/aws-iam-create-access-key.png)

Select Application running outside AWS and click Next:

![](./assets/aws-iam-create-access-key-2.png)

Enter a description for your key and click Create access key:

![](./assets/aws-iam-create-access-key-3.png)

Copy both access keys to a safe place - we’ll need them later in the lesson, and **they can't be viewed again later:**

![](./assets/aws-iam-create-access-key-4.png)

Click the **Done** button (bottom-right), but don’t close the browser. If you didn’t click the ‘Show’ option, it will give you a reminder to snag your key, as it will not be available again later!

On to creating an S3 bucket…

## Create an S3 Bucket for the `catcollector` App

Click the search bar at the top of the screen, enter **S3** and click the **S3** link under the **Services** section.

![](./assets/aws-s3.png)

Buckets are containers for stuff you store in S3.

Typically, you would want to create an S3 bucket for each web application you develop that needs to use S3.

Let’s click the orange **Create bucket** button to get started:

![](./assets/aws-s3-create-bucket.png)

You’ll have to enter a _globally unique_ **Bucket name**, then select the **Region** nearest to your job market:

![](./assets/aws-s3-create-bucket-2.png)

For the best performance, always be sure to select the nearest location to where your application will be hosted, e.g., Heroku. For this lesson, be sure to select the Region nearest you.

Scroll down to get to the following screen where you want to **UNCHECK** the **Block all public access** checkbox, then **CHECK** the **I acknowledge…** checkbox:

![](./assets/aws-s3-create-bucket-3.png)

Scroll all the way down the page and click the orange Create Bucket button!

![](./assets/aws-s3-create-bucket-4.png)

The bucket has been created!

However, we need to make sure that web browsers will be able to download cat images when using catcollector.

To do so, we need to specify a "bucket policy" that enables read-only access to a bucket’s objects.

Click on the bucket name:

![](./assets/aws-s3-bucket-name.png)

Click the Permissions tab at the top:

![](./assets/aws-s3-bucket-permissions.png)

Scroll down to Bucket Policy and click Edit:

![](./assets/aws-s3-bucket-policy.png)

After you have clicked the button to copy your Bucket ARN click on the Policy generator link:

![](./assets/aws-s3-bucket-policy-2.png)

A new tab will open in your browser.

Start entering the data as follows:

- Type of Policy is: S3 Bucket Policy
- Principal is: `*`
- AWS Service is: Amazon S3
- Actions: 1 Action is selected - the `GetObject` action
- Amazon Resource Name (ARN) is: The **Bucket ARN** you copied from the last page (it should look like `arn:aws:s3:::<your-bucket-name-here>` (replacing `<your-bucket-name-here>` with the name of your bucket). You also need to append a `/*` to the end of the ARN so that the final ARN reads as: `arn:aws:s3:::<your-bucket-name-here>/*`. Using `my-very-happy-cat-collector` as and example the final ARN would look something like: `arn:aws:s3:::my-very-happy-cat-collector/*`

![](./assets/aws-s3-bucket-policy-3.png)

With those values entered, click the yellow Add Statement button.

![](./assets/aws-s3-bucket-policy-4.png)

Once that is done, click the yellow Generate Policy button that’s in Step 3.

![](./assets/aws-s3-bucket-policy-5.png)

Copy ALL of the text inside the box, including the curly braces, then close the modal:

![](./assets/aws-s3-bucket-policy-6.png)

Close the AWS Policy Generator browser tab and return to the S3 Management Console browser tab. Paste in the text you just copied:

![](./assets/aws-s3-bucket-policy-7.png)

Scroll to the bottom of the page and click the orange **Save changes** button.

After saving you should see something like this at the top of the page:

![](./assets/aws-s3-bucket-policy-8.png)

Congrats! You now have an S3 bucket that, using the access keys, an application can upload files of any type to; that also allows any browser to download by using a known URL.

Now let’s get on with the app!

# Install & Configure - Boto 3

## Install `boto3`

The official Amazon AWS SDK (Software Development Kit) for Python is a library called [Boto3](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html). To install this in our app, we’ll need to add it to the `Pipfile` and `Pipfile.lock`. From your terminal, run:

```bash
pipenv install boto3
```

## Configure Credentials

Don’t ever put your Amazon AWS, or any other secret keys, in your source code!

If you do, and push your code to GitHub, it will be discovered within minutes and could result in a major financial liability!

Have I scared you? Good…

Let's check out the [**Boto3 docs**](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/credentials.html) to see all the options we have for configuring our AWS credentials in our app. You'll see that there are TONS. We'll be using this method: [**Environment Variables with specific names**](https://boto3.amazonaws.com/v1/documentation/api/latest/guide/credentials.html#environment-variables).

### Environment Variables in Django

To configure Boto3, we're going to need these 2 environment variables:

```
AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
```

Put your access keys here, make sure you do not use quotes or spaces after the `=` sign. By using these specific names we don't have to do any additional work to make use of them in our application.

We’re done with Boto3, on to the larger project…

# Add a `Photo` Model

## The Relationship

We’re going to add a `Photo` Model that will hold the URL of a cat image in S3.

The relationship looks like this: `Cat ----- Photo`
_A Cat has a Photo -and- A Photo belongs to a Cat_

## Define the `Photo` Model

To the **`models.py`** file we go:

```python
class Photo(models.Model):
  url = models.CharField(max_length=250)
  cat = models.OneToOneField(Cat, on_delete=models.CASCADE)

  def __str__(self):
    return f"Photo for cat_id: {self.cat_id} @{self.url}"
```

Nice and simple!

> Reminder: If a Model "belongs to" another Model, it must have a foreign key. More than one "belongs to" relationship - means more than one foreign key.

<details>
<summary>What needs to be done now?</summary>

Open the terminal and enter:

```bash
python3 manage.py makemigrations
python3 manage.py migrate
```

</details>

# Add the `add_photo` URL

We need to add a new `path()` to the `urlpatterns` list that will match the request sent when the user submits a photo.

In **`urls.py`**:

```python
urlpatterns = [
	...
  path('cats/<int:pk>/delete/', views.CatDelete.as_view(), name='cats_delete'),
  path('cats/<int:cat_id>/add-feeding/', views.add_feeding, name='add-feeding'),
  # new path below
  path('cats/<int:cat_id>/add-photo/', views.add_photo, name='add-photo'),
]
```

Pretty much like the `add-feeding` route!

Notice once again, we’re going to capture the cat’s `id` using a URL parameter named `cat_id`.

The server currently shows an error because we’ve referenced an `add_photo` _view function_ that doesn’t exist. Let’s take care of that next…

# Code the `add_photo` View Function

<details>
<summary>Where do we define the view functions?</summary>

**`views.py`**

</details>

First, we need to import three more things:

- the `boto3` library
- the `Photo` Model
- Python’s `uuid` utility that will help us generate random strings

```python
from django.shortcuts import render, redirect
from django.views.generic.edit import CreateView, UpdateView, DeleteView
from .forms import FeedingForm
# new/updated imports below
from .models import Cat, Toy, Photo
import uuid
import boto3

```

Next, we’re going to define a couple of variables we’ll use in the _view function_…

### Determine the correct AWS Service Endpoint

AWS has different endpoints (URLs) dependent upon the service (S3 in our case) and geographic region.

In regards to S3, [**the docs**](https://docs.aws.amazon.com/general/latest/gr/s3.html) point to these endpoints for the regions in the US:

- **`https://s3.us-east-1.amazonaws.com/`** (N. Virginia)
- `**https://s3.us-east-2.amazonaws.com/**` (Ohio)
- `**https://s3.us-west-1.amazonaws.com/**` (N. California)
- `**https://s3.us-west-2.amazonaws.com/**` (Oregon)

Pick the endpoint nearest to you and assign it to the `S3_BASE_URL` variable. In addition, assign your bucket name to a `BUCKET` variable as follows:

```python
# Add these "constant" variables below the imports
S3_BASE_URL = 'Your endpoint from above'
BUCKET = 'your-bucket-name-here'
```

**Make sure that you use YOUR S3 base URL and bucket name instead of** `'Your endpoint from above'` and \***\*`your-bucket-name-here`**.\*\*

Each file uploaded to S3 must have a unique URL.

We’ll be using `S3_BASE_URL`, `BUCKET` and a randomly generated key to build this unique URL.

The unique URL will also be saved in the `url` attribute in each `Photo` instance.

Finally, this is where the magic happens, we’ll review the code as we type it in:

```python
def add_photo(request, cat_id):
  # photo-file will be the "name" attribute on the <input type="file">
  photo_file = request.FILES.get('photo-file', None)
  if photo_file:
    s3 = boto3.client('s3')
    # need a unique "key" for S3 / needs image file extension too
		# uuid.uuid4().hex generates a random hexadecimal Universally Unique Identifier
    # Add on the file extension using photo_file.name[photo_file.name.rfind('.'):]
    key = uuid.uuid4().hex + photo_file.name[photo_file.name.rfind('.'):]
    # just in case something goes wrong
    try:
      s3.upload_fileobj(photo_file, BUCKET, key)
      # build the full url string
      url = f"{S3_BASE_URL}{BUCKET}/{key}"
      # we can assign to cat_id or cat (if you have a cat object)
      photo = Photo(url=url, cat_id=cat_id)
      # Remove old photo if it exists
      cat_photo = Photo.objects.filter(cat_id=cat_id)
      if cat_photo.first():
        cat_photo.first().delete()
      photo.save()
    except Exception as err:
      print('An error occurred uploading file to S3: %s' % err)
  return redirect('cat-detail', cat_id=cat_id)
```

Code like

```python
key = uuid.uuid4().hex + photo_file.name[photo_file.name.rfind('.'):]
```

is a great line of code to examine in smaller pieces within the debugger if it seems overwhelming.

On to the UI…

# Update the `detail.html` Template

We’re going to need to update the `**detail.html**` template to

- Show a cat image if one has been uploaded.
- Show a form used to upload photos.

## Display Cat Images

Find the `cat-container`. We're going to check if there is a photo associated with the cat. If there is we're going to display it instead of the placeholder image.

```html
<section class="cat-container">
  <div class="cat-img">
    {% if cat.photo.url %}
    <img
      src="{{cat.photo.url}}"
      alt="A photo of {{cat.name}}"
      class="usr-img"
    />
    {% else %}
    <img src="{% static 'images/sk8r-boi-cat.svg' %}" alt="A skater boy cat" />
    {% endif %}
  </div>
</section>
```

## Form for Uploading Photos

In this same section, right after the `cat-actions` `<div>` we need to add our new form which will upload files to the server:

```html
    <div class="cat-actions">
      <a href="{% url 'cat-update' cat.id %}" class="btn warn">Edit</a>
      <a href="{% url 'cat-delete' cat.id %}" class="btn danger">Delete</a>
    </div>
    {% comment %} New markup below {% endcomment %}
    <h3>Change {{ cat.name }}'s photo</h3>
    <p id="file-name"></p>
    <form
      action="{% url 'add-photo' cat.id %}"
      enctype="multipart/form-data"
      method="POST"
    >
      {% csrf_token %}
      <label class="btn secondary">
        Choose photo
        <input type="file" name="photo-file" id="file-input">
      </label>
      <button type="submit" class="btn submit">Upload Photo</button>
    </form>
  {% comment %} New markup above {% endcomment %}
  </div>
</section>
```

When using HTML forms to upload files, it’s important to add the `enctype="multipart/form-data"` attribute to the `<form>` tag.

Other than `{% csrf_token %}`, it’s pretty much a generic form that is typically used to upload files to any web app.

This form needs a touch of JS to actually show the name of the file we're uploading. Thankfully we already have this file created, we're just adding to it. Here are the additions:

```jsx
const fileInput = document.getElementById('file-input');
const fileName = document.getElementById('file-name');

fileInput.addEventListener('change', (evt) => {
  const fileToUpload = evt.target.files[0].name;
  if (fileToUpload) {
    fileName.innerText = fileToUpload;
  } else {
    fileName.innerText = '';
  }
});
```

Another refresh:

![](./assets/upload-photo.png)

Looking good!

# Test it Out!

Try uploading your favorite cat pic.

Success!

![](./assets/cat-pic.png)

# Update `index.html`

Just a quick change here to display the user's uploaded image if it exists. The new code belongs in the `div` with the `card-img-container` class:

```html
<div class="card-img-container">
  {% if cat.photo.url %}
  <img src="{{cat.photo.url}}" alt="A photo of {{cat.name}}" class="usr-img" />
  {% else %}
  <img src="{% static 'images/sk8r-boi-cat.svg' %}" alt="A skater boy cat" />
  {% endif %}
</div>
```

And there we go!

![](./assets/index-cat-pic.png)

# Check it Out in Admin

Don’t forget to register the new `Photo` model so that you can use the built-in Admin app to add/edit/delete model instances.

Update the `**admin.py**` file to look like this:

```python
from django.contrib import admin
# Import your models here
from .models import Cat, Feeding, Toy, Photo

# Register you models here
admin.site.register(Cat)
admin.site.register(Feeding)
admin.site.register(Toy)
admin.site.register(Photo)
```

Then browse to https://127.0.0.1:8000/admin and click through to view the added photo.

You’ll see something like this:

![](./assets/admin-photo.png)

Note how the photo’s url is formed.

Congrats on uploading files to Amazon S3!
