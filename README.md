*This project has been created as part of the 42 curriculum by jbarbay, agan, yliew.*

# Webserv

## Description

Webserv is a fully custom **HTTP/1.1 server written from scratch in C++98** to replicate Nginx. The goal of the project is to understand, at a low level, how a web server works: parsing raw HTTP requests, managing client connections, serving static content, handling file uploads, running CGI scripts, and returning accurate HTTP responses—all while staying non-blocking and crash-free under any condition.

The server:
- Is configured through **Nginx-inspired configuration file**, allowing multiple virtual ervers, each listening on its own `host:port`.
- Serves a **fully static website**, including directory listing (autoindex), default index files, and custom redirections.
- Supports the **GET, POST and DELETE** HTTP methods.
- Allows **clients to upload files** to a configured storage location.
- Executes **CGI scripts** (Python, PHP) based on file extension, including support for chunked request bodies and CGI output without `Content-Length`.
- Provides **default error pages** when none are configured.
- Relies on a **single `epoll()` loop** to multiplex all client and listening sockets, ensuring the server never performs a blocking read/write outside of an I/O readiness notification.

## Instructions

### Compilation

```sh
make        # build the webserv executable
make clean  # remove object files
make fclean # remove object files and the executable
make re     # rebuild from scratch
```

The project is compiled with `c++` using the `-Wall -Wextra -Werror -std=c++98` flags.

### Usage

```sh
./webserv [path/to/config_file]
```

If no configuration file is given, a default configuration is used.

Sample configuration files are provided in [config_files/](config_files/):

- [config_files/default.conf](config_files/default.conf): a single-purpose configuration demonstrating static file serving, directory listing, redirections, file uploads, file deletion, and CGI execution (Python and PHP).
- [config_files/multiple_servers.conf](config_files/multiple_servers.conf): multiple virtual servers sharing the same or different `host:port` pairs.

Example configuration block:

```nginx
server
{
	listen 8080;
	host 127.0.0.1;
	server_name example.com;
	body_max_length 1024;

	root wwwroot/;
	index index.html;
	error_page 400=errors/400.html 404=errors/404.html;

	location /
	{
		root wwwroot/;
		autoindex on;
		allowed_methods POST GET;
		index index.html;
	}

	location /cgi-bin/
	{
		cgi_ext .py .php;
		cgi_exec /usr/bin/python3 /usr/bin/php-cgi;
	}
}
```

Within each `location` block you can configure: allowed HTTP methods, redirections, the
root directory the URL is mapped to, directory listing (autoindex), the default index file,
file upload destination, and CGI extension/interpreter mapping.

### Testing

The server was tested with:
- A standard web browser (navigating, submitting forms, uploading and deleting files).
- `telnet` / `curl`, to inspect raw requests and responses.
- `Nginx`, to compare headers and behaviour for equivalent requests.
- Custom scripts to stress test the server and verify it keeps running under load and
  with malformed requests.
