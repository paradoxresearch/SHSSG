# Configuring and Installing Nginx

## Contents

- [Configuring and Installing Nginx](#configuring-and-installing-nginx)
  - [Contents](#contents)
  - [Introduction](#introduction)
  - [Installation of Nginx](#installation-of-nginx)
    - [Update the Package Manager](#update-the-package-manager)
    - [Install Nginx](#install-nginx)
    - [Verify the Install](#verify-the-install)
  - [Basic Configuration of Nginx](#basic-configuration-of-nginx)
    - [Structure of a Basic Nginx Configuration File](#structure-of-a-basic-nginx-configuration-file)
    - [Example Configuration File](#example-configuration-file)
  - [Setting up Nginx as a Reverse Proxy](#setting-up-nginx-as-a-reverse-proxy)
    - [Configuring a Basic Reverse Proxy](#configuring-a-basic-reverse-proxy)
  - [Testing and Applying Configuration Changes](#testing-and-applying-configuration-changes)
    - [Test Configuration Changes](#test-configuration-changes)
    - [Apply Configuration Changes](#apply-configuration-changes)
  - [Important Nginx Concepts](#important-nginx-concepts)
    - [Virtual Hosts (Server Blocks)](#virtual-hosts-server-blocks)
    - [Document Root](#document-root)
    - [Load Balancing](#load-balancing)

## Introduction

**Nginx** is like a highly skilled Swiss Army knife for web-related tasks. Its primary job is to be a web server, which means it takes requests from web browsers (like Chrome or Firefox) and delivers the website content back to them. Think of it as the main post office for your website, handling all incoming and outgoing mail.

But Nginx can do much more! It can also act as a **reverse proxy**. Imagine a large company with several internal departments. Instead of customers contacting each department directly, they talk to a receptionist (the reverse proxy). The receptionist then forwards their requests to the correct internal department. Similarly, Nginx as a reverse proxy sits in front of your actual web applications, providing an extra layer of security and control.

Furthermore, Nginx can function as a **load balancer**. If our busy website is like a popular restaurant, a load balancer is like a maître d' who evenly distributes incoming customers among all the available tables (your server resources), preventing any single table from getting overwhelmed.

Finally, Nginx can also work as an **HTTP cache**, which is like having a memory of frequently requested website elements. If someone asks for the same information again, Nginx can quickly provide the cached version without needing to fetch it from the source again, making things much faster.

Individuals might use Nginx for their personal websites or home servers because it's lightweight and efficient. Businesses, especially those with high-traffic websites or complex web applications, rely on Nginx for its performance, scalability (ability to handle growth), and flexibility in managing their web infrastructure.

## Installation of Nginx

### Update the Package Manager

Before you install any new software on your server, it's a good practice to update the package lists. Think of this like checking the latest inventory catalog of a store before you place an order. This ensures that your system has the most recent information about available software versions and their dependencies.

To do this, we'll use the `apt` command, which stands for "Advanced Package Tool." It's a powerful command-line tool used for managing software packages on Debian-based systems like Ubuntu.

Open your terminal on your server and run the following command:

```bash
sudo apt update
```

You might be prompted to enter your password. The sudo command allows you to run commands with administrative privileges, which are necessary for installing software.

Once this command finishes running, your package lists will be updated.

### Install Nginx

To install Nginx, we'll use the apt install command followed by the name of the package we want to install, which is nginx.

So, the command you'll need to run in your terminal is:

```bash
sudo apt install nginx
```

The `apt` tool will then reach out to the software repositories, download the necessary Nginx packages, and install them on your server. You might be asked to confirm whether you want to proceed with the installation by typing `y` and pressing Enter.

### Verify the Install

Let's make sure Nginx was installed correctly and is running. This is like checking if our new front desk manager has reported for duty and is ready to work.

We can check the status of the Nginx service using the `systemctl` command. `systemctl` is another essential command-line tool in modern Linux systems used to manage services. Services are background processes that run on your system, providing various functionalities. Nginx runs as a service.

To check the status of the Nginx service, run the following command in your terminal:

```bash
systemctl status nginx
```

This command will give you information about the Nginx service, including whether it's currently running (usually indicated by the word "active" in green) and any recent logs.

## Basic Configuration of Nginx

The main control center for Nginx is its configuration directory, which is typically located at `/etc/nginx/`. Inside this directory, you'll find various files, but the primary one we'll focus on initially is `nginx.conf`. Think of nginx.conf as the main blueprint for how Nginx operates globally.

To see the contents of this directory, you can use the `ls` command in your terminal:

```bash
ls /etc/nginx/
```

You'll likely see a few items, including:

- `nginx.conf`: The main configuration file.
- `conf.d/`: A directory for additional configuration snippets.
- `sites-available/`: A directory containing configuration files for individual websites or server blocks (we'll talk about these later).
- `sites-enabled/`: A directory containing symbolic links to the configuration files in `sites-available/` that are currently active.

For now, let's focus on the `nginx.conf` file. You can view its contents using the cat command or a text editor like nano:

```bash
cat /etc/nginx/nginx.conf
```

or

```bash
sudo nano /etc/nginx/nginx.conf
```

### Structure of a Basic Nginx Configuration File

When you open the `nginx.conf` file, you'll notice it's organized into several blocks or sections, much like different departments in a company or different zones in a building plan. The most important of these are:

- `main` **block (global settings)**: This is the outermost block and contains directives that affect the entire Nginx server. These settings are like the overall rules and regulations for the entire building. For example, it might specify the user that Nginx processes run under.

- `http` **block**: This block encloses all directives related to HTTP (the protocol used for web communication) functionality. Think of this as the main area dedicated to handling web traffic within our building. It can contain settings that affect how Nginx handles web requests and responses.

- `server` **block**: Within the `http` block, you can have one or more `server` blocks. Each `server` block defines the configuration for a specific website or virtual host. It's like having individual apartments within our building, each with its own specific settings. A `server` block typically specifies which port Nginx should listen on (usually port 80 for standard HTTP or port 443 for HTTPS), the server name (the domain name of the website), and how to handle requests for that specific domain.

- `location` **block**: Inside a `server` block, you can have multiple `location` blocks. These blocks define how Nginx should handle requests for specific URLs or URI patterns within a website. Think of these as individual rooms within an apartment, each serving a different purpose (e.g., the `/images` location might handle requests for image files).

### Example Configuration File

The `sites-available` directory contains configuration files for individual websites or server blocks. When Nginx is first installed, it usually includes a default configuration file named default within the `/etc/nginx/sites-available/` directory. This file contains the basic settings for a default website that Nginx can serve.

To see the contents of this default server block configuration file, you can use the cat command or a text editor like nano:

```bash
cat /etc/nginx/sites-available/default
```

or

```bash
sudo nano /etc/nginx/sites-available/default
```

When you open this file, you'll see a server block. Within this block, you'll likely find some important directives, including:

- `listen`: This directive specifies the port that this server block will listen on for incoming connections. You'll probably see `listen 80 default_server;`, which means it's listening on the standard HTTP port (80) and is the default server for any requests that don't match a more specific `server_name`.

- `server_name`: This directive specifies the domain names or hostnames that this server block should respond to. You might see `server_name _;`, where the underscore `_` acts as a wildcard, meaning it will respond to any hostname that doesn't match another server block.

- `root`: This directive specifies the root directory where the website's files are located. When a request comes in for a specific file, Nginx will look for it within this directory. You might see something like root `/var/www/html;`.

---

If you were to open the default configuration file, you might see something like this:

```Nginx
server {
        listen 80 default_server;
        listen [::]:80 default_server ipv6only=on;

        root /var/www/html;
        index index.html index.htm;

        # server_name can be a hostname, wildcard, or regex
        server_name _;

        location / {
                try_files $uri $uri/ =404;
        }

        #error_page 404 /404.html;

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #       include snippets/fastcgi-php.conf;
        #
        #       # With php-fpm (fpm-fcgi)
        #       fastcgi_pass unix:/var/run/php/php7.4-fpm.sock;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #       deny all;
        #}
}
```

Let's break down some of the key parts we mentioned:

- `listen 80 default_server;` **and** `listen \[::]:80 default_server ipv6only=on;`: These lines tell Nginx to listen for incoming HTTP requests on port 80. The `default_server` part indicates that this server block will handle requests that don't specifically match any other `server_name`. The second line is for IPv6 addresses.

- `root /var/www/html;`: This line specifies that the web files for this default website are located in the `/var/www/html` directory. If someone requests the homepage (`/`), Nginx will look for the `index.html` or `index.htm` file in this directory.

- `index index.html index.htm;`: This line tells Nginx which files to serve as the default page if a user accesses a directory. It will try to serve `index.html` first, and if it doesn't find it, it will try `index.htm`.

- `server_name _;`: As we discussed, the underscore here acts as a wildcard, meaning this server block will respond to any hostname that isn't explicitly defined in another `server` block.

- `location / { ... }`: This is a location block that matches all requests (`/`). Inside this block, the `try_files` directive tells Nginx to first try to serve the requested URI as a file, then as a directory, and if neither exists, it returns a 404 error (Not Found).

You'll also see some commented-out sections (lines starting with #). These are examples of more advanced configurations, such as handling PHP files. We won't focus on those right now.

## Setting up Nginx as a Reverse Proxy

Think back to our analogy of the receptionist in a large company. A **reverse proxy** acts as a middleman between clients (like web browsers) and one or more backend servers (where your actual applications are running). Instead of clients directly accessing the backend servers, they communicate with the reverse proxy (Nginx). The reverse proxy then forwards these requests to the appropriate backend server and, once it receives a response, sends it back to the client.

Why would we want to do this? There are several key benefits:

- **Improved Security**: The reverse proxy can hide the actual IP addresses and details of your backend servers, making it harder for attackers to target them directly. It can also handle tasks like SSL encryption/decryption, freeing up your backend servers.
- **Load Balancing**: As we discussed earlier, Nginx can distribute incoming requests across multiple backend servers. This prevents any single server from being overwhelmed and improves the overall performance and reliability of your application.
- **Caching**: The reverse proxy can cache frequently accessed content, serving it directly to clients without needing to forward the request to the backend every time. This can significantly speed up response times and reduce the load on your backend servers.
- **Centralized Access Control**: You can configure access rules and security policies at the reverse proxy level, making it easier to manage and enforce them across your applications.

In our initial setup, we'll focus on the basic function of forwarding requests to a backend application. Let's say you have a simple web application running on the same server but listening on a different port, for example, port `3000`. We can configure Nginx to receive requests on the standard HTTP port (80) and then forward those requests to your application running on port 3000.

### Configuring a Basic Reverse Proxy

We'll be working within the `/etc/nginx/sites-available/default` file again. You'll need to open it with administrative privileges using a text editor like `nano`:

```bash
sudo nano /etc/nginx/sites-available/default
```

Inside the `server` block, you'll likely see the `location / { ... }` block. This block currently handles requests for the root path (`/`) of your server. We're going to modify this block to tell Nginx to forward these requests to our hypothetical backend application running on port `3000`.

To do this, you'll replace the content within the `location / { ... }` block with the `proxy_pass` directive. This directive specifies the address of the backend server to which Nginx should forward the requests.

Here's how the modified location block should look:

```Nginx
location / {
    proxy_pass http://localhost:3000;
}
```

In this line:

- `location /` tells Nginx that this rule applies to all requests coming to the root path of your server (e.g., `http://your_server_ip/`).
- `proxy_pass http://localhost:3000;` tells Nginx to forward these requests to the HTTP server running on the same machine (`localhost`) on port `3000`. If your backend application was running on a different server, you would replace `localhost:3000` with the IP address and port of that server (e.g., `http://192.168.1.100:8080`).

After making this change, save the file and exit the text editor. In `nano`, you can do this by pressing `Ctrl+O` (write out), then Enter, and then `Ctrl+X` (exit).

---

Now that you've configured the basic `proxy_pass` directive, let's explore some other important directives that often go hand-in-hand with setting up a reverse proxy. Think of these as additional instructions you might give to our receptionist to handle calls more effectively.

One crucial directive is `proxy_set_header`. When Nginx forwards a request to the backend server, it also sends along various HTTP headers, which contain information about the client's request (like their browser type, IP address, etc.). Sometimes, the backend application needs specific headers to function correctly or to know the original details of the client's request since it's now coming from the proxy server.

The `proxy_set_header` directive allows you to modify or add headers that Nginx sends to the backend. Here are a couple of common examples:

- `proxy_set_header Host $http_host;`: This line tells Nginx to pass the original `Host` header from the client's request to the backend server. The `$http_host` variable holds the hostname that the client used to access your server. This is important because the backend application might be serving multiple websites based on the `Host` header.

- `proxy_set_header X-Real-IP $remote_addr;`: This line adds a new header called `X-Real-IP` and sets its value to the client's actual IP address, which is stored in the `$remote_addr` variable. This is useful for the backend application to know who actually made the request, even though it came through the proxy.

- `proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;`: This header is used to track the chain of proxies that a request has passed through. `$proxy_add_x_forwarded_for` appends the client's IP address to any existing `X-Forwarded-For` header.

You would typically add these `proxy_set_header` directives within the location block where you have the `proxy_pass` directive. For our example, the `location / { ... }` block in your `/etc/nginx/sites-available/default` file might now look like this:

```Nginx
location / {
    proxy_pass http://localhost:3000;
    proxy_set_header Host $http_host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
}
```

These are just a few examples, and the specific headers your backend application needs will depend on its configuration.

## Testing and Applying Configuration Changes

### Test Configuration Changes

Before we apply any changes we've made to the Nginx configuration, it's essential to **test the configuration for any syntax errors**. Even a small typo in the configuration file can prevent Nginx from starting or cause unexpected behavior.

Nginx provides a built-in command to test the configuration:

```Bash
sudo nginx -t
```

When you run this command, Nginx will check the syntax of your configuration files (including `nginx.conf` and any files in `sites-available` and `sites-enabled`). If there are no errors, you'll typically see output like this:

```shell
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

If there are any errors, Nginx will tell you the specific file and line number where the error occurred, which will help you troubleshoot and fix it. **It's crucial to run this command every time you make changes to your Nginx configuration files**.

### Apply Configuration Changes

To apply the changes you've made to the Nginx configuration files, you need to reload the Nginx service. Reloading tells Nginx to reread its configuration files and apply the new settings without completely stopping and restarting the service. This is generally preferred over a full restart because it minimizes any potential downtime for your web services.

You can reload the Nginx service using the `systemctl` command again, but this time with the reload option:

```bash
sudo systemctl reload nginx
```

After running this command, Nginx should apply your new reverse proxy configuration.

It's worth noting the difference between reload and restart:

- `reload`: Tells the running Nginx processes to reread the configuration files and apply the new settings gracefully. The existing worker processes continue to handle current connections using the old configuration until they are finished. New connections will use the new configuration.

- `restart`: Completely stops all running Nginx processes and then starts new processes with the new configuration. This can cause a brief interruption in service.

For most configuration changes, a `reload` is sufficient. A full `restart` is usually only necessary if there are fundamental changes to the Nginx process itself or in more complex scenarios.

## Important Nginx Concepts

### Virtual Hosts (Server Blocks)

The first concept we'll touch upon is **Virtual Hosts**, also known as **Server Blocks** (which we've already encountered in the configuration files). Think of a single Nginx server as a large apartment building. Each **virtual host** is like an individual apartment within that building, identified by its own address (domain name) and potentially listening on a specific port.

With virtual hosts, you can host multiple websites or run different reverse proxy configurations on a single physical server. Nginx uses the `server_name` directive within each `server` block to determine which website or proxy configuration should handle a particular incoming request based on the domain name requested by the client.

For example, you could have one server block configured for `yourdomain.com` and another for `anotherdomain.net`, both running on the same server and listening on the standard HTTP port 80. When a user visits `yourdomain.com`, Nginx will use the configuration defined in the server block with `server_name yourdomain.com;`. Similarly, requests for `anotherdomain.net` will be handled by its corresponding `server` block.

This allows you to efficiently manage multiple web presences from a single server instance, making it cost-effective and easier to manage.

### Document Root

While less directly relevant when Nginx is primarily acting as a reverse proxy, it's a fundamental concept when Nginx is serving static website content directly.

Think of the **document root** as the main filing cabinet or directory on your server where the website's files (like HTML files, CSS stylesheets, JavaScript files, images, etc.) are stored. When a user requests a specific resource from your website (e.g., `yourdomain.com/about.html` or `yourdomain.com/images/logo.png`), Nginx looks within the specified document root directory to find and serve that file to the user's browser.

The `root` directive in a `server` block configuration tells Nginx where this main filing cabinet is located for that particular website or virtual host. For example, in our default server block, you might have seen a line like `root /var/www/html;`. This means that for the default website, Nginx will look for files within the `/var/www/html` directory. If a user requests `yourdomain.com/index.html`, Nginx will look for the `index.html` file inside `/var/www/html/`.

When Nginx is acting as a reverse proxy, it doesn't typically serve static content itself. Instead, it forwards the requests to the backend application, which then handles the serving of content. In this case, the `root` directive in the Nginx configuration might point to a placeholder directory or might not be as critical to the reverse proxy functionality itself. However, it's still an important concept to understand for when you might want Nginx to serve static files alongside your proxied application or for configuring regular websites.

### Load Balancing

We touched on this earlier, but let's delve a little deeper into how Nginx can act as a load balancer.

Imagine our popular restaurant again. If we only had one chef in the kitchen, they could quickly become overwhelmed when many customers place orders at the same time. A **load balancer** in this scenario acts like a maître d' who strategically distributes the incoming orders (web requests) across multiple chefs (backend servers). This ensures that no single chef is overloaded, leading to faster service and a better experience for the customers.

Nginx can be configured to distribute incoming traffic across multiple backend servers that are all running the same application. This provides several benefits:

- **Improved Performance**: By distributing the load, each backend server handles fewer requests, leading to faster response times for users.
- **Increased Reliability**: If one of the backend servers fails, the load balancer can direct traffic to the remaining healthy servers, ensuring that your application stays online and available.
- **Scalability**: As your application grows and handles more traffic, you can easily add more backend servers, and Nginx can distribute the load across them.

There are different methods Nginx can use to distribute the load, such as:

- **Round Robin**: Requests are distributed to the backend servers in a sequential order (server 1, then server 2, then server 3, then back to server 1, and so on).
- **Least Connections**: Requests are sent to the backend server with the fewest active connections.
- **IP Hash**: Requests from the same client IP address are always directed to the same backend server.

To configure load balancing in Nginx, you would typically define an **upstream** block, which lists the backend servers, and then use the `proxy_pass` directive within a `location` block to point to this upstream group instead of a single server.

While setting up a full load balancing configuration is a more advanced topic, understanding the core concept is crucial for managing scalable and highly available web applications.
