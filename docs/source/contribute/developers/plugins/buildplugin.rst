How to Build a ThinkUp Plugin
=============================

First, run the script which will automatically generate the required folder structure and default files with some
standard methods every plugin will need.

Navigate to ThinkUp's root directory in a terminal and type the following command:

::

    ./extras/dev/makeplugin/makeplugin NameOfYourPlugin

Where NameOfYourPlugin is the name of the plugin you want to create, e.g. Twitter, Facebook etc. Once the script is
done, navigate to the webapp/plugins/ folder, and you'll see a newly-created folder there which contains all your
new plugin's files.

If you load ThinkUp and go to the Settings area, you'll see your plugin listed there with the default plugin icon.
Activate your new plugin and click on its name to view its settings page.

ThinkUps .gitignore file ignores all plugins by default to enable development of non core plugins. So you will need to
add an exception of the form:

!webapp/plugins/your_plugin_name

to the .gitignore file.

Icons
=====
You will need to add two icons that represent the service you are making the plugin for to the plugins assets folders,
these images are normally the same as the services favicon.

Image 1 should be 48x48 pixels and called service_name_icon.png

Image 2 should be 16x16 pixels and called favicon.png

Setting Various Details
=======================

You then need to edit plugin_name.php in the plugins controller file and set the description (line 5), this is what
will appear below the plugin name on the settings page. This is normally 1 sentence describing what the plugin
captures and displays for example:

Capture and display YouTube videos.

Then set the path to the icon (line 7) and author (line 9).

In this file on line 44 you can specify if this plugin runs before or after the insight generator plugin by setting the
second parameter of registerCrawlerPlugin() to true to run before the insight generator plugin and false to run after
it.

Then edit account.index.tpl in the plugins view folder and set the filename of the plugins icon on line 5.

You then need to set the class descriptions in all of the autogenerated files.


Retriving OAuth Tokens
======================
The first class to write is the PluginConfigurationController.

This presents a form to the user where they can enter any details needed to carry out authentication.

Start by creating a file in the docs/source/userguide/settings/plugins/ folder called plugin_name.rst

And then write a step by step guide for the user on how to configure the plugin and include information on what data
the plugin captures.

You can then link to this guide by editing the path on line 47 to:

userguide/settings/plugins/plugin_name

and you'll probably want to remove the message on line 45.

The HTML for the configuration page can be found in view/account.index.tpl, you'll probably want to copy a template
for this from one of the existing plugins.

This page needs to tell the user what the plugin does and provide them with some basic instructions on how to configure
it.


The input boxes shown to the user are controlled by the authControl() function in the PluginConfigurationController.
You should also add the names of any required parameters to the constructor of the [PluginName]Plugin.php class along
with the name of the folder that this plugin lives in.

With the ability for the user to enter the details you require to retrieve the OAuth tokens you then need to write the
code to retrieve them, this is normally done in a function called setUpPluginNameInteractions() in the
PluginConfigurationController class.

The first thing you need to do in this function is to retrieve the values of any form data the user has entered
from the options array that it passed in.

You should then create the redirect URI that the authenticating service will send the user back to and then build the
link the user needs to go to in order to retrieve the OAuth tokens.


You now need to handle the user being redirected back to ThinkUp with the OAuth tokens. When this happens you will need
to exchange the client id, client secret and the code you get back from the service for an OAuth token and possibly also
a refresh token. The function to do this should be located in the plugins crawler class.

At this point you will need to start making requests to the services API, all API calls should be made through the
APIAccessor class located in the model folder.

This normally has 2 functions one which makes a GET request to the API and one which makes a POST request.

With the tokens obtained you will need to get some basic information about the user so you can store the OAuth tokens
in the database you should atleast obtain a relevant user id, user name and full name for the user.

Storing the access tokens is normally handled by the saveAccessTokens() function. In here you should check if an
instance of this plugin already exists for the user ID and if so check if an owner instance exists or not and insert
the required data accordingly.

If an instance does not exist you will have to create one and well as an owner instance.

Finally you will need to insert the user into the database if they don't already exist.

Getting the Data
================

With the ability for the user to configure the plugin completed, it is now time to work on getting the data from the
service.

The first thing to consider is what data you want to collect and how this fits into ThinkUps current data model. You'll
probably want to discuss the data you want to collect and how you will store it on the developer mailing list before
you start writing the code.

With the data you will collect and how it will be stored decided you can begin writing the PluginNameCrawler class.

First create some global variables in the constructor. You'll want an instance, logger, access token and API Accessor.

Next fill in the initializeInstanceUser method, this method will be called before every crawl and its main purposes are
to verify that the OAuth tokens you have are still valid and if not refresh them and retrieve more detailed information
about the instance user in the process.

Then it is time to write the main method for your plugin, the one which actually goes out to the service and retrieves
the data. This method is generally called fetchInstanceUserPosts/Videos/Images, which ever is appropriate for your
plugin.

This method will normally page through results from the API and store them in the database.

If you are capturing comments / replies for a post, don't forget that you will need to also store details of the user
who made the comment / reply in the database.


Tying It All Together
=====================

You can now tie all of your work together with the final class to write the PluginNamePlugin class.

This class has a function called crawl() that tells ThinkUp what to do when the user initiates a crawl.

In this method you will need to get the plugin options so you have access to the OAuth tokens  and then retrieve the
logged in user from the database.

Next get the instances for this plugin for the logged in user and then crawl for each one of them.

This normally involves first checking the OAuth tokens are still valid and then calling your main crawling method.

Testing Your Plugin
===================
ThinkUp uses a test driven development approach and so you must  write tests to prove the correctness of your code.

All API calls should be intercepted and handled locally, you can do this by writing a mock APIAccessor, the basis for
it can be found in the tests/classes folder.

This will then need to be included in any test files which test a class that makes external API calls.

Data that the real API would return should be stored in the /apidata folder. The file name should be the url that the
real call would go out to with & replace by - and the prefix common to all calls removed.

The easiest way to create these file is to set CREATE_FILES=1 the first time you run your tests. This will then
create blank files with the appropriate names in your apidata folder. You then just need to place the create data
the real API would return in these files.

Setting TEST_DEBUG=1 can also be useful when running the tests as you will be able to see which local files are being
read and print out debugging statements.


When testing the plugin configuration controller it is important to note that your new plugin will not be registered
automatically so in the buildPluginOptions() and buildController() functions you will need to register your plugin
manually like so:

$builder_plugin = FixtureBuilder::build('plugins', array('name' => 'youtube', 'folder_name' => 'youtube',
'is_active' => 1) );
// Set the plugin ID (the id of the last insert to the database (the call above) )
$plugin_id = $builder_plugin->columns['last_insert_id'];


The test class for the PluginNamePlugin is normally quite short and tests that the constructor works correctly.

You may encounter issues where the apidata files have names that are over 200 characters long, this limitation exists
to enable people to run the test suite on Windows. To work around it you should

1) Try to reduce the filenames size, if you insert IDs into the filename from data obtained by previous API queries
you can modify the data returned and shorten the values of parameters returned.

2) If the filenames are still too long you will need to hash them using something like MD5 which outputs 32 character
strings. Detect if a call to a URL which is too long is about to be made in the mock API Accessor and then replace the
URL with the hashed version of the filename, and rename the file to its hashed name.


Another potential issue is that your API calls may use dynamically changing values based on things like the date. To
work around this detect if a call to a URL which has dynamically changing values is about to be made in the mock API
accessor and replace the dynamically changing value with a constant. Make sure to rename the APIdata file to have this
constant value also.

You will need to add the names of your new test files to the tests/all_plugin_tests.php file and the init.tests.php file
.
