**Building a Facebook application with Rails and hosting it in Heroku**
=======================================================================


Introduction
------------

Facebook has given a big step in simplicity with its _Graph API_ and the creation of the _Open Graph protocol_, now more than ever it's easier to read and write data from and to the socalled social graph.

You could turn your webpage into a fully-featured Facebook-like page, just like if you were inside Facebook, you can give your users the ability to sign in with their Facebook credentials, customize your users' experience with parameters taken from their Facebook profiles, you could add a _Like_ button to every object in your page as images, songs, articles, etc., tell your users which friends of theirs have liked your content, and a lot of things more, oh, I forgot to mention something, all of this with total ease. Yes, I know you could do it with Facebook Connect, but I said "with ease".

Ok, let's get started with the fun.

A brief explanation of the Facebook application architecture
------------------------------------------------------------

Well sorry if I didn't jump to the actual funny part, but we really need to know the first things first, it'll save us a lot of time in the long run. I swear.

A Facebook application is actually two applications: A container application and a web application.

When we create a Facebook application in the Facebook site what we are actually creating is the container of our web app.

Let's think of our web application like if it were a painting, Facebook itself would be the stand and the container would be the canvas, now let's call the container a [Canvas Application](http://developers.facebook.com/docs/guides/canvas/).

As you can see they both are actually two different things:  

* The stand and the canvas are hosted in Facebook
* Our web application, which is not different than any other application we've made before, is not hosted in any Facebook server; we must store it in our own server or a hosting service. 

For both of them to look like if they were only one application you have to associate them and once we do so our [Canvas Application](http://developers.facebook.com/docs/guides/canvas/) and our web app will begin to interact with each other.

Well, all of this let us with a process of three phases:  

* Creation
* Association
* Configuration

Actually, the association is part of the configuration, but I like to think of the process this way to keep things separate.

##**Creation**

####Facebook Canvas Application

This is the easiest part, you just have to go to the Facebook Developer site and [setup a new application](http://www.facebook.com/developers/createapp.php). 

> screenshot of the form

Input whatever application name you want in the _Application Name_ field, like "My Facebook App", then agree the terms and click the _Create Application_ button, after passing the _Security Check_ you'll be presented with the _Basic Information_ of you brand new application.

Cool, we've created our [Canvas Application](http://developers.facebook.com/docs/guides/canvas/), now we'd need to associate it with an existing web app.

####Web application
For the moment, we'll assume we already have a web application we'd like to associate with Facebook and we'll assume we are hosting it in **_http://my\_facebook\_app.heroku.com/_**

###**Association**

_**NOTE**: In case you closed the previous window you can access all your applications in the [My Applications](http://www.facebook.com/developers/apps.php) page._ Click the application we just created and click the **Edit Settings** button.

Now, click the **_Facebook Integration_** button in the settings page. 

> screenshot of the settings page

I'd like you to pay attention to the Canvas section, here we'll find all the settings we need to associate our existing web app with our [Canvas Application](http://developers.facebook.com/docs/guides/canvas/).

* **Canvas Page**  
	- It's an unique access point for your application, say you named your application _My Facebook App_, your Canvas Page might be http://apps.facebook.com/my\_facebok\_app/.
	
	By the way, sometimes you'll have to struggle to find an available name, it's like when you want to buy a .com domain, sometimes the one you want have been already bought by another person, it's unavailable, it's something like that.

* **Canvas URL**
	- This is the URL of our web application, in this case **_http://my\_facebook\_app.heroku.com/_** (the trailing slash is mandatory).

Now, when our users browses **_http://apps.facebook.com/my\_facebook\_app/_**, Facebook will show them our [Canvas Application](http://developers.facebook.com/docs/guides/canvas/) (an HTML page) which contains an [IFrame](http://www.w3schools.com/tags/tag_iframe.asp) (inline frame) that in turn shows **_http://my\_facebook\_app.heroku.com/_**. This kind of applications are called **IFrame Canvas Applications**.

       Canvas application                    Web application
          in Facebook                      in our own servers
    +----------------------+       -->   +--------------------+
    | Facebook chrome      |     /       |       hello!       |
    |  +-----------+ <-    |   /         |                    |
    |  |   hello!  |    \==|===          |                    |
    |  |           |       |             |                    |
    |  |   Canvas  |    /==|===          |                    |
    |  +-----------+ <-    |   \         |                    |
    |                      |     \       |                    |
    +----------------------+       -->   +--------------------+
	ToDo: make an infograhic

Fig1. An IFrame Canvas application is actually an IFrame surrounded by the Facebook chrome. The IFrame points to our web app URL.

Let's try it with our new application. 

Without any fear, change the **Canvas URL** to _http://www.heroku.com/_, now open the **Canvas URL** in your browser. What do you see? Yes, the Heroku site, this is why it looks like your web application is inside Facebook.

####**Canvas Type**

* FBML (Facebook Markup Language) Canvas Applications
* IFrame Canvas Applications

As of today, {the date}, Facebook still allow us to create FBML Canvas Applications but they will be deprecated by the end of 2010. [Click here to see the announcement](http://developers.facebook.com/blog/post/402).

###**Configuration**

Now it's time edit all the other settings to customize your application: description, logo, icon, etc. It's a very straight-forward phase, so I won't touch it here.

###**The interaction between Canvas Applications, web applications and Facebook**

Facebook has several mechanisms to let our web applications interact with the data of its social graph, more precisely, with the data of the users of our application, among these mechanism are:

* [Graph API](http://developers.facebook.com/docs/api) - A powerful yet simple RESTful API
* [XFBML]() - It enables you to incorporate FBML into your websites and IFrame Canvas Applications.

Every time you want an user authorizes your application and every time you want to access your users data you'll be using the [Graph API](http://developers.facebook.com/docs/api) in one way or other.

Here's a diagram of how Facebook applications interact with the [Graph API](http://developers.facebook.com/docs/api) and XFBML.


1.User request http://apps.facebook.com/my_facebook_app  
2.Facebook returns HTML code with the Facebook chrome and the IFrame  
3.The IFrame request the Canvas URL to our server  
4a.Our web app make a request to the Graph API (say the name of the current user)  
4b.Our web app contains XFBML (a Login and a Like button)  
5a.The Graph API respond with the users data in JSON/XML  
5a. Facebook responds with the HTML to show the Login and Like buttons  
6.Our app builds an HTML response to show the name of the current user and it is shown in the IFrame  


                                   1  
    +----------------------+ ==============> +-----------+   
    | Facebook chrome      |                 |           |  
    |  +-----------+       |       2         |    FB     | ----  
    |  |   hello!  |       | <============== |           |    |  
    |  |           |       |                 |           |    |  
    |  |   Canvas  |       |                 +-----------+    |  
    |  +--+----+---+       |                           ^      |  
    |     ^    |           |                           |      |  
    +-----|----|-----------+                           |      |  
          |    |______________                         | 4    | 5  
          |________________    \                       |      |  
                  6         \    \  3                  |      |  
                              \    \                   |      |  
                                \    \                 |      |  
                                 |    `--->  +-----------+    |      
                                 |           |           |    |  
                                  \          |    Our    |    |  
                                    `------- |   server  | <--'  
                                             +-----------+  
    ToDo: make an infographic

If our application requests the Graph API via Javascript the communication would be directly with the Facebook servers.

1.Our application contains the following Javascript code:

    <script>
	      FB.api('/me', function(response) {
	          alert(response["name"]);
	      });
	</script>

It's fired by the user with a button.

2.Facebook returns the response in JSON and its parsed by the callback function and it alerts with the name of the current user.

    +----------------------+     
    | Facebook chrome      |              +-----------+  
    |  +-----------+       |    1         |           | 
    |  |   hello!  | ------|----------->  |    FB     | 
    |  |           |       |              |           | 
    |  |   Canvas  | <-----|------------  |           | 
    |  +-----------+       |    2         +-----------+                           
    |                      |                             
    +----------------------+ 
    ToDo: make an infographic

Well, It would seem it wasn't a brief explanation but believe me, it really was.

The Facebook's Graph API
------------------------


The Graph API enables you to read and write objects and connections in the Facebook social graph. It's so easy to use and it's very clean. If you want to fully understand how does the API actually works you have to know about the [RESTful](http://tomayko.com/writings/rest-to-my-wife) architecture, although it isn't completely necessary, I'd really recommend you to research about it.

The main concept to take into account is _object_, we have to think of every single thing in Facebook as _objects_: photos, events, comments, friends, tags, groups, etc. and we must know that every object in Facebook has an unique identifier, an ID.

Ok, let's query the Graph API with our browser. Click the following link to see the information of the Miles Davis page.

[http://graph.facebook.com/57796847423](http://graph.facebook.com/57796847423)

Now you should be seeing the response in [JSON](http://en.wikipedia.org/wiki/JSON) format. Among other things you will see the following information:

	{
	   "id": "57796847423",
	   "name": "Miles Davis",
	   "picture": "http://profile.ak.fbcdn.net/hprofile-ak-snc4/hs170.ash2/41611_57796847423_9040_s.jpg",
	   "link": "http://www.facebook.com/MilesDavis",
	   "category": "Musicians",
	   "username": "MilesDavis",
	   "genre": "Jazz",
	   "bio": ...
	   "fan_count": 363730
	}
	
Now, we're going to query the API passing the _username_ as parameter:

[http://graph.facebook.com/MilesDavis](http://graph.facebook.com/MilesDavis)

We get the same response, maybe there are new fans between calls but the structure is the same.

As I said earlier, that is a page, and the Graph API exposes certain public and private information of the page, In the [Graph API reference > Page](http://developers.facebook.com/docs/reference/api/page) we will find the access points to such information, Facebooks calls them Connections.

The reference says we can access a connection called _posts_, let's try it:

[http://graph.facebook.com/MilesDavis/posts](http://graph.facebook.com/MilesDavis)

Right now I see something like this:

    {
       "data": [
          {
             "id": "57796847423_126368887412272",
             "from": {
                "name": "Miles Davis",
                "category": "Musicians",
                "id": "57796847423"
             },
             "message": "Drummer Lenny White on Bitches Brew",
             ...
    }

It's an array of _Post_ objects, and the [Graph API reference > Post](http://developers.facebook.com/docs/reference/api/post) page say there's a connection called _Comments_ and that I can access them with the following URL:

[http://graph.facebook.com/57796847423_126368887412272/comments/](http://graph.facebook.com/57796847423_126368887412272/comments/)

Cool, we are seeing the comments of the post with ID=57796847423_126368887412272.

This is basically how the Graph API works, if you want to know more about it please go to the [Graph API overview page](http://developers.facebook.com/docs/api).

Well, so far, we've been accessing public information, if we want to access private information we won't be able to do it this way, Facebook only allow us to access private information if we have a Facebook application and such application must be explicitly authorized to access such information.

Who authorizes our application? neither Facebook nor the FCC but the **USERS**, nobody else, and they are the only ones with the power to deauthorize it.

###**Authentication**

Facebook uses the [OAuth 2.0 protocol](http://wiki.oauth.net/OAuth-2) for authentication and authorization.

    Authentications means "It's me, and here's my proof"  
    Authorization means "Now that you know who I am, would you let me do this or that?"  

The [Facebook Authentication](http://developers.facebook.com/docs/authentication/) page explains the authentication process very well, but I'll try to summarize the process with a simple graphic:

1.Register your application to get an app ID and secret

2.https://graph.facebook.com/oauth/authorize?
     client\_id=app_id&
     redirect\_url=http://localhost:3000/oauth_callback

3.If the user authorizes your application, we redirect the user back to the redirect URI you specified 
   http://localhost:3000/oauth_redirect?code=the\_code

4.Exchange the code for an oauth access token in the following URL
   https://graph.facebook.com/oauth/access_token?
	 client\_id=...&
     redirect\_uri=http://localhost:3000/oauth_callback
     client_secret=...&
     code=...

5.Facebook will request the callback url with the access token
	 http://localhost:3000/oauth_callback?access_token=3gergve5gesrdsgs
	
6.Store it in a DB, cookie or whereever you want, it must be used to make requests on behalf of the user.


    +----------------------+     2
    | Facebook chrome      |------------> +-----------+
    |  +-----------+       |     3        |           |
    |  |           | <-----|------------  |    FB     |
    |  |           |       |    4         |           |
    |  |   Canvas  | ------|------------> |           |
    |  +-----^-----+       |    5         +-----+-----+
    |        |-------------|-------------------/                          
    +----------------------+ 
    ToDo: make an infographic

Ok, now our application has been authenticated and the user has authorized it, a question raises: 

    What kind of information would my application access? 

The answer is: 

    Only the general information: email, name, etc.

This happens when we don't specify what kind of permissions we're asking for, so Facebook sets the default permissions.

Here is how we ask the user for extended permissions:

    https://graph.facebook.com/oauth/authorize?
        client_id=...&
        redirect_uri=http://www.example.com/callback&
        scope=user_photos,user_videos,publish_stream

Once we get the access token our application will be able to access all the photos, videos and even write to the users's wall.

That was basically how authentication and authorization works in Facebook.

Now we have all we need to start creating real Facebook applications and the most important part, make them in Ruby.

Ruby and Facebook (The status of gems and plugins)
--------------------------------------------------

I've found that most of the existing gems and Rails plugins are still using [Facebook Connect](http://en.wikipedia.org/wiki/Facebook_Platform#Facebook_Connect) and the [Old REST API](http://developers.facebook.com/docs/reference/rest/) instead of the new [Graph API](http://developers.facebook.com/blog/post/377) and that some of them doesn't work with Ruby 1.9 and/or Rails 3.

The gems we are going to use in this tutorial are proven to work with Ruby 1.9.x and Rails 3.0.0, I'd like to share with you as well the gems I tested in the process of creation of this tutorial:

* **Gems**
	- [FBGraph](http://github.com/nsanta/fbgraph)
	- [miniFB](http://github.com/appoxy/mini_fb)
	- [Koala](http://github.com/arsduo/koala)
	- [Mogli](http://github.com/mmangino/mogli)

* **Plugins**
	- [Facebooker2](http://github.com/mmangino/facebooker2)

* **Middlewares**
	- [OmniAuth](http://github.com/intridea/omniauth)
	- [rack-facebook](http://github.com/carlosparamio/rack-facebook)


**Making the simplest Facebook application possible**
-----------------------------------------------------


The fun part has finally arrived, in this section we'll make a very simple web application using the [Sinatra Framework](http://www.sinatrarb.com/) and we will integrate it with Facebook. I'll assume that you're using Ruby 1.9.x. and that you have [Git](http://book.git-scm.com/2_installing_git.html) installed.

If you are using any other version of Ruby I'd suggest you to [install Ruby 1.9.x with RVM](http://amerine.net/2010/02/24/rvm-rails3-ruby-1-9-2-setup.html). If you're going to use [RVM](http://rvm.beginrescueend.com/) (Ruby Version Manager) you won't need to prefix the following commands with _sudo_ (Unix-like OSs only).

Let's install some gems. 

If you don't already do so, I do really recommend you to use [Bundler](http://gembundler.com/), it's the default gem dependency manager in Rails 3 and you'll fall in love with it as soon as you use it, well, at least I did.

	$ sudo gem install bundler

We need to host our web application somewhere, let's do it the easy and fast way, let's	use [Heroku](http://www.heroku.com). We'll need to install the Heroku command-line tool, a gem that will allow us to create and deploy Heroku applications with ease.

	$ sudo gem install heroku

Of course, you'll need to sign-up for a Heroku account and then configure the command-line tool, it's really a very simple process, [click here to see the full instructions](http://docs.heroku.com/heroku-command) to do so.

We are all set up, let's get started!

The first step will be to [create a Canvas application](http://www.facebook.com/developers/createapp.php) in Facebook, we already touched this earlier.

I have named "Faceboku" but you can name it whatever you want to.

These would be our application settings:

	**Canvas Page** = http://apps.facebook.com/faceboku/  
	**Canvas URL** &nbsp;= http://localhost:4567 (the standard port used by Sinatra application)  
	**Canvas Type** = IFrame

If you don't see the Canvas Type option, don't worry, there's a possibility Facebook has got rid of it by the time you're reading this tutorial.

As you may realize, you won't be able to use the same Canvas Page than me, just choose another one. We're using http://localhost:4567 as our Canvas URL because that way we'll be able to test our Facebook application before we deploy it to Heroku.

What's next? our Sinatra application.

Create a directory called _simplest\_fb\_app_ wherever you want in your system, I'll do it in my home directory:

	$ mkdir ~/simplest_fb_app

then create the following files on it:

	* **faceboku.rb** (use the name of your Canvas application)
	* **config.ru** (Rack config file)

If you installed Bundler, now we'd need to create a file called _Gemfile_, here we'll declare the dependencies of our application, but we don't need to make it manually, Bundler can make it for us:

	$ bundle init
	Writing new Gemfile to ~/simplest_fb_app/Gemfile

_Take into account that the gem is called Bundler while the command-line utility is called **bundle**_

Now we're going to declare the dependencies, edit the Gemfile with the text editor you want to let it look like this:

	source :gemcutter

	gem 'sinatra', '1.0'
	gem 'omniauth', '0.0.5'

Tell Bundler to check if we have all the dependencies installed:

	$ bundle check
	The following gems are missing
	* sinatra (1.0)
	Install missing gems with `bundle install`

Bundler is telling us that we don't have all the dependencies resolved, let's fix it:

	$ bundle install
	Fetching source index for http://rubygems.org/
	Using...a lot of gems
	Using rack (1.2.1) 
	Installing omniauth (0.0.5) 
	Installing sinatra (1.0) 
	Using bundler (1.0.0) 
	Your bundle is complete! Use `bundle show [gemname]` to see where a bundled gem is installed.

What we gain using Bundler? There're a lot of advantages, first, we can be sure that you and me are using exactly the same versions of gems, second, Bundler creates a file called Gemfile.lock, take a look at it, you'll see a tree of dependencies, third, Bundler makes easy to deploy to different environments.

NOT FINISHED

Making a Facebook application with Rails 3
------------------------------------------

...