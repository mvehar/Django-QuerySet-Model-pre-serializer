# Django-QuerySet-serialize

Basic serializer of Django models/QuerySets to object, which can be used for JSON conversion.

Use at yor own risk. Code is provided "as is".

```python
from django.forms.models import model_to_dict
from django.db.models import Model 
from django.db.models.fields import Field 
from django.db.models.query import QuerySet 
from django.db.models.fields.related import RelatedField, ForeignObject

def qsSerialize(model, depth=1):
    if depth == 0: return None;

    if(isinstance(model, QuerySet)):
        if model.count() == 1:
            return qsSerialize(model.first(), depth)

        return qsSerialize(list(model.all()), depth)

    if(isinstance(model, list)):
        if depth == 1:
            return None

        return [qsSerialize(m, depth-1) for m in model]

    if isinstance(model, Model):
        fields = model._meta.get_fields()
        obj = {}

        for field in fields:
            if isinstance(field, Field):
                if  not isinstance(field, RelatedField):
                    obj[field.get_attname()] = getattr(model, field.get_attname())

                if isinstance(field, RelatedField):
                    if isinstance(field, ForeignObject):
                        obj[field.get_attname()] = qsSerialize(getattr(model, field.get_attname()), depth-1)

                    else:
                        obj[field.get_attname()] = qsSerialize(getattr(model, field.get_attname()).all(), depth-1)

        return obj

    #return original object
    return model
```
