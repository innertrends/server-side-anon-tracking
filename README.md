# Server Side Anonymous User Ids

The most reliable and safe way to identify anonymous users without compromising their data is through the use of Secure and HttpOnly cookies.

These steps should be taken in order to set the anonymous id of users on the server side:

1. Generate the HttpOnly cookie

Only server-side scripts can create HttpOnly cookies. Making the HttpOnly cookie and setting it as a JS POST message requires a specialized file hosted on your server and accessible from the outside world.

Here's a PHP code snippet that demonstrates the process:

```
<?php
if (isset($_COOKIE["wau"]))
    {
        echo "<script>window.top.postMessage (JSON.stringify({ \"uuid\": \"".$_COOKIE["wau"]."\" }), \"*\");</script>";
    }
    else
    {
        $uuid = sprintf( '%04x%04x-%04x-%04x-%04x-%04x%04x%04x',mt_rand( 0, 0xffff ), mt_rand( 0, 0xffff ), mt_rand( 0, 0xffff ), mt_rand( 0, 0x0fff ) | 0x4000, mt_rand( 0, 0x3fff ) | 0x8000, mt_rand( 0, 0xffff ), mt_rand( 0, 0xffff ), mt_rand( 0, 0xffff ));
        $expire = date("D, d M Y H:i:s",time()+3600000)." GMT";
        header ("Set-Cookie: wau=$uuid; expires=$expire;path=/; domain=innertrends.com; httpOnly; SameSite=Strict");
        echo "<script>window.top.postMessage (JSON.stringify({ \"uuid\": \"".$uuid."\" }), \"*\");</script>"; 
    }
?>
```



2. Add the following code to the web website source code, in the **header** or above any analytics tracking.

```
window.dataLayer = dataLayer||[];
window.addEventListener('message', function (e) {
    var message = "";
    try {
        message = JSON.parse(e.data);
    } catch (ex) {
        //console.error(ex);
    }
    if (message !== "") 
    {
        var listenerData = JSON.parse(e.data);
        if (typeof listenerData.uuid != "undefined")
        {
            dataLayer.push(listenerData);
            //or set as variable that can be used in any tracking library
        }
    }
});

var iframe = document.createElement('iframe');
iframe.style.display = "none";
iframe.src = "https://YOURDOMAIN.COM/PATH_TO_COOKIE_FILE";
document.body.appendChild(iframe);
```

**Important:** Replace https://YOURDOMAIN.COM/PATH_TO_COOKIE_FILE with the path to the file created at step 1.