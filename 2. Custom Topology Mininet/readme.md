# Membuat Custom Topology pada Mininet
## Topology Mininet Dengan 2 Host dan 2 Switch
1. Masuk pada instance EC2 yang telah terinstall Mininet.
```sh
ssh -i .ssh/labsuser.pem ubuntu@ip_instance
```
2. Membuat file `custom_topo_2sw2h.py` dengan perintah `nano` untuk membuat custom
topologi dengan 2 switch dan 2 host. Isikan dengan kode berikut.
```sh
#!/usr/bin/env python

"""Custom topology example

Two directly connected switches plus a host for each switch:

   host --- switch --- switch --- host

Adding the 'topos' dict with a key/value pair to generate our newly defined
topology enables one to pass in '--topo=mytopo' from the command line.
"""

from mininet.topo import Topo
from mininet.log import setLogLevel, info

class MyTopo( Topo ):

    def addSwitch(self, name, **opts ):
        kwargs = { 'protocols' : 'OpenFlow13'}
        kwargs.update( opts )
        return super(MyTopo, self).addSwitch( name, **kwargs )

    def __init__( self ):
        "Create MyTopo topology..."
        
        # Inisialisasi Topology
        Topo.__init__( self )

        # Tambahkan node, switch, dan host
        info( '*** Add switches\n')
        s1 = self.addSwitch('s1')
        s2 = self.addSwitch('s2')

        info( '*** Add hosts\n')
        h1 = self.addHost('h1', ip='10.1.0.1/24')
        h2 = self.addHost('h2', ip='10.1.0.2/24')
     
        info( '*** Add links\n')
        self.addLink(s1, h1, port1=1, port2=1)
        self.addLink(s1, s2, port1=2, port2=1)
        self.addLink(s2, h2, port1=2, port2=1)

topos = { 'mytopo': ( lambda: MyTopo() ) }
```
3. Menjalankan mininet tanpa controller menggunakan custom topology yang telah dibuat.
```sh
sudo mn --controller=none --custom custom_topo_2sw2h.py --topo mytopo --mac --arp
```
4. Membuat alur flow agar h1 dapat terhubung dengan h2.
```sh
mininet> sh ovs-ofctl add-flow s1 -O OpenFlow13 "in_port=1,action=output:2"
mininet> sh ovs-ofctl add-flow s1 -O OpenFlow13 "in_port=2,action=output:1"
mininet> sh ovs-ofctl add-flow s2 -O OpenFlow13 "in_port=1,action=output:2"
mininet> sh ovs-ofctl add-flow s2 -O OpenFlow13 "in_port=2,action=output:1"
```
5. Mengecek apakah setiap host sudah saling terkoneksi menggunakan perintah
`pinggall` pada console mininet.
```sh
mininet> pingall
```
6. Menguji koneksi h1 dengan h2.
```sh
mininet> h1 ping -c2 h2
```
## Tolology Mininet Loop Dengan 3 switch dan 6 host (Manual Flow)
1. Masuk pada instance EC2 yang telah terinstall Mininet.
```sh
ssh -i .ssh/labsuser.pem ubuntu@ip_instance
```
2. Membuat file `custom_topo.py` dengan perintah `nano` untuk membuat custom
topologi dengan 3 switch dan 6 host (loop). Isikan dengan kode berikut.
```sh
#!/usr/bin/env python

"""Custom topology example

Two directly connected switches plus a host for each switch:

   host --- switch --- switch --- host

Adding the 'topos' dict with a key/value pair to generate our newly defined
topology enables one to pass in '--topo=mytopo' from the command line.
"""

from mininet.topo import Topo
from mininet.log import setLogLevel, info

class MyTopo( Topo ):

    def addSwitch(self, name, **opts ):
        kwargs = { 'protocols' : 'OpenFlow13'}
        kwargs.update( opts )
        return super(MyTopo, self).addSwitch( name, **kwargs )

    def __init__( self ):
        "Create MyTopo topology..."
        
        # Inisialisasi Topology
        Topo.__init__( self )

        # Tambahkan node, switch, dan host
        info( '*** Add switches\n')
        s1 = self.addSwitch('s1')
        s2 = self.addSwitch('s2')
        s3 = self.addSwitch('s3')

        info( '*** Add hosts\n')
        h1 = self.addHost('h1', ip='10.1.0.1')
        h2 = self.addHost('h2', ip='10.1.0.2')
        h3 = self.addHost('h3', ip='10.2.0.3')
        h4 = self.addHost('h4', ip='10.2.0.4')
        h5 = self.addHost('h5', ip='10.3.0.5')
        h6 = self.addHost('h6', ip='10.3.0.6')
        
        info( '*** Add links\n')
        self.addLink(s1, h1, port1=2, port2=1)
        self.addLink(s1, h2, port1=3, port2=1)
        self.addLink(s1, s2, port1=4, port2=2)
        self.addLink(s1, s3, port1=1, port2=3)
        self.addLink(s2, h3, port1=3, port2=1)
        self.addLink(s2, h4, port1=4, port2=1)
        self.addLink(s2, s3, port1=1, port2=4)
        self.addLink(s3, h5, port1=2, port2=1)
        self.addLink(s3, h6, port1=1, port2=1)

topos = { 'mytopo': ( lambda: MyTopo() ) }
```
3. Menjalankan mininet tanpa controller menggunakan custom topology yang telah dibuat.
```sh
sudo mn --controller=none --custom custom_topo.py --topo mytopo --mac --arp
```
4. Membuat alur flow agar agar setiap host dapat saling terkoneksi satu
sama lain.
```sh
sh ovs-ofctl add-flow s1 -O OpenFlow13 "in_port=1,action=output:2,3"
sh ovs-ofctl add-flow s1 -O OpenFlow13 "in_port=2,action=output:1,3,4"
sh ovs-ofctl add-flow s1 -O OpenFlow13 "in_port=3,action=output:1,2,4"
sh ovs-ofctl add-flow s1 -O OpenFlow13 "in_port=4,action=output:2,3"
sh ovs-ofctl add-flow s2 -O OpenFlow13 "in_port=1,action=output:3,4"
sh ovs-ofctl add-flow s2 -O OpenFlow13 "in_port=2,action=output:3,4"
sh ovs-ofctl add-flow s2 -O OpenFlow13 "in_port=3,action=output:1,2,4"
sh ovs-ofctl add-flow s2 -O OpenFlow13 "in_port=4,action=output:1,2,3"
sh ovs-ofctl add-flow s3 -O OpenFlow13 "in_port=1,action=output:2,3,4"
sh ovs-ofctl add-flow s3 -O OpenFlow13 "in_port=2,action=output:1,3,4"
sh ovs-ofctl add-flow s3 -O OpenFlow13 "in_port=3,action=output:1,2"
sh ovs-ofctl add-flow s3 -O OpenFlow13 "in_port=4,action=output:1,2"
```
5. Mengecek apakah setiap host sudah saling terkoneksi menggunakan perintah
`pinggall` pada console mininet.
```sh
mininet> pingall
```
6. Menguji koneksi h1 dengan h3.
```sh
mininet> h1 ping -c2 h3
```
