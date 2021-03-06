## Look Ma, no OS! Interactive Demo
This repository contains the demo material for my talk entitled:

**Look Ma, no OS! Unikernels and Their Applications**

The slides can be found here:

http://slides.com/technolo-g/look-ma-no-os-unikernels-and-their-applications

There 4 very simple demos:
- MirageOS (with Jitsu)
- Rumprun
- Runtime JS
- LING

## Initial setup
These demos are based on Vagrant. They should work with both VMware
Fusion 7 and VirtualBox (even on Windows!). The only real prereq is to
have Vagrant installed. If you would like to mount the directory as a
shared NFS mount in the VM just enable NFS on your system and uncomment
the shared folder line.

Once you have Vagrant + NFS installed run the following command to bring
up the machine and SSH to it (X11 fowarding is enabled):
```
vagrant up
vagrant ssh
```

**Note:** VirtualBox is dog slow at this point in time. All of the
instructions clone the repos to the local filesystem within the virtual
machine, but even still when running in VirtualBox please have patience.
If it's too slow then just purchase VMware Fusion. You won't regret that
move if you are a frequent Vagrant user as it is much smoother and
quicker.

## MirageOS
The first and coolest demo is of a static site running on MirageOS; summoned just in time.
The way it works is that we compile and package our unikernel as a Xen VM. Then we will
fire up a DNS daemon based on Jitsu (https://github.com/mirage/jitsu). The way Jitsu works is:
- A user attempts to resolve a domain against the daemon (www.example.org)
- Jitsu checks Xen to see if the configured unikernel is running
- If it is running, the daemon returns the A record and updates its TTL on the unikernel
- If it is not running, the daemon will start the unikernel and then return the A record
- After 2x the configured TTL for the A record passes, the unikernel is suspended (or deleted in our example)
- Times for the initial request are about .5 seconds which includes recieving the DNS request, starting the unikernel, and responding with a record
- Subsequent requests take approximate .02 seconds as the unikernel is already running

The site it is serving is based on the output of Jekyll (https://jekyllrb.com/). We just take those static files
and bundle them up with an OCaml based webserver in order to serve the static assets. In order to get this demo running,
please follow these steps:

```
# Running from the shared directory is slow on VMware and
# terrible on VirtualBox so we'll clone it to the local fs.
git clone https://github.com/technolo-g/lookma
cd lookma/mirageos/scripts

# Prepare the system
./0_prepare.sh

# Activate Opam
eval `opam config env`

# Build our unikernel
./1_build-unikernel.sh

# Start the Jitsu DNS server
./2_jitsu.sh

# Show that there are no VMs running
sudo xl list

# Test it out (in another terminal)!
dig @127.0.0.1 jitsu.unikornel.com 
sudo xl list

# Browse to the site
open http://mirageos.unikornel.com # Not Jitsu
# or
open http://jitsu.unikornel.com
```


## Rumprun
In order to demonstrate the Rumprun unikernel, we will stand up a very basic WordPress stack. This involves:
- Preparing the system
- Building and starting the MySQL unikernel
- Building and starting the PHP unikernel
- Building and starting the NGINX unikernel
- Running through the WordPress install

**Note:** MySQL is not tuned and is very, very slow at this
time. I beleive this is just our implementation. Even though the install
times out, it does actually complete and the site will load afterwards.

In order to get this demo going, please perform the following steps:
```
# Running from the shared directory is slow on VMware and
# terrible on VirtualBox so we'll clone it to the local fs.
git clone https://github.com/technolo-g/lookma
cd lookma/rumpkernel/scripts
./0_prepare.sh

# Fire up MySQL
./1_mysql.sh

# Fire up PHP
./2_php.sh

# Fire up NGINX
./3_nginx.sh
```

You may now browse to `http://rumprun.unikornel.com` and complete the WordPress installation. Please note that it does go pretty slow right now.

## Runtime JS
There are two demos in the `runtimejs` directory:
- hello-world: This is a very basic demo that will build a unikernel that outputs the text 'Hello StrangeLoop!' on the console out.
- webserver: This is a simple webserver written in JS that responds to all requests on port 9000 with a 200 response code and the text 'Hello StrangeLoop!'

In order to build and run the demos run the following commands (from within the Vagrant box):
```
# Running from the shared directory is slow on VMware and
# terrible on VirtualBox so we'll clone it to the local fs.
git clone https://github.com/technolo-g/lookma
cd lookma/runtimejs/scripts
./0_prepare.sh

# Run the hello-world demo:
cd ~/lookma/runtimejs/scripts
./demo-helloworld.sh
cd ../hello-world && npm start

# Run the webserver demo:
cd ~/lookma/runtimejs/scripts
./demo-webserver.sh
cd ../webserver && npm start
```

## LING (Erlang on Xen)

The LING demo is a basic webserver serving static content via an
Erlang/OTP application. In this demo we compile the application and
it's dependencies with `rebar` (the binary is included in this repo) and
then package the tool using `railing` (also included here). The
unikernel is then run on Xen and you are dropped into the Erlang shell
on the system with the application started.

Getting this demo running is as simple as the rest:
```
# Running from the shared directory is slow on VMware and
# terrible on VirtualBox so we'll clone it to the local fs.
git clone https://github.com/technolo-g/lookma
cd lookma/ling/scripts

# The machine has everything needed so no setup!
# Build the unikernel
./1_build.sh

# Run the unikernel
./2_run.sh

# See the results!
open http://ling.unikornel.com
```

## Links used to create these demos:
### MirageOS
- From Jekyll to Unikernel in 50 Lines: http://amirchaudhry.com/from-jekyll-to-unikernel-in-fifty-lines/
- Just In Time Summoning of Unikernels: https://github.com/mirage/jitsu
- My First Unikernel: http://roscidus.com/blog/blog/2014/07/28/my-first-unikernel/
- Mirage Skeleton Repo: https://github.com/mirage/mirage-skeleton

### Rumprun
- Tutorial: https://github.com/rumpkernel/wiki/wiki/Tutorial%3A-Serve-a-static-website-as-a-Unikernel
- Rumprun Packages: https://github.com/rumpkernel/rumprun-packages

### RuntimeJS
- RuntimeJS Homepage: http://runtimejs.org/

