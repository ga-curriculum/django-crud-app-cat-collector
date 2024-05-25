# ![Django CRUD App - Cat Collector - Concepts](./assets/hero.png)

**Learning objective:** By the end of this lesson, students will understand the structure and philosophy behind Django's design, noting its built-in functionalities and strict conventions.

## Understanding framework philosophies: Express vs. Django

### Review of Express framework

Express is characterized as a minimalist framework, offering just enough tools to get started but not much beyond the basics. This approach provides the flexibility to define routes, map controller actions, and render dynamic views according to your project's needs. The lack of rigid structure means file naming and organization are left largely to the developer's discretion. When additional functionality is required, it typically involves integrating and configuring middleware or outside libraries. This setup is ideal if you appreciate a "build-it-your-way" philosophy.

### Introduction to Django: A full-featured framework

Transitioning to Django, you'll encounter a contrastingly different environment. Django is a full-featured framework, designed to include a comprehensive suite of built-in functionalities. It adheres to a strict set of *conventions*, guiding you on how to structure your code, name your files, and even how to handle database operations. This structured approach is accompanied by numerous helper classes and methods, simplifying tasks that in Express might require additional packages.

For those familiar with frameworks like Express, Django might initially seem restrictive. However, this structure aims to ensure scalability and maintainability of applications. It's important to note that becoming familiar with Django's many features can be more time-consuming compared to grasping the basics of Express. Nevertheless, learning the essentials of Django is manageable and can greatly accelerate development once you are accustomed to its paradigms.

### The request/response cycle in Django

If you have experience with Express, you've learned that interacting with a web application involves a series of HTTP requests and responses:

- User actions like clicking links or submitting forms generate HTTP requests.
- These requests are handled by a server's routing system, which directs them to the appropriate code.
- The code then executes CRUD operations. Depending on the operation, the server might render dynamic content for 'Read' operations or issue redirects for 'Create', 'Update', or 'Delete' operations.

In Django, the HTTP request/response cycle is managed by built-in components, which simplifies much of the setup process. This integration can greatly benefit newcomers to web development or anyone who wants to simplify their application’s infrastructure.

tktk - Hunter this is the image from the notion content, should we update it?
![MVT Design Pattern](./assets/mvt.png)
