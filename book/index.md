---
title: Book an Appointment
layout: single
permalink: /book/
---

Choose the appointment type that matches what you need. Each type has
its own availability and questions. Some types may share time: once a
time is booked through one type, Google Calendar removes that time from
the others as well.

All appointment types remain listed here continuously. The linked Google
Calendar page is authoritative for the dates and times currently offered.
If it shows no times, no appointments of that type are currently configured.

{% assign bookings = site.data.bookings %}
{% for group in bookings.groups %}
## {{ group.label }}

  {% for service_id in group.services %}
    {% assign service = bookings.services[service_id] %}
- [{{ service.title }}]({{ service.path | relative_url }}) — {{ service.summary }}
  {% endfor %}
{% endfor %}
