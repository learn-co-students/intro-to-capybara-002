# Intro To Capybara

## Objectives

1. Understand what an integration test is compared to unit and controller tests.
2. Understand how to read a Capybara test of response HTML.
3. Understand how to read a Capybara test that interacts with HTML.
4. Understand how to read a Capybara test that submits an HTML form.

### Advanced Concept Warning

First, Integration Testing and Capybara are very advanced topics in code and it's super awesome amazing that you're reading about it! It's okay if not everything in this lesson makes total sense, we're just trying to introduce you to as much of modern application development as possible. Since we write a lot of our Sinatra and Rails web application tests for Learn using RSpec and Capybara, we thought it might just be helpful to give you some context with which to read the tests and their output. You're awesome, we love you.

## What is an Integration Test
There are three basic levels of testing that correspond to the different levels of our application stack in a Model-View-Controller architecture.

![Web Application Stack and Tests](https://dl.dropboxusercontent.com/s/k2ypcn86btb6ajo/2015-09-29%20at%204.14%20PM.png)

Don't worry if you don't know exactly what a model or a controller or a view are, you probably don't need too in-depth of an understanding to continue this lesson and learn a bit about Capybara. If you're curious, the following definitions should be enough for now:

**Database:** Databases persist or save data from our application. An application backed by a database can better manage huge volumes of data, relate the data to each other, search for specifc data, and more. Best of all, if our application crashes, the database will still store the data so when we fix our application and start it up again, it'll have all the data it used to because the database was saving it seperately.

**Models:** Models provide an object-oriented abstraction or metaphor or object for the data in our application. Whether stored in a database or just in-memory, any unit of code, generally a class, whose primary responsibility is dealing with data, whether reading it, or writing it, or searching it, is considered a model. You've probably built a bunch of models already, even if they weren't attached a database.

**Controllers:** Controllers provide the main interface and application logic. They deal with things like "Where should the data for this feature come from?" or "What should happen if the CLI receives "browse" input from the user?" or "What HTML should be sent to the user when they visit the /about page?". In large scale MVC applications, controllers are represented by classes, but really, lots of `bin` files or procedure oriented methods, the ones that seem to just call a bunch of other methods, could be considered controllers.

**Views:** Any code that is responsible for presenting data or output to the user, from methods that use a bunch of `puts`, to HTML, to ERB templates, could be considered a View. Views present information to the user. They provide an interface and context for the user to interact with our applications. In Web Applications, Views are generally represented by ERB templates that generate HTML.

**User/Browser:** The top of our application pyramid is finally the user. Whether describing the person using our application, the interface they use such as a Command Line, Voice, or HTML, or the program they use to even access our application, such as a Browser like Chrome or Safari, or a Native app, our application is responsible for delivering the user an experience on some sort of pre-existing platform.

### Unit Tests

All the way at the bottom of our application is our database. We rarely interact with the database directly and generally build objects, called models, that encapsulate database interactions. These units of work that are model oriented and provide interfaces for reading and writing data to the database get tested via Unit tests. Unit tests should isolate the expected data behavior regardless of interface.

### Controller Tests

A level up from our database and models are our controllers. In a web application controllers are responsible for responding to the request by assembling all the data and HTML that will compose our response. We test controllers with controller tests. A controller test should be solely responsible for making sure that an HTTP request returns the expected HTTP response. Controller tests should not test HTML or forms, but rather, that the controller behaved as expected.

### Integration Tests

Integration tests are the highest-level of test, they are closest to describing how a user will actually interact with our application. Commonly referred to as End-To-End Tests, Integration tests should flex your entire application stack and rarely isolate components or behaviors. They are perfect for spec'ing high level user interactions with HTML and forms. We're going to be learning how to write and read an integration test using a library called Capybara within an RSpec test suite.

*Note: You will generally not be required to write tests in Learn. However, you will be required to read a test and understand what functionality the test suite is anticipating and testing. If you can write a test, we believe you can read a test.*

## Integration Testing with Capybara

Consider a simple Web Application that shows the user a form asking for their name and then when they submit the form, the application will greet the user based on the name they inputted.

The Form at GET "/"
![Form](https://dl.dropboxusercontent.com/s/1zocl86jv9qguth/2015-09-29%20at%206.00%20PM%20%281%29.png)

After submitting the form, the response at POST '/greet'
![Response](https://dl.dropboxusercontent.com/s/83o4onopkwquuve/2015-09-29%20at%206.01%20PM.png)

Colloquially we could express the tests for this application as follows:

```
When a user visits '/'
  they should see a greeting
  they should see a form with a name field

When a user submits the greeting form
  they should fill in the name with "Avi"
  they should click submit
  they should see "Hi Avi, it's nice to meet you!"
```

It is exactly these sort of behaviors, conditions, and expectations that Capybara make very easy to build, read, and test.

### Capybara Setup

Using Capybara for integration tests in Rails or Sinatra (or really any rack based framework) just means modifying our `spec/spec_helper.rb` or our testing environment to actually load capybara.

File: `spec/spec_helper.rb`
```ruby
# Load RSpec and Capybara
require 'rspec'
require 'capybara/rspec'
require 'capybara/dsl'

# Configure RSpec
RSpec.configure do |config|
  # Mixin the Capybara functionality into Rspec
  config.include Capybara::DSL
  config.order = 'default'
end

# Define the application we're testing
def app
  # Load the application defined in config.ru
  Rack::Builder.parse_file('config.ru').first
end

# Configure Capybara to test against the application above.
Capybara.app = app
```

With a lot of syntax, all that code is doing in our `spec_helper` is configuring `RSpec` and our tests to be able to use all the wonderful methods and interactions Capybara provides.

The most important part of the configuration is the last line where we explicitly tell `Capybara` that the `app` we're testing against is defined in `config.ru`.

### Our Application and Tests

Now that our test suite is setup to use Capybara, let's start writing some tests for our application. Everything is already built and included with this lesson if you fork and clone it or you can rebuild it on your own by copying the code into the files.

#### GET '/' - Our index with the greeting form.

![Form](https://dl.dropboxusercontent.com/s/1zocl86jv9qguth/2015-09-29%20at%206.00%20PM%20%281%29.png)

We need to build a Sinatra route to respond to requests at `GET '/'`. We know that when that page is working, it'll at least include the text `Welcome!` in an `<h1>` tag.

Let's write this test in `spec/application_integration_spec.rb`.

File: `spec/application_integration_spec.rb`
```ruby
require 'spec_helper'

describe "GET '/' - Greeting Form" do
  it 'welcomes the user' do
    visit '/'
    expect(page.body).to include("Welcome!")
  end
end
```

Most of that code is actually vanilla RSpec. Capybara provides two new methods, `visit` and `page`.

##### Capybara `visit`

The `visit` method navigates the browser used during the test to a specific URL. It is equivalent to a user typing a URL into their browser's location bar.  The argument it accepts is a string for the URL you want to test. Since we want to test our `'/'` root URL, we say `visit '/'` and Capybara will load that page within our test.

##### Capybara `page`

The `page` method in capybara exposes the "session" or "browser" that is conceptually (and literally) being used during the test. The `page` method gives you a `Capybara::Session` object that represents the browser page the user would actually be looking at if they typed in `'/'` (or whatever argument you last passed to `visit`).

As such, `page` responds to a lot of methods that represent actions a user would do on a page, such as `click_link`, `fill_in`, and `body`.

The `page.body` will basically dump out all the HTML in the current page as a string.

Our test is telling Capybara to visit the `/` URL. It then sets the expectation that the body of the page returned should include at least the text `Welcome!` somewhere.

#### Passing the First Test

File: `./app.rb`
```ruby
class Application < Sinatra::Base
  get '/' do
    erb :index
  end
end
```

`./app.rb` is our main application file, defining the controller that will power this web application. We create a class `Application` and inherit from `Sinatra::Base` to give this class all the web super-powers needed to transform the ruby class into a sinatra controller.

Within our `Application` controller, we use the Sinatra DSL to create a `GET` route at the `/` URL. We tell our application that in response to HTTP GET requests to `/`, render the ERB template or HTML as the response. We're just saying that the content for this response, what to send back to the user's browser, is located in a file `views/index.erb`. This is a Sinatra provided functionality to render ERB templates located in the `views` directory. Let's look at the file.

File: `views/index.erb`
```erb
<h1>Welcome!</h1>
```

Our ERB view `views/index.erb` (Sinatra will automatically look for the `.erb` extension when you call `erb` in the controller), just contains HTML to make our test pass.

The final piece of the puzzle is a `config.ru` file to put everything together and start our Sinatra application. This is the file `shotgun` or `rackup` will read to start your local application server and it's also the file our test suite is using to define our application, remember `Rack::Builder.parse_file('config.ru').first`?

File: `./config.ru`
```ruby
require 'sinatra'

require_relative './app'

run Application
```

In a basic application like this example, our `config.ru` `require`s the `sinatra` gem. It then `require_relative` the required file `app.rb` that defines our main `Application` controller. Finally, we `run` our `Application` controller to start our web application.

If we now start our application with `shotgun` and open `http://localhost:9393` in our browser we'll see our welcome message.

![Welcome](https://dl.dropboxusercontent.com/s/bue5icj3yuz6iol/2015-09-29%20at%208.56%20PM.png)

And when we run our test suite, we'll see:

```
$ rspec

GET '/' - Greeting Form
  welcomes the user

Finished in 0.03438 seconds (files took 0.43261 seconds to load)
1 example, 0 failures
```

If you an error output, don't worry, read it and see if you can figure out what's going on and fix it. If not, just move on, **this lesson is about understanding how to read a Capybara test**, not write them or debug them.

#### Adding the Greeting Form Requirement

Great, step 1, get a basic test and application working, is done. Let's now add tests for the HTML form that will allow a user to provide their name for the greeting. When our tests pass, our form at `/` will look like:

![Form](https://dl.dropboxusercontent.com/s/1zocl86jv9qguth/2015-09-29%20at%206.00%20PM%20%281%29.png)

So let's describe an expectation for that HTML in our test.

Edit: `spec/application_integration_spec.rb`
```ruby
require 'spec_helper'

describe "GET '/' - Greeting Form" do
  # Code from previous example
  it 'welcomes the user' do
    visit '/'
    expect(page.body).to include("Welcome!")
  end

  # New test
  it 'has a greeting form with a user_name field' do
    visit '/'

    expect(page).to have_selector("form")
    expect(page).to have_field(:user_name)
  end
end
```

Our new test has a similar setup to the first test, we need to tell capybara to visit the page at '/'. Once that is done, we can set some expectations against the `page` object that represents the user looking at the homepage in their browser. We can simply assert that the `page` has an HTML selector for `form`, meaning that the page contains an HTML element that matches the `form` tag.

Where does this magic `have_selector` matcher come from? That's right, Capybara has added that to RSpec. Capybara `page` objects respond to methods that relate intimately to HTML and the DOM (Document Object Model) that defines web applications. You can literally as the `page` object, hey, do you have HTML that matches the following selector? Pretty amazing, right?

The second expectation is similar, `expect(page).to have_field(:user_name)
`. We're saying that we expect the `page` to have a form field called `user_name`. We get to be even more semantic with the `have_field` matcher that will make sure the HTML in `page` contains a form input with either an `ID` or `name` attribute that matches the argument, in this case `:user_name`.

After editing the integration test and saving it, if we run our tests according to the current code, we'll see:

```
rspec

GET '/' - Greeting Form
  welcomes the user
  has a greeting form with a user_name field (FAILED - 1)

Failures:

  1) GET '/' - Greeting Form has a greeting form with a name field
     Failure/Error: expect(page).to have_selector("form")
       expected #has_selector?("form") to return true, got false
     # ./spec/application_integration_spec.rb:12:in `block (2 levels) in <top (required)>'

Finished in 0.03963 seconds (files took 0.42466 seconds to load)
2 examples, 1 failure

Failed examples:

rspec ./spec/application_integration_spec.rb:9 # GET '/' - Greeting Form has a greeting form with a name field
```

Our first test for welcoming the user still passes, but our new test fails. Let's zoom in on the meaningful part of the failure message.

```
Failure/Error: expect(page).to have_selector("form")
  expected #has_selector?("form") to return true, got false
```

The failure message is telling us that we expected the page to have an HTML form and it doesn't, so it's failing. Let's add our HTML form to our view and make this test pass.

Edit: `views/index.erb`
```erb
<h1>Welcome!</h1>

<form action="/greet" method="POST">
  <label for="user_name">Name:</label>

  <p><input type="text" name="user_name" id="user_name" /></p>

  <input type="submit" value="Submit"/>
</form>
```

Don't worry about the specifics of the HTML form, but know that it is building an HTML form that when submitted by the user clicking on the Submit button, will create an HTTP POST request to `/greet`, submitting whatever the user typed into the form text `<input>` field nested in the form that happens to be `name`d `user_name` (I know confusing, but it's `name` attribute is equal to `user_name`).

Run your tests now or reload your browser and you should see the form and your tests passing.

#### Processing the Form and Greeting the User

The final step of our greeting application is to teach our application how to accept the form we previously built when the user actually fills it in and clicks submit. We know that we want to outcome to be a custom greeting message based on what they type into the `user_name` field in the form. If you typed in `Avi` and clicked submit, we'd expect the page in our browser to look like:

![Response](https://dl.dropboxusercontent.com/s/83o4onopkwquuve/2015-09-29%20at%206.01%20PM.png)

Let's write those tests.

Add the following to the end of: `spec/application_integration_spec.rb` (under the closing `end` of the first `describe` block).

```ruby
describe "POST '/greet' - User Greeting" do
  it 'greets the user personally based on their user_name in the form' do
    visit '/'

    fill_in(:user_name, :with => "Avi")
    click_button "Submit"

    expect(page).to have_text("Hi Avi, nice to meet you!")
  end
end
```

This new test is trying to mimick a user visiting our greeting form, filling in their name, clicking the submit button, and what they should see in response. Because of the amazing `RSpec` DSL mixed in with `Capybara`, our test basically just says that.

We `visit '/'` to load the form into the `page` object.

Then we use the Capybara method `fill_in` to fill in the input field `user_name` with `Avi`.

Finally, we `click_button "Submit"` to submit the form. That HTML interaction, submitting a form, will trigger a new HTTP request in the Capybara session and `page` object. Just like when you submit a form, the browser loads a new page and you see new content, when Capybara submits a form, just like when we call `visit`, the `page` object is appropriately updated. `page` no longer contains the original greeting form, but rather, after `click_button`, `page` now has the response to the greeting form.

For the response to submitting the greeting form, we can `expect` the `page` to `have_text` `"Hi Avi, nice to meet you!"`. The user filled in their user_name as "Avi", the resulting greeting should mention that. `have_text` is another really friendly and semantic Capybara matcher for testing HTML text value explicitly.

After adding this test, if we run our test suite, we'll see:

```
$ rspec

GET '/' - Greeting Form
  welcomes the user
  has a greeting form with a user_name field

POST '/greet' - User Greeting
  greets the user personally based on their user_name in the form (FAILED - 1)

Failures:

  1) POST '/greet' - User Greeting greets the user personally based on their user_name in the form
     Failure/Error: expect(page).to have_text("Hi Avi, nice to meet you!")
       expected #has_text?("Hi Avi, nice to meet you!") to return true, got false
     # ./spec/application_integration_spec.rb:24:in `block (2 levels) in <top (required)>'

Finished in 0.05962 seconds (files took 0.42668 seconds to load)
3 examples, 1 failure

Failed examples:

rspec ./spec/application_integration_spec.rb:18 # POST '/greet' - User Greeting greets the user personally based on their user_name in the form
```

Our first two tests relating to `GET '/'` pass, but our new test is failing. Zooming in on the failure:

```
1) POST '/greet' - User Greeting greets the user personally based on their user_name in the form
   Failure/Error: expect(page).to have_text("Hi Avi, nice to meet you!")
     expected #has_text?("Hi Avi, nice to meet you!") to return true, got false
```

When submitting a POST request to `/greet`, the same configuration we programmed into our HTML form, if the `user_name` was filled in with `Avi`, we expected the resulting page to contain the text `"Hi Avi, nice to meet you!"`. Unfortunately, at this point, it does not.

To make this pass we need to add two things. Currently, our Sinatra application only responds to a single HTTP request, `GET` requests to `/`. We need to teach it to respond to `POST` requests to `/greet`. Let's do that.

Edit: `./app.rb`
```ruby
class Application < Sinatra::Base
  # Old route from previous tests
  get '/' do
    erb :index
  end

  # New route to respond to the form submission
  post '/greet' do
    erb :greet
  end
end
```

Using the Sinatra `post` method, we create a response for requests to `POST '/greet'`. That response should be the HTML contained in the `views/greet.erb` template, just like the HTML response of our first route was contained in `views/index.erb`.

The next step is to build our view in `views/greet.erb`. The point of this view is to use the data the user submitted in the previous form within our HTML to produce a personalized greeting for the user.

File: `views/greet.erb`
```erb
<h1>Hi <%= params[:user_name] %>, nice to meet you!</h1>
```

That HTML and ERB will satisfy our test.

If you are unfamiliar with the `params` object and how it relates to froms and inputs, that's totally fine. The tl;dr is that all the information the user submitted in the form is available to your code within a hash `params`. `params[:user_name]` returns the text the user put into the form input field `user_name`. `<%= params[:user_name] %>` uses ERB's embedded ruby to dynamically insert the value of `params[:user_name]` into the HTML of the response.

With this code in place, our tests should pass and if we refresh our browser and submit the form it should behave as expected.

That's it, you totally crushed Capybara and Integration testing.

## Summary

We learned about:

1. Integration Tests - Tests that exercise the outermost part of our application code from the perspective of our user, actually interacting with the application and interface they interact with, like a web browser and HTML.
2. Capybara - A ruby library that works with RSpec to allow you to write extremenly powerful, simple, and semantic Integration Tests for Web Applications.
3. `visit`, `page` - Capybara methods for controlling the test user's browser and introspecting on the current state of the page they are looking at.
4. `have_selector`, `have_field`, `have_text` - Capybara matches for ensuring that the page contains certain matching HTML or text.
5. `fill_in`, `click_button` - Capybara methods for mimicking user activity like filling in form fields or clicking buttons.
6. A simple Sinatra Form Application - We also built a simple sinatra application that has two routes, a GET and a POST, displaying a form, accepting the form input, and sending a dynamic response.

Great job!
