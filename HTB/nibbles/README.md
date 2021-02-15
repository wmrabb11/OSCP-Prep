# user
  - nmap directory for initial scans
  - ports 22 and 80 are open, SSH looks up to date
  - WEBSITE
    - landing page is just a "hello world"
    - in the HTML comments -> '/nibbleblog'
    - apache, html, php
    - there's a "feed.php" that just displays some XML
    - reading through the "view source" of the page -> link to "/nibbleblog/admin/js/..." -> went to "/nibbleblog/admin" and found files listed
    - found an admin page: "/nibbleblog/admin.php"
    - looks like [this](https://packetstormsecurity.com/files/133425/NibbleBlog-4.0.3-Shell-Upload.html) will help us once we get logged in
    - guessed "admin:admin", "admin:password", "admin:nibbles" and the last one worked
    - verified that the version is the same one in the link above (4.0.3) by going to the settings panel and scrolling down to the bottom (it shows "Nibbleblog 4.0.3")
    - upload a PHP rev shell just like in the linked article
    - start a nc listener
    - visit `/nibbleblog/content/private/plugins/my_image/image.php`
  - nibbler shell caught
  - user pwned

# root
  - weird "monitor.sh" that just calls "bash" in nibblers home directory
  - grabbed the "personal.zip" file from nibbler home directory, the monitor script is different
  - `sudo -l` yields `User nibbler may run the following commands on Nibbles:
    (root) NOPASSWD: /home/nibbler/personal/stuff/monitor.sh`
  - we have write access to monitor.sh
    - I don't think the monitor.sh script is just supposed to be "bash", but it is rated as an easy box
    - when I went to start this box (I have a VIP subscription, so I can start retired boxes), it was already started
    - I'm thinking that the person working on this before me forgot to clean up plus I didn't think I'd need to reset the box
    - either way, we know we have write access to that file, and we can run the script as root
  - with the monitor script just having "bash" in it, running `sudo /home/nibbler/personal/stuff/monitor.sh` gives us a root shell
  - root pwned
