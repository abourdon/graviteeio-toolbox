# [Gravitee.io API Management](https://gravitee.io/products/apim/) CLI commands

## Prerequisites 

Have a ready to run [NodeJS](https://nodejs.org/en/) and [NPM](https://www.npmjs.com/) based environment.
To install yours, [NVM](https://github.com/nvm-sh/nvm) could be a good option:

```bash
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash
$ nvm install 12.0
```

> **Note:** At the time of writing, the latest NVM version is v0.34.0. Feel free to update it according to the current latest one.

Finally, install the desired dependencies:

```bash
$ npm install
```

## CLI command execution

In addition to be executed directly from the [gravitee-cli.sh](../gravitee-cli.sh) CLI, any command is actually a NodeJS script that can be executed as follow:

```bash
$ node <script>.js [OPTIONS]
```

For instance:

```bash
$ node list-apis.js --url <APIM URL> --username <APIM username> --password <APIM password> --query-filter products
```

For more details about `[OPTIONS]`, ask for help:
```bash
$ node <script>.js -h
```

## CLI command list

Here are existing commands :
- [`count-apis.js`](./count-apis.js): Count the number of APIs based on user search predicate.
- [`create-endpoint.js`](./create-endpoint.js): Create an API endpoint.
- [`create-endpoint-group.js`](./create-endpoint-group.js) : Create an endpoint group for APIs that match user predicate.
- [`enable-endpoints.js`](./enable-endpoints.js) : Enable (or disable) API endpoints based on user predicate.
- [`enable-logs.js`](./enable-logs.js) : Enable (or disable) detailed logs on APIs that match user predicate.
- [`extract-api-quality.js`](./extract-api-quality.js) : Extract API quality compliance as CSV content.
- [`import-api.js`](./import-api.js) : Import existing API (update) depending a search. Import only if search returns exactly one result.
- [`list-activated-logs-apis.js`](./list-activated-logs-apis.js) : List all APIs with activated detailed logs. 
- [`list-apis.js`](./list-apis.js) : List all registered APIs by displaying their name and context path.
- [`list-apis-quality.js`](./list-apis-quality.js) : List all registered APIs by displaying their name, context path and quality.
- [`list-applications.js`](./list-applications.js) : List all registered Applications by displaying their name, context path and quality.
- [`list-inactive-ldap-users.js`](./list-inactive-ldap-users.js): List inactive LDAP users.
- [`list-labels.js`](./list-labels.js) : List labels defined on APIs.
- [`list-non-subscribed-apis.js`](./list-non-subscribed-apis.js) : List all APIs with no active subscription by displaying their name, context path, owner name and owner email.
- [`list-non-subscribed-applications.js`](./list-non-subscribed-applications.js) : List all applications with no active subscription by displaying their name, owner name and owner email.
- [`transfer-ownership.js`](./transfer-ownership.js): Transfer ownership for APIs or applications.

## CLI command development example

```js
const CliCommand = require('./cli-command');
const { flatMap } = require('rxjs/operators');
const util = require('util');

const NO_DELAY_PERIOD = 0;

/**
 * List all registered APIs by displaying their name, context path, owner name and owner email.
 *
 * @author Aurelien Bourdon
 */
class ListApis extends CliCommand {

    constructor() {
        super(
            'list-apis',
            {
                'filter-by-name': {
                    describe: "Filter APIs against their name (insensitive regex)",
                    type: 'string'
                },
                'filter-by-context-path': {
                    describe: "Filter APIs against context-path (insensitive regex)",
                    type: 'string'
                },
                'filter-by-primary-owner': {
                    describe: "Filter APIs against its primary owner name or address (insensitive regex)",
                    type: 'string'
                },
                'filter-by-endpoint-group-name': {
                    describe: "Filter APIs against endpoint group name (insensitive regex)",
                    type: 'string'
                },
                'filter-by-endpoint-name': {
                    describe: "Filter APIs against endpoint name (insensitive regex)",
                    type: 'string'
                },
                'filter-by-endpoint-target': {
                    describe: "Filter APIs against endpoint target (insensitive regex)",
                    type: 'string'
                },
                'filter-by-plan-name': {
                    describe: "Filter APIs against plan name (insensitive regex)",
                    type: 'string'
                },
                'filter-by-policy-technical-name': {
                    describe: 'Filter APIs against their policy technical names (insensitive regex) (see https://docs.gravitee.io/apim_policies_overview.html for more details)'
                }
            }
        );
    }

    definition(managementApi) {
        managementApi
            .login(this.argv['username'], this.argv['password'])
            .pipe(
                flatMap(_token => {
                    return this.hasBasicsFiltersOnly() ?
                        managementApi.listApisBasics({
                            byName: this.argv['filter-by-name'],
                            byContextPath: this.argv['filter-by-context-path'],
                            byPrimaryOwner: this.argv['filter-by-primary-owner']
                        }, NO_DELAY_PERIOD) :
                        managementApi.listApisDetails({
                            byName: this.argv['filter-by-name'],
                            byContextPath: this.argv['filter-by-context-path'],
                            byEndpointGroupName: this.argv['filter-by-endpoint-group-name'],
                            byEndpointName: this.argv['filter-by-endpoint-name'],
                            byEndpointTarget: this.argv['filter-by-endpoint-target'],
                            byPlanName: this.argv['filter-by-plan-name'],
                            byPolicyTechnicalName: this.argv['filter-by-policy-technical-name']
                        });
                }),
            )
            .subscribe(this.defaultSubscriber(
                api => this.displayRaw(util.format('[%s, %s, %s <%s>] %s',
                    api.id,
                    api.context_path,
                    api.owner.displayName,
                    api.owner.email,
                    api.name
                ))
            ));
    }

    hasBasicsFiltersOnly() {
        return Object.keys(this.argv)
            .filter(argv => argv.startsWith("filter-by")
                && argv !== 'filter-by-name'
                && argv !== 'filter-by-context-path'
                && argv !== 'filter-by-primary-owner'
            ).length === 0;
    }
}

new ListApis().run();
```

## Add your own CLI command

All the technical stuff is handled by the [`CliCommand`](lib/cli-command.js) class. Then to add your own CLI command, you just have to inherit from it and only define the specific part of your command (i.e., its name and process definition by overriding the associated methods as shown above).