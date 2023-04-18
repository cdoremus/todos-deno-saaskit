# SaaSKit Production Deployment

## Prereqs

- This section (SasSKit Production Deployment) assumes that a local development
  environment has been setup and tested
  - Supabase DB and Auth has been done
  - Stripe has been setup and configured
  - **TODO:** There might be some Prod-specific configurations that need to be
    done

## Authentication

### Create a Supabase account

- Create the account

### Create and Populate a Supabase Database

- The app's data tables
  - todos table
  - customers table
- Click on the Database button on the Supabase dashboard
- Select Tables icon -> Create New Table
- Create todos and cusomers tables

### Setting up Supabase Auth

- Authenticate with email and password
  - Deno SaaSKit currently supports email, but there are other auth strategies
    that we plan on supporting in the future
  - Email authentication is configured by default when you create a new project.
  - Email auth restricts each user to only create, read, update and delete their
    data
  - Create Policies
    - Customers table (links the built-in auth users table with your app's
      customers table)
      - Configuration -> Policies
      - New Policy -> Customers table
      - Enter a policy name ("Enable all operations for users based on user_id")
      - Allowed Operations -> select All
      - Targeted Roles -> select Authenticated
      - USING expression: (auth.uid() = user_id)
      - WITH CHECK expression: (auth.uid() = user_id)
      - Click Review -> Save Policy
    - Todos table
      - Same policy configuration as the customers table

## Payments

### Create a Stripe Account

- Stripe CLI account creation does it manually

### Add a bank to your Stripe account

- Select a Bank from the list on
  https://dashboard.stripe.com/account/onboarding/bank-account
- Login to your bank's online account and authorize an account for use by Stripe

### Install and activate the Stripe CLI

- Follow install instructions on https://stripe.com/docs/stripe-cli
- Run `stripe login` on the command-line and complete the login in your browser
  with key exchange
- Once setup, authentication lasts for 90 days

## Activate a Stripe Premium Account

- **IGNORE FOR NOW:** Asher's (iuioiua) recent PR automates this by invoking
  `deno task init:stripe` (See https://github.com/denoland/saaskit/pull/93)

  1. Automatically creates the "Premium tier" product.
  2. Automatically creates the default billing portal configuration used in the
     /dashboard/manage-subscription route.
  3. Prints the "Premium tier" product price ID to be copied into constants.ts.
- Manual activation of the Premium tier using the CLI

```bash
stripe products create --name="Premium tier" --default-price-data.unit-amount=500 --default-price-data.currency=usd --default-price-data.recurring.interval=month --description="Unlimited todos"
```

## Deno Deploy

### Automatic Deployment using a Github Link

- Activate Deno Deploy app in Github
  - Go to personal account
  - Select Applications -> Deno Deploy
  - Select Repository: Select repositories -> Choose your project's repo
- Activate the link to the Deno app in the DD dashboard
  - Go to the Deno dashboard and create a new project
- Select Automatic deployment
  - Settings -> Git Integration
  - Select the repo and entry file (`main.ts`)

### Deployment using a Github Action

- Create a `deploy.yml` file in `.github/workflows` of your repo with this
  content:

```yml
# Deploy this project to Deno Deploy
name: Deploy
on: [push]

jobs:
  deploy:
    name: Deploy
    runs-on: ubuntu-latest
    permissions:
      id-token: write # Needed for auth with Deno Deploy
      contents: read # Needed to clone the repository

    steps:
      - name: Clone repository
        uses: actions/checkout@v3

      - name: Install Deno
        uses: denoland/setup-deno@main
        # with:
        #   deno-version: 1.32.4

#      - name: Build site
#        run: deno run -A ./build.ts

      - name: Upload to Deno Deploy
        uses: denoland/deployctl@v1
        with:
          project: deno-saaskit-todos # deploy project name
          entrypoint: main.ts
          # root: dist
          import-map: import_map.json
          exclude: .git/** .gitignore .vscode/** .github/** README.md .env .example.env

          # dry-run: true
```

- Configure Supabase and Stripe environmental variables found in the `.env` file
  to values used in production in the Deno Deploy Dashboard
  - Deno Deploy Dashboard -> select your SaaSKit project -> Settings ->
    Environmental Variables
  - Add the names and values of the variables in the `.env` file
    - TODO: Change to prod-specific values

## Deploy to a Virtual Private Server

See https://deno.com/blog/npm-and-deno-anywhere

### Setting up Docker

- Create a Docker Hub account if you don't have one

- Create a `Dockerfile` for your project
  - services needed for both Supabase and Stripe
- Create `.dockerignore` file to exclude files and directories you don't want
  deployed
  - `README.md`, `.env`, `.example.env`, `LICENSE` files
  - `.vscode`, `.git`, `.github` folders

### AWS Lightsail

- Create an AWS account if you don't have one

- Create a AWS Lightsail `docker-compose.yml`

- Goto AWS Lightsail console

- Click Containers -> Create container service
  - Half way down the page, click "Setup your first Deployment" and select
    "Specify a custom deployment".

- Enter a container name of your choosing

- In Image, be sure to use {{ username }}/{{ image }} that you have set in your
  Docker Hub. For this example, it is lambtron/deno-on-aws-lightsail.

- Click "Add open ports" and add the value 8000

- In Public Endpoint, select your container name

- Show screenshot of filled-out "Setup your first deployment" form

- Click "Create container service"

- After deployment is completed, click on the public address link to see your
  app in the browser

### Digital Ocean

- Create an Digital Ocean account if you don't have one
- Install the doctl CLI

- Create a DO `docker-compose.yml`

- Build, Tag, and Push your Docker image to Digital Ocean Container Registry

- Deploy to Digital Ocean via SSH
