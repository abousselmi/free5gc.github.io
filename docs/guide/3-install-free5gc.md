# Installation

## A. Prerequisites

1. Linux Kernel Version
    * In order to use the UPF element, you must use the `5.0.0-23-generic` or `5.4.x` version of the Linux kernel.  free5gc uses the [gtp5g kernel module](https://github.com/free5gc/gtp5g), which has been tested and compiled against that kernel versions only. If you installed Ubuntu 20.04, the version looks like 5.4.x. To determine the version of the Linux kernel you are using:

    ```bash
        $ uname -r
        5.4.0-65-generic
    ```

You will not be able to run most of the tests in [Test](https://github.com/free5gc/free5gc/wiki/Test) section unless you deploy a UPF.

2. Golang Version
    * As noted above, free5gc is built and tested with Go 1.17.8
    * To check the version of Go on your system, from a command prompt:

    ```bash
        go version
    ```

    * If another version of Go is installed, remove the existing version and install Go 1.17.8:

    ```bash
        # this assumes your current version of Go is in the default location
        sudo rm -rf /usr/local/go
        wget https://dl.google.com/go/go1.17.8.linux-amd64.tar.gz
        sudo tar -C /usr/local -zxvf go1.17.8.linux-amd64.tar.gz
    ```

    * If Go is not installed on your system:

    ```bash
        wget https://dl.google.com/go/go1.17.8.linux-amd64.tar.gz
        sudo tar -C /usr/local -zxvf go1.17.8.linux-amd64.tar.gz
        mkdir -p ~/go/{bin,pkg,src}
        # The following assume that your shell is bash
        echo 'export GOPATH=$HOME/go' >> ~/.bashrc
        echo 'export GOROOT=/usr/local/go' >> ~/.bashrc
        echo 'export PATH=$PATH:$GOPATH/bin:$GOROOT/bin' >> ~/.bashrc
        echo 'export GO111MODULE=auto' >> ~/.bashrc
        source ~/.bashrc
    ```

    * Further information and installation instructions for `golang` are available at the [official golang site](https://golang.org/doc/install).

3. Control-plane Supporting Packages

```bash
sudo apt -y update
sudo apt -y install mongodb wget git
sudo systemctl start mongodb
```

* **WARNING: MongoDB 5.0+ requires a CPU with AVX support. Or downgrade your MongoDB to 4.4**

    * see https://www.mongodb.com/community/forums/t/mongodb-5-0-cpu-intel-g4650-compatibility/116610/2

    * see also [docker-library/mongo#485 (comment)](https://github.com/docker-library/mongo/issues/485#issuecomment-891991814)

4. User-plane Supporting Packages

```bash
sudo apt -y update
sudo apt -y install git gcc g++ cmake autoconf libtool pkg-config libmnl-dev libyaml-dev
```

5. Linux Host Network Settings

```bash
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -o <dn_interface> -j MASQUERADE
sudo iptables -A FORWARD -p tcp -m tcp --tcp-flags SYN,RST SYN -j TCPMSS --set-mss 1400
sudo systemctl stop ufw
```

## B. Install Control Plane Elements

1. Clone the free5GC repository
    * To install the latest stable build (v3.2.1):

    ```bash
        cd ~
        git clone --recursive -b v3.2.1 -j `nproc` https://github.com/free5gc/free5gc.git
        cd free5gc
    ```

    * **(Alternatively)** to install the latest nightly build:

    ```bash
        cd ~/free5gc
        git checkout main
        git submodule sync
        git submodule update --init --jobs `nproc`
        git submodule foreach git checkout main
        git submodule foreach git pull --jobs `nproc`
    ```

2. Compile network function services in `free5gc`
    * To do so individually (e.g., AMF only):

    ```bash
        cd ~/free5gc
        make amf
    ```

    * To build all network functions:

    ```bash
        cd ~/free5gc
        make
    ```

## C. Install User Plane Function (UPF)

1. As noted above, the GTP kernel module used by the UPF requires that you use Linux kernel version `5.0.0-23-generic` or `5.4.x`.  To verify your version:

```bash
uname -r
```

2. Retrieve the 5G GTP-U kernel module using `git` and build it

```bash
git clone -b v0.6.8 https://github.com/free5gc/gtp5g.git
cd gtp5g
make
sudo make install
```

3. Build the UPF (you may skip this step if you built all network functions above):

   * to build using make:

   ```bash
   cd ~/free5gc
   make upf
   ```

4. Customize the UPF as desired. The UPF configuration file in `run.sh` is `free5gc/config/upfcfg.yaml`.

## D. Install WebConsole

1. Before building WebConsole, install nodejs and yarn packages first:

```bash
sudo apt remove cmdtest
sudo apt remove yarn
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get update
sudo apt-get install -y nodejs yarn
```

2. Build WebConsole

   * to build using make:

   ```bash
   cd ~/free5gc
   make webconsole
   ```

   * **(Alternatively)** to build manually:

   ```bash
   cd ~/free5gc/webconsole/frontend
   yarn install
   yarn build
   rm -rf ../public
   cp -R build ../public
   cd ..
   go build -o bin/webconsole server.go
   ```

   **Note: 2GB or more of OS memory is recommended. WebConsole may be failed to build if memory is less then 1GB.**
