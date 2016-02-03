---
layout: post
title:  Introducing OSM for the dyslexic
date:   2016-02-01 00:00:00
summary : First version of #OSM for #dyslexic
categories: maps
tags: OSM maps dyslexic
featured: true
---

A GIS application to make OpenStreetMap more accessible by dyslexics, mitigating common reading errors

# What is the appplication going to do?
The primary objective of the application is to mitigate typical reading errors dyslexic people do when reading and browsing OpenStreetMap.

# Which data sets will it use?

The proposed appplication use at least the following dataset from OpenStreetMap:

## Boundaries

Boundaries mark the borders of areas, mostly political, but also of other administrative areas. The application will use administrative boundaries as basemap (boundary = administrative , admin_level from 1 to 10)

## Highways 

Highways in OpenStreetMap are roads, routes, ways , or thoroughfares on land which connects one location to another and has been paved or otherwise improved to allow travel by some conveyance, including motorised vehicles, cyclists, pedestrians, horse riders, and others (but not trains – see Railways for further details). The application will use highways as basemap.

## Buildings

Houses, factories and ruined buildings. The app will use buildings as basemap.

## Aeroways

Describes the fixed physical infrastructure associated with the air travel, including airports, runways, helipads, and terminal buildings. The app will use aeroways as basemap.

## Amenities

They represent useful and important facilities for visitors and residents, an assortment of community facilities including toilets, telephones, banks, pharmacies and schools. The app will use amenities as identificable items.

## Addresses

Addresses for the building or facility. The app will use addresses as idenificable items.

## Shops

Places selling retail products or services. The apps will use shops as identificable items.


# Development status
Check it on the project official web site:

<http://www.osm4dys.org>


# Advantages of the app compared to existing ones

## OSM Direction tool for Visually Impaired
From <http://wiki.openstreetmap.org/wiki/OSM_Direction_tool_for_Visually_Impaired>

The application is a Keyboard accessible Directions tool using OpenSteetMap for Visually
Impaired people. It works on the principle such that,after user enters source and
destination as his/her query in text boxes, the route or walking/driving directions are
resulted as output not only on maps but also as detailed text explaining entire routeto be
easily readable.

## Look and Listen Map
From <http://wiki.openstreetmap.org/wiki/OSM_for_the_blind>

"Look and Listen Map" is an online map and routing service for blind, visually impaired
and sighted persons based on OSM data. This application will use a Braille display as
output device and/ora voice that reproduce the text on the screen.

## HaptoRender
From <http://wiki.openstreetmap.org/wiki/HaptoRender>

HaptoRender is planned to be a renderer that uses OSM data to create tactile maps for
blind and visually impaired persons.
HaptoRender will have several advantages compared with commercial tactile maps:
Phisical maps are cheaper, because there will not be any license cost for the map data.
If one part of a map is obsolete, the part can be reproduced.

## Commercial software for dysleciscs

### Read&Write 
From <http://www.texthelp.com/UK>

Read&Write is an easy-to-use flexible toolbar containing support features to make
reading, writing and research easier for the dyslexic. The software has been designed to
address some of the issues that people with dyslexia face daily, namely reading
difficulties, writing difficulties, problems with spelling and general literacy support, but
does not work on images.

### BrowseAloud
From <http://www.browsealoud.com/uk/>

BrowseAloud adds speech, reading and translation support to a website facilitating
access and participation for those people with print disabilities, dyslexia, low literacy and
mild visual impairments.

# Our contribution
Despite many applications were developed specifically for dyslexics, none of them is a
GIS application.

The concept of styling maps for dislexics is an innovative point of view for increasing
acessibility of the GIS (open) data. None of the free basemaps are accesible-ready for
dislexics. They are:

* feature-rich,
* color-rich,
* rich of text following complex paths.

Dyslexics feel better when:

* text is capitalized and always horizontal,
* same colors represent same things,
* few information are on the screen.

None of the existing GIS application has the capability to:

* Describe and tell (audio) where the user is
* Describe and tell the path made by the user around the map
* Describe and tell what the user is currently watching
* Describe and tell the user informations about every single feature on a map