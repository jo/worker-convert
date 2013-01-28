# Worker Convert

This worker converts images with imagemagick.


## Prerequisites

This worker depends on `imagemagick`. Make shure the `convert` program is in the path.

## installation

    git clone git://github.com/null2/worker-convert.git
    cd worker-convert
    npm install


## Configuration

Configuration is done in a worker configuration document inside the target database.
The worker only process if there exists such a configuration file.

A Worker Configuration File might look like this:

    {
      "_id": "worker-config/convert",
      "_rev": "9-a653b27246b01cf9204fa9f5dee7cc64",
      "formats": [ "jpg", "jpeg", "png", "gif", "tif", "tiff", "bmp", "svg" ],
      "versions": {
        "thumbnail": {
          "format": "png",
          "arguments": [
            "-thumbnail", "135x135",
            "-colorspace", "sRGB"
          ]
        }
      }
    }

You can update the config live so that all future processings will take the new configuration.

Note that attachment names which starts with a version name, followed by a '/' are ignored.

`formats` must be an array containing the image formats the worker should handle,
strictly speaking an array of file extensions.

You can specify one or many `versions` to be generated.
The version name will be part of the image, eg. for an image named 'image.jpg' there will be
generated a thumbnail named `thumbnail/image001.png`.

Each version can specify a target `format`, `png` in that case,
and can specify individual `arguments`, which must be an array containing
the arguments passed to imagemagick. The `convert` invocation in this case would look like

    convert image.jpg -thumbnail 135x135 -colorspace sRGB outfile.png


## Worker Status Document

The worker stores a status document inside the target database.
The worker stores its last update seq here and can resume at the point it stopped.

    {
      "_id": "worker-status/convert",
      "_rev": "1-1922b5623d07453a753ab6ab2c634d04",
      "last_update_seq": 34176
    }


## Document Status Object

The worker updates a status object inside the document.
This makes it supereasy to monitor worker status as well as
it keeps an atomic lock when many workers listen to the same database.
Images are prozessed only once by comparing the attachments _revpos_ property
with the revpos property of the status object.

The status object of the worker looks like this:

    "worker_status": {
      "convert": {
        "image.jpg": {
          "status": "completed",
          "revpos": 160
        }
      }
    }

The status field can be _triggered_, _completed_ or _error_.

The worker status is scoped by the worker name in order to have many workers
processing the same document.
The worker status is also per attachment to process only attachments which had changed.


## Running the Worker

To start, this needs the following environment variables set:

    COUCH_SERVER=http://example.org COUCH_DB=mydb npm start


## License & Copyright

(c) 2013 Johannes J. Schmidt, null2 GmbH, Berlin

Licensed under the Apache License 2.0.
