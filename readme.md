# Javel3

A new version of javel but for bah-asm.

There are some cool new features!

## Documentation
1. [The javel3 server](#the-javel3-server)
2. [The templating system](#the-templating-system)



### The Javel3 server
![structure](assets/struct.svg) **javel3** *(extend http_server)* The javel3 HTTP server instance.
- ![structure](assets/var.svg) **certFile**: `str` Path to your SSL certificate (PEM file).
- ![structure](assets/var.svg) **keyFile**: `str` Path to your SSL private key (PEM file).
- ![structure](assets/var.svg) **routes**: `[]javel3_route*` Array containing all your routes.
- ![structure](assets/var.svg) **port**: `int` The port on which the server should start listenning.
- ![structure](assets/var.svg) **portTLS**: `int` The port on which the server should start listenning for TLS connection (only if .certFile and .keyFile provided).
- ![structure](assets/var.svg) **maxRequestSize**: `int` The maximum request size that the server should receive.
- ![structure](assets/func.svg) **launch**: `launch()` Launch the javel3 HTTP server. This function will block the calling thread.

---

![structure](assets/struct.svg) **javel3_route** A javel3 route.
- ![structure](assets/var.svg) **base**: `str` Path to the root of your route in your file-system.
- ![structure](assets/var.svg) **path**: `str` Path to your route in the URL.
- ![structure](assets/var.svg) **cacheMaxAge**: `int` If >= 0, the number of seconds that this route's request should be kept in cache (will not work if you set a custom handle).
- ![structure](assets/var.svg) **disableParsing**: `bool` Will disable HTML parsing (will not work if you set a custom handle). This is set to false by default.
- ![structure](assets/func.svg) **handle**: `function(javel3_route*, http_request*, http_response*)` Custom handle (useful for making API...).
- ![structure](assets/func.svg) **error**: `function(javel3_route*, int, http_request*, http_response*)` Errors handle, first int argument is the http error code that must be sent to the client as `resp.code`.


### The templating system

By default, any route with de default handle and .disableParsing set to false, will treat .html files as template files. 

Template files allow you to mix client side Javascript and server side Bah inside the same file. Anycode that is executed server side cannot be seen by the client.

Anytime the template file is saved, the Javel3 server will automaticaly recompile it. This enables you to make hotfixes to dynamic webpages without restarting or recompiling the server. Note that this uses the JIT compiler, if you #define any function declared inside your bah project, you will be able to call it.

For the jit compiler and the templating system to work, the `/opt/bah-asm/` bah libraries must be present on the system.  

There is three way of insertting Bah code:
- `<script type="bah"> your bah code </script>`<br/>
    Useful to include files or declare functions / global variables.
- `<embed src="your bah code" type="bah" />`<br/>
    Useful to execute bah code inline Anything that is `echo()`ed isnisd here will be rendered in place (as would php do). The code is treated as a function body.
    Inside and embed statement, you can use `request` and `response` to access the http_request and http_response of the current connection.
- `bah.myFunction()` inside your javascript code.
    Note that any js argument used when calling the function will be converted to the arguments of the Bah function definition. Any function that is called inside the javascript code will be exposed to the client as a javascript binding. 
    
    **Note that if a function is not called inside the static html page, it will not be exposed.**


Inside the Bah code, you can interact with the client with few functions:
- `echo(s str)` to insert some text in the page where the script is located. This will only work before the page is sent to the client.
- `eval(js str)` will evaluate the javascript code inside the browser of the client.

#### Example

The following example will let you update a variable `nb` by the value of the `ammount` input. All calculations are done server side in Bah.
(This is really dumb but shows the three ways of declaring bah code).
```html
<html lang="en">
<head>
    <title>Adder</title>
</head>
<body>
    <script type="bah">
        #import "time.bah"
        nb = 0

        prefixStr = "Score: "
        increaseNb(ammount int) {
            if ammount < 1 {
                eval("alert(\"You cannot add an ammount less than 1.\")")
                return
            }

            nb += ammount

            if nb < 0 {
                nb = 0
            }

            eval("nbdiv.innerText = \""+prefixStr+intToStr(nb)+"\"")
        }
    </script>

    <div id="nbdiv">
        <embed src="echo(prefixStr+intToStr(nb))" type="bah">
    </div>

    <input type="number" id="ammount" value="1">
    <button onclick="bah.increaseNb(ammount.value)">Add</button>

    <p>In this example, it is impossible from the webpage to achieve a negative score.</p>
</body>
</html>
```