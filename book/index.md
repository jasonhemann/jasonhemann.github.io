---
title: Book an Appointment
layout: single
permalink: /book/
---

Choose the appointment type that matches what you need. Each type has
its own availability and questions. Some types may share time: once a
time is booked through one type, Google Calendar removes that time from
the others as well.

Only appointment types that are currently open appear here. A direct
course or assignment link remains useful when its booking window is
upcoming or closed.

{% assign bookings = site.data.bookings %}
{% assign open_count = 0 %}
{% for group in bookings.groups %}
  {% assign group_open = false %}
  {% for service_id in group.services %}
    {% assign service = bookings.services[service_id] %}
    {% if service.status == "open" %}
      {% assign group_open = true %}
    {% endif %}
  {% endfor %}
  {% if group_open %}
## {{ group.label }}

    {% for service_id in group.services %}
      {% assign service = bookings.services[service_id] %}
      {% if service.status == "open" %}
        {% assign open_count = open_count | plus: 1 %}
- [{{ service.title }}]({{ service.path | relative_url }}) — {{ service.summary }}
      {% endif %}
    {% endfor %}
  {% endif %}
{% endfor %}

{% if open_count == 0 %}
No self-service appointment types are open right now. Please use the
durable link supplied by your course or assignment to see when its
booking window will open.
{% endif %}
