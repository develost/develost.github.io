---
layout: page
title: Identification Tiles Specification
description: The official specification of Identification Tiles Technology
theme: green
---

version: 0.1

status: working on it

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


## Java/Android/Osmdroid

{% highlight java %}


import android.graphics.Bitmap;
import android.graphics.Color;

public class ThreeUUIDs{
    private int itemsNumber = 0;
    private String uuid1 = "";
    private String uuid2 = "";
    private String uuid3 = "";
    private static final int SIDE_LEN = 4;
    private static final String PADDING = "00000000";
    //private String test;

    public static String bin2hex(final String str){
        String retval = "";
        String temp4bits = "";
        for (int i=0;i<str.length();i+=4){
            temp4bits = str.substring(i,i+4);
            if (temp4bits.equalsIgnoreCase("0000")) {retval += "0";}
            else if (temp4bits.equalsIgnoreCase("0001")) {retval += "1";}
            else if (temp4bits.equalsIgnoreCase("0010")) {retval += "2";}
            else if (temp4bits.equalsIgnoreCase("0011")) {retval += "3";}
            else if (temp4bits.equalsIgnoreCase("0100")) {retval += "4";}
            else if (temp4bits.equalsIgnoreCase("0101")) {retval += "5";}
            else if (temp4bits.equalsIgnoreCase("0110")) {retval += "6";}
            else if (temp4bits.equalsIgnoreCase("0111")) {retval += "7";}
            else if (temp4bits.equalsIgnoreCase("1000")) {retval += "8";}
            else if (temp4bits.equalsIgnoreCase("1001")) {retval += "9";}
            else if (temp4bits.equalsIgnoreCase("1010")) {retval += "a";}
            else if (temp4bits.equalsIgnoreCase("1011")) {retval += "b";}
            else if (temp4bits.equalsIgnoreCase("1100")) {retval += "c";}
            else if (temp4bits.equalsIgnoreCase("1101")) {retval += "d";}
            else if (temp4bits.equalsIgnoreCase("1110")) {retval += "e";}
            else if (temp4bits.equalsIgnoreCase("1111")) {retval += "f";}
            else {retval+="-"+temp4bits+"-";}
        }
        return retval;
    }

    public static int getFeaturesCount(final String str){
        String dataToUse = str.substring(1,3);
        int retval = 0;
        if (dataToUse.equalsIgnoreCase("00")) {retval = 0;}
        else if (dataToUse.equalsIgnoreCase("01")) {retval = 1;}
        else if (dataToUse.equalsIgnoreCase("10")) {retval = 2;}
        else if (dataToUse.equalsIgnoreCase("11")) {retval = 3;}
        else {retval = 0;}
        return retval;
    }

    public static String extractRawBits(final String str){
        String retval = "";
        retval += str.substring(3,3+21);
        retval += str.substring(25,25+23);
        retval += str.substring(49,49+23);
        retval += str.substring(73,73+23);
        retval += str.substring(97,97+23);
        retval += str.substring(121,121+23);
        retval += str.substring(145,145+23);
        retval += str.substring(169,169+23);
        retval += str.substring(193,193+23);
        retval += str.substring(217,217+23);
        retval += str.substring(241,241+23);
        retval += str.substring(265,265+23);
        retval += str.substring(289,289+23);
        retval += str.substring(313,313+23);
        retval += str.substring(337,337+23);
        retval += str.substring(361,361+23); // until the end
        return retval;
    }

    public static String addConstantsBits(final String str){
        String retval = "";
        retval += str.substring(0,48);
        retval += "0100";
        retval += str.substring(48,60);
        retval += "10";
        retval += str.substring(60,122);

        retval += str.substring(122+0,122+48);
        retval += "0100";
        retval += str.substring(122+48,122+60);
        retval += "10";
        retval += str.substring(122+60,122+122);        

        retval += str.substring(122+122+0,122+122+48);
        retval += "0100";
        retval += str.substring(122+122+48,122+122+60);
        retval += "10";
        retval += str.substring(122+122+60,122+122+122); // until the end
        return retval;
    }

    public static ThreeUUIDs generateFromBitmap(final Bitmap image, final int posTileX, final int posTileY){
        int stdPosX = posTileX / SIDE_LEN * SIDE_LEN; // works because of integer division
        int stdPosY = posTileY / SIDE_LEN * SIDE_LEN; // works because of integer division
        int currentColor = 0;
        String bigBinaryString = "";

        // extract 4x4 rgb colors
        for (int i=0; i<SIDE_LEN; i++){
            for (int j=0; j<SIDE_LEN; j++){
                currentColor = image.getPixel(stdPosX+j,stdPosY+i);
                String r = Integer.toBinaryString(Color.red(currentColor));
                String g = Integer.toBinaryString(Color.green(currentColor));
                String b = Integer.toBinaryString(Color.blue(currentColor));
                bigBinaryString += PADDING.substring(r.length()) + r + PADDING.substring(g.length()) + g + PADDING.substring(b.length()) + b;
            }
        }

        ThreeUUIDs retval = new ThreeUUIDs();
        if (bigBinaryString.substring(0,1).equalsIgnoreCase("1")){
            int feturesCount = getFeaturesCount(bigBinaryString);
            retval.setItemsNumber(feturesCount);
            String strippedBits = extractRawBits(bigBinaryString);
            String allBits = addConstantsBits(strippedBits);
            String hexString = bin2hex(allBits);

            String uuid1 = null;
            String uuid2 = null;
            String uuid3 = null;

            if (feturesCount >= 1){
                uuid1 = hexString.substring(0,8) + "-" + hexString.substring(8,12) + "-" + hexString.substring(12,16) + "-" + hexString.substring(16,20) + "-" + hexString.substring(20,32);
            }
            if (feturesCount >= 2){
                uuid2 = hexString.substring(32+0,32+8) + "-" + hexString.substring(32+8,32+12) + "-" + hexString.substring(32+12,32+16) + "-" + hexString.substring(32+16,32+20) + "-" + hexString.substring(32+20,32+32);
            }
            if (feturesCount >= 3){
                uuid3 = hexString.substring(64+0,64+8) + "-" + hexString.substring(64+8,64+12) + "-" + hexString.substring(64+12,64+16) + "-" + hexString.substring(64+16,64+20) + "-" + hexString.substring(64+20,64+32); // until the end
            }
            retval.setUUIDs(uuid1,uuid2,uuid3);
            //retval.setTest(retval.getItemsNumber() + ": " + strippedBits.length() + " " + allBits.length() );
            //retval.setTest("DEMO");
            //retval.setTest(retval.getItemsNumber() + ": " + uuid1 + "\n" + uuid2.substring(0,8) + " " + uuid3.substring(0,8) );
            // ok
        }else{
            retval.setItemsNumber(-1);
            //retval.setTest("Error items number");
            // error
        }
        return retval;
    }

    //public void setTest(String test){
    //    this.test = test;
    //}

    public void setUUIDs(String uuid1, String uuid2, String uuid3){
        this.uuid1 = uuid1;
        this.uuid2 = uuid2;
        this.uuid3 = uuid3;
    }

    public String getUUID(int i){
        String retval = null;
        if      (i == 1) {retval = uuid1;}
        else if (i == 2) {retval = uuid2;}
        else if (i == 3) {retval = uuid3;}
        else             {retval = "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX";}
        return retval;
    }

    public ThreeUUIDs(){
    }    

    public ThreeUUIDs(ThreeUUIDs aThreeUUIDs){
        this.itemsNumber = aThreeUUIDs.getItemsNumber();
        this.uuid1 = aThreeUUIDs.getUUID(1);
        this.uuid2 = aThreeUUIDs.getUUID(2);
        this.uuid3 = aThreeUUIDs.getUUID(3);
    }

    //public String getTest(){
    //    return test;
    //}

    private void setItemsNumber(final int itemsNumber){
        this.itemsNumber = itemsNumber;
    }



    public int getItemsNumber(){
        return itemsNumber;
    }

};

{% endhighlight %}
