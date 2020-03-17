---
marp: true
theme: totez
---
<!-- _class: invert lead -->
# Advanced ActiveRecord Queries

---
# Scopes

---
### Rails way

```ruby
class PlayerCharacter < ApplicationRecord
  scope :dead -> { where(status: "dead") }
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
### Chaining scopes with arguments

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
# Merged SQL

```SQL
SELECT * FROM `player_characters`
WHERE `player_characters`.`status` = "alive"
AND   `player_characters`.`status` = "dead";
```

---
# Default Scopes
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
## Getting yourself out of a pickle

```ruby
PlayerCharacter.unscoped.actually_totally_dead
```

---
<!-- _class: invert -->
# Joins

---
# Eager Loading

---
# Enum

---
# Dynamic methods

---
# Method Chaining

---
# Counts && Such

---
# Helpers

---
# END