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

# Naming conventions

The rules follow the Slippy map tilenames, so:

1. Tiles are 256×256 pixel PNG files
1. Each zoom level is a directory, each column is a subdirectory, and each tile in that column is a file
1. Filename(url) format is /zoom/x/y.png

The slippy map expects tiles to be served up at URLs following this scheme, so all tile server URLs look pretty similar.

# Internal representation of a tile

There are some restriction on PNG formats, to store all the required informations it is mandatory to use PNG24 or PNG32.

A 256x256 pixel PNG is divided into a 64x64 grid, with cells of 4 pixels.

| An UUID is                                                     |  128 bits |
| We know that all features use UUID version 4                   |  122 bits |
| We need to store 3 UUIDs v4 in each cell                       |  366 bits |
| We use first bit of red channel to store utility information   |  382 bits |
| Store the number of features, maximum is 3 so we need 2 bits   |  384 bits |
| In a 4x4 square of PNG (looseless) we have three 8bit channels |  384 bits |
| Unused bits                                                    |  000 bits |
{: .table-bordered}

# How to handle Identification Tiles client-side

At the time of writing the technology is not include in any of the existing and wide-user client libraries.

The following are some examples of working code in various programming languages.

## javascript

{% highlight javascript %}


/*********************************************************************************************
 * From points to 3 uuids
 * @param points 7x7 point extracted from an idcanvas
 *********************************************************************************************/
_utils.fromPoints2uuids = function(points){
    var uuids = null;
    try{
        var k = 0;
        var found = false;
        var foundI = 0;
        var foundJ = 0;
        for (var i=0; i<7&&!found;i++){
            for (var j=0;j<7&&!found;j++){
                // start index
                k = (i*7+j)*4;
                var redChannel = _utils.pad("00000000",points.data[k].toString(2),true)
                if (redChannel[0] === '1'){
                    foundI = i;
                    foundJ = j;
                    found = true;
                    //alert("found i: " + foundI + " j: "+ foundJ);
                }
            }
        }
        if (found){
            var bitstring = ""
            for (var i = foundI; i<foundI+4;i++){
                for (var j = foundJ; j<foundJ+4;j++){
                    k = (i*7+j)*4;
                    //var r = utils.pad("000",points.data[k].toString(10),true);
                    //var g = utils.pad("000",points.data[k+1].toString(10),true);
                    //var b = utils.pad("000",points.data[k+2].toString(10),true);
                    //var message = "k:"+ k + " " + r + "-" + g + "-" + b + "\n";
                    //alert(message);
                    bitstring += _utils.pad("00000000",points.data[k].toString(2),true)
                    bitstring += _utils.pad("00000000",points.data[k+1].toString(2),true)
                    bitstring += _utils.pad("00000000",points.data[k+2].toString(2),true)
                }
            }
            // from bitstring to interesting bits
            var counterBin = bitstring.substring(1,3);
            var counter = parseInt(counterBin,2);

            var i = 3;
            var j = 8;
            var k = 0;
            var interestingBits = bitstring.substring(i,j);
            for (var k=1; k<48;k++){
                i=j;
                j+=8;
                if (k%3 === 0){
                    interestingBits += bitstring.substring(i+1,j);
                }else{
                    interestingBits += bitstring.substring(i,j);
                }
            }
            //message += "Found " + counter + " features, len:" + interestingBits.length; ;
            if (counter > 0){
                //message += "IDENTIFIED " + counter + " FEATURE" + (counter===1 ? '':'S') + "\n";
                // from interesting bits to binRepresentation
                var binRepresentation = "";
                binRepresentation += interestingBits.substring(0,48);
                binRepresentation += "0100";
                binRepresentation += interestingBits.substring(48,60);
                binRepresentation += "10";  // this is constant part
                binRepresentation += interestingBits.substring(60,122);

                binRepresentation += interestingBits.substring(122+0,122+48);
                binRepresentation += "0100";
                binRepresentation += interestingBits.substring(122+48,122+60);
                binRepresentation += "10";
                binRepresentation += interestingBits.substring(122+60,122+122);

                binRepresentation += interestingBits.substring(122+122+0,122+122+48);
                binRepresentation += "0100";
                binRepresentation += interestingBits.substring(122+122+48,122+122+60);
                binRepresentation += "10";
                binRepresentation += interestingBits.substring(122+122+60,122+122+122);

                // from binRepresentation to 3 UUID version 4
                var uuid1bin = binRepresentation.substring(0,128);
                var uuid2bin = binRepresentation.substring(128,256);
                var uuid3bin = binRepresentation.substring(256,384);

                var uuid1 = _utils.bin2hex(uuid1bin);
                var uuid2 = _utils.bin2hex(uuid2bin);
                var uuid3 = _utils.bin2hex(uuid3bin);

                uuid1 = uuid1.substring(0,8) + "-" + uuid1.substring(8,12) + "-" + uuid1.substring(12,16) + "-" + uuid1.substring(16,20) + "-" + uuid1.substring(20,32);
                uuid2 = uuid2.substring(0,8) + "-" + uuid2.substring(8,12) + "-" + uuid2.substring(12,16) + "-" + uuid2.substring(16,20) + "-" + uuid2.substring(20,32);
                uuid3 = uuid3.substring(0,8) + "-" + uuid3.substring(8,12) + "-" + uuid3.substring(12,16) + "-" + uuid3.substring(16,20) + "-" + uuid3.substring(20,32);

                if (counter === 1){
                    uuids = [uuid1];
                }else if (counter === 2){
                    uuids = [uuid1,uuid2];
                }else if (counter === 3){
                    uuids = [uuid1,uuid2,uuid3];
                }
            }else{
                // counter = 0, simpley return uuids which is null
            }
        } else {
            // if not found, simply return uuids, which is null
        }
    } catch(e) {
            // to nothing, simply return uuids, which is null
    }
    return uuids;
 }



{% endhighlight %}


