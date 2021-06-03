---
title: "Formulario de búsqueda en Flask"
date: 2018-03-25 21:22:48 -0500
excerpt_separator: <!-- more -->
comments: true
categories:
- Python
---

En [la entrada anterior](http://blog.vero4ka.info/blog/2018/03/25/como-hacer-listview-en-flask/) hicimos un ``ListView`` para mostrar un listado de objetos que pertenecen a un modelo específico.

Ahora miramos cómo agregar un formulario de búsqueda, para que nuestro usuario pueda filtrar los objetos por sus campos:

<!-- more -->

![](/images/tech/passengers-search-form.png)

Extendemos la clase ``ListView``, agregando el atributo que se refiere al formulario de búsqueda (``search_form``) y un método que genera una petición de SQLAlchemy a partir de los campos del formulario:

`base.py`:

```python
class ListView(BaseView):
    methods = ['GET', 'POST']
    template_name = None
    context_object_name = None
    paginate_by = None
    search_form = None
    model = None

    def search(self, search_form, object_list):
        # Idea from http://stackoverflow.com/a/14876320/914332
        for field in search_form:
            try:
                key, op = field.name.split('__', 2)
            except ValueError:
                # raise Exception('Invalid filter')
                # it also may be csrf_token or submit field
                continue
            column = getattr(self.model, key, None)
            if not column:
                raise Exception('Invalid filter column: %s' % column)
            if field.data:
                if op == 'in':
                    filt = column.in_(field.data.split(','))
                else:
                    try:
                        attr = filter(
                            lambda e: hasattr(column, e % op),
                            ['%s', '%s_', '__%s__']
                        )[0] % op
                    except IndexError:
                        raise Exception('Invalid filter operator: %s' % op)

                    data = field.data

                    if attr in ('ilike', 'like'):
                        data = '%%%s%%' % field.data

                    filt = getattr(column, attr)(data)
                object_list = object_list.filter(filt)
        return object_list

    def get_context(self):
        context = {}
        context_object_name = self.context_object_name or 'object_list'
        object_list = self.get_objects()

        # Search form
        if self.search_form:
            search_form = self.search_form(request.form)
            context['search_form'] = search_form
            if request.method == 'POST' and search_form.validate():
                object_list = self.search(search_form, object_list)
        # Pagination
        # ... (see previous post)
        return context
```

La versión completa de ``ListView`` junto con la vista de ejemplo se puede encontrar aquí: [ListView en Flask](http://blog.vero4ka.info/blog/2018/03/25/como-hacer-listview-en-flask/).

El el siguiente paso agregamos un formulario de búsqueda a nuesra vista:

`views.py`:

```python
class PassengersListView(ListView):
    template_name = 'passengers/list.html'
    context_object_name = 'passengers'
    search_form = PassengersSearchForm
    model = Passenger
```

El código del formulario es siguiente:

`forms.py`:

```python
from flask_wtf import Form
import wtforms

class SearchForm(Form):
    submit = wtforms.SubmitField('Filtrar')

    def __init__(self, *args, **kwargs):
        super(SearchForm, self).__init__(*args, **kwargs)

        for field in self:
            field.validators.append(wtforms.validators.Optional())


class PassengersSearchForm(SearchForm):
    id__eq = wtforms.StringField()
    name__ilike = wtforms.StringField()
    email__ilike = wtforms.StringField()
```

Para que la búsqueda funcione, los campos del formulario deben coincidir con los campos
del modelo ``Passenger`` ([ver el modelo](/blog/2016/03/18/listview-en-flask)),
acompañados con operadores de comparación de SQLAlchemy:

```
    eq para ==
    le para <=
    lt para <
    ne para !=
    in
    like
    ilike
```

El listado de operadores se puede encontrar aquí:
[sqlalchemy.orm.properties.ColumnProperty.Comparator](http://docs.sqlalchemy.org/en/rel_0_8/orm/internals.html#sqlalchemy.orm.properties.ColumnProperty.Comparator).

Luego agregamos el formulario a la plantilla, asociada a la vista:

`list.html`:

```html
{% extends "layouts/base.html" %}
{% import "bootstrap/wtf.html" as wtf %}

{% block content %}
<table class="table">
  <tbody>
    <form class="form form-horizontal" method="post" role="form" action="{{ request.path }}">
      {{ search_form.csrf_token }}
      <tr>
        <td>{{ wtf.form_field(search_form.id__eq, form_type='inline') }}</td>
        <td>{{ wtf.form_field(search_form.name__ilike, form_type='inline') }}</td>
        <td>{{ wtf.form_field(search_form.email__ilike, form_type='inline') }}</td>
        <td></td>
        <td>{{ wtf.form_field(search_form.submit, form_type='inline') }}</td>
      </tr>
    </form>
  {% for passenger in passengers %}
    <tr>
      <td><a href="{{ url_for('passengers.passenger_detail', pk=passenger.id) }}">{{ passenger.id }}</a></td>
      <td>{{ passenger.name }}</td>
      <td>{{ passenger.email }}</td>
      <td>{{ passenger.phone }}</td>
    </tr>
  {% endfor %}
  </tbody>
</table>
{% endblock %}
```html

En este ejemplo usamos [Flask-Bootstrap](https://pythonhosted.org/Flask-Bootstrap/forms.html) para renderizar los formularios con las clases de [Bootstrap](http://getbootstrap.com/).
