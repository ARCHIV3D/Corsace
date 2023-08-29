Please use this repository when you are changing any of the code for Corsace projects.

# Before Getting Started
On WSL/Unix:
```
git clone https://github.com/Corsace/Corsace
```
On Windows (Use admin cmd):
```
git clone -c core.symlinks=true https://github.com/Corsace/Corsace
```

Install node-modules:
```
npm i
```
**Note: Please ensure that your node version is 16.6.0 or higher in order to use any of the discord features in this repository.**

## Getting Started

Duplicate `config/default.json` to `config/user/$USER.json`, `$USER` being your system username [(accessible via node's process.env.USER or USERNAME)](https://github.com/tusharmath/node-config-ts#using-files).
The values in your personal `config/user/$USER.json` config file will be referred to as `config` from now on.

### osu! API

`config.osu.v1.apiKey`

You can obtain your osu! API V1 key [here](https://osu.ppy.sh/p/api/)

```
config.osu.v2.clientId
config.osu.v2.clientSecret
```

You will need to create a "New OAuth Application" at the bottom of https://osu.ppy.sh/home/account/edit.

The callback URL should be set to:
```
config.corsace.publicUrl + /api/login/osu/callback
```

```
config.osu.bancho
```

You can obtain your osu IRC password from the same place as the osu OAuth Application at https://osu.ppy.sh/home/account/edit#irc.

If your account is a bot account, then make sure to have `botAccount` set to true; otherwise, make set it to false.

### Database

#### Setup

`config.database`

##### Via Docker (Recommended)

We are shipping a production-like `docker-compose.yml` file. You can start only the database service using: `docker-compose up -d database` or `npm run database`.

The database will listen on `127.0.0.1:3306`, with `corsace` being the database name, username and password.

##### Manual MariaDB Setup

If you do not want to use Docker, you will need to install [MariaDB](https://mariadb.org/) and create an empty database, named whatever you like. 

It can be as simple as running:
```
mysql -u root -p
MySQL> create database <new_db_name>; 
```

Make sure to update `config.database` to reflect your choice of database name and credentials.

#### Seeding the database

Create and seed the whole Corsace database using: `NODE_ENV=development npm run -- typeorm migration:run -d ormconfig`

### Object Storage/S3

We use S3-compatible object storage for storing and serving mappacks, configured in `config.s3`.

While we target Cloudflare R2, any S3 provider should work as long as they support multipart uploads and pre-signed URLs.

We use three buckets:
- `team-avatars` is a public bucket that stores team avatars, can be served by a CDN without authentication
- `mappacks` is a public bucket that stores public mappacks, can be served by a CDN without authentication
- `mappacks-temp` is a private bucket that stores private mappacks that should not have public access  
  Generated mappacks are first uploaded to this bucket, users are given access through pre-signed URLs.  
  Private mappacks are not meant to be permanently stored, a lifecycle policy should be added to that bucket to automatically delete objects after 1 day.  
  Mappacks that should become public get moved to the `mappacks` bucket.

#### Cloudflare R2

Go to the [Cloudflare R2 dashboard page](https://dash.cloudflare.com/?to=/:account/r2). Enable your plan if you haven't already (good luck exceeding free limits).

Create the `mappacks` and `team-avatars` buckets and enable their R2.dev subdomains, or associate a custom domain for each.

Create the `mappacks-temp` bucket and add an object lifecycle rule to delete objects after 7 days (leave prefix empty).

Set hostname to `<cloudflare account id>.r2.cloudflarestorage.com`, and obtain S3 credentials from https://dash.cloudflare.com/?to=/:account/r2/api-tokens. **Make sure you give the token `Edit` permissions instead of the default `Read` permissions.**

### Discord

#### Setup

`config.discord`

This is the most time-consuming part of the setup. 
You will need the following:

##### Enable Developer Mode
Check the option at 
```
User Settings > Appearance > Advanced > Developer Mode
```

This will allow you to right click users, roles, channels, etc to copy their IDs.

##### A Discord Server
Create a new Discord Server if you don't have one already. All it needs to have is a single channel.
Create a "staff" role and give it to yourself.

Right-click your server name and "Copy ID". Paste this into `config.discord.guild`.

Right-click your staff role and "Copy ID". You can either create a role for each corresponding role in the config, OR 
paste that role ID into the following config values to give yourself god-tier permissions.
```
config.discord.roles.corsace.corsace
config.discord.roles.corsace.core
config.discord.roles.corsace.headStaff
config.discord.roles.corsace.staff
```
and then into every other "staff" role in the config.

##### Discord Application
Go to https://discord.com/developers/applications and create a "New Application".

###### Client
You will need to add the "Client ID" and "Client Secret" to the config as follows:
```
discord: {
    ...,
    clientId: "<Client ID>",
    clientSecret: "<Client Secret>",
}
```

###### OAuth2
Head to the OAuth2 section of the bot and add the following redirect URL:
```
config.corsace.publicUrl + /api/login/discord/callback
```

Also add a redirect URL with your bot's specific Client ID that looks like:
```
https://discordapp.com/oauth2/authorize?&client_id=<CLIENT ID>&scope=bot&permissions=8
```
Follow this link to add your bot to your server.

###### Bot
Head to the Bot section of the bot and copy your bot token. 
Paste it into `config.discord.token`

Ensure you enable the `Server Members` and `Message Content` intents under the **Privileged Gateway Intents** subsection before usage, the bot will not start otherwise, and you will be provided a `[DISALLOWED INTENTS]` error.

### Centrifugo

We use Centrifugo for real-time notifications. You can find the documentation [here](https://centrifugal.dev/docs/getting-started/introduction).

#### Setup

On Unix:
Run `npm run centrifugo` to start the centrifugo server. It will be available at `http://localhost:8001` by default, unless you change the port in the config files.

On WSL/Windows OR if the above doesn't work:
Install the binary from [latest releases](https://github.com/centrifugal/centrifugo/releases), and add it to the root folder of this project.
Afterwards, run `npm run centrifugo:local` to start the centrifugo server. If you want to change the port, change the `-p` flag in the repective script in `package.json`, and your config file's api URL.

## Development

Run `npm run dev`, if you only want to run one of the projects, refer to the scripts in `package.json`.
To run the project without the api, use `npm run dev-client`.
