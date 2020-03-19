---
marp: true
theme: totez
---
<!-- _class: invert lead -->
# Advanced ActiveRecord Queries

---
<!-- _class: invert -->
# Scopes

---

### Rails way

```ruby
class PlayerCharacter < ApplicationRecord
  scope :dead, -> { where(status: "dead") }
end
```

---
### Becomes

```ruby
class PlayerCharacter < ApplicationRecord
  def self.dead
    where(status: "dead")
  end
end
```

---
### Chainable && Specific

```ruby
class PlayerCharacter < ApplicationRecord
  scope :dead, -> { where(status: "dead") }
  scope :dead_and_half_orc, -> do
    dead.where(race: "half-orc")
  end
end
```

---
### Scopes with Arguments

```ruby
class PlayerCharacter < ApplicationRecord
  scope :dead, -> { where(status: "dead") }
  scope :created_before, -> (time) do
    where("created_at < ?", time)
  end
end
```

---
### Chaining scopes 
#### with arguments

```ruby
  PlayerCharacter.created_before(3.days.ago).dead
```

---
### Produces this SQL

```SQL
SELECT * FROM player_characters
WHERE `player_characters`.`created_at` < (time)
AND `player_characters`.`status` = "dead";
```

---
### Merging Scopes

```ruby
class PlayerCharacter < ApplicationRecord
  scope :dead,  -> { where(status: "dead") }
  scope :alive, -> { where(status: "alive") }
end

PlayerCharacter.alive.dead
```

---
### Merged SQL

```SQL
SELECT * FROM `player_characters`
WHERE `player_characters`.`status` = "alive"
AND   `player_characters`.`status` = "dead";
```

---
<!-- _class: invert -->

# Default Scopes

---
![Devil Tool](https://media1.giphy.com/media/3o7ypFfZoWqCgnmJRS/giphy.gif?cid=6104955e7b3ddad7ebbca6bdd3f73fa0ec5b05ec1ce1e4ac&rid=giphy.gif)
### The Devil's tool

---
### Them Defaults

```ruby
class PlayerCharacter < ApplicationRecord
  default_scope { where.not(status: "dead") }
end
```

---
### Default SQL

```SQL
SELECT * FROM `player_characters`
WHERE `player_characters`.`status` != "dead";
```

---
# Merging Defaults

```ruby
class PlayerCharacter < ApplicationRecord
  default_scope { where.not(status: "dead") }
  scope :actually_totally_dead, -> { where(status: "dead") }
end
```
---
### Calling

```ruby
PlayerCharacters.actually_totally_dead
```
---
### Defaults are prepended

```SQL
SELECT * FROM `player_characters`
WHERE `player_character`.`status` != "dead"
AND   `player_character`.`status` = "dead";
```

---
![Get Out](https://media1.giphy.com/media/s0UqJX0nvBQs/giphy.gif?cid=6104955efcab03d02d10786bd7264f170cc5dd9e8704295b&rid=giphy.gif)
### Getting yourself out of a pickle

---
### unscoped

```ruby
PlayerCharacter.unscoped.actually_totally_dead
```

---
<!-- _class: invert -->
# Joins

---
### Given
```ruby
class PlayerCharacter < ApplicationRecord
  belongs_to :user
  has_many :player_classes
end

class PlayerClass < ApplicationRecord
  belongs_to :player_character
  has_many :spells
end

class Spell < ApplicationRecord
  belongs_to :player_class
end
```

---
### SQL Fragment

```ruby
PlayerCharacter.joins(
  "LEFT OUTER JOIN player_classes "            \
    "ON player_classes.player_character_id = " \
    "player_character.id"
)
```
---
### Provides

```SQL
SELECT player_characters.* FROM player_characters
LEFT OUTER JOIN player_classes ON
player_classes.player_character_id = player_character.id;
```
---
### Hash/Array Relationship Style

```ruby
PlayerCharacter.joins(:user)
```
---
### The SQLs

```SQL
SELECT player_characters.* FROM player_characters
INNER JOIN users ON player_characters.user_id = user.id;
```

---
### Joining Multiple Associations

```ruby
PlayerCharacter.joins(:user, :player_classes)
```

---
### Executes

```SQL
SELECT player_characters.* FROM player_characters
INNER JOIN users ON 
  player_characters.user_id = user.id
INNER JOIN player_classes ON 
  player_classes.player_character_id = player_characters.id;
```

---
### Join nested relationships

```ruby
PlayerCharacter.joins(player_classes: :spells)
```

---
### Nested Join SQL Produces

```SQL
SELECT player_characters.* FROM player_characters
INNER JOIN player_classes ON 
   player_classes.player_character_id = player_characters.id;
INNER JOIN spells ON
  spells.player_class_id = player_class.id;
```

---
### Specifying Conditions on nested associations 

---
### With SQL fragment

```ruby
PlayerCharacter.joins(player_classes: :spells).
  where("spells.name = ?", "Toll of the dead")
```

---
### Nested Style

```ruby
PlayerCharacter.joins(player_classes: :spells).
  where(spells: { name: "Toll of the dead" })
```

---
<!-- _class: invert -->
# Eager Loading

---
### n + 4 life

```ruby
@new_round_transactions.each do |t|
  if t.bridge_notes_balance.to_f > 0
    ...
    @company.debts.each do |d|
      if d.convertible == @nrb.security
        d.issuances.each do |debt_issuance|
        ...
      end
    end
  end
end
```

---
### Includes

```ruby
PlayerCharacter.includes(:player_classes)

# With nested associations
PlayerCharacter.includes(player_classes: :spells)

# With nested association conditions
PlayerCharacter.includes(player_classes: :spells).
  where(spells: { name: "Toll of the dead" })
```

---
### SQL Produced
<!-- includes creates 3 queries for all the associations --->

```SQL
SELECT `player_character`.* FROM `player_characters`;

SELECT `player_classes`.* FROM `player_classes`;

SELECT `spells`.* FROM `spells` 
WHERE `spells`.`name` = "Toll of the dead";
```

---
### Refences

```ruby
PlayerCharacter.includes(:player_classes).
  where("player_classes.name = ?", "cleric").
  references(:player_classes)
```

---
<!-- _class: invert -->
# Dynamic methods

---
### find_by_column

```ruby
PlayerCharacter.find_by_name("Nacia")
```

---
### Regular SQL

```SQL
SELECT `player_characters`.* FROM `player_characters`
WHERE `player_characters`.`name` = "Nacia"
LIMIT 1;
```
---
### find_or_create_by

```ruby
PlayerCharacter.find_or_create_by(name: "Nacia")
```
---
### Dynamic Creation

```SQL
SELECT `player_characters`.* FROM `player_characters`
WHERE `player_characters`.`name` = "Nacia"
LIMIT 1;

BEGIN
INSERT INTO `player_characters` (created_at, ,,,,)
VALUES ("2020-02-18 05:32:00 ")
COMMIT
```

---
### find_or_intialize_by

```ruby
PlayerCharacter.find_or_initialize_by(name: "Nacia")
```

---
### Dynamic Initialization

```SQL
SELECT * FROM `player_characters`
WHERE `player_characters`.`name` = "Nacia"
LIMIT 1;
```

---
<!-- _class: lead -->
### Counts && Such
Any ActiveRecord::Relation object can have `count` called on it.

```ruby
PlayerCharacter.count
```

---
### Count SQL

```SQL
SELECT COUNT(*) FROM `player_characters`;
```

---
### More Advanced Relations

```ruby
PlayerCharacters.includes(:player_classes).
  where(name: "Nacia", player_classes: { name: "cleric" }).
  count
```

---
<style scoped>
  code {
    font-size: 0.6em;
  }
</style>

### Produces

```SQL
SELECT COUNT(`player_characters`.`id`) AS `count_all` 
FROM `player_characters`
LEFT OUTER JOIN `player_classes` 
ON `player_classes`.`player_character_id` = `player_character`.`id`
WHERE (
  `player_characters`.`name` = "Nacia" AND
  `player_classes`.`name` = "cleric
);
```

---
### Count By Column
<!-- See how many records are in the DB with an age set -->

```ruby
PlayerCharacter.count(:age)
```

---
### SQL

```SQL
SELECT COUNT(`player_characters`.`age`)
FROM `accounts`;
```
---
### Averages

```ruby
PlayerCharacter.average(:age)
```

---
### Average SQL

```SQL
SELECT AVG(`player_characters`.`age`) 
FROM `player_characters`;
```

---
### Minimum

```ruby
PlayerCharacter.minimum(:age)
```

---
### Min SQL

```SQL  
SELECT MIN(`player_characters`.`age`) 
FROM `player_characters`;
```

---
### Max

```ruby
PlayerCharacter.maximum(:age)
```

---
### Max SQL

```SQL  
SELECT MAX(`player_characters`.`age`) 
FROM `player_characters`;
```

---
### Sum

```ruby
PlayerCharacter.sum(:death_saves)
```

--- 
### Sum SQL 

```SQL
SELECT SUM(`player_characters`.`death_saves`)
FROM `player_characters`;
```

---
<!-- _class: invert -->
# Using SQL Strings

---
### find_by_sql

Returns instances of Spell ActiveRecord class. 


```ruby
Spells.find_by_sql(
  "SELECT `spells`.* FROM `spells`"                     \
  "INNER JOIN `player_classes` ON "                     \
  "`spells`.`player_class_id` = `player_classes`.`id` " \
  "WHERE `player_classes`.`id` = 3 "                    \
  "ORDER BY `spells`.`level` DESC"
)

#=> [#<Spell id: 42, level: 0..>, #<Spell id: 67, level: ..]
```

---
### select_all
This requires that you write valid SQL for your DB


```ruby
PlayerCharacter.connection.select_all(
  "SELECT name, height FROM player_characters " \
  "WHERE height = '198 cm';"
)
#=> [
#     { name: "Thornok", height: "198 cm" }, 
#     { name: "Terry Crews", height: "198 cm" },
#     ...
#   ]
```
---
### ids

```ruby
PlayerCharacter.where(user_id: 1337).ids
#+> [5, 7, 31]
```
---
### Executes

```SQL
SELECT `player_characters`.`id` FROM `player_characters`
WHERE `player_characters`.`user_id` = 1337;
```

---
### pluck
And it's magnificence

```ruby
PlayerCharacter.where(user_id: 1337).pluck(:name)
```

---
### Replaces code like this

```ruby
PlayerCharacter.where(user_id: 1337).map(&:name)
```

---
### Produces

```SQL
SELECT `player_characters`.`name` FROM `player_characters`
WHERE `player_characters`.`user_id` = 1337;
```

---
<!-- _class: invert -->
# Helpers

---
### to_sql

```ruby
PlayerCharacter.joins(:player_class).
  where(player_class: { name: "cleric" }).to_sql

#=> SELECT * FROM `player_characters`
#   INNER JOIN `player_classes`
#   ON `player_classes`.`player_character_id` = 
#     `player_characters`.id
#   WHERE `player_classes`.`name` = "Cleric";
```

---
### Explain

```ruby
PlayerCharacter.where(name: "Nacia").explain
```

---
<!-- _class: lead -->

### Demo Time
because every database is different with its explain command. 

---
<!-- _class: invert -->

![END](https://media0.giphy.com/media/5bo7UYW69cYQZA4tOF/giphy-downsized.gif?cid=6104955ea0f3a143677049fc052aa667bfca29993a6aaebe&rid=giphy-downsized.gif)

# Fin