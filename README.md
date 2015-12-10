
#**MOVE.BG** project specification
-------

##architecture

Wordpress is used for administration of content published by the move.bg team.
Wordpress is extended with custom admin functionality to meet the move.bg-specific content needs & structure.
Content from wordpress is retrieved via wp-api plugin. We are not relying on wordpress for anything else except the content published by the move.bg team.
We want to have a separate multi-page application (MPA) with its own routing, token-based authentication, and a few custom features, so we can have more freedom in extending this application with custom functionalities now and in the future. Trying to inject everything in the wordpress ecosystem is more expensive and developer-hostile in the long run.

##taxonomy

###post
move.bg is basically an interactive media, so their primary entity is a post. This post is mostly like a standard wordpress post with an author, html content, featured image and tags. 
Extra fields are: 

* subtitle
* content-type (live debate, online discussion, expert opinion, etc)
* primary parent - a program or a rubric
* secondary parents - an array of programs or rubrics

---

###program
move.bg creates their own prototypes for reform they call "programs". When admins create a post from WP, they can associate that post with one of their existing programs. A program, is an entity that has:

* name
* short description
* branded square image
* cover image
* optional primary post associated with it (people go to this post when they click "learn more about the program")

---

###rubric
move.bg has a series of branded initiatives. for example "добрият пример", "челен сблъсък", "въпрос на месеца". we call those rubrics. a rubric is an entity identical to the program in every aspect. When admins create a post they can associate it with a rubric.

---

---

---


##routing

####/ (home)
a list of the latest posts

####/programs
* a list of all programs
* a list of posts associated with a program

####/rubrics
* a list of all rubrics
* a list of posts associated with a rubric

####/programs/:program-name
shows the program name, short description, featured image, "learn-more" button and a list of posts associated with the program

####/rubrics/:rubric-name
shows the rubric name, short description, featured image, "learn-more" button and a list of posts associated with the rubric

####/:post-slug
for example:

	http://move.bg/obama-did-something-funny
It is important for me that we have this clean, minimalistic routing. In the wordpress backend we will need to perform an additional check against static route names (rubrics, about, contact, etc) and handle post slug generation accordingly.

####/:static-page-name
* about
* contact
* donations
* testimonials


*that is all there is to routing!*

---


##api.move.bg

###overview

This is our backend that is separate from the WP setup. We seek the liberty to feely extend the functionality of our application outside of wordpress while minimizing developer cost and maximizing development friendliness.
Most of the endpoints are just funnelling data from wp to the user.
The API is used in two ways.

* through javascript, after a view has been loaded
* through the view when it is being requested

so for example, if the browser hits:

	move.bg/rubrics/chelen-sblasak

the view needs to send a request to:

	api.move.bg/rubrics/chelen-sblasak
which would return the rubric model. and also to:

	api.move.bg/posts/?parentSlug=chelen-sblasak
The data from both endpoints needs to be served to the view along with the html template.

###authentication
We need a single sign on solution that would be integrated with facebook, google & disqus.
SSO is very important because we want to use it in other move.bg projects as well.
This authentication is separate from the wordpress authentication system. Move.bg admins will access wp separately.

###endpoints

####/posts

routeParams (all optional):

**orderBy** - "latest"(default), "popular"
**parentType** - "any"(default), "program" or "rubric"
**parentSlug** - "" (default), slug(id) of a program or a rubric
**offset** - 0 (default) number of posts to skip
**limit** - 10 (default) number of posts to show

endpoint returns data: 

	[{postObject},{postObject}]

and headers:

* TOTAL-LENGTH (of posts)

**postObject** contains:

* title
* subtitle
* contentType
* parent (a program or a rubric object)
* author (an object with name, photo & link to profile)
* featuredImage
* viewCount
* commentsCount
* specialContent (an array of strings designating special content like "voting", "video", etc)

basically everything except the content

####/posts/:post-slug

returns an **extendedPostObject** that also includes:

* html content


####/parents

returns an array of all rubrics and programs


####**POST** /subscribe/:parent-slug
The user can subscribe to a rubric or a program.
**requires authentication**

#### /polls/:post-slug
Returns the poll objects for a given post.

####**POST** /polls/:post-slug/:poll-id
Posts the user answer to a poll. This can also work as editing the answer.
**requires authentication**

#### /support-widgets/:post-slug
Returns the support widget objects for a given post.

####**POST** /support-widgets/:post-slug/:support-widget-id
User "likes"/"supports" something that is presented in the widget.

#### **DELETE** /support-widgets/:post-slug/:support-widget-id
User removes their support.



---

##MPA requirements

* routing setup
* viewData served in views
* handlebars or similar template engine to be implemented

---

##Wordpress admin features

###1. Programs tab
This is a custom post type. Admins can create new programs with titles, descriptions, featured images & branded square images. They can optionally select a post to be associated as a primary post for the program.

###2. Rubrics tab
Identical with the programs tab.

###3. Extend Posts

* subtitle
* primary parent (select rubric or a projects)
* secondary parents (select multiple rubrics or projects)
* widgets to be inserted (still unsure what is the best way to do that)


###4. Extend Authors
* social profiles
* photos
* job titles
* bios

---

##Other admin features 

We need an admin interface to manage custom widgets & featues like the polls & the support widgets. We will have more such features in the future.
We also need to have an interface where the admin can explore the user database and make queries for segmentation, so that they can get a list of user emails & data, that they can then use for mass emails.




