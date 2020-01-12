This assumes that you have Docker (version 17.05 or greater)
and Docker Compose (version 1.6.0 or greater) already installed.

### Prepare things

1. Download the `szurubooru` source:

    ```console
    user@host:~$ git clone https://github.com/rr-/szurubooru.git szuru
    user@host:~$ cd szuru
    ```
2. Configure the application:

    ```console
    user@host:szuru$ cp server/config.yaml.dist server/config.yaml
    user@host:szuru$ edit server/config.yaml
    ```

    Pay extra attention to these fields:

    - secret
    - the `smtp` section.

    You can omit lines when you want to use the defaults of that field.

3. Configure Docker Compose:

    ```console
    user@host:szuru$ cp doc/example.env .env
    user@host:szuru$ edit .env
    ```

    Change the values of the variables in `.env` as needed.
    Read the comments to guide you. Note that `.env` should be in the root
    directory of this repository.

### Running the Application

1. Configurations for ElasticSearch:

    You may need to raise the `vm.max_map_count`
    parameter to at least `262144` in order for the
    ElasticSearch container to function. Instructions
    on how to do so are provided
    [here](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#docker-cli-run-prod-mode).

2. Running the application

    Download containers:
    ```console
    user@host:szuru$ docker-compose pull
    ```

    For first run, it is reccomended to start the databases seperately:
    ```console
    user@host:szuru$ docker-compose up -d elasticsearch
    user@host:szuru$ docker-compose up -d sql
    ```
    Then wait approx. 2 minutes before starting all containers.
    This gives time for the databases to initalize their storage
    structure.

    To start all containers:
    ```console
    user@host:szuru$ docker-compose up -d
    ```

    To view/monitor the application logs:
    ```console
    user@host:szuru$ docker-compose logs -f
    # (CTRL+C to exit)
    ```

    To stop all containers:
    ```console
    user@host:szuru$ docker-compose down
    ```

### Additional Features

1. **CLI-level administrative tools**

    You can use the included `szuru-admin` script to perform various
    administrative tasks such as changing or resetting a user password. To
    run from docker:

    ```console
    user@host:szuru$ docker-compose run api ./szuru-admin --help
    ```

    will give you a breakdown on all available commands.

2. **Using a seperate domain to host static files (image content)**

    If you want to host your website on, (`http://example.com/`) but want
    to serve the images on a different domain, (`http://static.example.com/`)
    then you can run the backend container with an additional environment
    variable `DATA_URL=http://static.example.com/`. Make sure that this
    additional host has access contents to the `/data` volume mounted in the
    backend.

3. **Setting a specific base URI for proxying**

    Some users may wish to access the service at a different base URI, such
    as `http://example.com/szuru/`, commonly when sharing multiple HTTP
    services on one domain using a reverse proxy. In this case, simply set
    `BASE_URL="/szuru/"` in your `.env` file.

    Note that this will require a reverse proxy to function. You should set
    your reverse proxy to proxy `http(s)://example.com/szuru` to
    `http://<internal IP or hostname of frontend container>/`. For an NGINX
    reverse proxy, that will appear as:

    ```nginx
    location /szuru {
        proxy_http_version 1.1;
        proxy_pass http://<internal IP or hostname of frontend container>/;

        proxy_set_header Host              $http_host;
        proxy_set_header Upgrade           $http_upgrade;
        proxy_set_header Connection        "upgrade";
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Scheme          $scheme;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Script-Name     /szuru;
    }
    ```

4. **Preparing for production**

    If you plan on using szurubooru in a production setting, you may opt to
    use a reverse proxy for added security and caching capabilities. Start
    by having the client docker listen only on localhost by changing `PORT`
    in your `.env` file to `127.0.0.1:8080` instead of simply `:8080`. Then
    configure NGINX (or your caching/reverse proxy server of your choice)
    to proxy_pass `http://127.0.0.1:8080`. We've also
    [included an example config](./nginx.vhost.production).