PUGXGeneratorBundle Documentation
=================================

This version of the bundle requires Symfony 2.3.  
It depends on Bootstrap 2.3.x and FontAwesome 3.2.1

For Symfony 2.2, please switch to 2.2 branch.

A small note on branches and tags: version numbers are not necessarily consistent
with Symfony ones. We know we should have started versioning by something like ``0.1`` or
``1.1``, but we cannot change that decision now, for compatibility issues.


## Installation

1. [Download PUGXGeneratorBundle](#1-download-pugxgeneratorbundle)
2. [Enable the Bundle](#2-enable-the-bundle)
3. [Usage](#3-usage)
4. [Layout](#4-layout)
5. [Pagination](#5-pagination)
6. [I18n](#6-i18n)
7. [Filters](#7-filters)
8. [Sorting](#8-sorting)

### 1. Download PUGXGeneratorBundle

**Using composer**

Add the following lines in your composer.json:

```
{
    "require-dev": {
        "pugx/generator-bundle": "2.3.x-dev"
    }
}

```
Of course, if you already have other dependencies in your "require-dev", adapt lines.
If your "require" section contains "sensio/generator-bundle", you can delete it (since
it is required by "pugx/generator-bundle".

Run composer to download the bundle:

``` bash
$ php composer.phar require pugx/generator-bundle:2.3.x-dev
```

Notice that if your composer.json requires "sensio/generator-bundle", you can delete it (since
it is required by "pugx/generator-bundle").

### 2. Enable the bundle

Enable the bundle in the kernel:

``` php
<?php
// app/AppKernel.php

public function registerBundles()
{
    // ...
    if (in_array($this->getEnvironment(), array('dev', 'test'))) {
        // ...
        $bundles[] = new PUGX\GeneratorBundle\PUGXGeneratorBundle();
    }
}
```

### 3. Usage

This bundle brings a new command, ``pugx:generate:crud``, that is similar to ``doctrine:generate:crud``.
You can get help, like any other Symfony command, just typing

``` bash
$ php app/console pugx:generate:crud --help
```

### 4. Layout

This bundle is ready to be used with [Bootstrap 2.3](https://github.com/twbs/bootstrap/tree/v2.3.2) and
with [Font Awesome 3](http://fortawesome.github.io/Font-Awesome/3.2.1/)

So, you can download Bootstrap (and, optionally, Font Awesome) and put it in your bundle.
Then, you can use a simple layout, like this one:

``` html+php
<!DOCTYPE html>
<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
        <title>{% block title '' %}</title>
        <link rel="shortcut icon" href="{{ asset('favicon.ico') }}" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        {% stylesheets
            '@AcmeDemoBundle/Resources/public/css/bootstrap.css'
            '@AcmeDemoBundle/Resources/public/css/font-awesome.css'
            '@AcmeDemoBundle/Resources/public/css/bootstrap-responsive.css'
        %}
        <link rel="stylesheet" href="{{ asset_url }}">
        {% endstylesheets %}
        {% block stylesheets %}
        {% endblock %}
    </head>
    <body>
        <div class="navbar navbar-fixed-top">
            <!-- put your nav bar here -->
        </div>
        <div class="container">
            {% block body %}
            {% endblock %}
        </div>
        <script src="http://code.jquery.com/jquery.min.js"></script>
        {% javascripts
            '@AcmeDemoBundle/Resources/public/js/bootstrap.js'
            '@AcmeDemoBundle/Resources/public/js/acme.js'
        %}
        <script type="text/javascript" src="{{ asset_url }}"></script>
        {% endjavascripts %}
    </body>
</html>
```

If you want the confirm delete functionality, you can add the following Javascript code,
based on jQuery, in one of you files (e.g. ``acme.js`` in layout above):

``` js
$(document).ready(function() {
    /* delete confirm */
    $('form#delete').submit(function(e) {
        var $form = $(this);
        var $hidden = $form.find('input[name="modal"]');
        if ($hidden.val() != 1) {
            e.preventDefault();
            $('#delete_confirm').modal('show');
            $('#delete_confirm').find('button.btn-danger').click(function() {
                $('#delete_confirm').modal('hide');
                $hidden.val(1);
                $form.submit();
            });
        }
    });
});
```

If you want more consistent boostrap forms, you can use a theme like this one:

```jinja
{% block form_row %}
{% spaceless %}
    <div class="control-group{% if errors|length > 0 %} error{% endif %}">
        {{ form_label(form) }}
        {{ form_widget(form) }}
        {{ form_errors(form) }}
    </div>
{% endspaceless %}
{% endblock form_row %}

{% block form_errors %}
{% spaceless %}
    {% if errors|length > 0 %}
        {% if compound %}
            <div class="alert alert-error">
                {% for error in errors %}
                    <div>{{
                        error.messagePluralization is null
                            ? error.messageTemplate|trans(error.messageParameters, 'validators')
                            : error.messageTemplate|transchoice(error.messagePluralization, error.messageParameters, 'validators')
                    }}</div>
                {% endfor %}
            </div>
        {% else %}
            {% for error in errors %}
                <span class="help-inline">{{
                    error.messagePluralization is null
                        ? error.messageTemplate|trans(error.messageParameters, 'validators')
                        : error.messageTemplate|transchoice(error.messagePluralization, error.messageParameters, 'validators')
                }}</span>
            {% endfor %}
        {% endif %}
    {% endif %}
{% endspaceless %}
{% endblock form_errors %}

{% block date_widget %}
{% spaceless %}
    {% if widget == 'single_text' %}
        {{ block('form_widget_simple') }}
    {% else %}
        <div {{ block('widget_container_attributes') }}>
            <div class="row">
                {{ date_pattern|replace({
                    '{{ year }}':  '<div class="col-xs-2">' ~ form_widget(form.year) ~ '</div>',
                    '{{ month }}': '<div class="col-xs-2">' ~ form_widget(form.month) ~ '</div>',
                    '{{ day }}':   '<div class="col-xs-1">' ~ form_widget(form.day) ~ '</div>',
                })|raw }}
            </div>
        </div>
    {% endif %}
{% endspaceless %}
{% endblock date_widget %}

{% block time_widget %}
{% spaceless %}
    {% if widget == 'single_text' %}
        {{ block('form_widget_simple') }}
    {% else %}
        {% set vars = widget == 'text' ? { 'attr': { 'size': 1 }} : {} %}
        <div {{ block('widget_container_attributes') }}>
            <div class="row">
                <div class="col-xs-1">{{ form_widget(form.hour, vars) }}</div>{% if with_minutes %} <div class="col-xs-1">{{ form_widget(form.minute, vars) }}</div>{% endif %}{% if with_seconds %} <div class="col-xs-1">{{ form_widget(form.second, vars) }}</div>{% endif %}
            </div>
        </div>
    {% endif %}
{% endspaceless %}
{% endblock time_widget %}

{% block widget_attributes %}
{% spaceless %}
    {% set classFound = false %}
    id="{{ id }}" name="{{ full_name }}"{% if read_only %} readonly="readonly"{% endif %}{% if disabled %} disabled="disabled"{% endif %}{% if required %} required="required"{% endif %}{% if max_length %} maxlength="{{ max_length }}"{% endif %}{% if pattern %} pattern="{{ pattern }}"{% endif %}
    {% for attrname, attrvalue in attr %}
        {% if attrname in ['placeholder', 'title'] %}{{ attrname }}="{{ attrvalue|trans({}, translation_domain) }}" {% else %}{{ attrname }}="{{ attrvalue }}" {% endif %}
        {% if attrname == 'class' %}
            {% set attrvalue = attrvalue ~ ' form-control' %}
        {% endif %}
    {% endfor %}
    {% if not classFound %}
        class="form-control"
    {% endif %}
{% endspaceless %}
{% endblock widget_attributes %}
```

If you put such theme file in ``src/Acme/DemoBundle/Resources/views/Form/theme.html.twig``,
you can use the ``--theme`` option of ``pugx:generate:crud`` command, like in this example:

``` bash
$ php app/console pugx:generate:crud \
    --entity=AcmeDemoBundle:Entity \
    --layout=AcmeDemoBundle::layout.html.twig \
    --theme=AcmeDemoBundle:Form:theme.html.twig \
    --with-write
```

### 5. Pagination

You likely want to use pagination in your modules.
If so, add [KnpPaginatorBundle](https://github.com/KnpLabs/KnpPaginatorBundle)
to your bundles and use ``--use-paginator`` flag in ``pugx:generate:crud`` command.

### 6. I18n

Generated templates support I18n. If you want to translate texts, you should enable
translation in your configuration:

```yaml
# app/config.yml
framework:
    # ...
    translator:      { fallback: "%locale%" }
```

Then you should create a translation file, in your preferred format.
Messages catalogue is named "admin". Here is an example of a working translation file
in YAML format, for Italian language:

```yaml
# src/Acme/DemoBundle/Resources/translations/admin.it.yml
"%entity% creation":             "Creazione %entity%"
"%entity% edit":                 "Modifica %entity%"
"%entity% list":                 "Elenco %entity%"
Actions:                         Azioni
Back to the list:                Torna alla lista
Confirm delete:                  Conferma eliminazione
Create:                          Crea
Create a new entry:              Crea nuovo
Delete:                          Elimina
"Do you want to proceed?":       "Procedere?"
Edit:                            Modifica
edit:                            modifica
Filter:                          Filtra
No:                              No
Reset filters:                   Azzera filtri
show:                            vedi
"Show/hide filters":             "Mostra/nascondi filtri"
this procedure is irreversible:  questa procedura è irreversibile
Yes:                             Sì
You are about to delete an item: Si sta per eliminare un elemento
```

### 7. Filters

If you want to use filters (like the ones in the old symfony 1 admin generator), add
[LexikFormFilterBundle](https://github.com/lexik/LexikFormFilterBundle) to your bundles.
Then, use the ``--with-filter`` flag in ``pugx:generate:crud`` command.

Since filters require some additional methods in generated controllers, moving them to
a generic ``Controller`` class (and extending it instead of Symfony default one)
could be a good idea.

Please notice that, for now, support for filters is experimental.
=======
To enable automatic opening/closing of filters, based on session, you can add following
code to your Javascript:

``` js
$(document).ready(function() {
    /* filter icon */
    $('button.filter').click(function() {
        var $icon = $(this).find('i');
        var target = $(this).attr('data-target');
        if ($icon.length) {
            var $div = $(target);
            if ($div.height() > 0) {
                $icon.attr('class', 'icon-angle-down')
            } else {
                $icon.attr('class', 'icon-angle-right')
            }
        }
    });
});
```

There is a known limitation for generation of relations in filter form class, so you
need to adapt field configuration by hand.

### 8. Sorting

You can add sorting in columns, by using ``--with-sort`` flag in ``pugx:generate:crud`` command.
If you do so, instead of simple labels, table headers will contain links to toggle sorting
ascending and descending.
