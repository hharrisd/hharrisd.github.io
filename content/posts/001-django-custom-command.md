---
title: "Working with Django Custom Command"
date: 2022-10-06T18:03:18-05:00
draft: false
cover:
    image: img/posts/001/bells.png
    alt: 'Django - Custom Commands'
    caption: 'Django - Custom Commands'
tags: ["python", "django", "custom", "command"]
categories: ["coding", "tech"]
ShowToc: true
---

## Introduction
Django built-in commands are a powerful tool for performing a lot of management actions like starting a project or an application, creating and migrating migrations, running the internal server, creating users, and many more.

Likewise, Django allows the creation of admin commands to implement custom functionalities, which is a great resource to execute tasks needed in certain moments. Those tasks could be for instance:

- Generating / Restoring backups
- Cleaning up databases
- Downloading reports
- Queuing messages 

Creating a custom command is pretty simple. To demonstrate it let’s assume this scenario: 

There is an application that stores information from Unsplash photografies. This application needs to request information from [Lorem Picsum, the Unsplash public API](https://picsum.photos) and store that information in a table, considering two ways to do it:
1. Just load the new data to the table.
2.  Delete the previous data in the table before loading the new one.

The model to store the data is:

```python
class Photo(models.Model):
    id = models.UUIDField(primary_key=True, default=uuid.uuid4, editable=False)
    author = models.CharField(max_length=200)
    width = models.IntegerField()
    height = models.IntegerField()
    url = models.URLField(max_length=200)
    download_url = models.URLField(max_length=200)
```

## Where to create custom commands?
Django will look for commands to register in each installed application in those directories: **management/commands**.

![Custom command directory](/img/posts/001/custom-command-directory.png)

The filename will be the name to call the command with the **manage.py** file. For this example, the command will be called ```python manage.py load-photo```.

The *load-photo.py* file contains a ```Command``` class that inherits from ```BaseCommand```. The three principal elements to implement the command are:
- **help** - Attribute to describe the command as a help text.
- **add_arguments()** - Method to parse the given arguments.
- **handle()** - Is the main method of the command and must be implemented.

```python
# General example
from django.core.management.base import BaseCommand, CommandError


class Command(BaseCommand):
    help = “Description”
    def add_arguments(self, parser):
	    pass
    def handle(self, *args, **options):
	    pass
```

## Parsing arguments

This method uses the library [argparse](https://docs.python.org/3/library/argparse.html) to parse the arguments and passes them to the handle method. In this example two arguments are defined, one positional (required) to indicate the number of images to load. The other is optional, to indicate if there will be a deletion of data before the load.

```python
def add_arguments(self, parser):
    # This is a positional argument
    parser.add_argument('quantity', type=int, help='Number of images to load from the API. 30 by default. Max 100.')

    # This is an optional argument
    parser.add_argument('--delete', type=bool, action=argparse.BooleanOptionalAction,
                        help='Deletes existing images before the load.')
```

## Handle method: The logic

The methods **validate_quantity**, **load_random_photos** and **deletes_existing_photos** are implemented to achieve the functionality required for this example.

```python
def validate_quantity(self, quantity: int) -> bool | CommandError:
    """Verifies that the number of image to fetch is between 1 and 100"""
    if quantity <= 0:
        raise CommandError('The numbers of image to load must be grater than 0.')

    if quantity > 100:
        raise CommandError('The numbers of image to load is up to 100.')

    return True

def load_random_photos(self, number_of_img: int) -> None:
    """Request items from Unsplash API and load them in the table."""
    response = requests.get(settings.UNSPLASH_IMAGES + str(number_of_img))
    if response.status_code != 200:
        raise CommandError(f'Error in request. Status: {response.status_code}.')

    saved_counter = 0
    for image in response.json():
        photo = Photo(author=image['author'], width=image['width'], height=image['height'], url=image['url'],
                        download_url=image['download_url'])
        photo.save()
        saved_counter += 1

    self.stdout.write(self.style.SUCCESS(f'{saved_counter} photos loaded successfully.'))

def deletes_existing_photos(self) -> None:
    """Deletes all records from the Photo models"""
    try:
        affected_records, _ = Photo.objects.all().delete()
        self.stdout.write(self.style.SUCCESS(f'Deleted {affected_records} records.'))
    except Exception:
        raise CommandError('Exception during deletion.')
```

The handle method has the responsibility to implement the logic that calls these aforementioned methods.

```python
def handle(self, *args, **options):
    """Executes the logic of the command."""
    if options['delete']:
        self.deletes_existing_photos()

    if self.validate_quantity(options['quantity']):
        self.load_random_photos(options['quantity'])
```

## Providing console output

Django recommends the use of [StringIO](https://docs.python.org/3/library/io.html#io.StringIO) to provide output to the console: It facilitates command testing and with the use of syntax coloring is easy to return an intuitive message for success or error among others.

## Testing the Command

The command can be tested using the method ```call_command``` from the ```django.core.management``` class.

```python
from io import StringIO
from django.core.management import call_command, CommandError
from django.test import TestCase


def execute_command(quantity: int, delete: bool = False) -> StringIO:
    out = StringIO()
    call_command('load-photo', quantity, delete=delete, stdout=out)
    return out


class ManagePhotoDataTest(TestCase):

    def test_command_load_output(self):
        out = execute_command(25)
        self.assertIn('25 photos loaded successfully.', out.getvalue())

    def test_command_delete_and_load_output(self):
        execute_command(25)
        out = execute_command(10, delete=True)
        self.assertIn('Deleted 25 records.', out.getvalue())
        self.assertIn('10 photos loaded successfully.', out.getvalue())

    def test_command_validates_negative_numbers_output(self):
        with self.assertRaises(CommandError):
            out = execute_command(-5)
            self.assertIn('The numbers of image to load must be grater than 0.', out.getvalue())

    def test_command_validates_up_to_100_output(self):
        with self.assertRaises(CommandError):
            out = execute_command(550)
            self.assertIn('The numbers of image to load is up to 100.', out.getvalue())

```

## Executing the command

![Terminal Example](/img/posts/001/terminal-example.png)

I hope this post helps you to understand this feature. For more detail about Custom Commands, I recommend reading Django’s [official documentation](https://docs.djangoproject.com/en/4.1/howto/custom-management-commands/).