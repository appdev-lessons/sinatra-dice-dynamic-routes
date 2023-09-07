# Dynamic path segments

In this project, we're going to improve the code for our dice rolling web app even more. We're going to add support for _all possible combinations_ of number of dice and type of dice — in other words, our app will be able to respond to an _infinite_ variety of HTTP requests! We'll pull this off by learning about **dynamic path segments**.

---

[Here is a video for this lesson](https://share.descript.com/view/qO2C2MH8le0). You should not rely entirely on the video. PLEASE READ the below lesson as you are going through the steps, since there is much more detail in the text than in the video. **Note:** In the lesson below there is now a target app and `rake grade` specs, which were not in the video.

## Create a codespace

We'll use the `appdev-projects/sinatra-dice-dynamic` repository for this project. 

This project includes automated tests, so click on this button to get started:

LTI{Load Sinatra Dice Dynamic assignment}(https://grades.firstdraft.com/launch)[S9ymPy6WCsn18gLbByVbZQ7k]{vfdtzJb5bLYqYwuqgeRKpc5d}(10)[Sinatra Dice Dynamic Project]

Then, fork the repository and create your codespace.

The starter code is the solution to the sinatra-dice-roll project. In other words, the starter code _already matches our target_:

[dice-roll.matchthetarget.com](https://dice-roll.matchthetarget.com/)

We've also added all the standard Sinatra boilerplate that we'll be using in all of our apps moving forward (`sinatra/reloader`, `better_errors`, a `views/` folder, a default `layout.erb` file, etc).

First `bin/dev`, and click around the live app preview to reacquaint yourself with it.

Then, study the starter code and make sure you understand it. Ask questions if not!

## Objective

In this project, we're going to add support for HTTP requests of the form:

```http
GET /dynamic/6/19 HTTP/1.1
```

```http
GET /dynamic/20/4 HTTP/1.1
```

```http
GET /dynamic/42/1337 HTTP/1.1
```

In general, we're going to support _all_ requests that match the **pattern** of:

```http
GET /dynamic/ANY_INTEGER/ANY_OTHER_INTEGER HTTP/1.1
```

Where `ANY_INTEGER` and `ANY_OTHER_INTEGER` are placeholders for any number.

That means we're going to support an _infinite_ variety of requests! It would be a lot of work to write an infinite number of routes in `app.rb`, so we're going to learn how to respond to all requests of that pattern with just one route — through the magic of **dynamic path segments**.

Go ahead and visit some of these routes in our target, you'll see they all work:

[dice-roll.matchthetarget.com/dice/6/19](https://dice-roll.matchthetarget.com/dice/6/19)

[dice-roll.matchthetarget.com/dice/20/4](https://dice-roll.matchthetarget.com/dice/20/4)

[dice-roll.matchthetarget.com/dice/42/1337](https://dice-roll.matchthetarget.com/dice/42/1337)

Run `rake grade` at a bash prompt and have a look at the specs; all but the specs associated with the `"dice/50/6"` route should be passing. That's because our goal is to:

1. extend our app to support _any_ request that matches the pattern; and
2. in doing so, not break the functionality that we already have.

Let's get started, and make our app match the target!

## Add /dynamic/50/6

In your live app preview, try visiting the path `/dynamic/50/6`. You should get the familiar "Sinatra doesn't know this ditty" 404 page, since we haven't set up a route for that path yet.

Now, let's make it work; but since it's so many dice (50 of them!), it would be tedious to create individual variables for each one. Instead, let's use [an `Array` and a loop in the view template, like we learned here](https://learn.firstdraft.com/lessons/105-sinatra-view-templates#looping-in-view-templates):

```ruby
# /app.rb

get("/dynamic/50/6") do
  @rolls = []

  50.times do
    die = rand(1..6)

    @rolls.push(die)
  end

  erb(:flexible)
end
```

```erb
<!-- /views/flexible.erb -->

<h1>50d6</h1>

<ul>
  <% @rolls.each do |a_roll| %>
    <li>
      <%= a_roll %>
    </li>
  <% end %>
</ul>
```

Re-visit the path `/dynamic/50/6` in your live app preview and verify that it works now. Is anything fuzzy about the action and view template? If so, now's a great time to ask questions about it!

---

**Checkpoint:**

- [Here you can see the changes that I've made since the last commit.](https://github.com/raghubetina/sinatra-dice-dynamic/commit/0091e405e085d54891c44f4c1893ab288ac84e36)
- [Here you can browse my entire codebase at this point in time.](https://github.com/raghubetina/sinatra-dice-dynamic/tree/0091e405e085d54891c44f4c1893ab288ac84e36)

## Making the route dynamic

In your live app preview, try changing the `50` in the second segment of the path to `100`, i.e. `/dynamic/100/6`, and re-submit the request. You'll see the familiar "Sinatra doesn't know this ditty" 404 page, since the request no longer matches any route that we have.

Rather than adding another route to solve this, let's make our existing route **dynamic**. Modify the route to look like this:

```ruby
# /app.rb

get("/dynamic/:number_of_dice/6") do
  @rolls = []

  50.times do
    die = rand(1..6)

    @rolls.push(die)
  end

  erb(:flexible)
end
```

Try visiting the path `/dynamic/100/6` in your live app preview again, and you should see that _it doesn't cause a 404 error anymore_. The request for:

```http
GET /dynamic/100/6 HTTP/1.1
```

is being **matched** by the route:

```ruby
get("/dynamic/:number_of_dice/6")
```

Try removing the colon (`:`) at the beginning of the second path segment:

```ruby
get("/dynamic/number_of_dice/6")
```

And then try re-submitting the request in your live app preview again. Now it's back to the 404 page.

Now put the colon (`:`) back, and try refreshing again. No more 404.

What's going on here? If we start a path segment with a `:`, then it becomes a **dynamic path segment**. Then, that segment is like a wildcard; _anything in that segment will count as a match_.

You can visit `/dynamic/zebra/6`, `/dynamic/giraffe/6`, or `/dynamic/elephant/6` and they will all be routed to the same action, which is currently displaying 50 six-sided dice.

The first and third segments — `/dynamic` and `/6` — are _not_ flexible, so the user can't put anything they want in those segments. The request must literally match, exactly, in those segments. So if you visit `/zebra/100/6` or `/dynamic/100/giraffe`, you'll get 404s.

Okay, so now users can send any request that matches the pattern `GET /dynamic/ANY_INTEGER/6` and we will respond with 50 random 6-sided dice rolls, rather than a 404. Nice!

But what if we want to _use_ the integer they supply in the second path segment to determine the number of dice we respond with, instead of always responding with 50?

## The `params` hash

Within every action and view template, Sinatra automatically creates a very special `Hash` and puts it in a variable called `params` (short for "parameters").

The `params` hash is our one, all-purpose tool for receiving **user input**. Now that the second segment of the path is dynamic, and the user can put any value that they want in that spot, it is considered user input.

Let's get a sense of what `params` looks like by embedding it at the top of our view template:

```erb
<!-- /views/flexible.erb -->

<%= params %>

<h1>50d6</h1>

<ul>
  <% @rolls.each do |a_roll| %>
    <li>
      <%= a_roll %>
    </li>
  <% end %>
</ul>
```

And then visit `/dynamic/50/6` in your live app preview. You should see something like this:

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1689191557/raw-params_igsx2q.png)

We can see that, right now, the `params` hash contains one key-value pair:

```ruby
{"number_of_dice"=>"50"}
```

This is because when the request for `GET /dynamic/50/6` came in:

- Sinatra starts at the top of `app.rb` and proceeds downwards looking at each route to see if it matches the request `GET /dynamic/50/6`. It found a match in our dynamic route:

  ```ruby
	get("/dynamic/:number_of_dice/6")
	```
- Since the second segment is dynamic, Sinatra takes the value from the request and stores it in the `params` hash.
- But, as we know, every value in a hash must be stored under a key. Sinatra looks at what we named the dynamic segment in the route, after the `:`, and _uses that as the key_.
- Hence:

    ![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1689192603/params-hash-1_hstodw.png)


What do you predict will happen if you visit `/dynamic/100/6` or `/dynamic/zebra/6`?

Try it and see. Did the behavior match your predictions?

What do you predict will happen if you change the route to:

```ruby
get("/dynamic/:alice/6")
```

And then visit  `/dynamic/100/6` or `/dynamic/zebra/6`?

Try it and see. Did the behavior match your predictions?

Okay, now that we have a sense of how `params` is being constructed, let's actually use them to make our actions smarter!

## Using parameters

So now, with our dynamic route:

```ruby
get("/dynamic/:number_of_dice/6")
```

We can visit `/dynamic/100/6` and see the `params` hash in the view template. But we currently still are displaying 50 dice rather than 100. Let's fix that.

Since the value `100` is in the `params` hash, we can access it and use it the same way we've done many times before — with `.fetch`:

```erb
<!-- /views/flexible.erb -->

<%= params.fetch("number_of_dice") %>

<h1>50d6</h1>

<ul>
  <% @rolls.each do |a_roll| %>
    <li>
      <%= a_roll %>
    </li>
  <% end %>
</ul>
```

You should now see the 100 displayed at the top of the live app preview. But, let's make better use of it, back in the action:

```ruby
# /app.rb

get("/dynamic/:number_of_dice/6") do
  @num_dice = params.fetch("number_of_dice").to_i

  @rolls = []

  50.times do
    die = rand(1..6)

    @rolls.push(die)
  end

  erb(:flexible)
end
```

- I've created an instance variable, `@num_dice`, and assigned a value to it by `fetch`ing something out of the `params` hash.
- I also called `.to_i` on the value, since HTTP only has `String`s, so all values in the `params` hash come to us as `String`s. It's up to us to convert the values into other classes if it makes sense.

Now we can use `@num_dice` in the view template. Let's embed it right within the `<h1>` where the number of dice are supposed to be:

```erb
<!-- /views/flexible.erb -->

<h1><%= @num_dice %>d6</h1>

<ul>
  <% @rolls.each do |a_roll| %>
    <li>
      <%= a_roll %>
    </li>
  <% end %>
</ul>
```

Finally, we can update the action to use the value when creating the `Array` of random values:

```ruby
# /app.rb

get("/dynamic/:number_of_dice/6") do
  @num_dice = params.fetch("number_of_dice").to_i

  @rolls = []

  @num_dice.times do
    die = rand(1..6)

    @rolls.push(die)
  end

  erb(:flexible)
end
```

Now try visiting `/dynamic/13/6`, `/dynamic/2/6`, and `/dynamic/500/6`. They should all work!

---

**Checkpoint:**

- [Here you can see the changes that I've made since the last commit.](https://github.com/raghubetina/sinatra-dice-dynamic/commit/8d3e27ef27c779b55e0091242840a39eba455797)
- [Here you can browse my entire codebase at this point in time.](https://github.com/raghubetina/sinatra-dice-dynamic/tree/8d3e27ef27c779b55e0091242840a39eba455797)

## Multiple parameters

We can, and often will, receive multiple values in the `params` hash; and we'll need to `fetch` each one in order to make the action behave the way we want it to.

For example, let's make the type of dice also dynamic:

```ruby
get("/dynamic/:number_of_dice/:how_many_sides") do
```

Now the `params` hash looks something like this:

```ruby
{"number_of_dice"=>"5", "how_many_sides"=>"4"}
```

Let's create a second instance variable called `@sides` that makes use of this information:

```ruby
# /app.rb

get("/dynamic/:number_of_dice/6") do
  @num_dice = params.fetch("number_of_dice").to_i

  @sides = params.fetch("how_many_sides").to_i

  @rolls = []

  @num_dice.times do
    die = rand(1..@sides)

    @rolls.push(die)
  end

  erb(:flexible)
end
```

And update the last bit of the `<h1>`:

```erb
<!-- /views/flexible.erb -->

<h1><%= @num_dice %>d<%= @sides %></h1>

<ul>
  <% @rolls.each do |a_roll| %>
    <li>
      <%= a_roll %>
    </li>
  <% end %>
</ul>
```

And now our action is fully dynamic. With just one route, we can support any number of dice and any type of dice that the user requests — an infinite number of possibilities!

---

**Checkpoint:**

- [Here you can see the changes that I've made since the last commit.](https://github.com/raghubetina/sinatra-dice-dynamic/commit/fd95134dd72a076848c3814787eea0968e672665)
- [Here you can browse my entire codebase at this point in time.](https://github.com/raghubetina/sinatra-dice-dynamic/tree/fd95134dd72a076848c3814787eea0968e672665)

## Refactoring

We can now delete the original static routes we wrote, since our one dynamic route supports all possible combinations. We can then change the first segment of the dynamic route from `/dynamic` to `/dice`, and the links in our homepage and navigation work again.

[Just look at this beautiful commit:](https://github.com/raghubetina/sinatra-dice-dynamic/commit/013446160723cb727075fe2ecf6c8c37fe24f167) 

![](https://res.cloudinary.com/dmxgp9oq2/image/upload/v1689195466/deletions_d51be3.png)
{: .bleed-full }

Reducing the number of lines of code while increasing readability and preserving all functionality is a big win!

What we just did is called **refactoring**:

> In computer programming and software design, **code refactoring** is the process of restructuring existing computer code—changing the factoring—without changing its external behavior. Refactoring is intended to improve the design, structure, and/or implementation of the software (its _non-functional_ attributes), while preserving its functionality. Potential advantages of refactoring may include improved code readability and reduced complexity; these can improve the source code's maintainability and create a simpler, cleaner, or more expressive internal architecture or object model to improve extensibility.
>
> Wikipedia, "[Code refactoring](https://en.wikipedia.org/wiki/Code_refactoring){:target="_blank"}" 

Refactoring is a crucial part of the software development process. First, we write messy, clunky, repetitive, but easy to understand and most importantly _functional_ code; then, only after having wrapped our heads around the problem by _solving_ it, we sit back and take a moment to think about whether there might be a more readable or less complex or more performant solution.

However, once we have a working solution, it's often very tempting to just leave it alone; why mess with a good thing and risk introducing bugs? Especially if you're dealing with a large, old, complicated system that you didn't build entirely yourself; it can be irresponsible to refactor willy nilly if you don't fully understand what you're changing (see [Chesterton's fence](https://en.wikipedia.org/wiki/Wikipedia:Chesterton%27s_fence){:target="_blank"}).

And yet, we do need to make changes to our codebase over time; even if we're satisfied to never refactor, we at some point _have_ to add new features or make security patches. How do we make sure we don't inadvertently break anything or introduce bugs? No matter how good our quality assurance team is, it's not realistic to expect them to manually examine every user path through the app and verify that it still works, with every combination of possible inputs, every single time any developer makes any change.

The answer: **automated tests**. This is why developers invest so much time and effort in writing automated test suites (this is what you've been running every time you've done `rake grade`). Automated tests are nothing more than a separate Ruby script that, essentially, web scrapes our own app, clicks every link, fills out every form, with every possible combination of inputs, and makes sure that the app is doing the right thing under all scenarios. As you might imagine, this self-web-scraping script takes a long time to write; often, longer than a feature itself.

But, if you plan to maintain a codebase over the long term (not always true for e.g. a throwaway prototype), it is well worth the investment.

### Our automated tests

Being the savvy developers that we are, we built automated tests into this app to make sure we didn't break anything during refactoring! 

Visit the target once more, click around, and try entering some different integers in the URL path (e.g. `/dice/42/6`):

[dice-roll.matchthetarget.com](https://dice-roll.matchthetarget.com/)

Does everything in your live app preview seem to match the target? Then `rake grade` and make sure that all the tests are passing.

If all the tests are passing, then you are done with the required portion of this project. Nice job!

## Optional extra challenge

Can you refactor your Sinatra Rock, Paper, Scissors codebase to use only one action instead of three for the URLs `/rock`, `/paper`, and `/scissors`?

- Did you read this lesson without skimming?
- Yes
	- Great!
- No
	- Oh no! Dynamic path segments and `params` are absolutely crucial concepts. Please re-read it when you have more time.
{: .choose_best #did_you_read title="Did you read it?" points="2" answer="1" }
