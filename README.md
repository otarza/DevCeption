# DevCeption

![DevCeption](https://user-images.githubusercontent.com/8479569/33684572-23cad2ce-dae8-11e7-83e8-33f6bb0dd0ab.jpeg)

This is an inception style (**Docker** inside **Vagrant** inside **Host**) development environment built around **tmux/vim** workflow.

## Why?
We (Devs) use too many tools for our everyday tasks, from editors to webservers to terminals. This makes switching machines painful. In case of theft, data damage, or simply format-reinstall OS, you need to go through the pain of reinstall reconfigure everytime... Solution to this problem is to have all your dev tools inside a container that can be **created**, **destroyed**, **reproduced** on any machine without a hit to productivity.

With **DevCeption**, you can go from mac to linux to windows and always find your habitual dev environment. This is huge boost to productivity and versatility as Host OS becomes meaningless.

p.s. Also because docker on mac really sucks!

## Goodies

**docker**
- services
- - webserver
- - - nginx
- - - - php generic config (Laravel)
- - - - WordPress optmized config
- - - php5-fpm, php7-fpm
- - - mariadb
- - phpcs, wpcs
- docker-compose

---

**oh-my-zsh**
- plugins: git, history-substring-search, tmux

---

**tmux**

[![asciicast](https://asciinema.org/a/151667.png)](https://asciinema.org/a/151667)

---

**vim**
- vim-quantum
- ctrlp
- vim-vinegar
- vim-gitgutter
- vim-trailing-whitespace
- editorconfig-vim
- delimitMate
- syntastic
- indentLine
- vim-commentary
- vim-polyglot
- matchtagalways
- vim-buftabline
- vim-bracketed-paste
- vim-tmux-focus-events
- vim-tmux-clipboard
- vim-surround
- YouCompleteMe
- ultisnips
- vim-snippets

---

**nvm**
- node lts, gulp

---

**git**
- git-commit-template
- diff-so-fancy

## Host Requirements

- virtualbox
- vagrant
- ansible
- rsync

## Getting Started

### Step 1

```
git clone https://github.com/wearede/DevCeption
cd DevCeption
vagrant up
```

Vagrant will perform following operations:
- It will first download and then import the centos7 base box.
- It will rsync www and docker configuration files to the newly created centos7 base box.
- Then using ansible it will configure and install dev tools from playbooks.

... grab a coffee, it will take some time on a slow connection.

**Note:** `Install vim plugins` task could take a while, but if it hangs see *Common issues* section below.

### Step 2

Lets configure ssh on host machine for painless ssh-eing. On macOS you would do:

```
vi ~/.ssh/config
```
and paste
```
#
Host DevCeption
HostName 127.0.0.1
User vagrant
Port 2222
UserKnownHostsFile /dev/null
StrictHostKeyChecking no
PasswordAuthentication no
IdentityFile ~/Code/Vagrant/DevCeption/.vagrant/machines/DevCeption/virtualbox/private_key
IdentitiesOnly yes
LogLevel FATAL
```

obviously replacing `IdentityFile` path `~/Code/Vagrant/DevCeption` with the location of your *DevCeption* directory.

`:wq`

and now you can simply `ssh DevCeption` from your host machine.

### Step 3

We need to build docker webserver containers inside a DevCeption box - and while we are at it, we can take tmux out for a spin.

`ssh DevCeption` and once you are in type `ts webserver`

- new tmux session will start, type a. `cd /srv/docker/services`,
- lets divide window in two panels type b. `alt+\`,
- to navigate panels use `alt+h` or `alt+l`.

In the left panel `cd webserver` directory on the right one `cd phpcs`

- in the webserver directory run `sudo docker-compose up`
- and inside phpcs dir run `sudo docker build -t phphcs .` <- note the `.`

![tmux](https://user-images.githubusercontent.com/8479569/33686579-caa3639e-daee-11e7-897e-7547d61c07b3.png)

You can now quit iterm2 but tmux session will continue to run inside DevCeption box (in the background), you can always go back to it by typing `ta webserver`, and to see all running sessions type `tl`.

### Step 4

Lets connect to mariadb running inside DevCeption box, inside docker container, from our host machine.
On macOS sequelpro works really well for managing mariadb/mysql:

![sequelpro](https://user-images.githubusercontent.com/8479569/33715683-e336c1a2-db6c-11e7-9cb2-cecc872e252b.png)

- We are using ssh tunneling to DevCeption box.
- ssh key is the same as IdentityFile from step 2, e.g. `~/Code/Vagrant/DevCeption/.vagrant/machines/DevCeption/virtualbox/private_key`
- mariadb root password is defined in [docker-compose.yml](https://github.com/wearede/DevCeption/blob/master/configs/docker/services/webserver/docker-compose.yml#L10)

### Step 5

Finally we will see how to get a web project up and running:



## Back Up Your Work

It is important to understand that project files will live inside DevCeption box and will not be synced back to host machine automatically. You have to do this manually.

Run these commands on your host:

```
rsync -vriht DevCeption:/srv/docker/ ./configs/docker/ --delete
rsync -vriht DevCeption:/srv/www/ ./www --delete
```
**Warnning:** Running `vagrant up` or `vagrant provision` will overwrite project files inside your DevCeption box with the ones on your host. So don't forget to periodically bring back / backup changes to your host machine, especially if you decide to do `vagrant destroy` ;)

## Common Issues

- ansible task **Install vim plugins** hangs:
  - *Iterm2* settings disable `profile -> terminal -> environment -> set locale variable automatically`
  - *oh-my-zsh* uncomment `export LANG=en_US.UTF-8` in `.zshrc` 
- copying to host machine clipboard
  - *Iterm2* in general settings enable `Applications in terminal may access clipboard` (works for tmux)
