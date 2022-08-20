# django-template
### Django Template hosted on AWS EC2 with NGINX

[Blog Link](https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-20-04)

1. Log in ubuntu server
    ```console
    sudo ssh -i "ec2-keyname.pem" ubuntu@public_ip
    ```

1. Adding SSH key for github
    ```console
    cd ~/.ssh
    ```
    ```console
    ssh-keygen -o -t rsa -C "example@gmail.com.com"
    ```
    ```console
    cat id_rsa.pub
    ```

1. Updating Ubuntu
    ```console
    sudo apt update
    ```

1. Installing python and dev tools
    ```console
    sudo apt install python3-pip python3-dev libpq-dev nginx curl
    ```

1. Upgrade pip
    ```console
    sudo -H pip3 install --upgrade pip
    ```

1. Install virtualenv
    ```console
    sudo -H pip3 install virtualenv
    ```
1. Clone the project
1. Go to the directory
    ```console
    cd proj-dir
    ```

1. Create virtual environment
    ```console
    virtualenv env
    ```

1. Activate env
    ```console
    source env/bin/activate
    ```

1. Install requirements
    ```console
    pip3 install -r requirements.txt
    ```

1. Creating systemd Socket and Service Files for Gunicorn
    - The Gunicorn socket will be created at boot and will listen for connections. When a connection occurs, systemd will automatically start the Gunicorn process to handle the connection.
    ```console
    sudo nano /etc/systemd/system/gunicorn.socket
    ```

1. Update gunicorn.socket
    ```socket
    [Unit]
    Description=gunicorn socket

    [Socket]
    ListenStream=/run/gunicorn.sock

    [Install]
    WantedBy=sockets.target
    ```

1. Create gunicorn.services and update it.
    ```console
    sudo nano /etc/systemd/system/gunicorn.service
    ```
    ```console
    [Unit]
    Description=gunicorn daemon
    Requires=gunicorn.socket
    After=network.target

    [Service]
    User=ubuntu
    Group=www-data
    WorkingDirectory=/home/ubuntu/django-template
    ExecStart=/home/ubuntu/django-template/env/bin/gunicorn \
            --access-logfile - \
            --workers 3 \
            --bind unix:/run/gunicorn.sock \
            core.wsgi:application

    [Install]
    WantedBy=multi-user.target
    ```

1. Start gunicorn
    ```console
    sudo systemctl start gunicorn.socket
    ```

1. Enable gunicorn
    ```console
    sudo systemctl enable gunicorn.socket
    ```

1. Check status
    ```console
    sudo systemctl status gunicorn.socket
    ```

1. Check for the existence of the gunicorn.sock file within the /run directory:
    ```console
    file /run/gunicorn.sock
    ```

    - Output of the above command should look like
    ```bash
    Output
    /run/gunicorn.sock: socket
    ```

1. If any error found, check the Gunicorn socket’s logs by typing:
    ```console
    sudo journalctl -u gunicorn.socket
    ```

1. Testing Socket Activation
    - Currently, if you’ve only started the gunicorn.socket unit, the gunicorn.service will not be active yet since the socket has not yet received any connections. You can check this by typing:
    ```console
    sudo systemctl status gunicorn
    ```

1. To test the socket activation mechanism, we can send a connection to the socket through curl by typing:
    ```console
    curl --unix-socket /run/gunicorn.sock localhost
    ```

1. If the output from curl or the output of systemctl status indicates that a problem occurred, check the logs for additional details:
    ```console
    sudo journalctl -u gunicorn
    ```

1. Check your /etc/systemd/system/gunicorn.service file for problems. If you make changes to the /etc/systemd/system/gunicorn.service file, reload the daemon to reread the service definition and restart the Gunicorn process by typing:
    ```console
    sudo systemctl daemon-reload
    ```
    ```console
    sudo systemctl restart gunicorn
    ```
    - Also run the above two commands after change in code

### Configure Nginx to Proxy Pass to Gunicorn

1. Start by creating and opening a new server block in Nginx’s sites-available directory:
    ```console
    sudo nano /etc/nginx/sites-available/myproject
    ```

1. Configure the file
    ```console
    server {
        listen 80;
        server_name 3.110.56.75, domain.com(if available);

        location = /favicon.ico { access_log off; log_not_found off; }

        location / {
            include proxy_params;
            proxy_pass http://unix:/run/gunicorn.sock;
        }
    }
    ```

1. Save and close the file when you are finished. Now, we can enable the file by linking it to the sites-enabled directory:
    ```console
    sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled
    ```

1. Test your Nginx configuration for syntax errors by typing:
    ```console
    sudo nginx -t
    ```

1. If no errors are reported, go ahead and restart Nginx by typing:
    ```console
    sudo systemctl restart nginx
    ```

1. Finally, we need to open up our firewall to normal traffic on port 80. Since we no longer need access to the development server, we can remove the rule to open port 8000 as well:
    ```console
    sudo ufw delete allow 8000
    ```
    ```console
    sudo ufw allow 'Nginx Full'
    ```
