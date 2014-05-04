---
layout: post
title: "Playing with RDF"
description: "I want a nice ORM type thing for RDF."
category:
tags: ['rdf']
---
I’ve been playing with the master branch of tracker and i’m loving it – it looks like its finally reached the stage where I won’t just turn it straight off after a fresh install.

It now brings GNOME an RDF store with a SPARQL interface. Powerful joo-joo, but kinda scary if you haven’t seen it before. Most conversations about it lead to words like graphs, triples, ontologies… My eyes start to gloss over.. I need to learn by doing. So i’ve been playing with writing some python wrappers to hide tracker and just provide a familiar pythonic interface.

Any object type that tracker knows about will be available in python via the wonders of introspection. All properties of the class are available, with docstrings explaining the intent of the property and its type. Obviously you can get, set and delete and do simple queries. And behind the scenes are SPARQL queries in all their glory. Theres a lot still to do, but enough done that I can synchronise my address book to Tracker with Conduit (see my tracker branch).

So far it looks something like this (but its subject to very rapid change):

    import tralchemy
    from tralchemy.nco import Contact

    # add a contact
    c = Contact.create()
    c.fullname = "John Carr"
    c.firstname = "John"
    c.nickname = "Jc2k"
    c.commit()

    # find all the people called John
    for c in Contact.get(firstname="John"):
        print c.uri, c.fullname

    # subscribe to any contact changes
    def callback(subjects):
        print subjects
    Contact.notifications.connect("SubjectsAdded", callback)

    # Will probably be just:
    Contact.added.connect(callback)

While get() is a nice way to do simple queries, what if you wanted to do something a little more complicated. It always feels messy when you have SQL or SPARQL nested in other code. Existing SQL ORM tools are a great place to start at avoiding this, but i quite like the LINQ style generator-to-SPARQL. Something like:

    q = Query(Contact.firstname for Contact in Store if Contact.nickname == 'Jc2k')

or

    q = Query(c.firstname for c in Store if c is Contact and c.nickname == 'Jc2k')
