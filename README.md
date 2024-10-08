﻿# django_signals

# django_signals


Questions for Django Trainee at Accuknox

# Topic: Django Signals

# Question 1: By default are django signals executed synchronously or asynchronously? Please support your answer with a code snippet that conclusively proves your stance. The code does not need to be elegant and production ready, we just need to understand your logic.

# In thread app's signals.py
from django.db.models.signals import post_save
from django.dispatch import receiver
from django.utils.timezone import now
from myapp.models import MyModel

@receiver(post_save, sender=MyModel)
def my_handler(sender, instance, **kwargs):
    print(f"Signal received at {now()}")
    # Simulate a delay
    import time
    time.sleep(2)
    print(f"Handler finished at {now()}")

# In thread app's models.py

from django.db import models

class MyModel(models.Model):
    name = models.CharField(max_length=100)

    def save(self, *args, **kwargs):
        super(MyModel, self).save(*args, **kwargs)
        print("Model saved.")


# Explanation

1. When you save an instance of MyModel, the post_save signal is sent.
2. The my_handler function is called, which prints the time, waits for 2 seconds, and then prints the time again.
3. You’ll observe that "Model saved." is printed first, then "Signal received at..." and finally "Handler finished at...".

This synchronous behavior shows that Django waits for my_handler to complete before it finishes processing the save method.

# Question 2: Do django signals run in the same thread as the caller? Please support your answer with a code snippet that conclusively proves your stance. The code does not need to be elegant and production ready, we just need to understand your logic.

# In thread app's signals.py

import threading
from django.db.models.signals import post_save
from django.dispatch import receiver
from myapp.models import MyModel

@receiver(post_save, sender=MyModel)
def my_handler(sender, instance, **kwargs):
    print(f"Signal handler thread ID: {threading.get_ident()}")

# In thread app's models.py

from django.db import models

class MyModel(models.Model):
    name = models.CharField(max_length=100)

    def save(self, *args, **kwargs):
        print(f"Save method thread ID: {threading.get_ident()}")
        super(MyModel, self).save(*args, **kwargs)

# Explanation

1. When you save an instance of MyModel, it prints the thread ID where the save method is executed.
2. The signal handler prints the thread ID where the signal is processed.
3. You will find that both IDs are the same, indicating that the signal handler runs in the same thread as the caller.

# Question 3: By default do django signals run in the same database transaction as the caller? Please support your answer with a code snippet that conclusively proves your stance. The code does not need to be elegant and production ready, we just need to understand your logic.

# In thread app's signals.py

from django.db.models.signals import post_save
from django.dispatch import receiver
from django.db import transaction
from myapp.models import MyModel

@receiver(post_save, sender=MyModel)
def my_handler(sender, instance, **kwargs):
    print("Signal handler is running.")
    # Simulate a database modification
    with transaction.atomic():
        instance.name = "Changed in signal"
        instance.save()

# In thread app's models.py

from django.db import models
from django.db import transaction

class MyModel(models.Model):
    name = models.CharField(max_length=100)

    def save(self, *args, **kwargs):
        print("Save method is running.")
        super(MyModel, self).save(*args, **kwargs)


# Explanation

1. When you save an instance of MyModel, the save method prints "Save method is running."
2. The signal handler prints "Signal handler is running." and then modifies the instance in a transaction.
3. If you check the database after the save, you will see that the changes made by the signal handler are committed together with the original save if no exception occurs.

This shows that the signal handler operates within the same transaction as the save method by default, so changes are either all committed or all rolled back together.


# Topic: Custom Classes in Python

Description: You are tasked with creating a Rectangle class with the following requirements:

An instance of the Rectangle class requires length:int and width:int to be initialized.
We can iterate over an instance of the Rectangle class 
When an instance of the Rectangle class is iterated over, we first get its length in the format: {'length': <VALUE_OF_LENGTH>} followed by the width {width: <VALUE_OF_WIDTH>}

class Rectangle:
    def __init__(self, length: int, width: int):
        self.length = length
        self.width = width

    def __iter__(self):
        yield {'length': self.length}
        yield {'width': self.width}

rect = Rectangle(5, 3)

for attr in rect:
    print(attr)
