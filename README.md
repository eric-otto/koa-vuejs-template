# Koa / Vue.js SPA Template App

This app is a template application using Koa for a REST/JSON API server and Vue.js for a web client.

## Overview of Stack
- Server
  - Koa
  - PostgresSQL
  - [TypeORM](https://github.com/typeorm/typeorm)
  - Docker used for development PostgresSQL database and MailCatcher server
- Client
  - Vue.js
  - Webpack for asset bundling and HMR (Hot Module Replacement)
  - CSS Modules
  - Fetch API for REST requests
- Testing
  - Mocha
  - MailCatcher for development email delivery
- DevOps
  - Ansible playbook for provisioning (Nginx reverse proxy, SSL via Let's Encrypt, PostgresSQL backups to S3)
  - Ansible playbook for deployment

## Development Setup

1. Install the following:
   - [Node.js >= v7.8.0](https://nodejs.org/en/download/)
   - [Ansible >= 2.3.1.0](http://docs.ansible.com/ansible/intro_installation.html) (`pip install ansible --upgrade` to upgrade)
   - [Docker](https://docs.docker.com/engine/installation/)
2. Run `npm install && npm start`
3. Open browser and navigate to [http://localhost:5000](http://localhost:5000).

## Scripts

### `npm install`

When first cloning the repo or adding new dependencies, run this command.  This will:

- Install Node dependencies from package.json

### `npm start`

To start the app for development, run this command.  This will:

- Run `docker-compose up` to ensure the PostgreSQL and MailCatcher Docker images are up and running
- Run dotnet watch run which will build the app (if changed), watch for changes and start the web server on http://localhost:5000
- Run Webpack dev middleware with HMR

### `npm run migrate`

After making changes to Entity Framework models in `api/Models/`, run this command to generate and run a migration on the database.  A timestamp will be used for the migration name.

### `npm test`

This will run the xUnit tests in api.test/ and the Vue.js tests in client-web.test/.

### `npm run provision:prod`

 _Before running this script, you need to create an ops/hosts file first.  See the [ops README](ops/) for instructions._

 This will run the ops/provision.yml Ansible playbook and provision hosts in ops/hosts inventory file.  This prepares the hosts to recieve deployments by doing the following:
  - Install Nginx
  - Generate a SSL certificate from [Let's Encrypt](https://letsencrypt.org/) and configure Nginx to use it
  - Install Node.js
  - Install Supervisor (will run/manage the Node.js/Koa app)
  - Install PostgreSQL
  - Setup a cron job to automatically backup the PostgresSQL database, compress it, and upload it to S3.
  - Setup UFW (firewall) to lock everything down except inbound SSH and web traffic
  - Create a deploy user, directory for deployments and configure Nginx to serve from this directory

### `npm run deploy:prod`

_Before running this script, you need to create a ops/hosts file first.  See the [ops README](ops/) for instructions._

This script will:
 - Build release Webpack bundles
 - Package the Node.js/Koa application in Release mode (dotnet publish)
 - Run the ops/deploy.yml Ansible playbook to deploy this app to hosts in /ops/hosts inventory file.  This does the following:
  - Copies the build assets to the remote host(s)
  - Updates the `appsettings.json` file with PostgresSQL credentials specified in ops/hosts file and the app URL (needed for JWT tokens)
  - Restarts the app so that changes will be picked up

## Development Email Delivery

This template includes a [MailCatcher](https://mailcatcher.me/) Docker image so that when email is sent during development (i.e. new user registration), it can be viewed
in the MailCacher web interface at [http://localhost:1080/](http://localhost:1080/).

## Visual Studio Code config

This project has [Visual Studio Code](https://code.visualstudio.com/) tasks and debugger launch config located in .vscode/.

### Tasks

- **Command+Shift+B** - Runs the "build" task which builds the api/ project
- **Command+Shift+T** - Runs the "test" task which runs the xUnit tests in api.test/ and Mocha/Enzyme tests in client-web.test/.

### Debug Launcher

With the following debugger launch configs, you can set breakpoints in api/ or the the Mocha tests in client-web.test/ and have full debugging support.

- **Debug api/ (server)** - Runs the vscode debugger (breakpoints) on the api/ Node.js/Koa app
- **Debug client-web.test/ (Mocha tests)** - Runs the vscode debugger on the client-web.test/ Mocha tests

## Credit

The following resources were helpful in setting up this template:

- [Sample for implementing Authentication with a React Flux app and JWTs](https://github.com/auth0-blog/react-flux-jwt-authentication-sample)
- [Angular 2, React, and Knockout apps on ASP.NET Core](http://blog.stevensanderson.com/2016/05/02/angular2-react-knockout-apps-on-aspnet-core/)
- [Setting up ASP.NET v5 (vNext) to use JWT tokens (using OpenIddict)](http://capesean.co.za/blog/asp-net-5-jwt-tokens/)
- [Cross-platform Single Page Applications with ASP.NET Core 1.0, Angular 2 & TypeScript](https://chsakell.com/2016/01/01/cross-platform-single-page-applications-with-asp-net-5-angular-2-typescript/)
- [Stack Overflow - Token Based Authentication in ASP.NET Core](http://stackoverflow.com/questions/30546542/token-based-authentication-in-asp-net-core-refreshed)
- [SPA example of a token based authentication implementation using the Aurelia front end framework and ASP.NET core]( https://github.com/alexandre-spieser/AureliaAspNetCoreAuth)
- [A Real-World React.js Setup for ASP.NET Core and MVC5](https://www.simple-talk.com/dotnet/asp-net/a-real-world-react-js-setup-for-asp-net-core-and-mvc)
- My own perseverance because this took a _lot_ of time to get right 😁
