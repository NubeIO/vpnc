# Mirror VPN client

Nube-iO VPN client CLI for popular Linux distro and architecture. That support

- Auto setup and register VPN service on *Unix system
- Auto connect and lease VPN private IP
- Auto manage private DNS from VPN network

## Installation

Download the latest version in [vpnc/releases](https://github.com/NubeIO/vpnc/releases).

- `nubeio-vpnc.armv7.zip` for IoT device: RaspberryPi, BeagleBone, etc...
- `nubeio-vpnc.amd64.zip` for user computer: support Ubuntu 18/20, Debian 8/9/10

[dnsmasq](https://thekelleys.org.uk/dnsmasq/doc.html) is used as local DNS caching to resolve DNS from private VPN network. Currently, `dnsmasq` can be setup and auto download when installing `vpnclient` (only support `Ubuntu`/`Debian`/`Raspibian`/`Fedora`/`CentOS` in this time, if you are using another Linux distro, please install `dnsmasq` manually)

### Tips

For convenient, setup download tool [ghrd](https://github.com/zero88/gh-release-downloader) to quick download artifact by version on github.

This is setup script for Ubuntu/Debian distro
```bash
export GHRDVER=1.1.2 && sudo curl -L https://github.com/zero88/gh-release-downloader/releases/download/v$GHRDVER/ghrd -o /usr/local/bin/ghrd \
  && sudo chmod +x /usr/local/bin/ghrd \
  && sudo ln -sf /usr/local/bin/ghrd /usr/bin/ghrd \
  && sudo apt install jq -y \
  && unset GHRDVER
```

### Download and setup VPN client

This is a script that using `ghrd`.
You need to know VPN version and target system (to identify architecture edition) then subtitle to `VPNCVER` and `VPNCARCH` in a below script.

For example: `VPNCVER=v0.9.1` and `VPNCARCH=armv7` (currently, one of `armv7` and `amd64`)

```bash
export VPNCVER=v0.9.1 && export VPNARCH=armv7 \
   && ghrd -a ".*$VPNARCH.*" -x -r $VPNCVER -o /tmp nubeio/vpnc \
   && sudo mkdir -p /app \
   && sudo unzip -o -d /app /tmp/nubeio-vpnc*.zip \
   && ln -sf /app/nubeio-vpnc /usr/local/bin/nubeio-vpnc \
   && unset VPNCVER && unset VPNARCH
```

Verify:

```bash
$ nubeio-vpnc version
INFO : VPN version : v4.29-9680-rtm
INFO : CLI version : 0.9.1
INFO : Hash version: 2465e648
-------------------------------------------------------
```

## Usage

- `nubeio-vpnc -h` for more information

  ```bash
  $ nubeio-vpnc -h
  Usage: nubeio-vpnc [OPTIONS] COMMAND [ARGS]...

    CLI tool to install VPN Client and setup VPN connection

  Options:
    -h, --help  Show this message and exit.

  Commands:
    about        Show VPN software info
    add          Add new VPN Account
    connect      Connect to VPN connection by given VPN account
    delete       Delete one or many VPN account
    detail       Get detail VPN configuration and status by one or many accounts
    disconnect   Disconnect VPN connection
    install      Install VPN client and setup *nix service
    list         Get all VPN Accounts
    log          Get VPN log
    set-default  Set VPN default connection in startup by given VPN account
    status       Get current VPN status
    trust        Trust VPN Server cert
    uninstall    Stop and disable VPN client and *nix service
    upgrade      Upgrade VPN client
    version      VPN Version
  ```
  
- To interact with any command, please use `nubeio-vpnc <command> -h` to see help documentation. For example:

  ```bash
  nubeio-vpnc install -h
  Usage: nubeio-vpnc install [OPTIONS]

    Install VPN client and setup *nix service

  Options:
    --auto-startup            Enable auto-startup VPN service  [default: False]
    --auto-dnsmasq            Give a try to install dnsmasq  [default: False]
    --dnsmasq / --no-dnsmasq  By default, dnsmasq is used as local DNS cache. Disabled it if using default System DNS
                              resolver  [default: True]

    -dd, --vpn-dir TEXT       VPN installation directory  [default: ("/app/vpnclient" or from "env.VPN_HOME")]
    -dn, --service-name TEXT  VPN Service name  [default: nubeio-vpn]
    -ds, --service-dir TEXT   Linux Service directory
    -f, --force               If force is enabled, VPN service will be removed then reinstall without backup  [default:
                              False]

    -v, --verbose             Enables verbose mode
    -h, --help                Show this message and exit.
  ```

- To connect VPN server, you must provide
  - `VPN Host` HTTPS VPN server.
  - `VPN Port` Default is `443`.
  - `VPN Hub`  It is multi-tenant option, normally it is `customer` code.
  - `Authentication` A login credential to appropriate VPN host and VPN Hub. Credential type can be `password` or `cert`

- After connect VPN connection (See [#VPN workflow](#vpn-workflow)), VPN client will be registered as linux service, then you can manage it as normal linux service with some basic linux commands

  ```bash
  sudo systemctl status nubeio-vpn  # service status
  sudo systemctl restart nubeio-vpn # restart service
  sudo journctl -u nubeio-vpn
  ```

### VPN workflow

1. Normal workflow

```bash
# Setup VPN client service
$ nubeio-vpnc install # pass "--auto-dnsmasq" to try installing `dnsmasq` internally

# Add VPN account then open VPN session and start VPN service
$ nubeio-vpnc add <option for VPN connection> # use "-h" for more detail

# Show VPN status
$ nubeio-vpnc status

# Disconnect VPN connection and stop VPN service
$ nubeio-vpnc disconnect # pass "--disable" to not start VPN service when startup computer
```

2. For upgrade to new version, use:

```bash
$ nubeio-vpnc upgrade
```
**Note** It is hot reload regardless current state is in VPN session or not. Don't stop a script manually by `<Ctrl + C>` if don't want to break a network connection.

3. Uninstall VPN service, use:

```bash
# pass "-f" to remove completely vpnclient installation and data folder
# it still keep "dnsmasq" to resolve DNS for public domain. if want to restore computer network to origin state, pass "--no-keep-dnsmasq"
$ nubeio-vpnc uninstall
```

#### IoT device

- Must use `Client Certificate Authentication`
- Need `VPN device user`, `VPN device certificated` file, `VPN device private key` file
- 2 steps for quick install and setup:

  ```bash
  # Install VPN client and setup Linux service
  $ sudo nubeio-vpnc install
  # Add and connect to VPN account
  $ sudo nubeio-vpnc add -sh <vpn_server> -su <hub_name> -cd -ct cert -cu <vpn_device_user> -cck <vpn_device_certificated> -cpk <vpn_device_private_key>
  ```

- After that, please verify by commands:

  ```bash
  $ sudo nubeio-vpnc status

  INFO : VPN Application   : Installed - /app/vpnclient
  INFO : VPN Service       : qweio-vpn - active(running) - PID[4511]
  INFO : VPN Account       : cba - Connection Completed (Session Established)
  INFO : VPN IP address    : [{'addr': '10.0.0.6', 'netmask': '255.0.0.0', 'broadcast': '10.255.255.255'}]
  ```

#### User computer

- Use `Client Password Authentication`
- Need `VPN user`, `VPN password`, `VPN Customer hub` (a.k.a customer code, per hub per customer)
- If you manage cross VPN customer, then it's ideally to provide `VPN account` (VPN connection name) that equals to `VPN customer code`

  ```bash
  $ sudo nubeio-vpnc install

  # You can check log
  $ sudo nubeio-vpnc log -f

  # You can check status
  $ sudo nubeio-vpnc status

  # Put your password in `single quotes` 'your-password'
  $ sudo nubeio-vpnc add -sh <vpn_server> -su <customer_code_1> -ct password -cu <vpn_user> -cp <vpn_password>
  # You can add other VPN accounts
  # pass '-cd' option is make VPN client account is default for startup computer
  $ sudo nubeio-vpnc add -sh <vpn_server> -su <customer_code_n> -ct password -cu <vpn_user> -cp <vpn_password> -cd

  # Then you can switch among account by
  $ sudo nubeio-vpnc connect customer_code_n

  # To uninstall vpn service
  $ sudo nubeio-vpnc uninstall
  ```
