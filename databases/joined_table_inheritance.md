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


## Types of Inheritance Hierarchies

SQLAlchemy supports three types of inheritance hierarchies:

* **Single Table Inheritance**: several types of classes are represented by a single table
* **Concrete Table Inheritance**: each type of class is represented by independent tables
* **Joined Table Inheritance**: the class hierarchy is broken up among dependent tables, each class represented by its own table that only includes those attributes local to that class.


## Joined Table Inheritance

From the simple description above, we can see that the concept of joined table inheritance is the most suitable for our scenario. So what exactly happens in a joined table inheritance? Here, a distinct table is used to represent each class along a hierarchy of classes. When you query a particular subclass in the hierarchy, an SQL JOIN is rendered in its inheritance path. The default behavior when querying the base class is to include only the base table in the SELECT statement. The subclasses will be instantiated against the the base class using a discriminator column. 

If you are not familiar with SQL queries, you can read more about this in the [Getting Started with PostreSQL tutorial](getting_started_with_postgresql.md).


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

With the base class in place, we can now proceed to define the `Student`, `Teacher` and `Parent` child classes. Each of these subclasses contain columns that represent the attributes unique to the subclass they represent. Each table is also expected to contain a primary key column as well as a foreign key reference to the parent table.

The `polymorphic_identity` parameter is specified within the mapper arguments of each subclass. This value, which should be unique to each mapped class across the whole hierarchy, is used to populate the column designated by the `mapper.polymorphic_on` parameter established in the base mapper. Only one 'identity' per mapped class is allowed. The Object Relational Mapper (ORM) uses the value of the `polymorphic_identity` to determine which class a row belongs to when loading rows polymorphically. In our examples above, every row which represents a `Student` will have the value `student` in its `type` row. The same applies to the `Teacher` and `Parent` subclasses.

Notice that both the primary key and the foreign key which references the parent class both belong to the same column. You may be familiar with this kind of setup when dealing with multiple tables:


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

Above, we are using a foreign key to reference the `id` column in the `Student` table. Both the `id` column and the foreign key column are separate. There is absolutely nothing wrong with this design. In a polymorphic setup, it is very common that the foreign key constraint is established on the same column as the primary key itself, even though this is not required.

> Naturally, when dealing with joined inheritance primary keys, the `id` columns of the subclasses are not used to locate the individual objects of the subclasses, in this case the `Student`, the `Teacher` and the `Parent`. Only the value in the `user.id` is considered. `student.id`, among the others, are still valid and can be used to locate the joined row once the parent row has been determined within an SQL statement.

When you query against the `User`, a combination of the `User`, `Student`, `Teacher` and `Parent` objects are returned. They will automatically populate the `user.type` column with the correct discriminator value as is appropriate. 