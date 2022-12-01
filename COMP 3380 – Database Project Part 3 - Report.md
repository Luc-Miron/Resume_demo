
# COMP 3380 – Database Project Part 3 - Report


<img src="/Users/Luc/Documents/Active_Course_Folders/Comp3380/Project/Part3/1536849.png" width="20%" height="20%" align="right">

### Group Members: 
* **Alyssa Gregorash**
* **Henry Wong** 
* **Luc Miron**

#### Due November 27, 2022

## Data Summary

The data contains information about **Pokémon**, their **abilities**, **moves**, **evolution**, **types**, **forms**, and other related elements like the moves themselves and **items**.  The data originated from the main series Pokémon games from generations 1-8 inclusive.  Side games are not included. Generation 9 wasn’t released until the due date for part 2, and therefore was not included.  This data was provided by [Pokémon Essentials](https://pokemon-fan-game.fandom.com/wiki/Pokémon_Essentials), a fan-made tool used for making Pokémon fan games.  This data belongs to [Nintendo](http://www.nintendo.com) and the [Pokémon Company](http://www.pokemon.com) and they do not endorse its use here.  Supplementary data was sourced from [Bulbapedia](http://bulbapedia.com), a Pokémon wiki, and its data is similarly owned by [Nintendo](http://www.nintendo.com) and the [Pokémon Company](http://www.pokemon.com).

This data was chosen because our group had no previous experience working with a dataset, except for one member who was familiar with this data.  It was agreed upon as the best option since the data was large enough and it could easily lead to making interesting queries.  

The data is over `4,200 KB` with over `72,624` combined entries over all the tables.  `Pokémon_Move` is the largest table by far, having around twenty entries per form for over 900 Pokémon. The `sql` file for inserting all the entries into this table was split into six parts to improve performance during insertion.

<img src="/Users/Luc/Documents/Active_Course_Folders/Comp3380/Project/Part3/Pokemon_DB_ER.png">

## The Data Model

The initial model didn’t suffer when moved into a relational model, as it was laid out with the translation in mind.

Changes to the model betweeen parts 1 and 2 were mostly minimal.  One of the changes, was to add another attribute for the `Move`, `Ability`, and `Items` entities to include the internal game engine name as there were instances of duplicate in-game display names. By including the internal name, it was easier to construct our queries.

### The Data Model Breakdown

Some of the tables were obvious to create.  The `Pokémon` entity needed its own table as it is central to the database and has many more attributes than all other entities.  **Pokémon**, **Items**, **Abilities**, and **Moves** all have different functions within the game and so it should not be in the same table as those entities or grouped with another as a single entity. For example, it does not make sense for an _item_ to have an _‘hp’_ value.

`Ability` and `Move` entities were each their own table due to different in-game functionality. **Abilities** are considered passive powers while **Moves** are active actions for Pokémon. These two entities deserved their own table as Pokémon can share many combinations of _move_ and _ability_.

The `Egg group` entity also had its own table as many Pokémon belong to certain egg groups.

`Type` is an entity that shares a relation to both `Pokémon` and `Move`. Every _move_ has a _type_ associated with it and every _Pokémon_ can have one or more _types_. Due to this, `Type` was given its own table as we would have thousands of duplicate entries for a domain consisting of (currently) only 19  distinct values. 

`Status` has relationships with many other entities such as `Type`, `Move`, and `Items`. _Items_ can cure many _statuses_, _types_ can have immunity to many _statuses_, and _moves_ can inflict many _statuses_. As `Status` was referenced multiple times, having its own table was warranted. 

### The Data Model Difficulties

#### Part 1

One of the early hurdles we had to address was how we were going to incorporate and implement **Pokémon forms**. They were very diverse and more common than expected. _Forms_ are referenced in the game by sharing a _Pokedex number_ with their respective base form. At first, we considered making `Form` a weak entity but quickly realized that the _form_ of a _Pokémon_ can also change its _stats_, _abilities_, _moves_, etc..  We explored the idea of making the form a subclass of Pokémon. We determined this would make it challenging to model certain relations because any relations needingto a specific `Form` would also require a _form ID_ value. Since there are many Pokémon that do not have any form changes, we would have been showing null for thousands of attribute values, which was undesirable. Our final solution was to have a Pokémon's base form referred to as form 0 and every other form be given subsequent form numbers. 

> For example, relations like `Galarian Meowth` (pokedex #52, form #2) evolving into `Perrserker` (pokedex #863, form #0) was easy since the numbers could be accessed without the complexity of weak entities or subclass specifics. 

In the end, we decided that every entity for `Pokémon` would have two primary keys and every relation that can be formed with a specific _form of Pokémon_ would require these two values passed every time.

With `Form`, we had initially considered adding a `duration` attribute to it but struggled to find an appropriate entity or relation to attach it to. `Duration` was thought to be important at the time, but the attribute value for almost every form would be either _indefinite_ or _until changed back_ which would make the attribute uninteresting.   

We also had to consider if a `Pokémon` should include its _form name_. We had to ask ourselves, “should Alolan Vulpix be stored as `Alolan Vulpix` or broken up like name=`Vulpix` and formName=`Alolan`?”. Since the name was not used as any key, we merged the form name with the species name to make it more apparent for queries.  

Another attribute we decided not to add in our final ER model was the Pokémon’s `Gender Ratio`.  We decided it was not very interesting and it did not depend on form ID’s.

#### Part 2
The original data files we used from our source to build our database were not in the traditional `comma separated value` format.
 
While writing the parser was not difficult, we quickly found out that it was not as straightforward as simply "**counting lines**". There were many instances in our data where extra attributes (lines) would show up and disrupt the parser. In some cases, this forced us to go into the data file itself and make changes in order for our parses to function properly.

Another setback, was that some data was not present in the data files and had to be manually sourced from [Bulbapedia](http://bulbapedia.com). One such example is, moves that inflict status effects.  In these cases, one member opened a wiki and manually wrote the required `insert` statements.  

We had initially planned to use the in-game display name as the primary key for many entities but later found edge cases where the in-game display name was duplicated or had different descriptions and functions. For example, there were two _abilities_ called `As One` that do different things. Using the internal names `ASONEGRIMNEIGH` and `ASONECHILLINGNEIGH` was the best way to distinguish them.  

In our ER diagram, we had a relationship where an item can teach a move. We wanted to create a table for this relationship but since it would have a 1-to-1 relationship between `Item` and `Move`. We ultimately decided to have it attached to the Item entity as an attribute and have the items that do not teach any moves have a null value.  

At a logical level, _egg groups_ do not need a _form id_ since all _forms_ of a particular **Pokémon** will belong to the same **egg group**. When writing our insert statements, we quickly found that we had to include both primary keys from the `Pokémon` entity in order for it to work smoothly.  

While brainstorming ideas for queries, we thought about making a query that would display the `evolution line` of a **Pokémon**. The concept was to show all pre and post evolutions of a **Pokémon** as a type of lineage. We determined this type of query was not possible without multiple recursive calls or repeating calls inside a while loop. This would be further complicated with branching evolutions and forms that might result in an evolution **tree** or worse, a **cycle**! This idea was ultimately dropped in favor of a simplified version that returns all **Pokémon forms** that can evolve and what **Pokémon** they evolve into.

## Interesting Queries

* `stab` is one of our most interesting queries. **Stab moves** are learned by and share a _type_ with the _Pokémon_ learning them. There is an additional layer of complexity as **Pokémon** can have different forms and can change their `Type`. This becomes clear when we compare the _stab moves_ between different forms of _Pokémon #493_ for instance. The list of moves that a Pokémon can learn changes depending on their form number. Counting the number of _stab moves_ is similarly interesting.  
* `pokeMoveType` is set up in a way to account for both of a Pokémon’s types when deciding how effective a given move is. It will print whether the Pokémon is immune to, weak to, resistant to, or affected normally by a given move.
* `othertype` is a query that will list **Pokémon** that can learn moves from a given _type_ to which they do not belong.
* `st` shows the highest stats for **Pokémon** of each type by taking the sum of `hp`, `atk`, `def`, `satk`, `sdef`, and `sped` as **total base stats**. Do any **Pokémon** top both of their types?  Notice any patterns, such as mega forms being frequent?

## The Data Model in Another Form

This data set does not explicitly require the use of a relational database but it was the most appropriate due the amount of referencing between entities. Our database is static since we will not be frequently updating values. Therefore, it would naturally be more structured. We are not likely to be deleting entries, so with a relational model, we can ensure greater data integrity compared to **Graph** or **NoSQL** databases.

As mentioned earlier, we felt the Pokémon `evolution line` query would have clearly worked better within a _graph database_. The nature of _graphs_ and their ability to represent _trees_, _cycles_ and _paths_ would be better suited to this type of query. Displaying the `evolution line` is possible in a _relational database_ but we were not satisfied with how it would be represented. Further, the complexity of this query in a _relational database_ combined with the error checking required, makes it a daunting task, whereas, within a _graph database_, the query can be written in a single line.

## Would this be a good tool for others?

This **Pokémon** database may work well for individuals that are familiar with Pokémon. It may not cover topics such as region but it has enough depth to be interesting. Many examples we saw in class work with having only one primary key referenced for a relation between two entities. This database can be used as an example of how to make relations using more than one primary key.    

The main drawback is that it requires a bit of knowledge of the **Pokémon** universe. Certain entities can be confusing to the uninitiated, such as the difference between an ability and a move. Second, although the database size is modest by database standards, it may be too large for in-class exercises.
