---
layout: post
title:  "Extracting Rollercoaster Tycoon Sprites"
date:   2015-10-18
author: Kolijn
---
Recently I wanted to extract the [sprites](https://en.wikipedia.org/wiki/Sprite_%28computer_graphics%29) from [Rollercoaster Tycoon 2](https://en.wikipedia.org/wiki/RollerCoaster_Tycoon_2). For those that don't know the game, it is an older tycoon game, where you need to build up a theme park, with the specific focus on roller coasters. It is a 2D isometric game, but the view can be rotated.

The image data for most of the sprites are stored in a file called g1.dat. This files uses its own special format to store the image data. Luckily, the format is documented [here](http://tid.rctspace.com/#TECHINFO). [This guide](http://tid.rctspace.com/csg/g1.html) in particular was invaluable in writing the script. I'm repeating the info given there, but a bit more concise.

To obtain the g1.dat file you will have to buy the game from Amazon or [GOG](http://www.gog.com/game/rollercoaster_tycoon_2). There is also an open source implementation called [OpenRCT](https://openrct.net/), but you still need the original files.

### Extracting the sprites ###
The g1.dat file starts with 2 integers, containing 4 bytes each. The first integer is the number of sprites in the file, the second is the length of the file.

After these two integers there is an entry for each of the sprites. Each entry is 16 bytes long and contains:


<table style="width:50%">
	<tr><td class="td-name"><b>Name</b></td><td class="td-size"><b>Size (bytes)</b></td></tr>
	<tr><td class="td-name">Start Address</td><td class="td-size"><i>4</i></td></tr>
	<tr><td class="td-name">Sprite Width</td><td class="td-size"><i>2</i></td></tr>
	<tr><td class="td-name">Sprite Height</td><td class="td-size"><i>2</i></td></tr>
	<tr><td class="td-name">X Offset</td><td class="td-size"><i>2</i></td></tr>
	<tr><td class="td-name">Y Offset</td><td class="td-size"><i>2</i></td></tr>
	<tr><td class="td-name">Flags</td><td class="td-size"><i>2</i></td></tr>
	<tr><td class="td-name">Padding</td><td class="td-size"><i>2</i></td></tr>
</table> 

The start address is offset from the end of all the sprite entries. So the first sprite will have an address of 0. The width and height are obvious; they are the width and height of the final sprite. The <i>X Offset</i> and <i>Y Offset</i> are used when calculating where to draw a sprite occupying a specific tile. The flag is used to determine the sprite type and encoding, and the padding is to align the entry to 16 bytes. There are 5 flag types that I found in the file:

<table style="width:50%">
	<tr><td><b>Flag</b></td><td><b>meaning</b></td></tr>
	<tr><td>1 or 17</td><td>Plain sprite</td></tr>
	<tr><td>5 or 21</td><td>Compressed sprite</td></tr>
	<tr><td>8</td><td>Palette</td></tr>
</table>

#### Plain Sprites ####
Plain sprites are easy: The pixel data starts at the end of the entry data plus the offset. The pixel data contains <i>sprite_width</i> x <i>sprite_height</i> pixels, all of which are stored as a single byte. The values are indexes to the palette being used. To obtain the RGB value for the sprite, look up the RGB value in the palette.

#### Compressed Sprites ####
Compressed sprites are a bit harder:

The sprite is stored as a series of scan elements. There can be more than one element per line of the image, or even none at all. 

The image definition starts with a list of offsets for each scan element. The addresses are 2 bytes long and are relative to the start address of the sprite.

After the offset positions the scan elements are stored. Each element begins with two bytes: The first indicates the number of pixels, the second the offset from the start of the row. The top bit of the first byte also indicates if it is the last element of the row. If the bit is set, it is the last element. After the two bytes there are single bytes which are used to look up the RGB values.

#### Palettes ####
The file contains various palettes. The length of the palette is calculated by <i>Sprite Width</i> x 3. The palettes are stored as Blue, Green and Red values (BRG). 

![Palette]({{ site.url }}/images/rct_palette.png)

This is one of the palettes in the file, which is also used for decoding the sprites. The other palettes are smaller, often only containing one colour. I have no idea what they are for.



### The script ###

I wrote a script to extract any subset of the sprites to a png image. The script also writes the sprite metadata to a file, and writes the sprite number next to the sprite. It is available at [https://github.com/KolijnWolfaardt/RCT_tools/blob/master/exportSprites.py](https://github.com/KolijnWolfaardt/RCT_tools/blob/master/exportSprites.py).

![Example Sprite Sheet]({{ site.url }}/images/rct_example_sprites.png)


The script requires [numpy](http://www.numpy.org/) and [PIL](http://www.pythonware.com/products/pil/) to run.
The script has a bunch of flags that can be set:

<b>Filename</b> &nbsp;<code>-f or --file </code><br>
This sets the location of the g1.data file. Per default, the script will just search in the current directory.

<b>Sprite Start</b> &nbsp; <code>-s or --sprite-start </code><br>
Sets the sprite to start from

<b>Sprite End</b> &nbsp; <code>-s or --sprite-end </code><br>
Set the sprite to end. Must be larger than Sprite Start

<b>Output File Width</b> &nbsp; <code> --out-width</code><br>
Sets the width of the ouput file.

<b>Output File Height</b> &nbsp; <code> --out-height</code><br>
Sets the Height of the output file.

<b>Palette</b> &nbsp; <code>-p</code><br>
This can be used to set a customized palette file. The file has to be a set of comma seperated interger values. Each line represent one color, with a total of 255 in the file.

<b>Custom Colors</b> &nbsp; <code>-c1 and -c2</code><br>
There are certain sprites with colours determined by the user. These include parts such as the rollercoasters, tracks and their supports. There are two colours that can be customized, and these can also be set, by using the -c1 and -c2 flags.

<b>Verbose</b>&nbsp; <code>-v</code><br>
You can tell the script to produce more output by setting this flag.

#### First Run ####
The palettes that are used for the images are stored inside the g1.dat file. The first time the script is run it will store the palettes in the palettes folder. These will be re-used later to determine the colours. 

#### NFO File ####
The script also stores the following details about the sprites in an <code>.nfo</code> file in the ouput directory.
<ul>
<li>Output File Number</li>
<li>Sprite number in the file</li>
<li>Total Sprite number</li>
<li>Address in the file (as a hex value)</li>
<li>Sprite Width</li>
<li>Sprite Height</li>
<li>xOffset</li>
<li>yOffset</li>
<li>flags</li>
<li>imagePosX</li>
<li>imagePosY</li>
</ul>

These values are stored as comma seperated values. For example, the output for the image above looks like this:
<table>
<tr><td>0,</td><td>    0,</td><td>   1915,</td><td> 106b34,</td><td> 64,</td><td> 31,</td><td> -32,</td><td> 0,</td><td> 21,</td><td> 3,</td><td> 7</td></tr>
<tr><td>0,</td><td>    1,</td><td>   1916,</td><td> 106fb0,</td><td> 64,</td><td> 32,</td><td> -32,</td><td> -1,</td><td> 21,</td><td> 99,</td><td> 7</td></tr>
<tr><td>0,</td><td>    2,</td><td>   1917,</td><td> 107440,</td><td> 64,</td><td> 16,</td><td> -32,</td><td> 0,</td><td> 21,</td><td> 195,</td><td> 7</td></tr>
<tr><td>0,</td><td>    3,</td><td>   1918,</td><td> 1076a0,</td><td> 62,</td><td> 16,</td><td> -32,</td><td> -1,</td><td> 21,</td><td> 291,</td><td> 7</td></tr>
<tr><td>0,</td><td>    4,</td><td>   1919,</td><td> 1078e0,</td><td> 64,</td><td> 32,</td><td> -32,</td><td> -1,</td><td> 21,</td><td> 387,</td><td> 7</td></tr>
<tr><td>0,</td><td>    5,</td><td>   1920,</td><td> 107d70,</td><td> 64,</td><td> 32,</td><td> -32,</td><td> -1,</td><td> 21,</td><td> 483,</td><td> 7</td></tr>
<tr><td>0,</td><td>    6,</td><td>   1921,</td><td> 108210,</td><td> 62,</td><td> 16,</td><td> -30,</td><td> -1,</td><td> 21,</td><td> 579,</td><td> 7</td></tr>
</table>

[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help
[google]:      https://google.com

