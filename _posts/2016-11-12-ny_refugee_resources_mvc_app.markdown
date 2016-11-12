---
layout: post
title:  "My Very Cool app"
date:   2016-11-11 21:13:35 -0500
---


In deciding the domain for my MVC portfolio app I decided to continue in the vein of my previous Immigrant Stories CLI Gem and build a Ruby based application that sought to engage Migrants and in this case, Refugees.  The objective in making this app wasn't to create something revolutionary, which it is not, but to use subject matter important to me to create a web enabled app to further my understanding of Ruby and how it's used in Front and Backend development.  

The Sinatra unit is Flatiron’s first real taste of how to use Ruby in a fully integrated web environment and lays out the fundamentals of website design from the perspective of a developer.  As such, I found the unit pretty dense in terms of shear material (I'm sure I'm not alone in this), but the end result, having made a fully functioning (albeit bare bones) Web app was extremely rewarding.  

I envisioned the app I created for this project as a kind of Yelp for refugee aid/outreach organizations in New York for them to post their experiences with a particular organization to help others navigate these often-complex social services.  

The Sinatra Portfolio Project has the following requirements:

* Build an MVC Sinatra Application.
* Use ActiveRecord with Sinatra.
* Use Multiple Models.
* Use at least one has_many relationship
* Must have user accounts. The user that created a given piece of content should be the only person who can modify that content
* You should validate user input to ensure that bad data isn't created

The [NY Refugee Resources app](https://github.com/zenglue/NY-Refugee-Resources-MVC-Sinatra-App) conforms to the MVC Sinatra requirement by first of all utilizing the Sinatra web framework, a light weight framework that forms the backbone and base of code the application uses. The models or "M" in MVC in my app are represented by the Ruby classes User, Organization and UserExperience in the app (which satisfies the multiple model requirement).  Sinatra here has built many of the methods already used in my code, like all or initialize; code that is common across most Ruby classes.  Through ActiveRecord and the migration tables I generated for the SQL database, Sinatra is able to define my attr_accessors for me and initialize each object class with attributes taken from the column names made for the SQL database.  Initial data for the Organization class was fed into a seed file and migrated along with the table data for the SQL database using the Rake Gem.  For Sinatra to understand the primary and foreign keys established in the database and thus map the appropriate methods (like UserExperience.user.username in the app), what I do have to establish in my models are the has_many and subsequent belongs_to relationships.  This app has more than one has_many relationship because each instance of User and Organization has many UserExperience and of course UserExperience belongs to User and Organization.  ActiveRecord interacts with the data directly while Sinatra handles the Create, Read, Update, Delete (CRUD) actions defined in my controllers (ApplicationController and the controllers that inherit from it: UserController, OrganizationController and UserExperienceController) to pass them to ActiveRecord.  The controllers or "C" in MVC and the .erb files, which in this case are our views or "V' in MVC contain most of the Ruby code not auto-magically created by Sinatra.  Using the URL's defined in the controllers Get and Post requests and the matching paths to our .erb files locally, Sinatra passes the Ruby code in our controllers to the Embedded Ruby code in the views to render the HTML in the browser.  Finally, authentication is fulfilled by enabling sessions in Sinatra and in the case of this app, the BCrypt Gem is used to add an additional layer of security by "salting" our user passwords and encrypting and decrypting them as a user signs up and logs in.  Input validation, the last requirement, was accomplished by checking the rendered views and inputting data where appropriate.  This is possible through the use of the Rack Gem middleware and it's dependencies along with the Shotgun Gem (for interactive loading of our controllers and views) which allows us to create a local server, our mini Internet warehouse, to communicate with Sinatra and check Ruby code on the fly.  

The Ride continues...

  


