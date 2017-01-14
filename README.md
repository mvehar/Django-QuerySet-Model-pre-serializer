# Django-QuerySet-serialize

Basic serializer of Django models/QuerySets to object/array, which can be used for JSON conversion.

Used for Django 1.10.

Use at yor own risk. Code is provided "as is".

```python
from django.db.models import Model 
from django.db.models.fields import Field 
from django.db.models.query import QuerySet 
from django.db.models.fields.related import RelatedField, ForeignKey

def qsSerialize(model, depth=1):
    if depth == 0: return None;

    if(isinstance(model, QuerySet)):
        if model.count() == 1:
            return qsSerialize(model.select_related().first(), depth)

        return qsSerialize(list(model.select_related().all()), depth)
        
    if(isinstance(model, list)):
        if depth == 1:
            return None

        return [qsSerialize(m, depth-1) for m in model]

    #check if Model - isinstance can fail if different packages
    if hasattr(model, 'refresh_from_db'):

        fields = model._meta.get_fields(include_hidden=True)
        obj = {}


        for field in fields:
            if isinstance(field, Field):
                if not field.related_model:
                    obj[field.get_attname()] = getattr(model, field.get_attname())

                if field.related_model:
                    if isinstance(field,ForeignKey):
                        
                        if field.rel.to:
                            fkey = {}
                            fkey['name'] = field.rel.get_related_field().name
                            fkey['value'] =  getattr(model, field.get_attname())

                            related = field.rel.to._default_manager.get(**{fkey['name']: fkey['value']})
                            obj[field.name] = qsSerialize(related, depth-1)
                        else:
                            obj[field.name] = None

                    else:
                        obj[field.get_attname()] = qsSerialize(getattr(model, field.get_attname()).all(), depth-1)

        return obj

    return model

```
