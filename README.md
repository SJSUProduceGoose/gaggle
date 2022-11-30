# Gaggle

> /ˈɡaɡəl/
>
> `noun`
> 1. a flock of geese.
> 2. the docker compose file bringing together produce goose's backend and frontend repositories.

## Getting Started

Make sure you have `docker` and `docker-compose` installed and the **Docker** application is
running before you attempt to execute any commands that start with `docker-compose`.

### Cloning

After cloning the repository we need to initialize the submodules:

```bash
git clone git@github.com:SJSUProduceGoose/gaggle.git
cd gaggle
git submodule init
git submodule update
```

### Configure The Environment

Copy the `.env.example` file to `.env` and fill in the values.

```bash
cp .env.example .env
```

| Variable              | Description                                                                                                                                     | Values                                            |
|-----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------|
| ENVIRONMENT           | Used to determine levels of security and logging within the backend.                                                                            | Options: `production` / `staging` / `development` |
| DATABASE_URL          | Used to locate connect to the PostgreSQL database.                                                                                              | Not required                                      |
| JWT_SECRET            | Used to sign the JWT token. For security reasons you should generate a new key if deploying this online. Preferably a 64 character long string. | Not required                                      |
| BASE_URL_UI           | Configures knowledge about the public facing URL of the frontend.                                                                               | Default: http://localhost:8080                    |
| BASE_URL_API          | Configures knowledge about the public facing URL of the frontend.                                                                               | Default: http://localhost:8080/api                |
| STRIPE_PRIVATE_KEY    | The private key that will be used to connect to stripe. [See](#configuring-stripe)                                                              | BYOK                                              |
| STRIPE_SIGNING_KEY    | The signing key to validate webhooks from Stripe. [See](#configuring-stripe)                                                                    | BYOK                                              |
| POSITIONSTACK_API_KEY | Used to resolve addresses to coordinates. [See](#positionstack)                                                                                 | BYOK                                              |


> BYOK: *Bring Your Own Key*

#### Configuring Stripe

Stripe is used to process payments. You will need to create an account and generate a
private key and signing key. You can find more information [here](https://stripe.com/docs/keys).

##### Create a Stripe Account
1. Register [here](https://dashboard.stripe.com/register) and set up a Test account. There
   is no need to set up a live account at this time.
2. Go to the [Stripe dashboard](https://dashboard.stripe.com/test/dashboard) and click on
   `Developers` -> `API keys`.
3. Copy the `Secret key` and paste them in the `.env` file.
4. Set up the tax rates for your business. You can do so by going to the
   [Stripe Tax Rates Dashboard](https://dashboard.stripe.com/test/tax-rates).
   + To simplify things, you can [set up automatic taxes](https://stripe.com/tax)
     and Stripe will automatically calculate the tax rate based on the address
     of the customer.
   + Configure the "origin address" to somewhere in the same state as your
     business.
   + And set the default product tax category to "Food for Immediate Consumption".

##### Stripe Private Key

The private key is used to connect to the Stripe API. You can find this key in the Stripe
dashboard under `Developers > API Keys`.

##### Stripe Signing Key

The signing key is used to validate webhooks from Stripe and protect against replay attacks.

##### Using  stripe-cli

To find your signing key execute the following command:

```bash
# don't forget to replace the key with your own
docker-compose run stripe-cli listen --api-key=$STRIPE_PRIVATE_KEY --print-secret
```

Make sure to update the .env with this new value.

##### Using the Stripe Dashboard (Uncommon)

If you are publishing the project to a public facing server, you can set up the
webhook on Stripe and find the signing key under `Developers > Webhooks`.

You'll want to direct to `/api/webhook/stripe/` and only need to select the
`checkout.session.completed` event.

#### positionstack

The positionstack API is used to resolve addresses to coordinates. You can find your API key
in the [positionstack dashboard](https://positionstack.com/dashboard). You'll need to create
your own account and use the free plan.

### Building and Running Produce Goose

If you have any issues starting the cluster, please refer to the [troubleshooting](#port-already-in-use) section.

```bash
docker-compose up
```

### Configuring The Database

**NOTE**: This step requires the **backend** container to be running with docker compose.

If you run into errors, see the [Connecting to the Database](#connecting-to-the-database) section.

To build the necessary tables we can run the following command:

```bash
docker-compose exec backend python manage db build
```

To seed the database with some initial dummy data we can run the following command:

```bash
docker-compose exec backend python manage db populate
```

In the future if you ever need to reset the database you can run the following command:

```bash
# -d drop all tables before building
# -p repopulate the database with dummy data after building
docker-compose exec backend python manage db build -dp
```

### Configuring Stripe Objects

**NOTE**: This step requires the **backend** container to be running with docker compose.

To create all the necessary the Stripe objects we need to run the following command:

```bash
docker-compose exec backend python manage stripe setup
```

### Restarting The Backend

Once you have initiated the database and Stripe objects you'll need to restart the backend:

```bash
docker-compose restart backend
```

### Known Issues

#### Connecting to the Database

If you encounter errors connecting to the database, especially in the
[Configuring The Database](#configuring-the-database) stage, the following
instructions should fix things:

1. Stop the docker compose process, either by pressing `Ctrl+C` or by running
   `docker-compose down` if you launched the stack in detached mode
   (`docker-compose up -d`).
2. Delete the `/postgres` folder in the root of the project. Either by running
   `rm -rf postgres` or by deleting the folder in your Finder/file explorer.
3. Run `docker-compose down` once more
4. Run `docker-compose up` to start the stack again.

#### Port Already In Use

If you get errors from docker saying that a port is already in use, make sure
there are no other docker containers or processes running on ports
3000, 5432, 8080, or 8000.

Once you've ensured that there are no other processes or containers running on those ports,
then retry `docker-compose up`.

## Stripe

### Testing Credit Cards

When checking out, you can use the following credit cards to test the payment flow:

| Card Number         | CVC          | Expiration Date | Description |
|---------------------|--------------|-----------------|-------------|
| 4242 4242 4242 4242 | Any 3 digits | Any future date | Succeeds    |
| 4000 0000 0000 9995 | Any 3 digits | Any future date | Fails       |

## Rebuilding Containers (Not In Development Context)

If you make changes to the contents of the frontend or backend folders, or pull new
changes, you'll need to rebuild the docker containers in which changes occurred.

To rebuild the frontend container, run the following command:

```bash
docker-compose build frontend
```

To rebuild the backend container, run the following command:

```bash
docker-compose build backend
```

After rebuilding you'll need to restart all the docker containers:

```
docker-compose down
docker-compose up
```

### Pulling New Changes

If there are new changes upstream, you pull the changes with:

```bash
git pull
git submodule foreach git checkout main
git submodule foreach git pull
```

And then follow the [Rebuilding Containers](#rebuilding-containers-not-in-development-context)
instructions to rebuild the modified repository containers.

## Development

### Hybrid Mode (Recommended)

The hybrid mode is the recommended way to develop the frontend and backend. Due to performance restrictions,
developing with a dockerized Nuxt container was found to be quite slow.

Hybrid mode will spin up the postgres, stripe-cli, nginx, and backend (complete with hot reloading) containers.
And allow you to run the frontend locally on port 3000.

To start the hybrid mode, run the following command:

```bash
docker-compose -f docker-compose.yml -f docker-compose.hybrid.yml up
```

Make sure the `.env` file in your frontend folder is configured as follows:

```bash
NUXT_BASE_URL=http://localhost:8080/api
NUXT_PUBLIC_BASE_URL=http://localhost:8080/api
```

### 100% Docker (Not Recommended)

If you want to run the entire development context in docker, you can do so by running the following command:

```bash
docker-compose -f docker-compose.yml -f docker-compose.dev.yml up
```

The only caveat is that the frontend will feel considerably slower tha the
[hybrid alternative](#hybrid-mode-recommended), and you'll often have to
reload the page as the hot reloading is a little hit-and-miss.

## The End

Good luck,
 
May the Goose be with you.
