<h1>Mult-Tenant SaaS App Framework</h1>
<i>by Chris Cunningham</i>

I set out to build the Framework for multi-tenan SaaS applications in Ruby on Rails with PostgreSQL, Bootstrap, iQuery, Amazon file storage, Stripe payments, SendGrid email, etc.  This framework will make the creation of new SaaS applications quick and easy. 

It is my intention to make file storage options, email provider options, payment processor options, etc. all modular so that they can be easily exchanged per the needs or requirements of an application. Ultimately I will build a generator that will allow me to choose the components to add, perhaps even predefined views.

NOTE: $ signifies a command to be entered in the terminal. Do not include the $ or the space that follows it when typing the command.

<hr>

<h2>SaaS Part 1: Creating a Rails App In C9 With PostgreSQL</h2>


1)  the first thing we are going to do is create a workspace in Cloud9 where will be able to create, test, and manage our application.

Cloud9 combines a powerful online code editor with a full Ubuntu workspace in the cloud.  Cloud9 supports more than 40 languages, with class A support for PHP, Ruby, Python, JavaScript, Go, and more and enables developers to get started with coding immediately with pre-configured workspaces, collaborate with their peers, and preview in a browser.  As per their official blog, Cloud9 was acquired by Amazon in July 2016.

Go to Cloud 9 www.c9.io (sign up for a free account) 

    - Click on “Create a new workspace”
    - Give it a name
    - Then, create a new "Blank" workspace


2)  Now that we have a Linux Ubuntu development environment setup, we will install Ruby on Rails.

Ruby on Rails, or simply Rails, is a server-side web application framework written in Ruby under the MIT License. Rails is a model–view–controller (MVC) framework, providing default structures for a database, a web service, and web pages. It encourages and facilitates the use of web standards such as JSON or XML for data transfer, and HTML, CSS and JavaScript for display and user interfacing.  Reference: http://guides.rubyonrails.org 

The first thing we are going to do is create our Rails application in Cloud 9, using PostgreSQL.

    $ gem install rails


3) Now that we have installed Rails and its many components, we can create a new Rails application.  We're going to create our Rails application using PostgreSQL for our database.

PostgreSQL is a powerful, open source object-relational database system. It has more than 15 years of active development and a proven architecture that has earned it a strong reputation for reliability, data integrity, and correctness. It runs on all major operating systems, including Linux, UNIX, and Windows.  Reference:  www.postgresql.org

    $ rails new saas -d postgresql
	(replace saas with your app name)


4) Change directory into the app

    $ cd saas	
	(replace saas with your app name)


5) Start the PostgreSQL database

    $ sudo service postgresql start


6) Create the database

    $ psql -c "create database saas_development owner=ubuntu"
	(replace saas with your app name)

7) Start the PostgreSQL database

    $ psql

8) Enter the following, one line at a time, at the prompt:

     UPDATE pg_database SET datistemplate = FALSE WHERE datname = 'template1';

     DROP DATABASE template1;

     CREATE DATABASE template1 WITH TEMPLATE = template0 ENCODING = 'UNICODE';

     UPDATE pg_database SET datistemplate = TRUE WHERE datname = 'template1';

     \c template1

     VACUUM FREEZE;

     \q


9) We will create the database

    $ rails db:create


10) Now let's migrate the database:

    $ rails db:migrate
	(now we have a schema.rb file in the db directory)

<hr>

<h2>SaaS Part 2: Add a homepage and run locally (virtually)</h2>


1) We will create a home page, home controller and route:

      $ rails g controller home index


2) Open the routes.rb file found in the config directory and
     replace: get 'home/index'  with: root 'home#index'


3) To see your application running on the Cloud 9 virtual local server,

      $ railss


4) And then go to: https://your-app-name-username.c9users.io

<hr>

<h3>Sidebar: Running the application on virtual local server on Cloud 9</h3>

 
1) To startup and run the rails server in Cloud 9:

      $ rails s -b $IP -p $PORT

   Type Ctrl C to stop the server


2) Let's configure the Cloud 9 workspace to use a more simple command to run the server.
 
      $ echo -e "\nalias railss='rails server -b \$IP -p \$PORT'" >> ~/.bash_aliases

   CLOSE ALL terminals, open a new terminal and

      $ cd saas		(replace saas)

   Then, to run the server now just enter:

      $ railss

<hr>

<h2>SaaS Part 3: Adding Git & GitHub for Version Control</h2>

Version control is a system that records changes to a file or set of files over time so that you can recall specific versions later.

Git is the most popular version control system. It was created by Linus Torvalds in 2005 for development of the Linux kernel, with other kernel developers contributing to its initial development. Git is free software distributed under the terms of the GNU General Public License version 2.

GitHub is a web-based Git or version control repository and Internet hosting service. It is mostly used for code. It offers all of the distributed version control and source code management functionality of Git as well as adding its own features.


1) Add Git:

      $ git add -A
      
   then  make your first commit

      $ git commit -m"Initial commit"


2) GitHub is a web-based Git or version control repository and Internet hosting service. It is mostly used for code. It offers all of the distributed version control and source code management functionality of Git as well as adding its own features.

  - Go to github.com (create a free account) and create a new repository.
  - Scroll to the botton of the page where it says "…or push an existing repository from the 
    command line" and copy and paste the two commands (one at a time) into the terminal.

      $ git remote add origin git@github.com:your-user/repository-name.git

      $ git push -u origin master

   For subsequent pushes to github enter $ git push

<hr>

<h2>SaaS Part 4: Deploy app to Heroku for Production</h2>

Heroku is a cloud platform as a service supporting several programming languages that is used as a web application deployment model.

1) Go to heroku.com (create a free account)

    $ heroku login
	will be prompted for heroku account email and password

   Go back to heroku and click on the “New” button to create a new app.  Then scroll to the    
   bottom of page for "Existing Git repository" and add copy and paste the remote.

    $ heroku git:remote -a your-app-name            replace your-app-name

    $ git push heroku master

    $ heroku run rake db:migrate

Go to: https://your-app-name.herokuapp.com to see your running application.

  For subsequent pushes to heroku enter $ git push heroku master
  And whenever you are pushing changes that include changes to the database schema, enter 
  $ heroku run rake db:migrate
  
<hr>

<h2>SaaS Part 5: Adding SendGrid for transactional email</h2>


1) In the terminal, enter:

    $ heroku addons:create sendgrid:starter


2) Login to heroku.com open the dashboard, click on your app and you will see that SenGrid has
    been added.
	
    Click on SendGrid and you will be taken to https://app.sandgrid.com and asked to confirm your 
    email address.  Open your email and click to confirm.


3) In config/environment.rb at the bottom enter:

    ActionMailer::Base.smtp_settings = {
      :address => 'smtp.sendgrid.net', 
      :port => '587', 
      :authentication => :plain, 
      :user_name => ENV['SENDGRID_USERNAME'], 
      :password => ENV['SENDGRID_PASSWORD'], 
      :domain => 'heroku.com', 
      :enable_starttls_auto => true 
    }

Note that using the API key is the preferred method but for now we are just using the user name and password set by SendGrid.


4) Open config/environments/development.rb
      
   Below the line:  config.eager_load = false  add the following two lines:

    config.action_mailer.delivery_method = :test 
    config.action_mailer.default_url_options = { :host => 'your-app-name-username.c9users.io', :protocol => 'https'}
	(be sure to replace the c9 app-name and your c9 user name in the url)


5) Open config/environments/production.rb
     
   Below the line:  config.eager_load = true  add the following two lines:

    config.action_mailer.delivery_method = :smtp
    config.action_mailer.default_url_options = { :host => 'your-app-name.herokuapp.com', :protocol => 'https'}
	(be sure to replace the heroku app-name in the url)
	
