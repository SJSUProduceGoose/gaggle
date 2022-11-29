# Gaggle

> /ˈɡaɡəl/
>
> `noun`
> 1. a flock of geese.
> 2. the docker compose file bringing together produce goose's backend and frontend repositories.

## Cloning

After cloning the repository we need to initialize the submodules:

```bash
git clone git@github.com:SJSUProduceGoose/gaggle.git
cd gaggle
git submodule init
git submodule update
```

## Configure The Environment

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
| STRIPE_PRIVATE_KEY    | The private key that will be used to connect to stripe. [See](#stripe)                                                                          | BYOK                                              |
| STRIPE_SIGNING_KEY    | The signing key to validate webhooks from Stripe. [See](#stripe)                                                                                | BYOK                                              |
| POSITIONSTACK_API_KEY | Used to resolve addresses to coordinates. [See](#positionstack)                                                                                 | BYOK                                              |


> BYOK: *Bring Your Own Key*

### Stripe

Stripe is used to process payments. You will need to create an account and generate a
private key and signing key. You can find more information [here](https://stripe.com/docs/keys).

#### Create a Stripe Account
1. Register [here](https://dashboard.stripe.com/register).
2. Go to the [Stripe dashboard](https://dashboard.stripe.com/test/dashboard) and click on
   `Developers` -> `API keys`.
3. Copy the `Secret key` and paste them in the `.env` file.
4. Set up the tax rates for your business. You can do so by going to the
   [Stripe Tax Rates Dashboard](https://dashboard.stripe.com/test/tax-rates).

#### Stripe Private Key

The private key is used to connect to the Stripe API. You can find this key in the Stripe
dashboard under `Developers > API Keys`.

#### Stripe Signing Key

The signing key is used to validate webhooks from Stripe and protect against replay attacks.

#### Using  stripe-cli

To find your signing key execute the following command:

```bash
# don't forget to replace the key with your own
docker compose run stripe-cli listen --api-key=$STRIPE_PRIVATE_KEY --print-secret
```

Make sure to update the .env with this new value.

#### Using the Stripe Dashboard

If you are publishing the project to a public facing server, you can set up the
webhook on Stripe and find the signing key under `Developers > Webhooks`.

You'll want to direct to `/api/webhook/stripe/` and only need to select the
`checkout.session.completed` event.

### positionstack

The positionstack API is used to resolve addresses to coordinates. You can find your API key
in the [positionstack dashboard](https://positionstack.com/dashboard). You'll need to create
your own account and use the free plan.

## Building and Running Produce Goose

```bash
docker-compose up
```

## Configuring The Database

**NOTE**: This step requires the backend container to be running with docker compose.

To build the necessary tables we can run the following command:

```bash
docker compose exec backend python manage db build
```

To seed the database with some initial dummy data we can run the following command:

```bash
docker compose exec backend python manage db populate
```

In the future if you ever need to reset the database you can run the following command:

```bash
# -d drop all tables before building
# -p repopulate the database with dummy data after building
docker compose exec backend python manage db build -dp
```

## Configuring Stripe Objects

**NOTE**: This step requires the backend container to be running with docker compose.*

To create all the necessary the Stripe objects we need to run the following command:

```bash
docker compose exec backend python manage stripe setup
```