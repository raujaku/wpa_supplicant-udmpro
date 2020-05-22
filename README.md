# wpa_supplicant for UDM Pro
## For 1.7.*
 
### This is just a collection of everyones ideas and code which makes WPA_SUPPLICANT work on 1.7.*
 
*** FROM pbrah***

overview
This guide has primarily been written for authenticating to AT&T U-Verse using wpa_supplicant on a UDM Pro.  It is assumed that you've already retrieved your certificates from a modem supplied by AT&T.  If you have not, you can purchase a used modem on ebay, such as the NVG589 and then root it to get the certificates.  I had success using the following guide.

https://github.com/bypassrg/att

If the above link no longer works, I have also forked it to my GitHub below.

https://github.com/pbrah/att
*** 



# 1. decode credentials
### Credit: devicelocksmith

Download decoder v1.0.4: win, linux, mac

[Decoder](https://www.devicelocksmith.com/2018/12/eap-tls-credentials-decoder-for-nvg-and.html)


* Copy mfg.dat, attroot****.der,& attsubca****.der files to the same location as mfg_dat_decode.

* Run mfg_dat_decode. 

* You should get a file like this: EAP-TLS_8021x_****.tar.gz

* unzip the tarball and you should get a file like this: EAP-TLS_8021x_****.tar

* unzip the .tar file and you should get your .perm files and your wpa_supplicant.conf file




# 2.  scp your certs and wpa_supplicant.conf to the UDM Pro
### Credit: pbrah
```
$ scp -r *.pem root@192.168.1.1:/tmp/

    password:(YOUR PASSWORD)

    CA_001E46-****.pem                                                          100% 3926     3.8KB/s   00:00

    Client_001E46-****.pem                                                      100% 1119     1.1KB/s   00:00

    PrivateKey_PKCS1_001E46-****.pem                                            100%  887     0.9KB/s   00:00

$ scp -r wpa_supplicant.conf root@192.168.1.1:/tmp/

    wpa_supplicant.conf                                                         100%  680     0.7KB/s   00:00
```


# 3. ssh to the UDM Pro
### Credit: fryjr82 & pbrah

create a directory for the certs and wpa_supplicant.conf in the podman directory then copy the files over.

```
$ mkdir /mnt/data/podman/wpa_supplicant/

$ cp -arfv /tmp/*pem /tmp/wpa_supplicant.conf /mnt/data/podman/wpa_supplicant/
```


# 4. update the wpa_supplicant.conf files
### Credit: fryjr82 & pbrah

Do not run these more than once or you will end up with incorrect paths.

```
$ sed -i 's,ca_cert=",ca_cert="/etc/wpa_supplicant/conf/,g' /mnt/data/podman/wpa_supplicant/wpa_supplicant.conf

$ sed -i 's,client_cert=",client_cert="/etc/wpa_supplicant/conf/,g' /mnt/data/podman/wpa_supplicant/wpa_supplicant.conf

$ sed -i 's,private_key=",private_key="/etc/wpa_supplicant/conf/,g' /mnt/data/podman/wpa_supplicant/wpa_supplicant.conf
```

# 5. run the wpa_supplicant podman container 
### Credit: fryjr82 & pbrah

the podman run command below assumes you are using port 9 WAN.  If not, adjust accordingly.

```
$ podman run --privileged=true --network=host --name=wpa_supplicant-udmpro -v /mnt/data/podman/wpa_supplicant/:/etc/wpa_supplicant/conf/ --log-driver=k8s-file --restart=on-failure -detach -ti pbrah/wpa_supplicant-udmpro:v1.0 -Dwired -ieth8 -c/etc/wpa_supplicant/conf/wpa_supplicant.conf
```

# 6. connecting & starting
### Credit: GiulianoM

* Connect the ONT to Port 9 (WAN) on your UDM Pro
* Power cycle ONT
* ssh to the UDM Pro & start the container

```
$ podman start wpa_supplicant-udmpro
```

* wait 

## *troubleshooting
### Credit: pbrah
If you are having issues connecting after starting your docker container, the first thing you should do is check your docker container logs.
```
$ docker logs -f wpa_supplicant-udmpro
```

### **notes
The container will have to be re-started each time you reboot your UDM pro

```
$ podman start wpa_supplicant-udmpro
````

