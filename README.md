# Install dnscrypt-proxy on Ubuntu 18.04

Not exactly the best way to do it, but it works...

[Oficial documentation to install on Linux](https://github.com/jedisct1/dnscrypt-proxy/wiki/Installation-linux)

[Steps that I got inspired to get it to work](https://github.com/jedisct1/dnscrypt-proxy/issues/557#issue-346484267)

### Step by step: 
###### (ooh baby...)

1. Install *`net-tools`*:
    ```bash
    sudo apt install net-tools
    ```

2. Install *`dnscrypt-proxy`*:
    ```bash
    sudo add-apt-repository ppa:shevchuk/dnscrypt-proxy
    sudo apt install dnscrypt-proxy
    ```

3. Edit the file *`/etc/dnscrypt-proxy/dnscrypt-proxy.toml`*:

    Change the *`listen_addresses`* propertie to:
    ```bash
    listen_addresses = ['127.0.0.1:53', '[::1]:53']
    ```

4. Check if it's something running on port 53:
    ```bash
    sudo netstat -upln|grep ":53 "		# for UDP
    sudo netstat -tpln|grep ":53 "		# for TCP
    ```

5. Disable the *`systemd-resolved`*:
    ```bash
    sudo systemctl stop systemd-resolved
    sudo systemctl disable systemd-resolved
    ```
    Could be need to uninstall *`resolvconf`*:
    ```bash
    sudo apt remove resolvconf
    ```
    If something goes wrong, it can be reactivated with:
    ```bash
    sudo systemctl enable systemd-resolved
    sudo systemctl start systemd-resolved
    ```
    And you could reinstall *`resolvconf`*:
    ```bash
    sudo apt install resolvconf
    ```

6. Restart the machine.

7. After the boot, check if the system has the directory *`/etc/resolvconf`*:
    ```bash
    ls /etc/resolvconf
    ```

8. Case it exists, rename it:
    ```bash
    sudo mv /etc/resolvconf /etc/resolvconf.bk
    ```

9.  Rename the file *`/etc/resolv.conf`* also:
    ```bash
    sudo mv /etc/resolv.conf /etc/resolv.conf.bk
    ```

10. Create a new *`/etc/resolv.conf`* file:
    ```bash
    sudo touch /etc/resolv.conf
    ```

11. Edit the just created *`/etc/resolv.conf`* file with your favorite editor. It must contain only this 2 lines:
    ```bash
    nameserver 127.0.0.1
    options edns0 single-request-reopen
    ```
12. For now, will be necessary to rewrite the *`/etc/resolv.conf`* manualy every time that the conection restarts, including on boot.

13. On *`Ubuntu System Settings`* go to your network configuration (the engine button). In the new window that opens, go to the *`IPV4`* tab. Uncheck *`Automatic DNS`* and insert the address *`127.0.0.1`*.

14. For now, you'll need to start the *`dnscrypt-proxy`* manualy. On a terminal:

    ```bash
    sudo dnscrypt-proxy -config /etc/dnscrypt-proxy/dnscrypt-proxy.toml
    ```

15. You'll need to let it running on this terminal. Open another terminal (or another tab) and run:
     ```bash
    sudo dnscrypt-proxy -resolve google.com
    ```

16. If *`dnscrypt-proxy`* could resolve the Google address, we're good.

17. Also, you can verify with *`nslookup`*:
    ```bash
    nslookup google.com
    ```
    It should return something like:
    ```bash
    Server: 127.0.0.1
    Address: 127.0.0.1#53
    ...
    ```

18. On a terminal, install and start the *`dnscrypt-proxy`* service:
    ```bash
    sudo dnscrypt-proxy -service install -config /etc/dnscrypt-proxy/dnscrypt-proxy.toml
    ```
    and
    ```bash
    sudo dnscrypt-proxy -service start -config /etc/dnscrypt-proxy/dnscrypt-proxy.toml
    ```

19. Restart your machine and search for the *`dnscrypt-proxy`* process after boot:
    ```bash
    ps -aux | grep dnscrypt-proxy
    ```

20. I've made an alias on *`~/.profile`* to rewrite the *`/etc/resolv.conf`*:
    ```bash
    # ~/.profile
    alias almsdnscpcon="sudo sed -i 's/127.0.0.53/127.0.0.1\noptions edns0 single-request-reopen/g' /etc/resolv.conf"
    ```
    This way, I can call the alias from the terminal, to make the alteration on the *`/etc/resolv.conf`*, just typing:
    ```bash
    almsdnscpcon
    ```

<br><br>
### Important
You'll need to edit the file *`/etc/resolv.conf`* (step 20) on each boot...
