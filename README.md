Panda uploader
==============

Panda uploader allows you to upload videos from your applications to Panda. It uses an HTML5-based upload method or, if this is not supported, falls back to a Flash-based solution.

It works as a jQuery plugin, and requires requires JQuery version 1.3.

First of all
------------

Copy the `panda_uploader` directory to your app's public path.

Include the following declaration in your page, after loading jQuery. Replace `VERSION` with the correct number:

    <script src="/panda_uploader/jquery.panda-uploader-VERSION.min.js" type="text/javascript"></script> 

The above will load the necessary JavaScript code.

Generating access details
-------------------------

You will also need a the help of a server-side library to generate access details for you. On the examples below, you will notice the following snippet of JS code:

    var panda_access_details = {
        'access_key': 'your-access-key',
        'cloud_id': 'your-cloud-id',
        'timestamp': '2010-02-02T17:04:46+00:00',
        'signature': 'dC/M34sNc7v+oyhFRfVLEKpkVMNyhhyuyCECAQiR6nrUw='
    };

These access details are best generated by a support library. We recommend you use the ones we provide for several popular server-side languages: [Ruby](http://github.com/newbamboo/panda_gem), [PHP](http://github.com/newbamboo/panda_client_php) and [Python](http://github.com/newbamboo/panda_client_python).

For either of the above libraries, the process is the same: initialise a Panda instance and use the method `signed_params()` to generate your access details:

    # Ruby example
    ruby_access_details = panda.signed_params('POST', '/videos.json')

    # PHP example
    $php_access_details = $panda->signed_params('POST', '/videos.json');

    # Python example
    python_access_details = panda.signed_params('POST', '/videos.json')

Then convert the returned details to JSON and add them to your HTML template.

Simplest example
----------------

The following is the simplest working form that will upload a video:

    <form action="/path/to/action">
        <!-- the control will appear next to this, and the video ID will be stored here after the upload -->
        <input type="hidden" name="panda_video_id" id="returned_video_id" />

        <!-- a submit button -->
        <input type="submit" value="Upload video" />
    </form>
    <script>
    var panda_access_details = {
        'access_key': 'your-access-key',
        'cloud_id': 'your-cloud-id',
        'timestamp': '2010-02-02T17:04:46+00:00',
        'signature': 'dC/M34sNc7v+oyhFRfVLEKpkVMNyhhyuyCECAQiR6nrUw='
    };
    jQuery("#returned_video_id").pandaUploader(panda_access_details);
    </script>

This will render a fairly ugly form. We'll worry about the looks later. For now:

1. Click on "Choose file". You will be shown a file selection dialog.
2. Select a file to upload.
3. Click "Upload video" and the upload will start.

After the upload, Panda returns a unique ID that identifies your video. This will be automatically set as the value of the hidden field `#returned_video_id`, which was specified in the jQuery call. Then, the form will be finally submitted so your application can read this value and use it to reference the video later.


Adding a progress bar
---------------------

The example above is minimal, and has a very poor interface. At the moment, the user doesn't know how the upload process is going, or if it is working at all. A progress bar would be very appropriate here, and one is included by default.

To enable it, first create a DIV that will contain the bar:

    <!-- upload progress bar (optional) -->
    <div id="upload_progress" class="panda_upload_progress"></div>

And then let pandaUploader() know about it:

    jQuery("#returned_video_id").pandaUploader(panda_access_details, { upload_progress_id: 'upload_progress' });

Finally, the full example with all the controls would be:

    <form action="/player.php">
        <!-- the control will appear next to this, and the video ID will be stored here after the upload -->
        <input type="hidden" name="panda_video_id" id="returned_video_id" />

        <!-- upload progress bar (optional) -->
        <div id="upload_progress" class="panda_upload_progress"></div>

        <!-- a submit button -->
        <p><input type="submit" value="Upload video" /></p>

        <script>
        var panda_access_details = {
            'access_key': 'your-access-key',
            'cloud_id': 'your-cloud-id',
            'timestamp': '2010-02-02T17:04:46+00:00',
            'signature': 'dC/M34sNc7v+oyhFRfVLEKpkVMNyhhyuyCECAQiR6nrUw='
        };
        jQuery("#returned_video_id").pandaUploader(panda_access_details, {
            upload_progress_id: 'upload_progress'  // Optional
        });
        </script>
    </form>

Advanced usage
--------------

### Additional arguments

At the moment, the following arguments are supported:

* **`upload_progress_id`**: the ID of DIV that will contain the progress bar.
* **`api_host`**: alternative host for the Panda API. Must be set to `api.eu.pandastream.com` if you signed up for the EU region cloud.
* **`uploader_dir`**: path were the uploader files are located in the web server. By default "`/panda_uploader`"
* **`upload_strategy`**: see below.
* **`widget`**: force use of HTML5-based or Flash-based widget. By default, HTML5 widget is used if supported, falling back to Flash if not. See below.
* and several events: `onwidgetload`, `onchange`, `onprogress`, `onreadystatechange`, `onsuccess`, `onload`. See below.


### Upload strategies

This plugin allows for two "strategies" to upload the video file:

1. **Upload on submit**: the file will be uploaded when the form is submitted.
2. **Upload on select**: the file will be uploaded as soon as it is selected.

The default behaviour is upload on submit. If you wish to upload on select instead, you'll need to specify this explicitly:

    jQuery("#returned_video_id").pandaUploader(panda_access_details, {
        upload_strategy: new PandaUploader.UploadOnSelect()
    });

### Upload on submit

The "upload on submit" strategy also accepts an option:

* **`disable_submit_button`**: defaults to true. When true, it will disable the form's submit button until a video is selected.

To specify strategy options, do the following:

    jQuery("#returned_video_id").pandaUploader(panda_access_details, {
        upload_strategy: new PandaUploader.UploadOnSubmit({
            disable_submit_button: false
        })
    });

### Repeated uploads with "Upload on select"

When using the "upload on select" strategy, you have to bear this in mind: users may upload a file, then change their minds about it and upload another one in its place. If you want this to work correctly, you have to make sure that each separate upload uses a different cryptographic signature.

How to generate another signature?:

1. You should make an Ajax request to your own site to generate the signature
2. The `panda_access_details` parameter should be given as a function rather than a hash, so that this is regenerated for every upload

WHAT???

OK, ok, I know that's a bit too much. You are better off by having a look at [some example code](http://github.com/newbamboo/panda_example_php/blob/master/ajax/index.php). On it, the important bits are:

1. `pandaUploader` is passed a function `get_signed_params`
2. `get_signed_params` returns the latest generated parameters and starts an AJAX request to get new ones

I hope that helps...

### Flash widget

By default, the plugin checks if the browser supports Flash-less uploads using HTML5 features (supported in Gecko and Webkit browsers). If this is not the case, a Flash solution is used.

If you instead want to use Flash in all cases, you may specify so by using the `widget` option:

    jQuery("#returned_video_id").pandaUploader(panda_access_details, {
        widget: new PandaUploader.FlashWidget()
    });

The constructor of the widget accepts an argument: a hash of options to be passed on to swfupload. For example, if you want to show a different button, do:

    jQuery("#returned_video_id").pandaUploader(panda_access_details, {
        widget: new PandaUploader.FlashWidget({
            button_image_url : "/my-cool-button.png",
            button_width : 70,
            button_height : 30
        })
    });

All available arguments are documented at the [SWFUpload site](http://demo.swfupload.org/Documentation).

In addition to SWFUpload options, the Flash widget accepts one additional argument:

* **`add_filename_field`**: add a text field next to the widget. This will show the name of the currently selected file, mimicking the behaviour of a standard HTML file field. Defaults to `true`.

### Custom events

This plugin also provides a number of events where you can plug your own actions:

* onwidgetload: widget has been initialised and rendered on the page
* onchange: a new file has been selected
* onprogress: called multiple times when the file is being uploaded
* onreadystatechange: called when the server responds to preflight ([see W3C spec](http://www.w3.org/TR/XMLHttpRequest2)) and upload requests
* onload: file upload completed (successfully or not). Results are not yet available.
* onsuccess: operation completed successfully

Example:

    $('#returned_video_id').pandaUploader(panda_access_details, {
        onchange: function() {
            alert("New file selected");
        },
    });


Notes
-----

This code uses [Adam Royle's swfupload-jquery-plugin](http://github.com/ifunk/swfupload-jquery-plugin), which is in turn an interface to [SWFUpload](http://www.swfupload.org). Code is minified using the [Google Compiler](http://code.google.com/closure/compiler/).


Copyright (c) 2010 New Bamboo. Distributed under the terms of the MIT License. See LICENSE file for details