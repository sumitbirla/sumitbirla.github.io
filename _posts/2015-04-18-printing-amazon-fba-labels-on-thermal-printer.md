---
id: 630
title: Printing Amazon FBA labels on thermal printer
date: 2015-04-18T13:06:53-04:00
author: Sumit Birla
layout: post
guid: http://sumitbirla.com/?p=630
permalink: /2015/04/printing-amazon-fba-labels-on-thermal-printer/
image: /wp-content/uploads/2015/04/zebra-lp2844.jpg
categories:
  - Technology
---
<a href="http://sumit.tampahost.net/2015/04/printing-amazon-fba-labels-on-thermal-printer/zebra-lp2844/" rel="attachment wp-att-685"><img src="http://sumit.tampahost.net/wp-content/uploads/2015/04/zebra-lp2844-150x150.jpg" alt="" title="Zebra LP2844" width="150" height="150" class="alignright size-thumbnail wp-image-685" srcset="https://sumitbirla.me/wp-content/uploads/2015/04/zebra-lp2844-150x150.jpg 150w, https://sumitbirla.me/wp-content/uploads/2015/04/zebra-lp2844-300x300.jpg 300w, https://sumitbirla.me/wp-content/uploads/2015/04/zebra-lp2844.jpg 500w" sizes="(max-width: 150px) 100vw, 150px" /></a>Thermal printers such as the Zebra LP2844 have been around for decades and are used widely in fulfillment centers. They are simple, low-maintenance printers that don&#8217;t require ink and spit out shipping labels with very high contrast (just two colors &#8211; black and white). Zebra printers speak a very simple text-based language called EPL2. Amazon doesn&#8217;t provide the ability to print to EPL2 printers, however. They provide a PDF file that you must print on a regular printer, cut using scissors and stick on boxes using tape. Not very conducive to high-volume shipping. To add insult to injury, the resolution, dimensions and positioning of the labels on the PDF isn&#8217;t helpful either if you wish to print an image of the label to the thermal printer.

In order to be able to printer Amazon FBA labels on a thermal printers, I have written a script that extracts labels from the PDF, resizes and rearranges then, and combines back into a single PDF so that you can use your computer&#8217;s print dialog to print with acceptable results. The key to successful transmogrification of Amazon&#8217;s PDF is to reduce the number of times the labels are resized. Labels mostly contain text and lines and each time you resize such an image, aliasing artifacts appear. So in order to understand the layout of Amazon&#8217;s PDF, let&#8217;s analyze it using standard linux tools:

**Note:** the following information is for a shipment consisting for 5 packages (5 UPS labels and 5 box labels)

<pre>root@pinto:/home/dropbox/Dropbox/www# pdfinfo package.pdf 
Producer:       iText 2.1.7 by 1T3XT
CreationDate:   Fri Apr 17 13:52:52 2015
ModDate:        Fri Apr 17 13:52:52 2015
Tagged:         no
Form:           none
Pages:          5
Encrypted:      no
Page size:      612 x 792 pts (letter)
Page rot:       0
File size:      827702 bytes
Optimized:      no
PDF version:    1.4</pre>

The above output shows that Amazon created the PDF using iText library, standard letter size (very different from thermal printers 4&#8243; x 6&#8243; dimension). Note that the page size is 612 x 792 pts. Each postscript point is 1/72th of an inch by default.

The next step is to see what embedded images are present in the PDF:

<pre>root@pinto:/home/dropbox/Dropbox/www# pdfimages -list package.pdf 
page   num  type   width height color comp bpc  enc interp  object ID
---------------------------------------------------------------------
   1     0 image      86    18  gray    1   1  ccitt  no        29  0
   1     1 image    1400   800  index   1   8  image  no        30  0
   2     2 image      86    18  gray    1   1  ccitt  no        34  0
   2     3 image    1400   800  index   1   8  image  no        35  0
   3     4 image      86    18  gray    1   1  ccitt  no        38  0
   3     5 image    1400   800  index   1   8  image  no        39  0
   4     6 image      86    18  gray    1   1  ccitt  no        42  0
   4     7 image    1400   800  index   1   8  image  no        43  0
   5     8 image      86    18  gray    1   1  ccitt  no        46  0
   5     9 image    1400   800  index   1   8  image  no        47  0
</pre>

This shows there are two embedded images on each page. One has dimensions of 86&#215;18 and the other is 1400&#215;800. The larger one turns out to be our UPS shipping label. The smaller one is a [PDF417 barcode](http://en.wikipedia.org/wiki/PDF417) that appears on the box label. The rest of the box label is actual text placed on the PDF using the [iText library](http://en.wikipedia.org/wiki/IText).

<div id="attachment_644" style="width: 610px" class="wp-caption aligncenter">
  <br /><a href="http://sumit.tampahost.net/2015/04/printing-amazon-fba-labels-on-thermal-printer/fba_labels/" rel="attachment wp-att-644"><img aria-describedby="caption-attachment-644" src="http://sumit.tampahost.net/wp-content/uploads/2015/04/fba_labels-789x1024.png" alt="" title="Amazon FBA Labels" width="600" class="aligncenter size-large wp-image-644" style="border: solid 1px #666; box-shadow: 0 0 5px #ccc;" srcset="https://sumitbirla.me/wp-content/uploads/2015/04/fba_labels-789x1024.png 789w, https://sumitbirla.me/wp-content/uploads/2015/04/fba_labels-231x300.png 231w, https://sumitbirla.me/wp-content/uploads/2015/04/fba_labels.png 861w" sizes="(max-width: 789px) 100vw, 789px" /></a>
  
  <p id="caption-attachment-644" class="wp-caption-text">
    A page from the PDF provided by Amazon
  </p>
</div>

With above information, we can start writing our script to convert above PDF into something that can be printed on a thermal printer without sacrificing on the quality of the print much. Remember we must try to avoid resizing images as much as possible before sending the final print job.

  1. Extract the embedded shipping label images from Amazon PDF and save as PNG.
  2. Crop image to remove extra whitespace and rotate shipping label 90 degrees.
  3. Cut box labels from PDF and save as PNG having same width as the shipping label image.
  4. Combine the images to form a new PDF having resolution of 203 dpi (same as thermal printer).

The tools needed to run this script are ImageMagick, pdfimages and ghostscript.

<pre class="brush: ruby; title: ; notranslate" title="">#!/usr/bin/env ruby

require 'fileutils'

input_pdf = ARGV[0]
output_pdf = ARGV[1]
tmp_dir = "./fba_tmp"

# create directory to hold temporary images
Dir.mkdir(tmp_dir) unless Dir.exist?(tmp_dir)
system "rm -f #{tmp_dir}/*"

# extract images from input PDF
system "pdfimages -j #{input_pdf} #{tmp_dir}/ups"

# delete the *.pbm files because those are not the shipping label
system "rm #{tmp_dir}/*.pbm"

# Rotate and crop each shipping label.  Output images will be 800 x 1204
i = 0
Dir.glob(tmp_dir + "/*.ppm").each do |file|
i += 1
system "convert #{file} -gravity west -crop 86x100%+0+0 -rotate 90 #{tmp_dir}/shipping-#{i}.png"
end
system "rm #{tmp_dir}/*.ppm"

# Cut out the box labels from the PDF and resize to match shipping labels
system "convert -density 400 #{input_pdf} -gravity northwest -alpha off -crop 50x48%+0+150 -resize 800x1204 #{tmp_dir}/box.png"

# Combine images into a single PDF, each page is set to be 800x1204 to avoid any resizing
system "convert #{tmp_dir}/*.png -density 203 -repage 800x1204+0+0 #{output_pdf}"
</pre>

Run this script passing in page of PDF provided by Amazon and name of the processed PDF file:

<pre>./fba_labels.rb package.pdf output.pdf</pre>

Here is what the new PDF looks like in viewer:

<a href="http://sumit.tampahost.net/2015/04/printing-amazon-fba-labels-on-thermal-printer/new-layout/" rel="attachment wp-att-717"><img src="http://sumit.tampahost.net/wp-content/uploads/2015/04/new-layout-432x1024.png" alt="" title="New PDF layout" height="800" class="aligncenter size-large wp-image-717" srcset="https://sumitbirla.me/wp-content/uploads/2015/04/new-layout-432x1024.png 432w, https://sumitbirla.me/wp-content/uploads/2015/04/new-layout-126x300.png 126w" sizes="(max-width: 432px) 100vw, 432px" /></a>

The final step involves opening the PDF in any viewer and printing using the system print dialog. Be sure to select 4&#8243; x 6&#8243; as the page size, otherwise you will not get correctly sized labels.

<a href="http://sumit.tampahost.net/2015/04/printing-amazon-fba-labels-on-thermal-printer/print-dialog/" rel="attachment wp-att-690"><img src="http://sumit.tampahost.net/wp-content/uploads/2015/04/print-dialog.png" alt="" title="Print Dialog" width="600" class="aligncenter size-full wp-image-690" srcset="https://sumitbirla.me/wp-content/uploads/2015/04/print-dialog.png 793w, https://sumitbirla.me/wp-content/uploads/2015/04/print-dialog-300x197.png 300w" sizes="(max-width: 793px) 100vw, 793px" /></a>

Although the steps above look straight-forward, it took hours of experimentation to get all the parameters for \`convert\` right. Given the size and scope of Amazon fulfillment operations, they really should provide a way of printing to thermal printers. Some of the avenues I explored included using UPS api to fetch EPL2 label, however, you can&#8217;t retrieve a label for a shipment was created elsewhere. The second option was to recreate the labels using the EPL2 language, but there were too many unknowns of whether the various funky barcode symbologies used were needed or not, not to mention the amount of time it would take to implement it. Finally, this hack-job of a script turned out to be sufficient for the task at hand. No more scissors or tape to put labels on boxes &#8211; a huge time-saver!

Although the script is written in Ruby, it could very well be a bash script. If someone would write it, I will be happy to share it here for other who don&#8217;t want the Ruby dependency for a simple script.