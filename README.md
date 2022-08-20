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