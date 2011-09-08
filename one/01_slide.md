!SLIDE 
# Active Relation (ARel) #


!SLIDE bullets incremental
# ARel is ... #

* An OOP interpretation of Relational Algebra
* The basis of a new API for querying in ActiveRecord 3.x
* Fairly lacking in API documentation


!SLIDE small bullets incremental
# The ARel DSL ... #

* Implements closure under composition
  * (chainable methods w/ indentical behavior regardless of sequence)
* Follows a Functional pattern
  * (Relation objects are immutable)
* Represents relational query elements as Ruby object instances
* Cleanly maps to a wide variety of relational query operations


!SLIDE small bullets incremental
# The ActiveRecord 3.x Querying API ... #

* Is built around and benefits from the ARel API
* Is a little similar to, but not the same as the ARel API
* Substantially replaces the 2.x querying API
* Will Deprecate the older API in 3.1 and remove it entirely in 3.2
* Is fairly lacking in documentation
  * including how it is/isn't related to ARel


!SLIDE small
# Building SQL Queries With ARel #

    @@@ ruby
    person = Arel::Table.new(:people)
    
    person.project(Arel.sql('*')).to_sql
     => 'select * from "people"'

    person.project(person[:id], person[:name]).to_sql
      => 'SELECT "people"."id", "people"."name" FROM "people"'
    
    person.where(person[:active].eq(true)).
           join(phone).on(person[:id].eq(phone[:person_id])).
           project(person[:name],phone[:kind],phone[:phone]).
           to_sql
    => <<SQL
    SELECT "people"."name", "person_phones"."kind",
    "person_phones"."phone"
    FROM "people" INNER JOIN "person_phones" ON
    "people"."id" = "person_phones"."person_id"
    WHERE "people"."active" = \'t\'
    SQL


!SLIDE small
# Self-Join Queries In ARel

    @@@ ruby
    person = Arel::Table.new(:people)

    friend_person = person.alias

    friend = Arel::Table.new(:friends)

    person.join(:friend).on(
      person[:id].eq(friend[:person_id])).
    join(friend_person).on(
      friend[:person_id_friend].eq(friend_person[:id])).
    project(person[:name],friend_person[:name]).
    to_sql
    => <<SQL
    SELECT "people"."name", "people_2"."name"
    FROM "people" INNER JOIN 'friend' ON "people"."id" =
    "friends"."person_id" INNER JOIN "people" "people_2" ON 
    "friends"."person_id_friend" = "people_2"."id"
    SQL

!SLIDE small
# Column Aliases and SQL Expressions With Operators
_Note: Use Rails 3.1 (not 3.0) to try this out._

    @@@ ruby
    person = Arel::Table.new(:people)

    person.project(
      (person[:jars_of_goo] * person[:oz_per_goo_jar]).
      as('goo')
    ).to_sql
    => <<SQL
    SELECT "people"."jars_of_goo" * "people"."oz_per_goo_jar"
    AS goo
    FROM "people"
    SQL

!SLIDE small
# Grouping and Aggregate Computations

    @@@ ruby
    friend.project(
      friend[:person_id], Arel.sql('*').count.
      as('friend_count')).group(friend[:person_id]).to_sql
    => <<SQL
    SELECT "friends"."person_id", COUNT(*) AS friend_count
    FROM "friends"  GROUP BY "friends"."person_id"
    SQL


!SLIDE bullets incremental
# Integration with ActiveRecord

* ActiveRecord 3.x uses ARel under the hood
* The ActiveRecord 3.x query API is kinda' like, but not the same as the ARel API
* ActiveRecord 3.x exposes ARel via \<model>.arel_table

!SLIDE small
# Correlated Subqueries in ARel

    @@@ ruby
    person.project(
      Arel.sql('people.*'),
      person_phone.project(person_phone[:id].count
    ).where(person[:id].eq(person_phone[:person_id])).
      as('phone_count')).
    to_sql
    => <<SQL
    SELECT people.*, (
    SELECT COUNT("person_phones"."id")
    FROM "person_phones"
    WHERE "people"."id" = "person_phones"."person_id"
    ) phone_count
    FROM "people"
    SQL

!SLIDE small
# Querying ActiveRecord 3.x Query API and ARel

_Note: #& is supposed to be an alias for #merge, but did not work in my testing. Perhaps in Rails 3.2?_
    @@@ ruby
    Person.where(:active => true).joins(:phones).merge(
                       PersonPhone.where(:kind => 'home'))

    pt = Person.arel_table
    ft = Friend.arel_table
    fpt = pt.alias
_Notes: How do we get an outer join here?  Any way to get 'friends.*' without going behind ARel's back?
    sql=pt.project(
      Arel.sql('friends.*'),
      (fpt[:jars_of_goo] * fpt[:oz_per_goo_jar]).sum.
        as('oz_of_friend_goo')
    ).join(ft).on(pt[:id].eq(ft[:person_id])).
    join(fpt).on(ft[:person_id_friend].eq(fpt[:id])).
    group(pt[:id]).to_sql
    Person.find_by_sql(sql) => ...

!SLIDE smaller bullets
# Links

* http://magicscalingsprinkles.wordpress.com/2010/01/28/why-i-wrote-arel/
* https://github.com/rails/arel
* http://www.railsdispatch.com/posts/activerelation
* http://m.onkey.org/active-record-query-interface

!SLIDE
# The End #
