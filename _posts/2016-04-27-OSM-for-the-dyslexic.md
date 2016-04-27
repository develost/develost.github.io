---
layout: post
title:  OSM for the dyslexic first public release
date:   2016-04-27 00:00:00
summary : osm4dys Android app on play store
categories: maps
tags: OSM maps dyslexic
featured: true
---

A GIS application to make OpenStreetMap more accessible by dyslexics, mitigating common reading errors

The development of the application is at the end, and the app is on the play store.
[Check it out](https://play.google.com/store/apps/details?id=org.osm4dys.app)

# Use the application effectively

## Main idea of the application

![Main idea]({{ site.url }}/posts-images{{page.url}}/gui.png)


## Pan, zoom, identify, interact

We identified the behaviour of the application for each gesture.

| Gesture                                                    | Action performed                                                     |
| ![Pan]({{ site.url }}/posts-images{{page.url}}/pan_01.png) | Move a finger on the map - it generates a pan                        |
| ![Pinch]({{ site.url }}/posts-images{{page.url}}/zoom_01.png) | Pinch on the map - it generates a zoom out by a level                |
| ![Stretch]({{ site.url }}/posts-images{{page.url}}/zoom_02.png) | Stretch on the map - it generates a zoom in by a level               |
| ![Long press]({{ site.url }}/posts-images{{page.url}}/long_press_01.png) | Long press on the map - it generates an identify location command    |
| ![Long press]({{ site.url }}/posts-images{{page.url}}/long_press_01.png) | Long press on functionality - activates that functionality           |
{: .table-bordered}

## Specific functionalities

### Location history

![Location history]({{ site.url }}/posts-images{{page.url}}/location-history.png)

The location history functionality allows the user to recognize the path followed to reach the location he is currently looking at. When the user presses (long press) one of the entry the user is guided to that location.
A message on the notification area informs the user about the behaviour.

### Overlay mode

![Overlay mode]({{ site.url }}/posts-images{{page.url}}/overlay.png)

The ovelay mode allows the user to recognize what is around the location he is watching. Once recognized he can restore the previous visualization simply pressing the overlay button.

This funtionality can be used to navigate quickly from a place to another.

### Text2speech capability

![Text2speech capability]({{ site.url }}/posts-images{{page.url}}/text2speech.png)

The text2speech capabilty relyies on the android natice capability to syntetize speech from text. The use has to install the desired language on its device before using this functionality.

We think this is not a limitation because the text2speech capability is free of charge, and there are voices for many languages. The only operation the user has to do in do download the desired voice from google servers.

### Information about a location

![Information about a location]({{ site.url }}/posts-images{{page.url}}/info-location.png)

Once the user identified a location, a notification appear in the notivication area. After that he can know more about the location, pressing the informations button.

The maximum number of features present is three to avoid crowding. Features presented follows the rule “from generic to particular” and an information about the context is always returned (if present in original data) .

All the alphnumeric information are spaced and distinguised. A press on the text2speech capability makes the application start to tell all the information contained on the screen.

If text2speech is not configured, if the user presses the text2speec button the application will show the text to speech configuration page.

## The identification process in detail

| ![Identification]({{ site.url }}/posts-images{{page.url}}/id1.png)  | ![Identification]({{ site.url }}/posts-images{{page.url}}/id2.png) |
| 1. The user goes to the desired location   | 3. The user knows how many features hit |
| 2. The user presses a location on the map  | 4. The identification succededs: two features: Milan and a supermarket |
| ![Identification]({{ site.url }}/posts-images{{page.url}}/id3.png)  | ![Identification]({{ site.url }}/posts-images{{page.url}}/id4.png) |
| 5. The user preses the information button | 6. The user scrolls and reads if text2speech is enabled, a voice tells the content | 
