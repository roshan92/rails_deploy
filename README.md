# All package and actions to deploy rails app
=============================================
# Estimated time ~40 minutes
-----------------------------

* [Rails 5](http://rubyonrails.org/)
* [Nginx](https://www.nginx.com/resources/wiki/)
* [PostgreSQL](https://www.postgresql.org/)
* [Puma](http://puma.io/)
* [Ubuntu 16.04](https://www.ubuntu.com/)
* [Capistrano](http://capistranorb.com/)


### The tutorial work with server by [digitalocean.com]
##### Login and create a droplet Ubuntu 16.04x64 1gb cpu & 30gb disk

```
ssh root@xxx.xxx.xx

sudo apt-get install aptitude

sudo apt-get update

sudo aptitude update

sudo aptitude safe-upgrade
```
If you have error with locale
------------------------------
```
sudo locale-gen el_GR.UTF-8
```
##### Next add deploy user
```
adduser deploy
```

##### Give deploy root privileges
```
usermod -aG sudo deploy
```

##### Add Public Key Auth to Your Server.

```
## Use already created local SSH key and add to the server.
## Local Machine
cat ~/.ssh/id_rsa.pub | pbcopy
```

##### Paste SSH Key to Deployed User's Authorized Keys

```
## Server
$ su - deploy
$ mkdir ~/.ssh
$ chmod 700 ~/.ssh
$ nano ~/.ssh/authorized_keys 
# Paste Key
```

##### Change permissions back:
```
chmod 600 ~/.ssh/authorized_keys
```

##### Next, reload service ssh
```
sudo systemctl reload sshd
```

Test Login: ssh deploy@server_ip

#### Install additional packages from Rails
```
sudo apt-get install git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev python-software-properties libffi-dev nodejs
```


##### Install git & nginx
```
sudo apt-get install curl git-core nginx -y
```

##### Install version ruby language rbenv
```
cd
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
exec $SHELL
```


#### Install ruby build plugin for rbenv

```
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bashrc
exec $SHELL
```


###### Show ruby available versions

```
rbenv install -l

rbenv install 2.5.1

rbenv global 2.5.1

rbenv rehash

ruby -v
```

**Set default to not generate document for gem install**

```
echo 'gem: --no-document' >> ~/.gemrc
```

##### Next, install ruby gems version control system

```
gem install bundler

rbenv rehash
```


##### Install Rails

```
gem install rails -V --no-ri --no-rdoc

rbenv rehash
```


### Create ssh key and copy

```
ssh-keygen -t rsa

cat ~/.ssh/id_rsa.pub
```

##### Visit the bitbucket/github and add ssh, after test it

```
ssh -T git@bitbucket.com
```


#### Next, install PostgreSQL
```
sudo aptitude update

sudo apt-get install postgresql postgresql-contrib libpq-dev

```

###### Create user (this will be same user in your database.yml, username field for the production tag):

```
sudo -u postgres createuser -s <APPNAME>

```

###### Set password for APPNAME

```
sudo -u postgres psql
\password <APPNAME>

postgres=# CREATE DATABASE appname_production;
```

####

```
vim /etc/postgresql/9.5/main/pg_hba.conf
```

##### replace

local   all             postgres                                peer

###### to

local   all             postgres                                md5

###### restart postgresql
```
sudo service postgresql restart
```


**Add additional gem into Gemfile**

```
group :development do
    gem 'capistrano',         require: false
    gem 'capistrano-rbenv',     require: false
    gem 'capistrano-rails',   require: false
    gem 'capistrano-bundler', require: false
    gem 'capistrano3-puma',   require: false
end

gem 'puma'
```

#### Next, install gems

```
bundle

rbenv rehash
```

#### Create files from capistrano

```
cap install
```



Copy [config/deploy.rb](../master/config/deploy.rb) & [Capfile](../master/Capfile) files and paste in your rails app



```
git add -A
git commit -m "Set up Puma, Nginx & Capistrano"
git push origin master
```

###### Run capistrano init

```
cap production deploy:initial
```



#### make file /home/deploy/apps/app/shared/config/database.yml
```
production:
  adapter: postgresql
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  timeout: 5000
  encoding: unicode
  database: app_name
  username: <%= ENV['APPNAME_DATABASE_USERNAME'] %>
  password: <%= ENV['APPNAME_DATABASE_PASSWORD'] %>
```


#### Set ubuntu environment variable
```
rails secret

sudo vim /etc/environment
```

#### and add this line below

```
export SECRET_KEY_BASE=abb2e9e094bb8fb1aa216defed455caa0f9
export APPNAME_DATABASE_USERNAME=database_username
export APPNAME_DATABASE_PASSWORD=database_password
```


Example [config/nginx.conf](../master/config/nginx.conf) file, paste in your rails app


##### set nginx virtualhost
```
sudo rm /etc/nginx/sites-enabled/default

sudo ln -nfs "/home/deploy/apps/app_name/current/config/nginx.conf" "/etc/nginx/sites-enabled/app_name"

sudo service nginx restart
```


At the end, run

**If you want only to restart puma server**

`cap production deploy:restart`


**After deploy, publish change**

`git add -A`
`git commit -m "Deploy Message"`
`git push origin master`
`cap production deploy`


#### How to install let's encrypt [#1](https://github.com/roshan92/rails_deploy/issues/1)

If you have some problem, create new issue in this repo. I want to make this document better for the next guy, so let me know if there’s anything I can improve.

## Happy coding...


##### Bugs you might encounter and ways to solve.

**find_spec_for_exe': can't find gem bundler (>= 0.a) (Gem::GemNotFoundException)**

```
#will update the rubygems and will fix the problem.
gem update --system
```

##### Tips
Always ```sudo nginx -t``` to verify your config files are good.


**If you are uploading images, please remember to install  ImageMagick**

```
sudo apt-get install imagemagick
```

