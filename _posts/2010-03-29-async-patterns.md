---
layout: post
title: "Patterns for asynchronous javascript"
description: "Writing nice async javascript"
category: Coding 
tags: ['javascript', 'async']
---
Isn’t writing synchronous code nice?

    function do_stuff () {
      do_thing_one ();
      do_thing_two ();
      do_thing_three ();
    }

But synchronous is bad! Bad, bad, bad. So then came async. The simple pattern is to pass a callback to the function you are calling. It’s not as bad in JS because we just can do this:

    function do_stuff () {
      do_thing_one ( function (result) {
        do_thing_two ( function (result) {
          do_thing_three ( function (result) {
          });
        });
      });
    }

I’ve left out error handling. That really depends on the library.. But i imagine its messy. Now let’s see GIO style async.

    function do_stuff () {
      do_thing_one_async ( function (ar) {
        var result = do_thing_one_finish (ar);
        do_thing_two_async ( function (ar) {
          var result = do_thing_two_finish (ar);
          do_thing_three_async (function (ar) {
            var result = do_thing_three_finish (ar);
          });
        });
      });
    }

I like this a lot better than how i’d do it in python. But wouldn’t it be nice if you could write async code something like this?

    var do_stuff = async (function () {
      var result = yield do_thing_one ();
      yield do_thing_two ();
      yield do_thing_three ();
    });

Or even:

    var do_stuff = async (function () {
      var result = yield do_thing_one ();
      yield do_thing_two ();
      try{
        yield do_thing_three ();
      } catch (e) {
        print("Exception handled");
      }
    });

You can in [python](http://blogs.gnome.org/jamesh/2009/01/06/twisted-gio/). You can in [vala](http://blogs.gnome.org/juergbi/2009/09/18/closures-and-asynchronous-methods-in-vala/). And for JS? Well, I was going to say [now you can](http://github.com/Jc2k/whorl). But while I was looking for a good Vala link, I noticed Alex already did something like this [over a year ago](http://blogs.gnome.org/alexl/2008/09/16/async-io-made-easy-using-javascript/). D’oh.

What would be really nice is if the async wrappers could be generated automatically by GI. I had a first stab at this by simply parsing the GIR xml with E4X and providing an alternative import mechanism (thanks for the suggestion jdahlin). However to get full coverage i’d have to consider interfaces and inspect every object that implements an interface as it lands in JavaScript land to ensure it is wrapped. Ew.
