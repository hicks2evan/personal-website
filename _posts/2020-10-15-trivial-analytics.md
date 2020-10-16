---
title: "Trivial Analytics"
subtitle: "Extracting data about common Jeopardy! topics using Python"
published: true
image_url: "featured/2020-10-15.png"
---

The purpose of this post is to try to use some data analytics to answer a question that came up in a conversation between me and my trivia teammates. Before the coronavirus put our favorite bar trivia night on hold, my friends and I had a ritual of appearing there every Wednesday night at 7pm and answering 8 rounds of trivia questions on a variety of [geek pop-culture subjects](https://www.geekswhodrink.com/). The question that arose at our table was along these lines:

> What would you give to know the 10 most important topics to study for bar trivia?

It's pretty natural to start to try to answer this question with data. For our particular trivia game, if we had the data it would be great to know which *things* appear the most in questions and answers. Knowing, for example, that Beyonce is 10% more likely to appear in an audio round than Bruno Mars is a pretty critical piece of information for somebody with a limited amount of time to prepare to win that sweet, sweet $20 bar cash.

![Bruno Mars and Beyonce playing Jeopardy! side by side with audio equipment around them.](/assets/images/posts/2020-10-15/1.png)

Extracting this type of insight is 'non-trivial', even assuming a perfect world where I had access to a data set with my bar trivia game's questions and answers. No such data set exists. But, after some sleuthing I found a reddit poster sharing a data set with 200,000+ Jeopardy! questions [here](https://www.reddit.com/r/datasets/comments/1uyd0t/200000_jeopardy_questions_in_a_json_file/). Bad news for my trivia team, I don't have the resources to crack the code our game, but maybe we can learn something by asking an amended question of the Jeopardy! data:

> If you only could study 10 topics in preparation for Jeopardy!, which topics should you study?

## Working with the data
The first step was to set up the data in a way that would be easy for me to work with in this project. I spun up a mySQL database and added a `question` table to hold the question data from reddit. There were a number of columns that I imported, but mostly we care about the `question` and `answer` fields.

![Machine with worker loading television sets displaying Jeopardy! question values onto a conveyor belt, goes through a series of transforms to a console diplay with another worker observing output at a keyboard. Python logo, a symbol representing a CSV sheet are on the side of the machine.](/assets/images/posts/2020-10-15/2.png)

The Juptyer notebook for this post is set up to easily connect to a local mySQL database assuming it is set up a similar way. Python can connect to mySQL using a package called `mysql.connector`.


```python
import mysql.connector

def connectToMySQL():
  mydb = mysql.connector.connect(
    host="localhost",
    user="service",
    password="jeopardy!",
    database="jeopardy_db"
  )
  print("Connected.")
  print()
  return mydb


```

With a connection to the database open, you can execute normal SQL queries. Right away we are able to ask some fairly smart things if we know what we are looking for, like *show me five questions about Egypt*.


```python
mydb = connectToMySQL()
cursor = mydb.cursor()
cursor.execute("SELECT question, answer FROM question WHERE question like '%Egypt%' LIMIT 5")

for (question, answer) in cursor:
    print("Question: | " + question)
    print("Answer:   | " + answer)
    print()

cursor.close()
```

    Connected.
    
    Question: | 'Cleopatra's Needle is a short walk from this Egyptian Temple in the Metropolitan Museum of Art'
    Answer:   | the Temple of Dendur
    
    Question: | 'In 46 B.C. this Egyptian came with Caesar to Rome, where her statue was placed in the temple of Venus Genetrix'
    Answer:   | Cleopatra
    
    Question: | '"The Prince of Egypt"" featured Ralph Fiennes as the voice of this stubborn ruler'"
    Answer:   | the Pharaoh
    
    Question: | 'This city of east central Egypt is the southern half of the site of ancient Thebes'
    Answer:   | Luxor
    
    Question: | 'A short war between Israel & Egypt & Syria in October 1973 was named for this high holiday'
    Answer:   | Yom Kippur
    
    True



So now we have a database set up and we can write queries to ask it smart things. Like mentioned above, however, this requires us to know what we are asking. Questions like *what 10 things should I study* won't fly because we can't write a query yet for *things* we don't know we care about. We need some way to figure out what *things* in the questions are important.

## Named Entity Recognition
So what do I mean by *thing*? One naive solution to our problem might be to just look for common appearances of certain words. For example, if "America" appears regularly in questions, then that might be an important country to study, right? Well, practiced trivia players know that trivia is all about going more fine-grained than that. American History may be a very important subject to study, but at the end of the day, you may need to know some specifics about Hamilton that you may gloss over if you only study American History broadly. 

Consider another issue of the word count solution, it may tell you that it is quite important to know about "Alexander", but *Alexander-who?* Alexander Hamilton and Alexander the Great might both be important, but the word count solution doesn't tell us who is *more* important.

![Alexander Hamilton and Alexander the Great are hooked up by their craniums to an apperatus that has determined they are both named "Alexander".](/assets/images/posts/2020-10-15/3.png)

Another idea is to use the `category` of a question. That should help us get to the meat of what a question is about, but viewers of Jeopardy! will know well that the category is usually not useful, if not downright distracting. Categories like "African Geography" are way too broad to be useful. Meanwhile, many of the Jeapordy! categories are unique to the game, playful rhymes or word games.

In general, it looks like we are trying to extract people, places, times, etc. from the questions. In Natural Language Processing (NLP) there is a name for annotating this type of information, "Named Entity Recognition". Fortunately there are handy Python libraries out there like [spaCy](https://spacy.io/api/entityrecognizer) that can do the heavy lifting for this.



```python
# Reference https://towardsdatascience.com/named-entity-recognition-with-nltk-and-spacy-8c4a7d88e7da
import spacy
from spacy import displacy
from collections import Counter
import en_core_web_sm
nlp = en_core_web_sm.load()

def printAnnotation(q):
    doc = nlp(q)
    print([(X.text, X.label_) for X in doc.ents])

q = '"The Prince of Egypt" featured Ralph Fiennes as the voice of this stubborn ruler'
printAnnotation(q)
```

    [('The Prince of Egypt', 'WORK_OF_ART'), ('Ralph Fiennes', 'PERSON')]


Here we can see spaCy was able to identify to named entities in this question, "The Prince of Egypt" was labeled as a work of art (animated film, go watch it), and "Ralph Fiennes" was labeled as a person. This works pretty well generally, but it isn't perfect. For example, below it has decided that "Israel & Egypt & Syria" are an organization all-together. Hopefully these things will *come out in the wash* so to speak, but we should keep an eye out for misleading entities.



```python
q = "A short war between Israel & Egypt & Syria in October 1973 was named for this high holiday"
printAnnotation(q)
```

    [('Israel & Egypt & Syria', 'ORG'), ('October 1973', 'DATE')]


## Mapping Questions to Named Entities
Next thing to do to get an idea of which named entities occur frequently, we want to map questions and answer text to named entities. For this, we need a new table to track those entities and their labels, as well as a mapping table to handle the many to many relationship of question to named_entity. For those of you who aren't familiar with the idea of using mapping tables for many to many relationships in normalized databases, check out [this post](https://www.joinfu.com/2005/12/managing-many-to-many-relationships-in-mysql-part-1/).

![Entity relationship diagram describing the many to many relationship problem given two entities. It shows an example without mapping, and then with a mapping table.](/assets/images/posts/2020-10-15/4.png)

With that in place, we can set up some templated queries using template literals in Python. Establishing these common uses up front will allow us to get some reuse out of them as we go through the remainder of the project.


```python
# Query to get all questions from question table with limit and offset to paginate
get_all_questions = ("SELECT question_id, question, answer FROM question LIMIT %s OFFSET %s")

# Queries to wipe out tables before re-seeding
delete_mappings = "DELETE FROM question_entity_map"
delete_entities = "DELETE FROM entity"

# Queries to add named entities and mappings
add_entity = ("INSERT INTO entity (name, label) VALUES (%s, %s)")
add_mapping = ("INSERT INTO question_entity_map (question, entity) VALUES (%s, %s)")
```

The code below will seed our two new tables. It goes question by question and looks for named entities that it hasn't seen before and adds them to the `entity` table. It doesn't add the entity if it is a recurrence. In any case it adds a mapping to the question in the `question_entity_map` mapping table. First it needs to clear out whatever was seeded previously. I paginated the operation so that it doesn't do one **huge** insert, but this can be controlled by tweaking the limit and offset variables.

Go get a cup of coffee while this one runs.


```python
import regex
import unidecode
import spacy
from spacy import displacy
from collections import Counter
import en_core_web_sm
nlp = en_core_web_sm.load()

mydb = connectToMySQL()
cursor = mydb.cursor()

print("Deleting previously seeded records...")
cursor.execute(delete_mappings)
cursor.execute(delete_entities)
mydb.commit()

limit = 5000
start = 0
end = 200000
data_entities = dict()
all_data_entities = dict()
data_mappings = []

for i in range(int(start/limit), int(end/limit)):
  offset = i * limit
  print("Starting get for offset " + str(offset) + "...")
  get_data = (limit, offset)
  cursor.execute(get_all_questions, get_data)

  for (question_id, question, answer) in cursor:
    q = unidecode.unidecode(regex.sub("'", "", question + " " + answer))
    doc = nlp(q)
    for X in doc.ents:
        entity = X.text.lower()
        if not entity in data_entities:
          if not entity in all_data_entities:
            data_entities[entity] = X.label_
        data_mapping = (question_id, entity)
        data_mappings.append(data_mapping)

  cursor.executemany(add_entity, (list(data_entities.items())))
  mydb.commit()
  cursor.executemany(add_mapping, (data_mappings))
  mydb.commit()

  all_data_entities = {**all_data_entities, **data_entities} 
  data_entities.clear()
  data_mappings = []

print("Done, closing...")
cursor.close()
```

    Connected.
    
    Deleting previously seeded records...
    Starting get for offset 0...
    Starting get for offset 5000...
    Starting get for offset 10000...]
    ...
    Starting get for offset 190000...
    Starting get for offset 195000...
    Done, closing...
    
    True



With that in place, we should be able to query the mapping table for information about occurrences of certain named entities in questions. In fact, the entities with the highest count in the mapping table are those entities that appeared most commonly in questions. We can write a rudimentary query to try to answer our original question.


```python
mydb = connectToMySQL()
cursor = mydb.cursor()

cursor.execute("SELECT entity, COUNT(*) FROM question_entity_map GROUP BY entity ORDER BY COUNT(*) DESC LIMIT 10;")

for r in cursor:
    print(str(r))
    
cursor.close()
```

    Connected.
    
    ('first', 4385)
    ('one', 3134)
    ('2', 2453)
    ('u.s.', 2210)
    ('french', 1258)
    ('3', 1059)
    ('british', 958)
    ('1', 889)
    ('greek', 772)
    ('american', 766)
    
    True



We're getting closer. Not too surprisingly, we see frequent occurences of what look like geographic or linguistic designations. French, American, British, and Greek. There also appear to be cardinal numbers, as well as some ordinal numbers like First and Second. Let's look into the labels for some of these frequently appearing entities. To do this, we need to be able to pull back data from `question`, but also take into account the label in the `entity` table. We can write a join to look for questions containing named entities with those labels.


```python
# Join to pull data from question table and entity table linked by mapping table
get_question_by_entity_type = (
    "SELECT question.question, entity.label FROM question_entity_map " + 
    "LEFT JOIN question ON question.question_id = question_entity_map.question " +
    "LEFT JOIN entity ON question_entity_map.entity = entity.name " +
    "WHERE question_entity_map.entity = '%s' LIMIT 5;"
)
```

Using that we can query for various questions including entities that seem off, and see what their labels are.


```python
mydb = connectToMySQL()
cursor = mydb.cursor()

print('american:')
cursor.execute(get_question_by_entity_type % 'american')
for r in cursor:
    print(str(r))
print()

print('first:')
cursor.execute(get_question_by_entity_type % 'first')
for r in cursor:
    print(str(r))
print()

print('one:')
cursor.execute(get_question_by_entity_type % 'one')
for r in cursor:
    print(str(r))
print()

cursor.close()
```

    Connected.
    
    american:
    ('\'"American poet... became known as a leader of the Beat literary movement of the 1950s""\'"', 'NORP')
    ('\'"One of the most original and provocative American architects working today""\'"', 'NORP')
    ("'Sukkot, a Jewish festival, began as a harvest celebration & was a model for this centuries-old American holiday'", 'NORP')
    ("'This astronomer for whom a space telescope is named is honored in the American Scientists series'", 'NORP')
    ("'O Canada celebrates Canada Day on this date, 3 days before a big American holiday'", 'NORP')
    
    first:
    ("'Cows regurgitate this from the first stomach to the mouth & chew it again'", 'ORDINAL')
    ("'Karl led the first of these Marxist organizational efforts; the second one began in 1889'", 'ORDINAL')
    ('\'This "Modern Girl"" first hit the Billboard Top 10 with ""Morning Train (Nine To Five)""\'"', 'ORDINAL')
    ("'Warhol became the manager of this Lou Reed rock group in 1965 & produced their first album'", 'ORDINAL')
    ("'His first act after being sworn in as president of the Confederacy was to send a peace commission to Washington, D.C.'", 'ORDINAL')
    
    one:
    ("'In geologic time one of these, shorter than an eon, is divided into periods & subdivided into epochs'", 'CARDINAL')
    ('\'Teri Hatcher looked "shipshape"" as one of the singing ""mermaids"" who jumped on board this cruisin\' series in 1985\'"', 'CARDINAL')
    ('\'One edition calls this Darwin opus one of "the most readable and approachable"" of revolutionary scientific works\'"', 'CARDINAL')
    ('\'One edition calls this Darwin opus one of "the most readable and approachable"" of revolutionary scientific works\'"', 'CARDINAL')
    ("'If Joe DiMaggio's hitting streak had gone one more game in 1941, this company would have given him a $10,000 contract'", 'CARDINAL')

    True



It looks like there are some entity labels that we can rule out. "ORDINAL", "CARDINAL", and "NORP" appear to be presenting an issue already. At this point, one might start to wonder if it is less about entity labels we don't care about, and more about the few we *do* care about. 

Interestingly, the first appearance of a topic that feels truly *trivial* in nature is "The Clue Crew" with 363 occurrences in questions. Looking closely to see how it is labeled, "The Clue Crew" (the group from the Nancy Drew childrens book series) is an "ORG". Glancing through some of the common labels, it looks like we might care about "PERSON", "ORG" and "WORK_OF_ART" at least as a start.


```python
mydb = connectToMySQL()
cursor = mydb.cursor()

cursor.execute(
    "SELECT entity.name, COUNT(*) FROM question_entity_map " + 
    "LEFT JOIN entity ON question_entity_map.entity = entity.name " +
    "WHERE entity.label IN ('ORG','PERSON','WORK_OF_ART')" +
    "GROUP BY entity ORDER BY COUNT(*) DESC LIMIT 10;"
)

for r in cursor:
    print(str(r))

cursor.close()
```

    Connected.
    
    ('oscar', 356)
    ('congress', 213)
    ('nyc', 194)
    ('the clue crew', 175)
    ('shakespeare', 164)
    ('senate', 148)
    ('jesus', 137)
    ('nba', 129)
    ('lincoln', 129)
    ('nfl', 123)

    True



Looks like someone looking to up their Jeopardy! game (or any well-rounded individual I suppose), should spend some time reading up on the Oscars, the U.S. Congress, NYC, Shakespeare, and Jesus. Or at least, those are some highly frequent entities in questions. This gives a mostly satisfying answer to the original, but leaves a little bit to be desired.

Let me point out a couple of problems. "The Oscars", again, is a pretty broad category. Might be good to know it is worth going and memorizing, as far as award shows go, but it isn't quite as concrete as "Lincoln". Actually most of these results are organizations worth mentioning in a question, but not necessarily the topic of the question.

This also doesn't necessarily get at which topics are best to study for Jeopardy!. Some other factors could come into play. As an example, reading all of "Shakespeare" is a lot of work, wouldn't it be nice to know that he is only marginally more necessary than "Harry Potter", but slightly less niche?

![Cartesian coordinate diagram plotting categories of questions based on being more or less niche, and more or less frequent. The best categories to study would be both high frequency, and specific.](/assets/images/posts/2020-10-15/5.png)

## What next?

I hinted at it above, but it looks like there are a couple of fast follows for this analysis.

1. We need to figure out how to get to the "meat" of what a question is about. That way we can avoid tangential but highly occurring entities. I need to do some research on what is out there, but basically I want to be able to tell what is the central "Wikipediable" topic of every question, and count occurrences from there.
2. Wouln't it be really nice to know about how niche something is as a piece of data about the topic? If we had that, we could do some kind of composite rank. Highly occurring topics that are less broad would be the easiest to study quickly, thus rewarding the player the most.

I'm thinking I'll need to address those ideas in another post. Until then, play on.
