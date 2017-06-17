# png2pdf-worker-js
a proof-of-concept javascript worker for converting png to jpeg, without using canvas

I needed to convert very large png files to lower quality jpegs, client side on the browser. I found that loading the usual method of using the canvas (e.g. https://davidwalsh.name/convert-canvas-image) choked when your source image was several megabytes, and also locked up the browser half the time.

So I mated a jpg compressor with a png loader and came up with this worker.

Relies on these libraries:

* https://github.com/frumbert/png2jpeg-worker-js
* https://github.com/devongovett/png.js

It's a worker, so use it like this:

	var worker = new Worker("worker.js");

	worker.onmessage = function(e) {
		var blob = new Blob([e.data.data], {type: 'image/jpeg'} );
		var imageUrl = (window.URL || window.webkitURL).createObjectURL(blob);
		img = new Image();
		img.src = imageUrl;
		document.querySelector("body").appendChild(img);
	}

	// in my example I'm reading all png files from the root of a zip file, converting them to jpeg, then displaying them on the body

	var zip = new JSZip();  // https://github.com/Stuk/jszip

    	fetch("input.zip")
	.then(function(response) {
		return response.arrayBuffer();
	})
	.then(function(arraybuffer) {
		return zip.loadAsync(arraybuffer);
	}).then(function(zip) {
		zip.forEach(function(relativePath,file) {
			if (file.name.indexOf(".png") !== -1) {
				file.async("uint8array").then(function(data) {
					worker.postMessage({
						image: data,
						quality: 70
					});
				})
			}
		});
	});

There's certainly more you could do - such as better error handling... 

See included libraries for licensing information.