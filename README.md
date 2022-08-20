# django-template
Django Template hosted on AWS EC2 with NGINX

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