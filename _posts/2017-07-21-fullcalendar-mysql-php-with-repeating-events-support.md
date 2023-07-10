---
layout: post
title: Fullcalendar integration using JQUERY, MySQL & PHP with repeating events support
date:   2017-08-21 0600 +0200
tags: [general, javascript,php,mysql,fullcalendar, calendar]
category: javascript
---
Recently, I was working on a project that required populating events from a MySQL database to a Fullcalendar using PHP and JQuery.
The repeating events were set to repeat on a given frequency such as every <i>(n)</i> days/weeks/months/years. The client also needed a calendar that allowed completing single task in a repeating range. The following is my raw hack at tackling this problem. Any more efficient suggestions are welcome.

This solution is implemented using Laravel framework.

`laravel new events-app`

*Migration files*
`php artisan make:migration CreateEventsTable --create=events`

- Edit the migration file

```
Schema::create('events', function (Blueprint $table) {
            $table->increments('id');
            $table->string('title',100);
            $table->integer('repeats')->nullable()->default(0);
            $table->integer('freq')->nullable();
            $table->string('freq_term')->nullable();
            $table->date('start_date')->nullable();
            $table->date('end_date')->nullable();
            $table->string('status')->default('open'); //open, in-progress, closed, cancelled
            $table->text('notes')->nullable();
            $table->timestamps();

        });
```


Fullcalendar has a `dow` function that allows to repeat on specific days of the week. The problem is when the event  is set on a date range, when you use `dow`, then the event repeats forever in both past and future. Fullcalendar's end date is now exclusive and therefore I had to add 1 to the `for` loop the loops through the date range.