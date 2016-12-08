## Usage

* List current firewall rules and stop firewall
```
sudo iptables -L -n
```
* You can save existing firewall rules as follows:
```
sudo iptables-save > firewall.rules
```
* Finally, type the following commands to stop firewall and flush all the rules:
```
sudo iptables -F
sudo iptables -X
sudo iptables -t nat -F
sudo iptables -t nat -X
sudo iptables -t mangle -F
sudo iptables -t mangle -X
sudo iptables -P INPUT ACCEPT
sudo iptables -P FORWARD ACCEPT
sudo iptables -P OUTPUT ACCEPT
```


### issues

* iptables: Too many links. 

when `iptables -X` show  _iptables: Too many links._
```
iptables -F
iptables -X
```

### links

> http://blog.chinaunix.net/uid-26495963-id-3279216.html
> http://www.howtogeek.com/177621/the-beginners-guide-to-iptables-the-linux-firewall/
> http://19001989.blog.51cto.com/3447586/696540
> http://ipset.netfilter.org/iptables.man.html

