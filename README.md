# :tulip: Nitrogen 0.0.1 Draft Spec

A better way to seed your application.

#### Warning: This library is a Draft Specification. Not for Production use.

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
sprout :users
sprout :posts
```

You can also specify which models depend on the existance of other models when
defining your seeds.

```ruby
sprout :users do
  sprout :comments
  sprout :blog_posts do
    sprout :related_posts
  end
end

sprout :countries
```

## Defining Your Seeds

Nitrogen will use the name of the sprouts defined in the `seeds.rb` above to
find a Fertilizer with the same name.

Use a Fertilizer to define what data will be seeded and for which environments.
Some data may need to be available in your application all the time, while other
data might only be used in certain environments. You can configure this in your
model's Fertilizer like so:

```ruby
UserFertilizer < Nitrogen::Fertilizer

  all_environments do
    create(first_name: 'Jonny', last_name: 'Smith', email: 'jsmith@gmail.com')
    create(first_name: 'Sally', last_name: 'Jones', email: 'sjones@gmail.com')
    expire(first_name: 'Sammy', last_name: 'Lopez', email: 'slopez@gmail.com')
  end

  environment :development, :test do
    create(first_name: 'Admin', last_name: 'Test', admin: true)
  end

end
```

## Seeding Fake Development Data

You can use Fertilizers to generate fake data with the `factory_for` helper.
Nitrogen uses [FactoryGirl](https://github.com/thoughtbot/factory_girl) behind
the scenes to generate fake data. Nitrogen also provides access to
[Forgery](https://github.com/sevenwire/forgery) to create fake data.

```ruby
UserFertilizer < Nitrogen::Fertilizer

  all_environments do
    # Seed Production and Other Data
  end

  factory_for :development do
    first_name { Forgery(:person).first_name }
    last_name { Forgery(:person).last_name }
    email { Forgery(:person).email }
  end

end
```

You can randomize the the number of records that are created by supplying a
`frequency` option in your seed manifest. This can be useful for randomizing
the number of associated child records in a given relationship. In the
following example, a given User will have up to 4 Post records associated with
it. If no `frequency` is specified, the default behavior is to create one
dependent model per parent model.

```ruby
# config/seeds.rb
sprout :users do
  sprout :posts, frequency: 1..4
end
```

## Other Usage

Configure a seeded model to inherit from a pre-existing fertilizer. More on
Fertilizers below.

```ruby
# config/seeds.rb
sprout :users do
  sprout :addresses
  sprout :posts, frequency: 0..4
  sprout :reviews, fertilizer: :posts
end

sprout :categories
sprout :ratings
```

Define Fertilizers to populate your models. Fertilizers use `FactoryGirl` behind
the scenes. Define attributes and create fake data with `Forgery`. Fake data
will be automatically populated with basic Forgeries based on attribute data
type. Define custom forgeries for automatic use in your Fertilizer. Custom
forgeries are automatically assigned to the models attribute based on name.
Override any default data population by specifying a `nil` value for that
attribute.

Configure Fertilizer behavior for each environment, or run any setup code
necessary beforehand. Configure factories to run in specific environments.
Set a flag to determine which fields to find records by so duplicate seeds
are not created. Deprecate existing seeds to ensure they are no longer
present. Deprecated seeds are destroyed by provided attributes.

```ruby
# config/seeds/fertilizers/user_fertilizer.rb
UserFertilizer < Nitrogen::Fertilizer
  unique_by :email

  all_environments do
    create(first_name: 'Jonny', last_name: 'Smith', email: 'jsmith@gmail.com')
    create(first_name: 'Sally', last_name: 'Jones', email: 'sjones@gmail.com')

    deprecate(email: 'deprecated@gmail.com')
  end

  factory_for :development, :staging do
    first_name { Forgery(:person).first_name }
    last_name { Forgery(:person).last_name }
    email { Forgery(:person).email }
    aliases
  end

  forgery_for :aliases do
    dictionaries[:aliases].random
  end

end
```

`rails generate` hooks are also provided for the `model`, and `scaffold`
generators. A `rails generate seed` command is also provided to bootstrap
a new seed.

## Unresolved Items
* Defining Nitrogen's philosophy
* Defining Nitrogen's Fertilizer API for record creation
* Defining the number of seeds that are created

[![githalytics.com alpha](https://cruel-carlota.pagodabox.com/7f62cda8c7463b7a556e9085b8100926 "githalytics.com")](http://githalytics.com/josephjaber/nitrogen)