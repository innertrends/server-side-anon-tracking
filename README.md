# Server Side Anonymous User Ids

Secure and HttpOnly cookies are the most relais and safe method to identify anonymous users without compromising their data.

Please follow these steps in order to enable server side anonymous users tracking:

1. Generate the HttpOnly cookie

HttpOnly cookies can only be generated using server side scripts. Create a dedicated file on your server that is available on the internet that will create the HttpOnly cookie and set it as a JS POST message.

Here is an example on how to do this in PHP:

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