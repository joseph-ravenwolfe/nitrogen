# :tulip: Nitrogen 0.0.1 Draft Spec

#### Warning: This library is a Draft Specification. Not for Production use.

Nitrogen provides a simple and robust system for seeding your application.

## Features

#### Easy to Use

Comes with an installation task and hooks into your generators to start seeding
out of the box.

#### :heart:'s Ruby

Embraces a powerful configuration language you already know: *Ruby*. No more fiddling
with XML, YAML, HAML, or ALABAML. Ok, we made up ALABAML...

#### Clean

Nitrogen makes your `config/seed.rb` clean and easy to understand. It also provides
a canonical folder and file structure that is easy to digest.

#### Free Development Data

Fertilizers provide a structured and simple way to generate fake data for use in
development. This is a much faster and better way to bootstrap an application
rather than passing around production dumps.


## Getting Started

Add Nitrogen to your `Gemfile`:

```ruby
gem 'nitrogen'
```

Run `bundle` to install it.

After installing Nitrogen, run the following generator to get started.

```shell
rails generate nitrogen:install
```

This will set up the application's seed file along with a folder
structure for your Fertilizers.

## Defining Seed Order

Nitrogen provides a `sprout` helper to use in your `config/seeds.rb`file. Use
this to configure the models you would like to seed and in which order. You
can seed a model with the following.

```ruby
# config/seeds.rb
Nitrogen::Seeds.plant do
  sprout :users
  sprout :posts
end
```

You can also specify which models depend on the existance of other models when
defining your seeds.

```ruby
# config/seeds.rb
Nitrogen::Seeds.plant do
  sprout :users do
    sprout :comments
    sprout :blog_posts do
      sprout :related_posts
    end
  end
  
  sprout :countries
end
```

## Defining Your Seeds

Nitrogen will use the name of the sprouts defined in the `seeds.rb` above to
find a Fertilizer with the same name.

Use a Fertilizer to define what data will be seeded and for which environments.
Some data may need to be available in your application all the time, while other
data might only be used in certain environments. You can configure this in your
model's Fertilizer like so:

```ruby
# config/seeds/fertilizers/user_fertilizer.rb
class UserFertilizer < Nitrogen::Fertilizer

  all_environments do
    create(first_name: 'Jonny', last_name: 'Smith', email: 'jsmith@gmail.com')
    create(first_name: 'Sally', last_name: 'Jones', email: 'sjones@gmail.com')
  end

  environment :development, :test do
    create(first_name: 'Admin', last_name: 'Test', admin: true)
  end

end
```

Set a flag to determine which fields to find records by so duplicate seeds
are not created. Expire existing seeds to ensure they are no longer
present. Expired seeds are found and destroyed by the attributes provided.

```ruby
# config/seeds/fertilizers/user_fertilizer.rb
class UserFertilizer < Nitrogen::Fertilizer
  unique_by :email

  all_environments do
    create(first_name: 'Jonny', last_name: 'Smith', email: 'jsmith@gmail.com')
    expire(first_name: 'Sally', last_name: 'Jones', email: 'sjones@gmail.com')
    expire(first_name: 'Sammy', last_name: 'Lopez', email: 'slopez@gmail.com')
  end

end
```

## Seeding Fake Development Data

You can use Fertilizers to generate fake data with the `factory_for` helper.
Nitrogen uses [FactoryGirl](https://github.com/thoughtbot/factory_girl) behind
the scenes to generate fake data. Nitrogen also provides access to
[Forgery](https://github.com/sevenwire/forgery) to create fake data.

Provide the `factory_for` helper the number of records to create with a list
of environments you would like the factory to apply to.

```ruby
# config/seeds/fertilizers/user_fertilizer.rb
class UserFertilizer < Nitrogen::Fertilizer

  all_environments do
    # Seed Production and Other Data
  end

  factory_for 200, :development, :test do
    first_name { Forgery(:person).first_name }
    last_name { Forgery(:person).last_name }
    email { Forgery(:person).email }
  end

end
```

You can customize fake data by defining your own forgeries. Custom
forgeries are automatically assigned to the model's corresponding
attribute based on name.

```ruby
# config/seeds/fertilizers/user_fertilizer.rb
class UserFertilizer < Nitrogen::Fertilizer

  factory_for :development do
    # Declare other attributes...
    spy_alias
  end

  forgery_for :spy_alias do
    dictionaries[:spy_aliases].random
  end

end
```

You can randomize the number of records that are created by supplying a
`volume` option in your seed manifest. This can be useful for randomizing
the number of associated child records in a given relationship. In the
following example, a given User will have up to 4 Post records associated with
it. If no `volume` is specified, the default behavior is to create one
dependent model per parent model.

```ruby
# config/seeds.rb
sprout :users do
  sprout :posts, volume: 1..4
end
```

In addition to volume, you can also supply a `frequency` option to specify the
chance that any records will be created for an associated model. This may be
useful for realistically representing the volume of dependent models in your
data. Frequency can be supplied as a decimal proportion or a static integer
interval. In the following example, a given Post would have a 60% chance to
have an associated Comment. Posts that have any Comments at all are usually
interesting, and therefore prone to having anywhere from 10 to 50 Comments.
Every 10 Posts will make it into a Publication.

```ruby
# config/seeds.rb
sprout :posts do
  sprout :comments, frequency: 0.5, volume: 10..50
  sprout :publication, frequency: 10
end
```

You can also explicitly set the Fertilizer to be used by supplying a
`fertilizer` option.

```ruby
sprout :reviews, fertilizer: :posts
```

`rails generate` hooks are also provided for the `model`, and `scaffold`
generators. A `rails generate seed` command is also provided to bootstrap
a new seed.

## Unresolved Items
* Defining Nitrogen's Fertilizer API for record creation

[![githalytics.com alpha](https://cruel-carlota.pagodabox.com/7f62cda8c7463b7a556e9085b8100926 "githalytics.com")](http://githalytics.com/josephjaber/nitrogen)

