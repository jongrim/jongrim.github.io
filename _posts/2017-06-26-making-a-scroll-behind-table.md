---
layout: post
title: Making a Scroll Behind Table
tags: JavaScript Design Layout
---
I recently finished some work on my first group project at the Ga Tech Coding Camp and wanted to do a quick walkthrough on how I created a cool layout feature. Before I begin explaining the code, let me set up the scenario I was facing.

Our app, a music event search site, was currently receiving a list of events from Ticketmaster as the result of a search. We were displaying those results in two ways: a map with markers, and a table layout that listed each event and details. The table was below the map in the layout, and in the previous setup, when you scrolled to see the results lower in the table, the entire page would scroll. It occured to me, that this might not be the best user experience. It's pretty likely that a user could want to scroll the table results while still being able to see the map since the two give different information. One is location based, while the other is a pure list.

So I had my first two requirements for a fix: keep the map in view, and allow the table to scroll behind the map and off the page. There was only one more issue. The map is resizable using the jQuery UI plugin, so this meant that I couldn't count on the map taking up a fixed amount of space. So whatever the solution was, it had to also respond to resize events.

The basic steps I took to get this functioning were:
- Set the map content area to `position: fixed`, and `z-index: 1`
- Set the table content area to `position: relative`
- Create a function to compute the height of the map area, and then set the `top` property of the table area equal to this value. This function gets called once after the initial page load to set the table position, and again anytime the map is resized, as explained below.
- Set a listener for the 'resizestop' event on the resizable element. I found this by reading through the jQuery UI and was so happy to see it existed.

I haven't even mentioned that there is a fixed header on the site as well, but what's great about this solution is that it takes into account the space of the header already. Setting a fixed header with Bootstrap (which we were using) involves adding padding to the top of the body so everything accounts for the header space, so that value is built-in from the get go. You just have to figure out how much height the map is currently occupying and reset the table accordingly.

I ended up applying these concepts to other elements on the page as well. There is a footer that needs to be at the bottom of all the content, so I take the same approach: find the height of the map, find the height of the table, and that's where the top of the footer should be. Math for the win!

---
If interested, I've put together a simple [Codepen](https://codepen.io/jonjongrim/pen/EXbapj) that demonstrates this concept. Enjoy!
