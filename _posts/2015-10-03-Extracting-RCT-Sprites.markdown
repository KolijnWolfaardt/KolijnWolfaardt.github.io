---
layout: post
title:  "Extracting Roller Coaster Tycoon Sprites"
date:   2015-10-03
author: Kolijn
---
Recently I wanted to extract the [sprites](https://en.wikipedia.org/wiki/Sprite_%28computer_graphics%29) from [Roller Coaster Tycoon 2](https://en.wikipedia.org/wiki/RollerCoaster_Tycoon_2). For those that don't know the game, it is an older tycoon game, where you need to build up a theme park, with the specific focus on roller coasters. It is a 2D isometric game, but the view can be rotated. 

The image data for most of the sprites are stored in a file called g1.dat. This files uses its own special format to store the image data. Luckily, the format is documented [here](http://tid.rctspace.com/#TECHINFO). [This guide](http://tid.rctspace.com/csg/g1.html) in particular was invaluable in writing the script. I'm repeating the info given there, but more concise and with some examples.


## Extracting the sprites ##
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

After the offset positions the scan elements are stored. Each element begins with two bytes: The first indicates the number of pixels, the second the offset from the start of the row. The top bit of the first byte also indicates if it is the last element of the row. If the bit is set, it is the last element.

#### Palettes ####
The file contains various palettes. The length of the palette is calculated by <i>Sprite Width</i> x 3. The palettes are stored as Blue, Green and Red values (BRG). 

![Palette]({{ site.url }}/images/rct_palette.png)

The image shows one of the palettes in the file, which is also used for decoding the sprites. The other palettes are smaller, often only containing one colour. I have no idea what they are for.



## The script ##

I wrote a script to extract any subset of the sprites to a png image. The script also writes the sprite metadata to a file, and writes the sprite number next to the sprite. 

![Example Sprite Sheet]({{ site.url }}/images/rct_example_sprites.png)


The script has a bunch of flags that can be set:

There are certain sprites with colours determined by the user. These include parts such as the roller coasters, tracks and their supports. There are two colours that can be customized, and these can also be set, by using the -c1 and -c2 flags.


The palettes that are used for the images are stored inside the file. The first time the script is run it will store the palettes in the palettes folder. These will be re-used later to determine the colours. Storing the palettes on subsequent runs can be disabled with the -p switch. This can also be used to load your own custom palette.

[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help
[google]:      https://google.com

