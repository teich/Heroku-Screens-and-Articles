**Building a Facebook application with Ruby and hosting it in Heroku**
=======================================================================


Introduction
------------

With Facebook's _Graph API_ and the creation of the _Open Graph protocol_, it is now easier then ever before to read and write data from and to the "social graph".  Here's a few of the possibilities:

* You could turn your webpage into a fully-featured Facebook-like page, just like if you were inside Facebook.
* You can give your users the ability to sign in with their Facebook credentials and customize their experience with parameters taken from their Facebook profiles.
* You could add a _Like_ button to every object in your page as images, songs, articles, etc., and tell your users which friends of theirs have liked your content.

All of these can be achieved with ease. and in this article I'll show you how to get started.

A brief explanation of the Facebook application architecture
------------------------------------------------------------

First things first, we need to understand some of the underlying architecture of a facebook app.  It'll save us a lot of time in the long run. I swear.

A Facebook application is actually two applications: **A container application and a web application.**

When we create a Facebook application in the Facebook site what we are actually creating is the container of our web app.

Let's think of our web application like if it were a painting, Facebook itself would be the stand and the container would be the canvas, now let's call the container a [Canvas Application](http://developers.facebook.com/docs/guides/canvas/).

As you can see they both are actually two different things:  

* The stand and the canvas are hosted in Facebook
* Our web application, which is not different than any other application we've made before, is not hosted in any Facebook server; we're going to store it on Heroku.

As far as our customers know, it will look like a single application.  To get these components to work together properly it involves a process of three phases:  

* Creation
* Association
* Configuration

Actually, the association is part of the configuration, but I like to think of the process this way to keep things separate.

##**Creation**

####Facebook Canvas Application

This is the easiest part, you just have to go to the Facebook Developer site and [setup a new application](http://www.facebook.com/developers/createapp.php). 

![Create Application](https://github.com/envylabs/Heroku-Screens-and-Articles/raw/master/fb_tutorial/images/create_fb_app_form.gif)

Input whatever application name you want in the _Application Name_ field, like "My Application", then agree the terms and click the _Create Application_ button, after passing the _Security Check_ you'll be presented with the _Basic Information_ of your brand new application.

![Basic Information!](https://github.com/envylabs/Heroku-Screens-and-Articles/raw/master/fb_tutorial/images/basic_information.gif)

Cool, we've created our [Canvas Application](http://developers.facebook.com/docs/guides/canvas/), now we'd need to associate it with an existing web app.

####Web application
For the moment, we'll assume we already have a web application we'd like to associate with Facebook and we'll assume we are hosting it in **_http://my\_application.heroku.com/_**

###**Association**

_**NOTE**: In case you closed the previous window you can access all your applications in the [My Applications](http://www.facebook.com/developers/apps.php) page._ Click the application we just created and click the **Edit Settings** button.

Now, click the **_Facebook Integration_** button in the settings page. 

![Facebook Integration!](https://github.com/envylabs/Heroku-Screens-and-Articles/raw/master/fb_tutorial/images/facebook_integration.gif)

I'd like you to pay attention to the Canvas section, here we'll find all the settings we need to associate our existing web app with our [Canvas Application](http://developers.facebook.com/docs/guides/canvas/).

* **Canvas Page**  
	- It's an unique access point for your application, say you named your application _My Facebook App_, your Canvas Page might be http://apps.facebook.com/my\_facebok\_app/.
	
	By the way, sometimes you'll have to struggle to find an available name, it's like when you want to buy a .com domain, all the common names are often taken.

* **Canvas URL**
	- This is the URL of our web application, in this case **_http://http://my\_application.heroku.com/.heroku.com/_** (the trailing slash is mandatory).

Now, when our users browses **_http://apps.facebook.com/http://my\_application.heroku.com//_**, Facebook will show them our [Canvas Application](http://developers.facebook.com/docs/guides/canvas/) (an HTML page) which contains an [IFrame](http://www.w3schools.com/tags/tag_iframe.asp) (inline frame) that in turn shows **_http://my\_application.heroku.com/_**. These kind of applications are called **IFrame Canvas Applications**.

![Application in Canvas](https://github.com/envylabs/Heroku-Screens-and-Articles/raw/master/fb_tutorial/images/architecture/application_in_canvas.gif)

_An IFrame Canvas application is actually an IFrame surrounded by the Facebook chrome. The IFrame points to our web app URL._

Let's try it with our new application. 

Without any fear, change the **Canvas URL** to _http://www.heroku.com/_, now open the **Canvas URL** in your browser. What do you see? Yes, the Heroku site, this is why it looks like your web application is inside Facebook.

####**Canvas Type**

* FBML (Facebook Markup Language) Canvas Applications
* IFrame Canvas Applications

As of today, {the date}, Facebook still allow us to create FBML Canvas Applications but they will be deprecated by the end of 2010. [Click here to see the announcement](http://developers.facebook.com/blog/post/402).

###**Configuration**

Now it's time edit all the other settings to customize your application: description, logo, icon, etc. It's a very straight-forward phase, so I won't touch it here.

###**The interaction between Canvas Applications, web applications and Facebook**

Facebook has two mechanisms to let our web applications interact with the data of its social graph, more precisely, with the data of the users of our application:

* [Graph API](http://developers.facebook.com/docs/api) - A powerful yet simple RESTful API
* [XFBML]() - Enables you to incorporate FBML into your websites and IFrame Canvas Applications.

When your application wants to retrieve information from your app users you'll use the [Graph API](http://developers.facebook.com/docs/api) in one way or other.

Here's a diagram of how Facebook applications interact with the [Graph API](http://developers.facebook.com/docs/api)

1. User request http://apps.facebook.com/my_facebook_app  
2. Facebook returns HTML code with the Facebook chrome and the IFrame  
3. The IFrame request the Canvas URL to our server  
4. Our web asks for the current user's name to the Graph API
5. Facebook responds in JSON format
6. Our app builds an HTML response to show the name of the current user and it is shown in the IFrame  

![Graph API interaction](https://github.com/envylabs/Heroku-Screens-and-Articles/raw/master/fb_tutorial/images/architecture/graph_api_interaction.gif)

If our application requests the Graph API via Javascript the communication would be directly with the Facebook servers.  Here's an example:

Our application contains the following Javascript code which is by the user with a button.

    <script>
	      FB.api('/me', function(response) {
	          alert(response["name"]);
	      });
	</script>

Facebook returns the response in JSON and its parsed by the callback function and it alerts with the name of the current user.

![Call Graph API with JS](https://github.com/envylabs/Heroku-Screens-and-Articles/raw/master/fb_tutorial/images/architecture/call_graph_api_with_js.gif)

The Facebook's Graph API
------------------------

The Graph API enables you to read and write objects and connections in the Facebook social graph. It's quite easy to use, and sticks very closely to  [RESTful](http://tomayko.com/writings/rest-to-my-wife) architecture, which you might already be familiar with.

To understand the API, we have to think of every single thing in Facebook as _objects_: photos, events, comments, friends, tags, groups, etc. and we must know that every object in Facebook has an unique identifier, an ID.

Ok, let's query the Graph API with our browser. Click the following link to see the information of the Miles Davis page.

[http://graph.facebook.com/57796847423](http://graph.facebook.com/57796847423)

Now you should be see the response in [JSON](http://en.wikipedia.org/wiki/JSON), most likely something that looks like this:

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

We should get the same response.

The Graph API exposes certain public and private information of the page.  By consulting the [Graph API reference > Page](http://developers.facebook.com/docs/reference/api/page) can learn about additional access points to find different types of information (look under the **Connections** header). 

Once of these connections is called _posts_, let's try it:

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

It's an array of Mils Davis' _Post_ objects, and the [Graph API reference > Post](http://developers.facebook.com/docs/reference/api/post) page says there's also connection called _Comments_ and that I can access with the following URL:

[http://graph.facebook.com/57796847423_126368887412272/comments/](http://graph.facebook.com/57796847423_126368887412272/comments/)

Cool, we are seeing the comments of the post with ID=57796847423_126368887412272.

This is basically how the Graph API works.  To learn more please go to the [Graph API overview page](http://developers.facebook.com/docs/api).

So far we've been accessing public information, and unfortunately if we want to access private information we won't be able to do it this way.  Facebook only allows us to access private information if we have a Facebook application which is explicitly authorized by our users.  

###**Authentication**

Facebook uses the [OAuth 2.0 protocol](http://wiki.oauth.net/OAuth-2) for authentication and authorization.

    Authentications means "It's me, and here's my proof"  
    Authorization means "Now that you know who I am, would you let me do this or that?"  

The [Facebook Authentication](http://developers.facebook.com/docs/authentication/) page explains the authentication process very well, but I'll try to summarize the process with a simple graphic, let's say we have a server running in port 3000:

Register your application to get an app ID and secret

Redirect your users to the following URL:

	https://graph.facebook.com/oauth/authorize?
     client_id=app_id&
     redirect_url=http://localhost:3000/oauth_callback

If the user authorizes your application, Facebook will request the following URL:

	http://localhost:3000/oauth_callback?code=a_code_sent_by_facebook

Exchange the code for an oauth access token in the following URL

	https://graph.facebook.com/oauth/access_token?
	 client_id=...&
     redirect_uri=http://localhost:3000/oauth_callback
     client_secret=...&
     code=...

Facebook will request the callback url with the access token

	http://localhost:3000/oauth_callback?access_token=148664335156575|82da7d8e19b3eba2f0b0ffd9
	
Store it in a DB, cookie or wherever you want, it must be used to make requests on behalf of the user.

![Authorization process](https://github.com/envylabs/Heroku-Screens-and-Articles/raw/master/fb_tutorial/images/architecture/auth_process.gif)

Ok, now our application has been authenticated and the user has authorized it, but you might be wondering what information we have access to.

The answer is: 

    Only the general information: email, name, etc.

This happens when we don't specify what kind of permissions we're asking for, so Facebook sets the default permissions.

Here is how we ask the user for extended permissions:

    https://graph.facebook.com/oauth/authorize?
        client_id=...&
        redirect_uri=http://www.example.com/callback&
        scope=user_photos,user_videos,publish_stream

Once we get the access token our application will be able to access all the photos, videos and even write to the users's wall.

That's basically how authentication and authorization works in Facebook.

Now we know all we need to start creating real Facebook applications begin learning the most important part, how to make them in Ruby (and deploy them on Heroku).

**Making a simple Facebook application**
------------------------------------------

The fun part has finally arrived, we're about to make a simple Facebook application! How simple?  It will show a simple list of links to interesting articles, a Login button, and some Like buttons.  

I'll assume you're using Ruby 1.9.x. and that you have at least [Git](http://book.git-scm.com/2_installing_git.html) 1.6.x installed. [To see a guide to install Git in different platforms, please click here](http://book.git-scm.com/2_installing_git.html).

If you are using any other version of Ruby I'd suggest you to [install Ruby 1.9.x with RVM](http://amerine.net/2010/02/24/rvm-rails3-ruby-1-9-2-setup.html). If you're going to use [RVM](http://rvm.beginrescueend.com/) (Ruby Version Manager) you won't need to prefix the following commands with _sudo_ (Unix-like OSs only).

### Installing the needed gems. 

We'll need to make sure we have bundler:

    $ sudo gem install bundler

We'll also need to install the Heroku command-line tool, a gem that will allow us to create and deploy Heroku applications with ease.

	$ sudo gem install heroku
	
If you haven't signed up for Heroku yet, now would be a good time. [Click here to see the full instructions](http://docs.heroku.com/heroku-command) to do so.

#### What other gems are we going to use?

* [Sinatra](http://www.sinatrarb.com/), a [DSL](http://en.wikipedia.org/wiki/Domain-specific_language) for quickly creating web applications in Ruby.

#### Installing gems with Bundler

Its workflow is very simple:

    $ bundle init
    Writing new Gemfile to ~/simplest_fb_app/Gemfile
	
_Take into account that the gem is called Bundler while the command-line utility is called **bundle**_

Edit the Gemfile with your preferred text editor to let it look like this:

    source :gemcutter

	gem 'sinatra', '1.0'

Now we need to tell Bundler to check if we're missing the gems our application depends on, if so, tell it to install them:
	
	$ bundle check
	The following gems are missing
	...
	
    $ bundle install

If you want to know the advantages of using Bundler please [check this Yehuda Katz' post out](http://yehudakatz.com/2010/04/12/some-of-the-problems-bundler-solves/)

### **We are all set up, let's get started!**

The first step will be to [create a Canvas application](http://www.facebook.com/developers/createapp.php) in Facebook, I'll name it **Faceboku** but you can name it whatever you want to.

These would be our application settings:

**Site URL** = http://faceboku.heroku.com  
**Canvas Page** => http://apps.facebook.com/faceboku/ (instead of _faceboku_ you must use a different one)  
**Canvas URL** &nbsp;= http://localhost:4567 (the standard port used by Sinatra applications)  
**Canvas Type** = IFrame

_If you don't see the Canvas Type option, don't worry, there's a possibility Facebook has got rid of it by the time you're reading this._

We're using **http://localhost:4567** because that way we'll be able to test our Facebook application before we deploy it to Heroku.

What's next? our Sinatra application

Create a directory called _**simplest\_fb\_app**_ wherever you want in your system, I'll do it in my home directory:

	$ mkdir ~/simplest_fb_app

then create the following files and directories on it:

* **faceboku.rb** (you may use the name of your canvas application)
* **config.ru**
* **views/** (folder)
* **views/layout.erb**
* **views/index.erb**

I'll let these files look like:

**faceboku.rb**
	
	require 'sinatra'
	
	#Here you have to put your own Application ID and Secret
	APP_ID = '153304591365687'
	APP_SECRET = '7a7663099ccb62f180d985ba1252a3e2'

	get '/' do
	    @articles = []
	    @articles << {:title => 'Deploying Rack-based apps in Heroku', 
					  :link => 'http://docs.heroku.com/rack/'}
	    @articles << {:title => 'Learn Ruby in twenty minutes', 
	                  :link => 'http://www.ruby-lang.org/en/documentation/quickstart/'}
	    erb :index
	end

**config.ru** like this:

	require 'faceboku' #this is to load faceboku.rb

	run Sinatra::Application

**views/layout.erb**

	<!DOCTYPE HTML>
	<html xmlns:fb="http://www.facebook.com/2008/fbml">
		<head>
			<title>Faceboku</title>
			<style>
				body { font-family: Arial; font-size: 20px ;}
				a { text-decoration: none; color: #333; }
				.article { font-size: 1.2em; }
				.like_app { font-size: 0.8em }
			</style>
		</head>
		<body>
			<%= yield %>
			<div id="fb-root"></div>
			<script>
				window.fbAsyncInit = function() {

					FB.init({
						appId: '<%=APP_ID%>', 
						status: true,         // check login status
						cookie: true,         // set a cookie with the session (access token, user id, etc.)
				        xfbml: true           // parse XFBML (Login and Like buttons)
					});
				};

				(function() {
				  var e = document.createElement('script');
				  e.type = 'text/javascript';
				  e.src = document.location.protocol +
				    '//connect.facebook.net/en_US/all.js';
				  e.async = true;
				  document.getElementById('fb-root').appendChild(e);
				}());
			</script>

		</body>
	</html>

The [FB.init](http://developers.facebook.com/docs/reference/javascript/FB.init) method will initialize our application and it will save a cookie called _fbs\_APP\_ID_ (cookie: true) containing the access token, among other things. If we'd want to perform authorized requests on behalf of our users we'd need to include such access token in our Graph API requests.

**views/index.erb**

	<fb:login-button autologoutlink="true"></fb:login-button>
	<h3>Welcome to my list of interesting articles</h3>

	<table>
		<% @articles.each do |article| %>
		<tr>
			<td class='article'>- <a href='javascript:void(0);'><%=article[:title]%></a></td>
			<td><fb:like href="<%=article[:link]%>" layout="button_count" show_faces="false" /></td>
		</tr>
		<% end %>
	</table>

	<hr />
	<p><span class='like_app'>Do you like this application?</span></p>
	<p><fb:like /></p>

Wow, what an amazing application!, let's run it and browse it!

	$ ruby faceboku.rb

Open your browser and point it to [http://localhost:4567](http://localhost:4567).

![Sinatra application!](https://github.com/envylabs/Heroku-Screens-and-Articles/raw/master/fb_tutorial/images/sinatra.gif)

This is an easy application, it let your users to login with their Facebook credentials, it let them like your links and the application itself.

This is a Facebook application, which means we can see it in action in the Facebook site, let's try it, browse the following URL [http://apps.facebook.com/faceboku](http://apps.facebook.com/faceboku).

![Sinatra application inside Facebook!](https://github.com/envylabs/Heroku-Screens-and-Articles/raw/master/fb_tutorial/images/sinatra_on_facebook.gif)

And there it is, our super application right on Facebook.

Ok, but we can't serve our application from our local machine all the time, we need a server must always be up and running, nothing simpler than Heroku, it'll be a 4 steps process, let's do it:

Assuming we are in our application directory we'll need to turn it into a git one:
	
    $ git init

Add all the files and directories to source control:

    $ git add .

Commit them to your local repository:

    $ git commit -m "First version of my facebook application"

Deploy to Heroku:

	$ git push heroku master
	
Done! Our application has been deployed to Heroku, wouldn't you like to see it right away? simple:

	$ heroku open
	
Cool, it's right there, but wait, our Canvas application is still pointing to our local machine, we now want it to point to Heroku, let's change our Canvas URL:

![Change Canvas URL!](https://github.com/envylabs/Heroku-Screens-and-Articles/raw/master/fb_tutorial/images/change_canvas_url.gif)

Now let's browse to our application [http://apps.facebook.com/faceboku](http://apps.facebook.com/faceboku) and that's it!

[Download application](link_to_download_application_maybe_github)

In Conclusion
-------------

As you can see, building facebook applications with Ruby is simple and hosting with Heroku makes it even easier. However, if you're building a large application you may want to consult a more robust Web framework, other then Sinatra.  The most obvious choice here would be [Rails 3](http://rubyonrails.org/).

###Rails 3 and Facebook integration

I've found that most of the existing gems and Rails plugins are still using [Facebook Connect](http://en.wikipedia.org/wiki/Facebook_Platform#Facebook_Connect) and the [Old REST API](http://developers.facebook.com/docs/reference/rest/) instead of the new [Graph API](http://developers.facebook.com/blog/post/377) and some of them don't even work with Ruby 1.9 and/or Rails 3.

The following are proven to work with Ruby 1.9.x and Rails 3.0.0.

* **Gems**
	- [Cardinal Blue's rest-graph](http://github.com/cardinalblue/rest-graph)
	- [miniFB](http://github.com/appoxy/mini_fb)
	- [FBGraph](http://github.com/nsanta/fbgraph)
	- [Koala](http://github.com/arsduo/koala)
	- [Mogli](http://github.com/mmangino/mogli)

* **Plugins**
	- [Facebooker2](http://github.com/mmangino/facebooker2)

* **Middlewares**
	- [OmniAuth](http://github.com/intridea/omniauth)
	- [rack-facebook](http://github.com/carlosparamio/rack-facebook)
	
###Useful resources

Here's a few other resources you may find useful to get started integrating with Facebook, and hosting on Heroku.

#### Facebook
- [Facebook for Websites](http://developers.facebook.com/docs/guides/web)
- [Apps on Facebook](http://developers.facebook.com/docs/guides/canvas/)
- [Documentation](http://developers.facebook.com/docs/)
- [Developer Forum](http://forum.developers.facebook.net/)
- [Developer Roadmap](http://developers.facebook.com/roadmap)
- [Developer Blog](http://developers.facebook.com/blog/)
- [Test Console](http://developers.facebook.com/tools/console/) - A console to test XFBML and Graph API methods
- [URL Linter](http://developers.facebook.com/tools/lint/) - An Open Graph parser

####Sinatra
- [Sinatra Book](http://sinatra-book.gittr.com/)

####Rails 3
- [Railscasts](http://railscasts.com/tags/27)
- [Dive into Rails 3](http://rubyonrails.org/screencasts/rails3)
- [Rails Tutorial](http://railstutorial.org/)

####Heroku
- [What is Heroku?](http://docs.heroku.com/heroku)
- [Quickstart guide](http://docs.heroku.com/quickstart)