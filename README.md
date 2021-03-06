# Visible Wordpress Starter

A Docker Wordpress development environment by the team at [Visible](https://visible.vc/) and some awesome [contributors](https://github.com/visiblevc/wordpress-starter/graphs/contributors). Our goal is to make Wordpress development slightly less frustrating.

## Requirements

Well, to run a Docker environment, you will need Docker. The Dockerfile is only for an Apache+PHP+Wordpress container, you will need a MySQL container to run a website. We use Docker Compose 1.6+ for the orchestration.

---

## Getting started

This project has 2 parts: the Docker environment and a set of tools for theme development. To quickly get started, you can simply run the following:

```
# copy the files
git clone https://github.com/visiblevc/wordpress-starter.git
rm -rf .git Dockerfile run.sh README.md CHANGELOG.md ISSUE_TEMPLATE.md

# start the website at localhost:8080
docker-compose up
```

NOTE: If you run on MacOS with Docker in VirtualBox, you will want to forward the port by running this `VBoxManage controlvm vm-name natpf1 "tcp8080,tcp,127.0.0.1,8080,,8080"`. If you use another port than `8080`, change it in the command.

This repository does 2 things:

1. Include the files to create a wordpress Docker image (visiblevc/wordpress)
2. Include build tools to develop wordpress themes (gulp)

If you don't plan to build the Docker image yourself, you shouldn't care for 1. We publish the image on Docker Hub and you can grab it directly from there. That's why you can safely remove the Dockerfile and run.sh.

The reason we remove `.git`, `README.md` and `CHANGELOG.md` is because we assume you will start your own repository, named after your project. There is virtually no benefit keeping ties with our remote git repository.

---

### Documentation

We wrote a series of articles explaining in depth the philosophy behind this project:

- [Intro: A slightly less shitty WordPress developer workflow](https://visible.vc/engineering/wordpress-developer-workflow/)
- [Part 1: Setup a local development environment for WordPress with Docker](https://visible.vc/engineering/docker-environment-for-wordpress/)
- [Part 2: Setup an asset pipeline for WordPress theme development](https://visible.vc/engineering/asset-pipeline-for-wordpress-theme-development/)
- [Part 3: Optimize your wordpress theme assets and deploy to S3](https://visible.vc/engineering/optimize-wordpress-theme-assets-and-deploy-to-s3-cloudfront/)
- Part 4: Auto deploy your site on your server (coming)

### The Docker environment

The only thing you need to get started is a `docker-compose.yml` file:

```yml
version: '2'
services:
  wordpress:
    image: visiblevc/wordpress:latest
    links:
      - db
    ports:
      - 8080:80
      - 443:443
    volumes:
      - ./data:/data # Required if importing an existing database
      - ./wp-content/uploads:/app/wp-content/uploads
      - ./yourplugin:/app/wp-content/plugins/yourplugin # Plugin development
      - ./yourtheme:/app/wp-content/themes/yourtheme   # Theme development
    environment:
      DB_HOST: db
      DB_NAME: wordpress
      DB_PASS: root # must match below
      PLUGINS: >-
        academic-bloggers-toolkit,
        co-authors-plus,
        [WP-API-master]https://github.com/WP-API/WP-API/archive/master.zip,
      SEARCH_REPLACE: yoursite.com,localhost:8080
      WP_DEBUG: 'true'
  db:
    image: mysql:5.7 # or mariadb:10
    ports:
      - 3306:3306
    volumes:
      - data:/var/lib/mysql
    environment:
      MYSQL_ROOT_PASSWORD: root
volumes:
  data: {}
```

##### MySQL Credentials

- hostname: `db` (can be changed with the `DB_HOST` environment variable)
- username: `root`
- password: `root` (can be changed with the `MYSQL_ROOT_PASSWORD` and `DB_PASS` environment variables)
- database: `wordpress` (can be changed with the `DB_NAME` environment variable)
- admin email: `admin@${DB_NAME}.com`

##### WordPress Container Environment variables

- `DB_HOST` (optional): Defaults to `db`
- `DB_PASS` (required): Must match `MYSQL_ROOT_PASSWORD` of the mysql container
- `DB_NAME` (optional): Defaults to `wordpress`
- `DB_PREFIX` (optional): Defauts to `wp_`
- `ADMIN_EMAIL` (optional): Defaults to `admin@${DB_NAME}.com`
- `WP_DEBUG` (optional): Defaults to `false`
- `WP_DEBUG_DISPLAY` (optional): Defaults to `false`
- `WP_DEBUG_LOG` (optional): Defaults to `false`
- `THEMES` (optional): Comma-separated list of themes you want to install in either of the following forms:
  - `theme-slug`: Used when installing theme direct from WordPress.org.
  - `[theme-slug]http://themesite.com/theme.zip`: Used when installing theme from URL.
- `PLUGINS` (optional): Comma-separated list of plugins you want to install in either of the following forms:
  - `plugin-slug`: Used when installing plugin direct from WordPress.org.
  - `[plugin-slug]http://pluginsite.com/plugin.zip`: Used when installing plugin from URL.
- `MULTISITE` (optional): Set to `'true'` to enable multisite
- `PERMALINKS` (optional): String which will resolve to a valid WordPress permalink structure using [structure tags](https://codex.wordpress.org/Using_Permalinks#Structure_Tags). Defaults to `/%year%/%monthnum%/%postname%/`
- `SEARCH_REPLACE` (optional): Comma-separated string in the form of `current-url,replacement-url`.
    - When defined, `current-url` will be replaced with `replacement-url` on build (useful for development environments utilizing a database copied from a live site).
    - **IMPORTANT NOTE:** If you are running Docker on Mac or PC (using Docker Machine), your replacement url MUST be the output of the following command: `echo $(docker-machine ip <your-machine-name>):8080`
- `VERBOSE` (optional): Set to `true` to run build with verbose logging.

##### DB Container Environment variables

- `MYSQL_ROOT_PASSWORD` (required): Must match `DB_PASS` of the wordpress container

### Use `wp-cli`

You can access wp-cli by running `npm run wp ...`. Here are some examples:

```
npm run wp plugin install <some-plugin>
npm run wp db import /data/database.sql
```

### Working with databases

If you have an exported `.sql` file from an existing website, drop the file into the `data/` folder. The first time you run the container, it will detect the SQL dump and use it as a database. If it doesn't find one, it will create a fresh database.

If the SQL dump changes for some reason, you can reload the database by running:

```sh
docker exec wordpress /bin/bash "wp db import $(find /data/*.sql | head -n 1) --allow-root"
```

If you want to create a dump of your development database, you can run:

```sh
npm run wp db export /data --allow-root
```

Finally, sometimes your development environment runs on a different domain than your live one. The live will be `example.com` and the development `localhost:8080`. This project does a search and replace for you. You can set the `SEARCH_REPLACE: example.com,localhost:8080` environment variable in the `docker-compose.yml`.

---

## Development

You can find Development instructions in the [Wiki](https://github.com/visiblevc/wordpress-starter/wiki/Development).
