========================
DEP XXXX: Django model internalization XXXX
========================

:DEP: XXXX
:Author: Ashley Camba Garrido
:Type: Process
:Created: 2015-06-01
:Last-Modified: 2015-06-01


Abstract
========

This DEP describes the implementation of a model internalization backend API.

Motivation
==========

Internalization is a complex problem where there are several different but valid
approaches that depend on the specific constraints and problem at hand.

Currently there are several 3rd party apps that tackle the model internalization
with somewhat success. Currently I see two main problems:

 -  As a developer, once having decided upon a specific app for my model 
    translations in my project, I mostly(always?) have to patch/override/ 
    other 3rd party apps if I want to make them translatable
 -  As a 3rd party app developer, if I decide to make an app 'internacionalized'
    I might pick a certain method that is incompatible with what a project developer
    wants.
 -  Obviously not having a unified API makes developers have to learn a different 
    syntax for each project.



Implementation
==============

This feature could be implementing by having a TRANSLATIONS_BACKENDS system similar
to the django database settings or django storages that allow 3rd party developers 
create backends that provide hooks to be used where necesary, mainly on model creation,
and in the ORM.

Some ideas on how it could look like:

mixin?

    class ExampleTranslatable(Translatable, models.Model):
        age = models.IntegerField()
        title = models.TextField()

    class Meta:
        translatable_fields = ['title']

or more like the django admin register?

translations.register(ModelClass, TranslationBackend)


translation backend hooks:

class backend1.Backend():
    def translate_models(self, model):
        if model._meta.app_label == 'Foo':
            return True
        else:
            return False

        def on_model_loading(self, model):
            # Do magic!

    def save(self, model, save_args, save_kwargs):
    def delete(self, model, ...)
    def update(self, qs, ...)
    def fetch_translations(self, model):
    def prefetch_for_queryset(self, qs):
        return FetchFromFileIterator

orm:
directly as lookups?
   qs.filter(translations__en__title='Programmer').update(translations__en__title='Advanced programmer')

   ExampleTranslatable.objects.filter(translations__fi__title='Ohjelmoija')

or something more like language.activate() ?

In theory, not all backends implement work on the database. You could have backends that use a service
or a NoSQL database.

Problems:
========

    - Model setup (altering the fields dynamically, dynamic model creation)
    - Migrations (for third party apps a problem to be solved in Django)
    - ModelForms (translatable model form)
        def __init__(self, *args, **kwargs, from_lang='fi', edit_in='en')
    - Saving logic
    - Query filter intragtion?
    - Validation?
    - Fetching translations (in bulk with ModelIterator override)
    - Fetching translations when not loading from DB (MyModel(pk=1).translations should
                                                      fetch translations for the pk=1 obj)
