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