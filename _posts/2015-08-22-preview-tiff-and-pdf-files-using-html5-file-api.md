---
layout: post
title: "Preview TIFF and PDF files using HTML5 File API"
date: 2015-08-22 20:29:32 +0200
comments: true
category: blog
tags: [HTML5, FileReader, Javascript, Images, Canvas]
---

Sometimes you need to read files which are in the user's local machine. Here is where HTML [**File API**](http://www.w3.org/TR/FileAPI/) comes into play. The use case I'm going to explain is when you want to make a preview of an image which has not been uploaded anywhere yet, so you don't have the url of the image.
<!-- more -->
Also, if the file is TIFF or PDF the thing complicates. But the good news is that the preview is also possible. All the examples below make use of jQuery library. If you are interested how, continue reading.

### Preview web standard images
Below is the usual way of reading files using File API to make a preview of a standard **JPG** or **PNG** image.

``` html
<h2>Image Preview</h2>
<label>Select a file (jpg, jpeg, png)</label><br/>
<div><input type="file"/></div>
```

``` js
$(document).ready(function () {
	var fileTypes = ['jpg', 'jpeg', 'png'];  //acceptable file types
	$("input:file").change(function (evt) {
	    var parentEl = $(this).parent();
	    var tgt = evt.target || window.event.srcElement,
	                    files = tgt.files;

	    // FileReader support
	    if (FileReader && files && files.length) {
	        var fr = new FileReader();
	        var extension = files[0].name.split('.').pop().toLowerCase(); 
	        fr.onload = function (e) {
	        	success = fileTypes.indexOf(extension) > -1;
	        	if(success)
		        	$(parentEl).append('<img src="' + fr.result + '" class="preview"/>');
	        }
	        fr.onloadend = function(e){
	            console.debug("Load End");
	        }
	        fr.readAsDataURL(files[0]);
	    }   
	});
});
```

Note that first thing is to get the reference of the file/s on the change event. After that, we have to create an instance of **FileReader** to be able to read the file properly. Once we make sure that our browser supports FileReader and we have a reference to the file, then we read the file.

The outstanding aspect here is how we are reading the file with FileReader. Some methods are provided to read files. In this particular case we use ***FileReader.readAsDataURL(Blob|File)*** because the result property will contain the file/blob's data encoded as a data URL and that's what we are looking for in the `<img>` tag.


### Preview TIFF images
Displaying a tiff image on the web has been always a nightmare. And still there's no standard way of achieving it. For our example we'll use  [**seikichi's tiff.js library**](https://github.com/seikichi/tiff.js/tree/master).

Remember to add tiff js library.

`<script src="https://cdn.rawgit.com/seikichi/tiff.js/master/tiff.min.js"></script>`

Now, our fileTypes array will have support for tiff files only.

``` js
var fileTypes = ['tiff', 'tif'];
```

And, here is how we read and render tiff files.

``` js
if (FileReader && files && files.length) {
      var fr = new FileReader();
      var extension = files[0].name.split('.').pop().toLowerCase();
      fr.onload = function(e) {
        success = fileTypes.indexOf(extension) > -1;
        if (success) {
          //Using tiff.min.js library - https://github.com/seikichi/tiff.js/tree/master
          console.debug("Parsing TIFF image...");
          //initialize with 100MB for large files
          Tiff.initialize({
            TOTAL_MEMORY: 100000000
          });
          var tiff = new Tiff({
            buffer: e.target.result
          });
          var tiffCanvas = tiff.toCanvas();
          $(tiffCanvas).css({
            "max-width": "100px",
            "width": "100%",
            "height": "auto",
            "display": "block",
            "padding-top": "10px"
          }).addClass("preview");
          $(parentEl).append(tiffCanvas);
        }

      }
      fr.onloadend = function(e) {
        console.debug("Load End");
      }
      fr.readAsArrayBuffer(files[0]);
 }
```

The main difference here is now we are reading the file using ***FileReader.readAsArrayBuffer(Blob|File)*** and the image will be rendered on a HTML5 Canvas element.

### Preview PDF files
Obviously, PDF files are not web compliant, so we have to make use of external libraries which help us deal with these type of files. A very well-known is the [**Mozilla PDF JS library**](https://mozilla.github.io/pdf.js/). In our example we are going do a preview of the first page of the pdf file.

First, remember to include pdf js library in your html. You have to follow the steps described in the documentation to build the pdf.js project and generate the necessary pdf.js and pdf.worker.js files.

Now, our fileTypes array will have support for pdf files.

``` js
var fileTypes = ['pdf'];
```

Below is how we read the first page of a pdf file and render it on a HTML5 Canvas element.

``` js
if (FileReader && files && files.length) {
      var fr = new FileReader();
      var extension = files[0].name.split('.').pop().toLowerCase();
      fr.onload = function(e) {
        success = fileTypes.indexOf(extension) > -1;
        if (success) {
          console.debug("Parsing PDF document...");
          //PDFJS.workerSrc = '<path_to_pdf.worker.js>';
          PDFJS.getDocument(e.target.result).then(function getPdf(pdf) {
            //
            // Fetch the first page
            //
            pdf.getPage(1).then(function getPage(page) {
              var scale = 1.5;
              var viewport = page.getViewport(scale);

              //
              // Prepare canvas using PDF page dimensions
              //
              $(parentEl).append('<canvas id="pdf-image" class="preview"/>');
              var canvas = $(parentEl).find("canvas.preview").get(0);
              var context = canvas.getContext('2d');
              canvas.height = viewport.height;
              canvas.width = viewport.width;

              //
              // Render PDF page into canvas context
              //
              var renderContext = {
                canvasContext: context,
                viewport: viewport
              };
              page.render(renderContext);
            });
          });
        }

      }
      fr.onloadend = function(e) {
        console.debug("Load End");
      }
      fr.readAsArrayBuffer(files[0]);
 }
```

You can view or download the [**full example on this plunker**](http://plnkr.co/edit/o7j0segKRpjJHwhh8WGO?p=preview)
