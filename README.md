# tannit

Init system for services running in a Linux environment. Powered by tmux.

> [!WARNING]
> `tannit` is currently in early development stage, its not done and is not ready for use in production environments

also,

> [!IMPORTANT]
> No code is available on the repository, but it will be available soon.

## Why `tannit` (Usage)

1. Fast: Almost 0 delay for starting.
2. No daemons required: No custom `systemd` or `openrc` service is required to operate, `tmux` is the only dependency
3. Minimal RAM usage

## Why `tannit` (Name)

Original name was `NeoInit`, due to the overuse of `neo` in my projects, I moved off the old name
New name means

- tann (`true1ann`, me)
- init

## Installing `tannit`

> `tannit` can run from any directory, it doesn't have to be accessible by every user on the host machine, nor it is required to place it `/bin` or similiar. Just don't move `tannit` to other directories once atleast one service was launched

- Go to **Releases**
- Download `tannit` or `tannit.js`

> `tannit` is compiled, `tannit.js` requires to be ran with NodeJS

- Unzip to any folder on the host machine

## Configuring `tannit`

- Go to [`/example/tannit-config.yaml`](/example/tannit-config.yaml) and copy it over in the same folder as `tannit`
- Open the file, there you will find the basic declaration for a service.

### Magic variables

- `$UUID` - Service UUID (it will be visible in the configuration file too on the next config parse. ***DO NOT ADD IT YOURSELF***)
- `$ID` - Works the same as `$CWD` but with the `service.id` field
- `$CWD` - Working directory (so you can change just one variable if the service location changes (`service.cwd`))
- `$ENV` - Used as an object (e.g. `$ENV.PORT`)
- `.` - Used in paths, this is where `tannit`'s executable is located (for example `./tannit` points to `tannit` itself)

### YAML-based configuration syntax

```yaml
# For example if you want to make `tannit` config more modular
@include:
  - ./service1.yml
  - ./service2.yaml
  - ./service3.json # will be parsed too if the syntax is correct (see below)

services:
  - name: My website
    id: my-website # if contains a space or non-english character, this service will be ignored. (`[a-Z][0-9]-_`) if there are more than one service using the same ID, UUID will be added. (here: `my-website_d693ef9e`)
    cwd: ./services/myweb/ # trailing `/` is not required
    cmd: ["flask", "app.py"] # Python `venv`s are NOT activated automatically

    tannit-env:
      - python-venv: true # false by default
      - autostart: true # true by default
      - disable-logs: false # true by default
      - logs-location: ./logs/$UUID
      - restart: always # can be `always`, `never`, `on-failure`, `on-success`, `on-abort`, `on-abort-only`. `never` by default
      - restart-delay: 3000 # in ms
      - hidden: false # false by default, hides the service (healthcheck included) from the JSON list check up unless --hidden or --hidden-only is provided.
    env:
      PORT: 3000
      ...

    healthcheck:
      - name: dedicated-endpoint # if contains a space or non-english character, this field will be ignored (`[a-Z][0-9]-_`)
        type: http
        url: http://localhost:$ENV.PORT/api/healthcheck # UA: `annit-healthcheck/VERSION`
        interval: 60 # in s, 5 by default
        expect: 200 # HTTP response code, 200 by default
        timeout: 10 # in s, 10 by default
      
      - name: homepage
        type: http
        url: https://localhost:$ENV.PORT/
        interval: 60

    uuid: d693ef9e-c59e-46c9-9fed-6ad4f83d31d7 # added by tannit automatically. DO NOT CHANGE WHILE ANY SERVICE IS RUNNING (preferably dont change or add it yourself at all)
```

### JSON-based configuration syntax

> [!IMPORTANT]
> JSON doesn't allow comments. For information about 'what does this do' refer to the YAML version of the configuration

```json
{
  "@include": [
    "./service1.yml",
    "./service2.yml",
    "./service3.json"
  ],
  "services": [
    {
      "name": "My website",
      "id": "my-website",
      "cwd": "./services/myweb",
      "cmd": ["flask", "app.py"],
      "tannit-env": {
        "python-venv": true,
        "autostart": true,
        "disable-logs": false,
        "logs-location": "./logs/$UUID",
        "restart": "always",
        "restart-delay": 3000,
        "hidden": false
      },
      "env": {
        "PORT": 3000
      },
      "healthcheck": [
        {
          "name": "dedicated-endpoint",
          "type": "http",
          "url": "http://localhost:$ENV.PORT/api/healthcheck",
          "interval": 60,
          "expect": 200,
          "timeout": 20
        },
        {
          "name": "homepage",
          "type": "http",
          "url": "http://localhost:$ENV.PORT",
          "interval": 60
        }
      ],
      "uuid": "d693ef9e-c59e-46c9-9fed-6ad4f83d31d7"
    }
  ]
}
```

> `tannit` supports both, YAML and JSON. There are no differences in using one or another
