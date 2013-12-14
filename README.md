# ⚘ Nitrogen 0.0.1 Draft Spec

A better way to seed your application.

#### Warning: This library is a Draft Specification. Not for Production use.

## Installation

```ruby
gem 'nitrogen'
```

## Usage

Configure fertilizers and dependencies in `config/seeds.rb`. Nest configurations
for dependent models. Define the desired `:volume` for a dependent model to
appear. In the following example, a given `User` will have up to 4 `Post`
records associated with it. If no `:volume` is specified, the default behavior
is to create one dependent model. In addition to volume, the ability to specify
a desired `:frequency` is provided. Frequency can be supplied as a decimal
proportion or a static integer. In the following example, a given `User` has a
15% percent chance of having between 0 and 4 posts. In the following example,
every 10 users will have an associated address.

Configure a seeded model to inherit from a pre-existing fertilizer. More on
Fertilizers below.

```ruby
# config/seeds.rb
sprout :users do
  sprout :addresses, frequency: 10
  sprout :posts, volume: 0..4, frequency: 0.15
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

`rails generate` hooks are also provided for the `model`, and `scaffold` generators. A `rails generate seed` command is also provided to bootstrap a new seed.

[![githalytics.com alpha](https://cruel-carlota.pagodabox.com/7f62cda8c7463b7a556e9085b8100926 "githalytics.com")](http://githalytics.com/josephjaber/nitrogen)