# How to collect sosrpeort in Red Hat OpenShift nodes

The node needs capability to get registry.redhat.io/rhel8/support-tools image: 

~~~
$ ssh core@<node_ip>
$ sudo su -
# toolbox 
Trying to pull registry.redhat.io/rhel8/support-tools:latest...
Getting image source signatures
Checking if image destination supports signatures
Copying blob 8671113e1c57 done  
Copying blob 5dcbdc60ea6b done  
Copying blob b53695c4f6bf done  
Copying config a77781e294 done  
Writing manifest to image destination
Storing signatures
a77781e294f6fa21c7f472d601cf61faf557b21e264658b16285db0e708d5385
Spawning a container 'toolbox-root' with image 'registry.redhat.io/rhel8/support-tools'
Detected RUN label in the container image. Using that as the default...

# sosreport -k crio.all=on -k crio.logs=on
...
Your sosreport has been generated and saved in:
	/host/var/tmp/sosreport-ostest-master-0-2022-03-03-mpnouxr.tar.xz

 Size	57.17MiB
 Owner	root
 sha256	04c78bb6b30f47c9926fda9e14a6d08fee99eb2f304fa4b4296964e6b78cbf0d
~~~

Then logout from the `toolbox` and send the sosrpeort to your workstation/laptop:
~~~
# exit
# scp /var/tmp/sosreport-ostest-master-0-2022-03-03-mpnouxr.tar.xz root@IP-ADDR:~/
~~~
