# ActiveRecord Validations

ActiveRecord can validate our models for us before they even touch the
database.

We can use `ActiveRecord::Base` class methods like `#validates()` to set things
up.


# Objectives

After this lesson, you should be able to:

- Identify when validation occurs in the lifespan of an object
- Introspect on the `ActiveRecord::Errors` collection object on an AR instance
  - use `#valid?()`
  - use `#errors()`
- Generate `full_message`s for errors
- Check an attribute for validation errors
- Add a custom validation error to an AR model


# Context: Databases and Data Validity

## AR Validations Are Not Database Validations

Many relational databases such as PostgreSQL have data validation features that
check things like length and data type. In Rails, these validations are **not
used**, because each database works a little differently, and handling
everything in ActiveRecord itself guarantees that we'll always get the same
features no matter what database we're using under the hood.

## What is "invalid data"?

Suppose you get a new phone and you ask all of your friends for their phone
number again. One of them tells you, "555-868-902". If you're paying attention,
you'll probably wrinkle your nose and think, "Wait a minute. That doesn't sound
like a real phone number."

"555-868-902" is an example of **invalid data**... for a phone number. It's
probably a valid account number for some internet service provider in Alaska,
but there's no way to figure out what your friend's phone number is from those
nine numbers. It's a showstopper, and even worse, it kind of looks like valid
data if you're not looking closely.

## Validations Protect the Database

Invalid data is the bogeyman of web applications: it hides in your database
until the worst possible moment, then jumps out and ruins everything by causing
confusing errors.

Imagine the phone number above being saved to the database in an application
that makes automatic calls using the Twilio API. When your system tries to call
this number, there will be an error because no such phone number exists, which
means you need to have an entire branch of code dedicated to handling *just*
that edge case.

It would be much easier if you never have bad data in the first place, so you
can focus on handling edge cases that are truly unpredictable.

That's where validations come in.


# Basic Usage

For more examples of basic validation usage, see the Rails Guide for [Active
Record Validations][ar_validations]. Take a few minutes to browse the helpers
listed in Section 2.

```ruby
class Person < ActiveRecord::Base
  validates :name, presence: true
end

Person.create(name: "John Doe").valid? # => true
Person.create(name: nil).valid? # => false
```

[ar_validations]: http://guides.rubyonrails.org/active_record_validations.html

`#validates()` is our Swiss Army knife for validations. It takes two arguments:
the first is the name of the attribute we want to validate, and the second is a
hash of options that will include the details of how to validate it.

In this example, the options hash is `{ presence: true }`, which implements the
most basic form of validation, preventing the object from being saved if its
`name` attribute is empty.

# Lifecycle Timing

Before proceeding, keep the answer to this question in mind:

**What is the difference between `#new()` and `#create()`?**

If you've forgotten, `#new()` instantiates a new ActiveRecord model *without*
saving it to the database, whereas `#create()` immediately attempts to save it,
as if you had called `#new()` and then `#save()`.

**Database activity triggers validation**. An ActiveRecord model instantiated
with `#new()` will *not* be validated, because no attempt to write to the
database was made.

The only way to trigger validation without touching the database is to call the
`#valid?()` method.

For a full list of methods that trigger validation, see [Section
4][ar_callbacks_4] of the Rails Guide for Active Record Callbacks. Don't worry
about the rest of the information in that guide just yet; we'll go into
callbacks later!

[ar_callbacks_4]: http://guides.rubyonrails.org/active_record_callbacks.html#running-callbacks

# Validation Failure

Here it is, the moment of truth. What can we do when a record fails validation?

## How can you tell when a record fails validation?

**Pay attention to return values!**

By default, ActiveRecord does not raise an exception when validation fails.
The database operation method will simply return `false` and decline to update
the database.

Every database method has a sister method with a `!` at the end which will raise
an exception (`#create!()` instead of `#create()` and so on).

And of course, you can always check manually with `#valid?()`.

## Finding out why validations failed

To find out what went wrong, you can look at the model's `#errors()` object.

You can check all errors at once by examining `errors.messages`.

You can check one attribute at a time by passing the name to `errors` as a key,
like so:

```ruby
person.errors[:name]
```


# Displaying Validation Errors in Views

See [Section 8][ar_validations_8] of the Rails Guide for an example of how to
use the `ActiveModel::Errors#full_messages()` helper, reproduced here for
convenience:

[ar_validations_8]: http://guides.rubyonrails.org/active_record_validations.html#displaying-validation-errors-in-views

```erb
<% if @article.errors.any? %>
  <div id="error_explanation">
    <h2>
      <%= pluralize(@article.errors.count, "error") %>
      prohibited this article from being saved:
    </h2>

    <ul>
    <% @article.errors.full_messages.each do |msg| %>
      <li><%= msg %></li>
    <% end %>
    </ul>
  </div>
<% end %>
```

This constructs more complete-looking sentences from the more terse messages
available in `errors.messages`.


# Custom Validators

There are three ways to implement custom validators, with examples in [Section
6][ar_validators_6] of the Rails Guide:

- Subclassing `ActiveModel::Validator` and invoking with `#validates_with()`

  This approach is best when you want to do a whole bunch of validations on
  several different models. You can just call `validates_with MyValidator` on
  each of them.

- Subclassing `ActiveModel::EachValidator` and invoking with an inflected key in
  the options hash

  This is best for validating a single attribute on one model, especially one
  that you're already using builtin validators for.

  For example, in the Rails Guide, they define `EmailValidator` and then pass
  the `email: true` key-value pair to `#validates()` to invoke it.

- Calling `#validate()` (that's "validate" *without* an "s") with the name of an
  instance method

  Use this approach when you're not sure which to use. If you end up needing to
  use the same validation logic on a different model, you can easily extract the
  instance method into one of the ActiveModel classes and use `#validates()` or
  `#validates_with()` instead.

So, to recap:

- `Validator` and `validates_with` for doing many validations in one pass.
- `EachValidator` and `validates` for validating one specific attribute.
- `validate` for quick custom validations that you can extract later.

[ar_validators_6]: http://guides.rubyonrails.org/active_record_validations.html#performing-custom-validations
