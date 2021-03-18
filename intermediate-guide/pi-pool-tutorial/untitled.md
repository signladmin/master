# Create .img file

Put your micro sd card in your local machine and locate what it's called in /dev. For my laptop it is /dev/mmcblk0. Yours will most likely be different.

After locating move into the directory you wish to save the image to and create the image.

```text
sudo cat /dev/<your sd card> > pi-node.img
```

 cat is better than dd for this. cat will use all of your systems cpu cores, whereas dd uses one core. cat is faster 🙀

Once that completes we will use [PiShrink.sh](https://github.com/Drewsif/PiShrink) to deflate partitions and compress \(among a few other tricks\).

```text
install pishrinks.shwget https://raw.githubusercontent.com/Drewsif/PiShrink/master/pishrink.shchmod +x pishrink.shsudo mv pishrink.sh /usr/local/bin
```

```text
sudo pishrink.sh -az pi-node.img Pi-Node.img.gz
```

> pishrink.sh: Shrunk Pi-Node.img.gz from 7.5G to 1.3G ...

And there you have it! 🧙♂

Download [Pi-Node.img.gz](https://db.adamantium.online/Pi-Node.img.gz)​

