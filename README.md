# ⚘ Nitrogen 0.0.1 Draft Spec

A better way to seed your application.

#### Warning: This library is a Draft Specification. Not for Production use.

## Installation

```ruby
gem 'nitrogen'
```

## Usage

Configure fertilizers and dependencies in `config/seeds.rb`

```ruby
# config/seeds.rb
sprout :users do
  sprout :addresses
  sprout :posts, frequency: 0..4
  sprout :aliases, fertilizer: :addresses
end

sprout :inventory_items
sprout :shipments
```

Define Fertilizers. Fertilizers use `FactoryGirl` behind the scenes. Define attributes and create fake data with `Forgery`. Fake data will be automatically populated with basic Forgeries based on attribute data type. Define your own forgeries for automatic use in your Fertilizer.

```ruby
# config/seeds/fertilizers/user_fertilizer.rb
UserFertilizer < Nitrogen::Fertilizer

  all_environments do
    first_or_create(first_name: 'John', last_name: 'Smith', email: 'jsmith@gmail.com')
    first_or_create(first_name: 'Sally', last_name: 'Smith', email: 'ssmith@gmail.com')
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

== Copyright

Copyright (c) 2013 Joseph Jaber. See LICENSE.txt for
further details.

[![githalytics.com alpha](https://cruel-carlota.pagodabox.com/7f62cda8c7463b7a556e9085b8100926 "githalytics.com")](http://githalytics.com/josephjaber/nitrogen)