# Sum Repetitive Data In SQLAlchemy

Consider this data:

| Customer Name | Contribution |    Date     |
| ------------- | ------------ | ----------- |
|    Muthoni    |     100      | Jan 1, 2023 |
|    Njeri      |     1000     | Jan 2, 2023 |
|    Gitau      |     750      | Jan 3, 2023 |
|    Muthoni    |     300      | Jan 4, 2023 |
|    Gitau      |     250      | Jan 5, 2023 |
|    Gitau      |     700      | Jan 5, 2023 |
|    Njeri      |     1200     | Jan 6, 2023 |

We have three customers Muthoni, Njeri, and Gitau. They repeatedly make contributions (let us assume it is a self-help group) within January. At the end of the month, we may want to find out how much each person contributed. This data will now look like this at the end of the month:

| Customer Name | Contribution |
| ------------- | ------------ |
|    Muthoni    |     400      |
|    Njeri      |     2200     | 
|    Gitau      |     1700     |

In the aggregate contribution, I am not paying attention to the date. What I am interested in is the total contribution of each member. Now, in the context of SQLAlchemy, how can you achieve such a summation? How can we group the repetitive rows based on a customer's name and sum their contribution?


## Simple Models

A simple database module that captures customer contributions can look like this (I will use Flask for illustration):

```python
# models.py

from app import db
from flask_login import UserMixin
from datetime import datetime


class Customer(UserMixin, db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(64), index=True, unique=True nullable=False)
    registered_at = db.Column(db.DateTime, default=datetime.utcnow)
    contributions = db.relationship('Contribution', backref='customer', passive_deletes=True)



class Contribution(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    contribution = db.Column(db.Integer, default=1, nullable=False)
    date = db.Column(db.DateTime, default=datetime.utcnow)
    customer_id = db.Column(db.Integer, db.ForeignKey('customer.id'), on_delete='CASCADE')

```

In a one-to-many-relationship between the models, a customer can make multiple contributions. Looked another way, many contributions can belong to one customer in a many-to-one relationship. Alright, a customer has used the web app and we have the sample data shown above, how can we present it in a 'cleaner' manner?

## Render 'Clean' Customer Data

A view function that can be used to render this information can be created as follows:


```python
from app import app, db
from app.models import Customer, Contribution
from sqlalchemy import func


@app.route('/customer-contribution')
def customer_contribution():
    contributions = db.session.query(
        Contribution.customer_id, func.sum(
            Contribution.contribution)).group_by(Contribution.customer_id).all()
    return render_template('customer_data.html', contributions=contributions)
```

SQLAchemy provides the `func` namespace that is used to invoke SQL functions. In our case above, we want to invoke the `sum` function and apply it to all customer contributions. So, what is going on here? If you remember from above, it is a customer's name that keeps appearing multiple times. This customer can, and certainly does, make multiple contributions. 

We begin by specifying this `name` column from the `Contribution` model as `Contribution.customer_id`. This is possible because both models are related as specified by the `relationship` method found in the `Customer` model. Using `func`, we invoke the `sum` method on the `contribution` column of the `Contribution` model. Intentionally, this adds all data in the `contribution` column. But before we can finish, we want to ensure that the addition applies only to the said customer whose name keeps appearing multiple times. Therefore, we combine his contribution by using `group_by(Contribution.customer_id)`. What we have so far is the total contribution of a single customer. The `.all()` method is applied to return all customers in the database. In the end, we will have a list with customers' data such as `[('Gitau', 100), ('Njeri', 2200), ('Muthoni', 400)]`. This list is passed to the templates as a variable called `contributions`.


## Display The Data

We can display an individual customer's data on an HTML template as follows:

```html
<!-- customer.html -->

<div class="table-responsive">
    <table class="table table-striped">
        <thead>
            <tr>
                <th>Customer Name</th>
                <th>Contribution</th>
            </tr>
        </thead>
        <tbody>
            {% for contribution in contributions %}
                {% if contribution %}
                    <tr>
                        <td>{{ contribution.customer.name }}</td>
                        <td>{{ contribution[1] }}</td>
                    </tr>
                {% endif %}
            {% endfor %}
        </tbody>
    </table>
</div>
```

We loop through the `contributions` list to access the individual data we want. The Jinja2 templating engine allows for the use of double curly braces `{{  }}` to pass in dynamic data (data that can change). Remember, the customer's name can be gotten from `{{ contribution.customer.name }}`. His total contribution is not found in the database, so we can use an index to access the sum. The conditional `if` statement has been used to check if there is data in our query. If none exists, then nothing will be shown.