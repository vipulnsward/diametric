# Diametric

Diametric is a library for building schemas, queries, and transactions for
[Datomic][] from Ruby objects.

Currently, Diametric does not contain the logic for communicating with Datomic,
only for creating the schema, queries, and transactions.

## Usage

### Data API

```ruby
class Person
  include Diametric::Data

  attribute :name, String, :index => true
  attribute :email, String, :cardinality => :many
  attribute :birthday, DateTime
  attribute :iq, Integer
  attribute :website, URI
end

Person.schema
# Datomic transaction:
# [{:db/id #db/id[:db.part/db]
#   :db/ident :person/name
#   :db/valueType :db.type/string
#   :db/cardinality :db.cardinality/one
#   :db/index true
#   :db.install/_attribute :db.part/db}
#  {:db/id #db/id[:db.part/db]
#   :db/ident :person/email
#   :db/valueType :db.type/string
#   :db/cardinality :db.cardinality/many
#   :db.install/_attribute :db.part/db}
#  {:db/id #db/id[:db.part/db]
#   :db/ident :person/birthday
#   :db/valueType :db.type/instant
#   :db/cardinality :db.cardinality/one
#   :db.install/_attribute :db.part/db}
#  {:db/id #db/id[:db.part/db]
#   :db/ident :person/iq
#   :db/valueType :db.type/long
#   :db/cardinality :db.cardinality/one
#   :db.install/_attribute :db.part/db}
#  {:db/id #db/id[:db.part/db]
#   :db/ident :person/website
#   :db/valueType :db.type/uri
#   :db/cardinality :db.cardinality/one
#   :db.install/_attribute :db.part/db}]

Person.query_data(:name => "Clinton Dreisbach")
# Datomic query:
# [:find ?e ?name ?email ?birthday ?iq ?website
#  :from $ ?name
#  :where [?e :person/name ?name]
#         [?e :person/email ?email]
#         [?e :person/birthday ?birthday]
#         [?e :person/iq ?iq]
#         [?e :person/website ?website]]
# Args:
#   ["Clinton Dreisbach"]
#
# Returns as an array, [query, args].

Person.attributes
# [:dbid, :name, :email, :birthday, :iq, :website]

person = Person.new(Hash[*(Person.attributes.zip(results_from_query).flatten)])
# or
person = Person.from_query(results_from_query)

person.iq = 180
person.tx_data(:iq)
# Datomic transaction:
# [{:db/id person.dbid
#   :person/iq 180}]

person = Person.new(:name => "Peanut")
person.tx_data
# Datomic transaction:
# [{:db/id #db/id[:db.part/user]
#   :person/name "Peanut"}]
```

### Persisting Objects

With `Diametric::Persistence`, you can create objects that know how to store themselves to Datomic without using an external client.

```ruby
class Goat
  include Diametric::Persistence
  
  attribute :name, String, :index => true
end

Diametric::Persistence.connect('http://localhost:9000', 'test')
# database url and database alias

Goat.database = 'goats'
# will create database if it does not exist

goat = Goat.new(:name => 'Beans')
goat.dbid # => nil
goat.name # => "Beans"
goat.persisted? # => false
goat.new? # => true

goat.save
goat.dbid # => new id autogenerated
goat.name # => "Beans"
goat.persisted? # => true
goat.new? # => false

goats = Goat.where(:name => "Beans")
#=> [Goat(id: 1, name: "Beans")]

goat = Goat.first(:name => "Beans")
#=> Goat(id: 1, name: "Beans")
```


## Installation

Add this line to your application's Gemfile:

    gem 'diametric'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install diametric

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

[Datomic]: http://www.datomic.com
