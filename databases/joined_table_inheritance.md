# Joined Table Inheritance in SQLAlchemy

Have you ever wondered how you can load users from different tables in a single database? Let us say you are building an elearning application. The database features tables for a student, a teacher, a parent, and maybe an admin. These tables have some few common fields, such as name, email, and password. Individually, a student's table may contain fields like age and school. The teacher's table may contain the course they will be teaching, and the parent's table may contain the child's name. The admin's table may contain the admin's residence.

If you are already familiar with how to register and log in a user, you probably know that Flask login uses sessions to store the user's id. You begin by letting the database know what user you want to load.

```python

from app import login, db
from flask_login import UserMixin


@login.user_loader
def load_user(id):
    return Student.query.get(int(id))


class Student(UserMixin, db.Model):
    __tablename__ = 'student'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(64), index=True, unique=True)
    password = db.Column(db.String(64))
    age = db.Column(db.Integer)
    school = db.Column(db.String(64))


class Teacher(UserMixin, db.Model):
    __tablename__ = 'teacher'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(64), index=True, unique=True)
    password = db.Column(db.String(64))
    course = db.Column(db.String(64))


class Parent(UserMixin, db.Model):
    __tablename__ = 'parent'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(64), index=True, unique=True)
    password = db.Column(db.String(64))
    child = db.Column(db.String(64))


class Admin(UserMixin, db.Model):
    __tablename__ = 'admin'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(64), index=True, unique=True)
    password = db.Column(db.String(64))
    residence = db.Column(db.String(64))

```


The `load_user` function is a callback that Flask-Login calls when a student logs in. The function takes the student's id as an argument and returns the user object. Flask-Login then stores the student object in the session. However, as you can see, we are only loading a user from the `Student` table. What about the other tables? 

## Table of Contents

These are the items we will look at in detail as we try to understand the concept of joined table inheritance.

1. [Types of Inheritance Hierarchies](#types-of-inheritance-hierarchies)
2. [Joined Table Inheritance](#joined-table-inheritance)
3. [Relationships with Joined Inheritance](#relationships-with-joined-inheritance)
4. [Loading Inheritance Hierarchies](#loading-inheritance-hierarchies)
5. [Referring to Specific Subtypes on Relationships](#referring-to-specific-subtypes-on-relationships)
6. [Loading Objects with Joined Table Inheritance](#loading-objects-with-joined-table-inheritance)


## Types of Inheritance Hierarchies

SQLAlchemy supports three types of inheritance hierarchies:

* **Single Table Inheritance**: several types of classes are represented by a single table
* **Concrete Table Inheritance**: each type of class is represented by independent tables
* **Joined Table Inheritance**: the class hierarchy is broken up among dependent tables, each class represented by its own table that only includes those attributes local to that class.


## Joined Table Inheritance

From the simple description above, we can see that the concept of joined table inheritance is the most suitable for our scenario. So what exactly happens in a joined table inheritance? Here, a distinct table is used to represent each class along a hierarchy of classes. When you query a particular subclass in the hierarchy, an SQL JOIN is rendered in its inheritance path. The default behavior when querying the base class is to include only the base table in the SELECT statement. The subclasses will be instantiated against the the base class using a discriminator column. 

If you are not familiar with SQL queries, you can read more about this in the [PostreSQL tutorial](postgresql.md).


```python

class User(db.Model):
    __tablename__ = 'user'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(64), index=True, unique=True)
    password_hash = db.Column(db.String(64))
    type = db.Column(db.String(64))

    __mapper_args__ = {
        'polymorphic_identity': 'user',
        'polymorphic_on': 'type'
    }

class Student(User):
    __tablename__ = 'student'
    id = db.Column(db.Integer, db.ForeignKey('user.id'), primary_key=True)
    age = db.Column(db.Integer)
    school = db.Column(db.String(64))
    __mapper_args__ = {
        'polymorphic_identity': 'student',
    }


class Teacher(User):
    __tablename__ = 'teacher'
    id = db.Column(db.Integer, db.ForeignKey('user.id'), primary_key=True)
    course = db.Column(db.String(64))
    __mapper_args__ = {
        'polymorphic_identity': 'teacher'
    }


class Parent(User):
    __tablename__ = 'parent'
    id = db.Column(db.Integer, db.ForeignKey('user.id'), primary_key=True)
    child = db.Column(db.String(64))
    __mapper_args__ = {
        'polymorphic_identity': 'parent'
    }

```

The base class is `User` and the subclasses are `Student`, `Teacher`, and `Parent`. The `__mapper_args__` dictionary is used to specify the polymorphic identity and the polymorphic on column. The base class in a joined table inheritance hierarchy is configured  with additional arguments that will refer to the polymorphic discriminator column as well as Pthe identifier for the base class.

From the base class example above, an additional column `type` is established to act as the discriminator. It is configured using the `mapper.polymorphic_on` parameter. This column will store a value which will be used to indicate the type of object represented within the row. This column's value may be of any datatype, though string and integer are the most common.  The `polymorphic_identity` parameter is used to specify the value or the actual data that will be stored in the `type` column when an instance of the base class is created.

Something to note here is that a polymorphic discriminator expression is not strictly necessary, though it is required if polymorphic inheritance is to be used. All you need to do is to establish a simple column on the base table that will store the polymorphic identity.

> As of this writing, only one discriminator column or SQL expression may be configured for the entire inheritance hierarchy, typically on the base-most class in the hierarchy.

With the base class in place, we can now proceed to define the `Student`, the `Teacher` and the `Parent` child classes. Each of these subclasses contain columns that represent the attributes unique to the subclass they represent. Each table is also expected to contain a primary key column as well as a foreign key reference to the parent table.

The `polymorphic_identity` parameter is specified within the mapper arguments of each subclass. This value, which should be unique to each mapped class across the whole hierarchy, is used to populate the column designated by the `mapper.polymorphic_on` parameter established in the base mapper. Only one 'identity' per mapped class is allowed. The Object Relational Mapper (ORM) uses the value of the `polymorphic_identity` to determine which class a row belongs to when loading rows polymorphically. In our examples above, every row which represents a `Student` will have the value `student` in its `type` row. The same applies to the `Teacher` and the `Parent` subclasses.

Notice that both the primary key and the foreign key which references the parent class both belong to the same column. You may be familiar with the setup below when dealing with multiple tables:


```python

class Student(UserMixin, db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(64))
    posts = db.relationship('Post', backref='author', lazy='dynamic')



class Post(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    body = db.Column(db.String(140))
    student_id = db.Column(db.Integer, db.ForeignKey('student.id'))
```

We have used a foreign key to reference the `id` column in the `Student` table. Both the `id` column and the foreign key column are separate. There is absolutely nothing wrong with this design. In a polymorphic setup, it is very common that the foreign key constraint is established on the same column as the primary key itself, even though this is not required.

> Naturally, when dealing with joined inheritance primary keys, the `id` columns of the subclasses are not used to locate the individual objects of the subclasses, in this case the `Student`, the `Teacher` and the `Parent`. Only the value in the `user.id` is considered. `student.id`, among the others, are still valid and can be used to locate the joined row once the parent row has been determined within an SQL statement.

When you query against the `User`, a combination of the `User`, `Student`, `Teacher` and `Parent` objects are returned. They will automatically populate the `user.type` column with the correct discriminator value as is appropriate. 


## Relationships with Joined Inheritance

Relationships are fully supported in joined inheritance table. It works similarly to the `Student` and `Post` example shown above.


```python

class User(UserMixin, db.Model):
    __tablename__ = 'user'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(64), index=True, unique=True)
    password_hash = db.Column(db.String(64))
    type = db.Column(db.String(64))

    __mapper_args__ = {
        'polymorphic_identity': 'user',
        'polymorphic_on': 'type'
    }

class Student(User):
    __tablename__ = 'student'
    id = db.Column(db.Integer, db.ForeignKey('user.id'), primary_key=True)
    age = db.Column(db.Integer)

    school_id = db.Column(db.Integer, demployeeb.ForeignKey('school.id'))
    school = db.relationship('School', backref='student', lazy='dynamic')

    __mapper_args__ = {
        'polymorphic_identity': 'student',
    }


class School(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(140))

    learners = db.relationship('Student', backref='learner', lazy='dynamic')
```

If the foreign key is on a table corresponding to a subclass, the relationship should target the subclass. Above we have created a relationship between the `Student` and the `School`. The `Student` will have a `Student.school` attribute; the `School` will have the `School.learners` attribute that always loads against the join of the `user` and the `student` tables together.

How about the relationship between two parent tables? 

```python
class User(db.Model):
    __tablename__ = 'user
    # ...

    school_id = db.Column(db.ForeignKey('school.id'))
    school = db.relationship('School', back_populates='user')



class School(db.Model):
    __tablename__ = 'school'
    # ...

    learners = db.relationship('User', back_populates='school')
```

Above, the `user` table has a foreign key constraint back to the `school` table, hence this relationship is set up between `User` and `School`. The idea is to target the class in the hierarchy that also corresponds to the foreign key constraint.


## Loading Inheritance Hierarchies

When classes are mapped in inheritance hierarchies using the 'joined', 'single', or 'concrete' table inheritance styles, the usual behavior is that a query for a particular base class will also yield objects corresponding to  subclasses as well, hence the term 'polymorphic loading'.

Polymorphic loading comes with an additional problem of _which subclass attributes are to be queried upfront_, and _which are to be loaded later_. When an attribute of a particular subclass is queried up front, it can be used in the query as something to filter on, and it will be loaded when we get our objects back. On the other hand, if it is not queried up front, it gets loaded later when we first need to need to acceess it.

The `with_polymorphic()` function is used to provide a means of specifying which specific subclasses of a particular base class should be included in the query, which implies what columns and tables will be avaliable in the SELECT.

There are two variants to this functions:

- `mapper.with_polymorphic` in conjunction with `mapper.polymorphic_load` option. See [Setting `with_polymorphic` at mapper configuration time](#setting-with_polymorphic-at-mapper-configuration-time).
- Query-level such that we have `query.with_polymorphic()`. See [Using `with_polymorphic`](#using-with_polymorphic) subsection.

To understand the difference between `with_polymorphic()` function and the `query.with_polymorphic()` method, see [Setting `with_polymorphic` against a query](#setting-`with_polymorphic`-against-a-query) section.


### Using `with_polymorphic`


Normally, when a Query specifies the base class of an inheritance hierarchy, only the columns that are local to that base class are queried:


```python
db.session.query(User).all()
```

In single and joined table inheritance, only the columns local to `User` will be present in the SELECT statement. It is possible to get back instances of `Student`, `Teacher`, and `Parent` but they will not have the additional attributes loaded until we first access them, at which a point a lazy load is emitted. If we want to refer to columns mapped to `Student` or `Teacher` in our query that's against `User`, these columns are not directly available in both the single and joined table inheritance cases. This is because the `User` entity does not refer to these columns.

To solve these issues, the `with_polymorphic()` function provides a special `AliasedClass` that represents a range of columns across subclasses. This object can be used in a query like any other alias. When querried, it represents all the columns present in classes given.


```python
from sqlalchemy.orm import with_polymorphic

student_plus_teacher = with_polymorphic(User, [Student, Teacher])
result = db.session.query(student_plus_teacher)
result.all()
```

This is similar to using the SELECT statement below:

```sql
SELECT user.id AS user_id,
       student.id AS student_id,
       teacher.id AS teacher_id,
       user.name AS user_name,
       user.email AS user_email,
       user.type AS user_type,
       student.age AS student_age,
       teacher.course AS teacher_course
FROM user
       LEFT OUTER JOIN student
       ON user.id = student.id
       LEFT OUTER JOIN teacher
       ON user.id = teacher.id
```

You can see that the additional columns for `Student` and `Teacher` are included. This is also the case when you are using single table inheritance. You can also use the asterisk to indicate the inclusion of all subclasses.

```python
# Include column for Student
entity = with_polymorphic(User, Student)

# Include all mappped sub-classes
entity = with_polymorphic(User, '*')
```

### Using aliases with `with_polymorphic`

The function also provides 'aliasing


### Setting `with_polymorphic` at mapper configuration time

We have learnt that the `with_polymorphic()` function serves to load attributes from subclasses, as well as the ability to refer to the attributes from subclasses at query time. This can be referred to as 'eager loading'. When using the `with_polymorphic()` function, we would do this:

```python
from sqlalchemy.orm import with_polymorphic

student_plus_user = with_polymorphic(User, Student)
result = db.session.query(student_plus_user)
result.all()
```

We do have the option of configuring this function as an optional mapper argument. This will allow an entity to use a polymorphic load by default.

```python
class User(UserMixin, db.Model):
    __tablename__ = 'user'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(64), index=True, unique=True)
    email = db.Column(db.String(64), index=True, unique=True)
    password_hash = db.Column(db.String(64))
    type = db.Column(db.String(64))

    __mapper__args__ = {
        'polymorphic_identity': 'user',
        'polymorphic_on': type,
        'with_polymorphic': '*'
    }
```

We have used the asterisk to load all subclasses' columns. It is recommended that this option be used sparingly in joined table inheritance as it implies that the mapping will always emit an often large series of LEFT OUTER JOIN to many tables, which from an SQL perspective is not efficient. However, if you use the `with_polymorphic()` or `query.with_polymorphic()` will override the settings at the mapper level.

You can also polymorphically load a list of subset classes  using the `mapper.with_polymorphic`, just as you would with `query.with_polymorphic`. All you need to do is specify on each class that they should individually participate in the polymorphic loading by default using the `mapper.polymorphic_load` parameter.


```python
class Student(User):
    __tablename__ = 'student'
    id = db.Column(db.Integer, db.ForeignKey('user.id'), primary_key=True)
    age = db.Column(db.Integer)
    school = db.Column(db.String(64))
    __mapper_args__ = {
        'polymorphic_identiy': 'student',
        'polymorphic_load': 'inline'
    }
```

`mapper.polymorphic_load` paramater has been set to the value 'inline' which means that the `Student` class is part of the polymorphic load of the `User` class by default, exactly as though it had been appended to the `mapper.with_polymorphic` list of classes.


### Setting `with_polymorphic` against a query

The function `with_polymorphic()` is an upgrade to the query-level method `query.with_polymorphic()`. They both have the same purpose with the exception that the latter is not as flexible in its usage in that it only applies to the first entity of the query. It then takes effect for all occurences of that entity, so that the entity and its subclasses can be referred to directly, rathar than using an alias.


```python
db.session.query(User).with_polymorphic([Student, Teacher])
```

The `query.with_polymorphic()` mehtod has a more complicated job than the `with_polymorphic()` function, as it needs to correctly transform entities like `Student` and `Teacher` appropriately, but not interfere with other entities. It is recommended that you switch to `with_polymorphic()` if its flexibility is lacking.


### Polymorphic `Selectin` loading

There is an alternative to the `with_polymorphic` functions to eagerly load the additional subclasses on an inheritance mapping, especially when using the joined table inheritance. This is the use of polymorphic 'selectin' loading. It works similar to the `Select In` loading feature of relationship loading.

> Select In loading: the emitted SELECT statement has a much simpler structure than that of the subquery eager loading. It is the most simple and efficient way to eagerly load collections of objects.

```python
from sqlalchemy.orm import selectin_polymorphic

result = session.query(User).options(
    selectin_polymorphic(User, [Student, Teacher])
)
result.all()
```

Above, we have instructed a load of `User` to emit an extra SELECT per subclass by using `selectin_polymorphic` loader option. When the query is run, two additional SELECT statements will be emitted:

```sql
SELECT 
    user.id AS user_id,
    user.name AS user_name,
    user.email AS user_email,
    user.type AS user_type
FROM user
()

SELECT 
    student.id AS student_id,
    user.id AS user_id,
    user.type AS user_type,
    student.age AS student_Age,
    student.school AS student_school
FROM user JOIN student ON user.id = student.id
WHERE user.id IN (?, ?) ORDER BY user.id
(1, 2)

SELECT
    teacher.id as teacher_id,
    user.id As user_id,
    user.type AS user_type,
    teacher.course AS teacher_course
FROM user JOIN teacher ON user.id = teacher.id
WHERE user.id IN (?, ?) ORDER BY user.id
(3,)
```

Just like `with_polymorphic` function, we can configure `selectin_polymorphic` at mapper time for loading to take place by default.


```python
class User(db.Model):
    # ...
    type = db.Column(db.String(64))

    __mapper_args__ = {
        'polymorphic_identity': 'user',
        'polymorphic_on': 'type'
    }


class Student(User):
    # ...

    __mapper_args__ = {
        'polymorphic_identity': 'student',
        'polymorphic_load': 'selectin'
    }


class Teacher(User):
    # ...

    __mapper_args__ = {
        'polymorphic_identity': 'teacher',
        'polymorphic_load': 'selectin'
    }
```

Let us assume that `Student` has an additional relationship called `Student.homework`, that we would eagerly like to load. Unfortunately, `selectin_polymorphic` style of loading does not have the ability to refer to  the`Student` entity within the main query as filter, order by or other criteria, since this entity is not present in the initial query that is used to locate results. A walkaround we can employ to solve this is to apply loader options toward the `Student` which will take effect when the secondary SELECT is emitted.

```python
from sqlalchemy.orm import joinedload, selectin_polymorphic

result = session.query(User).options(
    selectin_polymorphic(User, Student),
    joinedload(Student.homework)
)
result.all()
```

The outcome of running the query above will emit three SELECT statements, though the one against `Student` will be as follows:

```sql
SELECT
    student.id as studentid,
    user.id AS user_id,
    user.type AS user_type,
    student.age AS student_age,
    student.school AS student_school,
    homework.id AS homework_id,
    homework.student_id AS homework_student_id,
    homework.data AS homework_data
FROM user JOIN student ON user.id = student.id
LEFT OUTER JOIN homework as ON student.id = h.student_id
WHERE user.id IN (?) ORDER BY user.id
(3,)

```

## Referring to Specific Subtypes on Relationships

Relationships are used to create a linkage between two mappings. When using `relationship()` where the target class is an inheritance hierarchy, the API allows that the join, eager load or any other linkage should target a specific subclass, alias, or `with_polymorphic` alias of that class hierarchy, rather than the class directly targeted by the `relationship()`.

```python
class User(db.Model):
    __tablename__ = 'user
    # ...

    school_id = db.Column(db.ForeignKey('school.id'))



class School(db.Model):
    __tablename__ = 'school'
    # ...

    learners = db.relationship('User', backref='school')
```

Above, you can see how a relationship has been constructed. A foreign key is used on the `User` to refer to the `School`. The `of_type` method allows the construction of joins along a `relationship()` path while narrowing the criterion to specific derived aliases or subclasses.

```python
result = db.session.query(School).join(School.learners.of_type(Student).filter(Student.name=='rahima'))
```

You can also join from `School` to the polymorphic entity that includes both `Student` and `Teacher` columns.

```python
student_and_teacher = with_polymorphic(User, [Student, Teacher])
result = db.session.query(School).join(School.user.of_type(student_and_teacher))
```

## Loading Objects with Joined Table Inheritance