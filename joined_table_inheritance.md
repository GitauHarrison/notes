# Working with Multiple Database Tables in Flask

Have you ever wondered how you can load users from different tables in a single database? Let us say you are building an elearning application. The database features tables for a student, a teacher, a parent, and maybe an admin. These tables have some few common fields, such as name, email, and password. Individually, a student's table may contain fields like age and school. The teacher's table may contain the course they will be teaching, and the parent's table may contain the child's name. The admin's table may contain the admin's residence.

If you are already familiar with how to register and log in a user, you probably know that Flask login uses sessions to store the user's id. You begin by letting the database know what user you want to load.

```python

from app import login


@login.user_loader
def load_user(id):
    return Student.query.get(int(id))
```

The `load_user` function is a callback that Flask-Login calls when a user logs in. The function takes the user's id as an argument and returns the user object. Flask-Login then stores the user object in the session. However, as you can see, we are only loading a user from the `Student` table. What about the other tables? 