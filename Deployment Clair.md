# Deployment Clair

*** Note *** : Clair use Postgresql 9.4+ database



**Docker Compose**

1) Download Clair files

```
$ mkdir -p clair/clair_config && cd clair
$ curl -L https://raw.githubusercontent.com/coreos/clair/master/contrib/compose/docker-compose.yml -o $PWD/docker-compose.yml
$ curl -L https://raw.githubusercontent.com/coreos/clair/master/config.yaml.sample -o $PWD/clair_config/config.yaml
```

- docker-compose.yml

```yml
version: '2'
services:
  postgres:
    container_name: clair_postgres
    image: postgres
    restart: unless-stopped
    environment:
      POSTGRES_PASSWORD: password
      POSTGRES_USER: postgres
      POSTGRES_DB: postgres

  clair:
    container_name: clair_clair
    image: quay.io/coreos/clair:v2.0.1
    restart: unless-stopped
    depends_on:
      - postgres
    ports:
      - "6060-6061:6060-6061"
    links:
      - postgres
    volumes:
      - ./tmp:/tmp
      - ./clair_config:/config
    command: [-config, /config/config.yaml]
```

2) Update config

```
$ sed 's/clair-git:latest/clair:v2.0.1/' -i docker-compose.yml && \
  sed 's/host=localhost/host=postgres password=password/' -i clair_config/config.yaml
```

3) Start Database

```
$ docker-compose up -d postgres
```

4) Download and load CVE

```
curl -LO https://gist.githubusercontent.com/BenHall/34ae4e6129d81f871e353c63b6a869a7/raw/5818fba954b0b00352d07771fabab6b9daba5510/clair.sql
```

5) Deploy Clair

```
$ docker-compose up -d clair
```

6) Download Klar

```
$ curl -L https://github.com/optiopay/klar/releases/download/v1.5/klar-1.5-linux-amd64 -o /usr/local/bin/klar && chmod +x $_
```

7) Scan Image (public image)

```
$ CLAIR_ADDR=http://localhost:6060 CLAIR_OUTPUT=Low CLAIR_THRESHOLD=10 \
  klar quay.io/coreos/clair:v2.0.1
```



**Docker**

1) Download Clair files

```
$ mkdir -p clair/clair_config && cd clair
$ curl -L https://raw.githubusercontent.com/coreos/clair/master/config.yaml.sample -o $PWD/clair_config/config.yaml
```

2) Docker run postgres & Clair

```
$ docker run -d -e POSTGRES_PASSWORD="" -p 5432:5432 postgres:9.6
$ docker run --net=host -d -p 6060-6061:6060-6061 -v $PWD/clair_config:/config quay.io/coreos/clair-git:latest -config=/config/config.yaml
```

3) Download Klar

```
$ curl -L https://github.com/optiopay/klar/releases/download/v1.5/klar-1.5-linux-amd64 -o /usr/local/bin/klar && chmod +x $_
```

4) Scan Image (public image)

```
$ CLAIR_ADDR=http://localhost:6060 CLAIR_OUTPUT=Low CLAIR_THRESHOLD=10 \
  klar quay.io/coreos/clair:v2.0.1
```

