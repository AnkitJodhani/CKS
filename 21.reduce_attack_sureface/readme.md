
# Reduce Attack surface

## Application
- keep up to date
- update linux kernel
- remove not needed packages

## Network
- Check & Close open ports
- Network behind firewall


## IAM
- Restrict user permission
- Run as non-root

## Kubernetes
- Node perpose: Run kubernetes not any other s/w
- Node Recycling: Node should be ephemeral


## See Open port
```bash

sudo netstat -ntpl

sudo lsof -i :22

ps aux

systemctl list-units

systemctl list-units --type=service | grep nginx

systemctl list-units --type=service | grep httpd

systemctl disable nginx
```

## Question
- Find and disable app which is litening on port 21

```bash
# find the name of service
sudo netstat -ntpl | grep -i ":21"

# find the name of service
sudo lsof -i :22

sudo systemctl list-units | grep -i "NAME_OF_SERVICE"

sudo systemctl status NAME_OF_SERVICE

sudo systemctl disable NAME_OF_SERVICE

```

## Linux users

```bash
# Current user
whoami

# List of all the users
cat /etc/passwd

# Change the user
su NAME_OF_USER

# switch to the root user
sudo -i

```