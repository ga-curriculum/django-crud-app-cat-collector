# ![Django CRUD App - Cat Collector - Setup](./assets/hero.png)

## Setup

Open your Terminal application and navigate to your `~/code/ga/lectures` directory:

```bash
cd ~/code/ga/lectures
```

Create a new repository on [GitHub](https://github.com/) named `django-crud-app-cat-collector`.

```bash
git clone https://github.com/<your-username>/django-crud-app-cat-collector.git
```

> 🚨 Do not copy the above command. It will not work. Your GitHub username will replace `<github-username>` (including the `<` and `>`) in the URL above.

Next, `cd` into your new cloned directory, `django-crud-app-cat-collector`:

```bash
cd django-crud-app-cat-collector
```

Initialize a new virtual environment inside your project directory and install Django:

```bash
pipenv install django
```

This command will create a new `Pipfile` and `Pipfile.lock` in your project directory, specifying Django as a dependency.

Activate the virtual environment:

```bash
pipenv shell
```

> With your virtual environment activated you should see a slight change in your terminal, with the virtual environment folder name listed to the left of your command line prompt.

Start a new Django project within your virtual environment:

```bash
django-admin startproject catcollector .
```

The `.` here starts this project inside of our current directory, instead of creating a new one.

Open the project's folder in your code editor:

```bash
code .
```

To deactivate the virtual environment when you're done, simply type:

```bash
exit
```
