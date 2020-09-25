# Carbon v2
How to set up a HTTPS forward proxy server on a linux server.

Install Go 1.14.9, copy the download link from https://golang.org/dl/

For Raspberry Pi: `wget https://golang.org/dl/go1.14.9.linux-armv6l.tar.gz`
Then extract it: `sudo tar -C /usr/local -xzf go1.14.9.linux-armv6l.tar.gz`
Then remove it: `rm go1.14.9.linux-armv6l.tar.gz`

Edit `/etc/profile` and add `PATH=$PATH:/usr/local/go/bin`

Then run `source /etc/profile`

Then run `apt install git` to install Git on the server

Then in the home directory create a folder called `cdy` and make a new file called `main.go` with the contents:

    package main
    
    import (
    	"github.com/caddyserver/caddy/caddy/caddymain"
    	 _ "github.com/caddyserver/forwardproxy"
    )
    
    func main() {
    	caddymain.EnableTelemetry = false
    	caddymain.Run()
    }

Then run 
`go mod init caddy` 
`go get github.com/caddyserver/caddy` 
`go build`
`cp caddy /usr/local/bin`
`sudo setcap cap_net_bind_service=+ep ./caddy`
`sudo ulimit -n 8192`

Create a file called `caddy.service` with the contents:

    [Unit]
    Description=Caddy HTTP/2 web server
    Documentation=https://caddyserver.com/docs
    After=network-online.target
    Wants=network-online.target systemd-networkd-wait-online.service
    
    [Service]
    Restart=on-abnormal
    LimitNOFILE=16384
    
    EnvironmentFile=/etc/default/caddy
    
    ExecStart=/usr/local/bin/caddy -log stdout -agree=true -conf=${CONFIG_FILE} -root=${ROOT_DIR}
    ExecReload=/bin/kill -USR1 $MAINPID
    
    KillMode=mixed
    KillSignal=SIGQUIT
    TimeoutStopSec=5s
    
    [Install]
    WantedBy=multi-user.target

Create the file `/etc/default/caddy` with the contents:

    CONFIG_FILE="/etc/caddy/caddyfile"
    ROOT_DIR="/var/www"

Create the directory `mkdir /etc/caddy` and also create the file `/etc/caddy/caddyfile` with the contents:

    domain.net {
	    tls you@email.com
	    root /var/www
	    forwardproxy {
		    hide_ip
		    hide_via
		}
	}

Create `mkdir /var/www`

Move the service file `cp caddy.service /etc/systemd/system/caddy.service`

Start, enable and check the status of the service with these commands:
`sudo systemctl start caddy.service`
`sudo systemctl enable caddy.service`
`sudo systemctl status caddy.service`

Then we're done.
