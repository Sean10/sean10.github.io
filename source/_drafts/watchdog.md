今天偶然在我的fedora跳板机上查man时，看到fedora的systemd.service里支持了watchdog的功能支持，所以看起来像是systemd把超级狗的系统功能也逐渐增加了集成，只有一句话我，牛B

根据目前了解到的情况来看，超级狗的功能来自内核

As of version 4.0 watchdog is able to make itself a real-time application.
It will lock all its pages in memory and set it�s scheduler to round-robin.