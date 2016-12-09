---
layout: post
title: Identification Tiles Specification 0.1 - released
date: 2016-10-04 00:00:00
summary: The official specification of Identification Tiles Technology version 0.1
categories: identification
tags: maps tile identification
featured: true
---

# Purpose
Identification tiles is a technique to embed IDs of features into tiles, so when the user interact with a tile, the interaction can be handled client-side to get the IDs of the underlying features.

# Applicability
The mandatory requisites for GIS data to be accessed trough Identification Tiles are:

1. have a not-null geometry
1. have an UUID version four as primary key or unique identifier

# Two identification processes, compared

| **Standard identification process (without identification tiles)**  | **Tiled identification process** |
| 0. the user is zooming/panning and stops on an area |  0. the user is zooming/panning and stops on an area |
| 1. the user selects which layer to identify, for example "buildings" | 1. the user clicks on the map |
| 2. the client software calculates coordinates which is an easy task | 2. the client software accesses to the identification tile on the clicked position  |
| 3. the client software asks the map server with layername and coordinates | 3. the client software accesses to the raw bytes of the identification tiles and get up to 3 UUIDs |
| 4. the map server queries the table "buildings" using the geometry trough a spatial index | 4. for each UUID asks (REST) a static web site for features informations |
| 5. spatial index if for Bounding box only so the database has to further process data to filter the result  | 5. the client present the information to the user |
| 6. the database passes informations to the mapserver   |  |
| 7. the mapserver passes information to the client  |  |
| 8. the client present the informations to the user  |  |
{: .table-bordered}



