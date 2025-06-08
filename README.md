ESC/POS virtual network printer 
*---------

This is a container-based ESC/POS network printer, that replaces paper rolls with HTML pages and a web interface.

The printer emulates a 80mm roll of paper.

![sample print](https://github.com/gilbertfl/escpos-netprinter/assets/83510612/8aefc8c5-01ab-45f3-a992-e2850bef70f6)

## Limits
This docker image is not to be exposed on a public network (see [known issues](#known-issues))

The print timeout is configurable via the `ESCPOS_TIMEOUT` environment variable (default is 10 seconds).

## Quick start

This project requires:
- A Docker installation (kubernetes should work, but is untested.)

To install v3.0 from source:

```bash
wget --show-progress https://github.com/gilbertfl/escpos-netprinter/archive/refs/tags/3.0.zip
unzip 3.0.zip 
cd escpos-netprinter-3.0
docker build -t escpos-netprinter:3.0 .
```

To run the resulting container:
```bash
docker run -d --rm --name escpos_netprinter -p 515:515/tcp -p 8080:80/tcp -p 9100:9100/tcp escpos-netprinter:3.0
```
It should now accept prints by JetDirect on the default port(9100) and by lpd on the default port(515), and you can visualize it with the web application at port 8080.  
For debugging, you can add port 631 to access the CUPS interface. The CUPS administrative username is `cupsadmin` and the password is `123456`.

## Using Docker Compose

A Docker Compose file is provided for easier deployment:

```bash
docker-compose up -d
```

The Docker Compose configuration:
- Maps the web interface to port 8080 on your host
- Sets a configurable timeout (default: 60 seconds)
- Creates persistent volumes for receipts and temporary files
- Automatically creates required directories if they don't exist

### Directory Structure

The Docker Compose setup creates and manages the following directories inside the container:
- `/app/web/receipts`: Stores the HTML receipts (persisted in a Docker volume)
- `/app/web/tmp`: Stores temporary files during processing (persisted in a Docker volume)

These directories are automatically created when the container starts, and their contents are preserved across container restarts through Docker volumes.

## Known issues
While version 3.0 is no longer a beta version, it has known defects:
- It still uses the Flask development server, so it is unsafe for public networks.
- While it works with simple drivers, for example the one for the MUNBYN ITPP047 printers, the [Epson utilities](https://download.epson-biz.com/modules/pos/) refuse to speak to it.

## Configuration Options

The following environment variables can be configured:

| Variable | Default | Description |
|----------|---------|-------------|
| ESCPOS_TIMEOUT | 10 | Timeout in seconds for print jobs |
| ESCPOS_DEBUG | false | Enable debug mode for more detailed logs |
| FLASK_RUN_DEBUG | false | Enable Flask debug mode |
| PRINTER_PORT | 9100 | JetDirect port for printer communication |