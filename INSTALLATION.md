# QPixel Installation

These instructions are for setting up a development instance of QPixel. QPixel is
built with Ruby on Rails.

In that guide it is assumed that you already have a Unix environment available
with Ruby  and Bundler installed. WSL works as well. Windows (core) has not been tested.

For an installation with **Docker** see the README.md in the [docker](docker) folder
for further instructions.

If you don't already have Ruby installed, use [RVM](https://rvm.io/) or
[rbenv](https://github.com/rbenv/rbenv#installation) to install it before following
these instructions.

## Prerequisites

For Debian-Based Linux:

```
sudo apt update
sudo apt install gcc make pkg-config
sudo apt install autoconf bison build-essential libssl-dev libyaml-dev libreadline-dev zlib1g-dev libncurses5-dev libffi-dev libgdbm-dev
sudo apt install mysql-server libmysqlclient-dev
```

For Arch-Based Linux:

```
sudo pacman -Syyu
sudo pacman -Sy gcc
sudo pacman -Sy make
sudo pacman -Sy ruby autoconf bison base-devel unixodbc
sudo pacman -Sy openssl
sudo pacman -S mariadb mysqld nodejs
```

For Mac:

```
xcode-select --install
brew install mysql bison openssl mysql-client
bundle config --global build.mysql2 --with-opt-dir="$(brew --prefix openssl)"
```

### Install JS runtime

If you already have Node.JS installed, you can skip this step. If not,
[download and install it](https://nodejs.org/en/download/) or for example
`sudo apt install nodejs`.

### Install Redis

If you haven't already got it, [download and install Redis](https://redis.io/download)
or for example `sudo apt install redis-server`.

### Install Imagemagick

If you haven't already installed Imagemagick, you'll need to [install it for
your system](https://imagemagick.org/script/download.php).

If you install Imagemagick from APT on a Debian-based system, you may need to
also install the `libmagickwand-dev` package.

`sudo apt install libmagick++-dev` should also work.

## Install QPixel

Clone the repository and `cd` into the directory:

    git clone https://github.com/codidact/qpixel
    cd qpixel

After downloading QPixel, you need to install all the dependencies. For that, you need to run

    bundle install

If Ruby complains, that the Bundler hasn't been installed yet, use `gem install bundler` and
then re-run the above command.

### Setting up the Database

If you weren't asked to set the root MySQL user password during `mysql-server` installation,
the installation is likely to be using Unix authentication instead. You'll need to sign into
the MySQL server with `sudo mysql -u root` and create a new database user for QPixel:

```sql
CREATE USER qpixel@localhost IDENTIFIED BY 'choose_a_password_here';
GRANT ALL ON qpixel_dev.* TO qpixel@localhost;
GRANT ALL ON qpixel_test.* TO qpixel@localhost;
GRANT ALL ON qpixel.* TO qpixel@localhost;
```

Copy `config/database.sample.yml` to `config/database.yml` and fill in the correct host,
username, and password for your environment. If you've followed these instructions (i.e. you
have installed MySQL locally), the correct host is `localhost` or `127.0.0.1`.

You'll also need to fill in details for the Redis connection. If you've followed these instructions,
the sample file should already contain the correct values for you, but if you've customised your
setup you'll need to correct them.


Set up the database:

    rails db:create
    rails db:schema:load
    rails r db/scripts/create_tags_path_view.rb
    rails db:migrate

You'll need to create a Community record and purge the Rails cache before you can seed the database.
In a Rails console (`rails c`), run:

```ruby
Community.create(name: 'Dev Community', host: 'localhost:3000')
Rails.cache.clear
```

After that you can call `rails db:seed` to fill the database with necessary seed data, such as
settings, help posts and default templates.

    $ rails db:seed
    Category: Created 2, skipped 0
    [...]

Now comes the big moment: You can start the QPixel server for the first time. Run:

    rails s

Open a web browser and visit your server, which should be running under [http://localhost:3000](http://localhost:3000).

### Create administrator account

You can create the first user account in the application through the "Sign up" route.
To upgrade the user account to an admin account, run `rails c` for a console, followed by:

```ruby
User.last.update(confirmed_at: DateTime.now, is_global_admin: true)
```

If you create more accounts, you can visit `http://localhost:3000/letter_opener` to access the
confirmation email and then promote your user to admin with
`rails r "User.last.update(is_global_admin: true)"`.

Reload the web browser and you should see the elevated access.

### New site setup

While being logged into your administrator account, go to [http://localhost:3000/admin/setup](http://localhost:3000/admin/setup).
Review the settings (if you want; you can change them later) and click "Save and continue" to complete
setting up the dev server.

### Configure Categories

Before you try to create a post we need to configure categories!
Go to `http://localhost:3000/categories/`

![img/categories.png](img/categories.png)

 Click "edit" for each category and scroll down to see the "Tag Set" field. This
 will be empty on first setup.

![img/tagset.png](img/tagset.png)

You will need to select a tag set for each category! For example, the Meta category can be
associated with the "Meta" tag set, and the Q&A category can be associated with "Main"

![img/tagset-selected.png](img/tagset-selected.png)

Make sure to click save for each one.<br>
<em>Note:</em> You may need to run `rails db:seed` again.

## Create a Post

You should then be able to create a post! There are character requirements for the
body and title, and you are required at least one tag.

![img/create-post.png](img/create-post.png)

And then click to "Save Post in Q&A"

![img/post.png](img/post.png)
