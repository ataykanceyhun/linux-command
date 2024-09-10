## IP Tables

> INSTALL

    sudo apt install iptables

> RETURN

    sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

> ADD

    sudo iptables -A INPUT -i eno1 -p tcp --dport 22 -s 10.100.22.0/26 -d 10.100.10.39 -j ACCEPT
    sudo iptables -A INPUT -i bond0 -p tcp --dport 22 -s 10.100.100.3 -d 10.100.100.2 -j ACCEPT

> SAVE

    sudo iptables-save | sudo tee /etc/iptables/rules.v4

> LIST

    sudo iptables -L

> LIST with Details

    sudo iptables -L -v

> LIST with Line Number

    sudo iptables -L --line-numbers


> DELETE with Line Number

    sudo iptables -D INPUT 4 / * Line Number */
