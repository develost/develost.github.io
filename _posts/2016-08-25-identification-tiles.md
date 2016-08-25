---
layout: post
title:  Host a GIS service without a GIS server
date:   2016-08-25 00:00:00
summary: The technology of Identification tiles revealed
categories: maps
tags: maps tile identification
featured: true
---

Have to host a map on a website? With identification tiles technology you can host your map
on a standard and static website:

 * high speed
 * support for identification
 * no need of any GIS Server

I have to say that this technology is quite new, in fact as far as I know it has been used only into two projects (both of them done by me).

Still interested into reading? Go ahead...

# Current status

The idea of moving the process of identification from server-side to client-side is something was something that I thought a lot about.

Now I can say this idea to be operative because has been implemented in multiple projects, for example:

 * OSM for the dyslexic (javascript / postgis, finished)
 * City for the dyslexic (native android app / postgis, in development)

# How it works

Itentification tiles are a structured directory of PNGs following the slippy map specification. Each PNG contains many feature IDs, where the features actually are so for the end-user application will be
computionally fast to get those informations simply obtaining the raw bits of the image.

My implementation divides each PNG 256x256 pixel in a 64x64 grid, with 4x4 pixel in each cell.
Using 8 bit as color dept you can store up to 3 UUIDs version 4 in each cell. Look at the following computation:

| An UUID is                                                     |  128 bits |
| We know that all features use UUID version 4                   |  122 bits |
| We need to store 3 UUIDs v4 in each cell                       |  366 bits |
| We use first bit of red channel to store utility information   |  382 bits |
| Store the number of features, maximum is 3 so we need 2 bits   |  384 bits |
| In a 4x4 square of PNG (looseless) we have three 8bit channels |  384 bits |
| Unused bits                                                    |  000 bits |
{: .table-bordered}

The computation shows that it is possible to store up to 3 UUIDs version four in a cell.

# An Identification tile

This is the identification tile 0/0/0.png the whole world at zoom level 0.

[!Identification tile of the world]({{ site.url }}/posts-images{{page.url}}/world.png)

This is a particular of the above image make you capable to see how identification information is stored into the tile.

[!Particular of an identification tile]({{ site.url }}/posts-images{{page.url}}.png)

# Something working

This technology has been implemented for the first time for the project "OSM for the dyslexic"
presented at 10th GEO European Projects Workshop 2016

 * [The map implementing Identification tiles](http://www.osm4dys.org/viewer/)
 * [GitHub pages of the project](https://github.com/osm-for-the-dyslexic)
 * [MyGeoss project](http://digitalearthlab.jrc.ec.europa.eu/mygeoss/results2.cfm)

# What you can do next

It depends on you of course... If you are interested into this technology do not hesitate to write me, for sure can be used in your website or android application.
