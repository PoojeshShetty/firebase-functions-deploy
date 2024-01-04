# Github actions to deploy firebase functions

In this repository, we have defined the functions deploy github workflow which deploys functions to your firebase account.

## How does it work

Before running the workflow, you will need a service account key to be added in the secret under the name GOOGLE_APPLICATION_CREDENTIALS. You can follow the [article](https://medium.com/@shettypoojesh/deploy-firebase-functions-using-github-actions-in-node-db5b955dddeb) to understand how to generate one.

Just a quick step that you will have to follow

1. Create a service account and give relevant permissions to it.
2. Download the keys
3. Copy the json contents are string using the command tool jq

   > cat <account-key-file.json> | jq -c

4. Paste the contents in the github actions secret variable with name as GOOGLE_APPLICATION_CREDENTIALS

Now moving forward, now lets break down the each steps used in the work flow

```
- uses: actions/checkout@v3
```

This step checks out to the current running branch of the repository

```
- name: Use Node.js ${{ matrix.node-version }}
    uses: actions/setup-node@v2
    with:
        node-version: ${{ matrix.node-version }}
    cache: "yarn"
```

Here we are setting up the node with node-version defined at the top of the workflow of value 18.x

```
- name: create-json
id: create-json
uses: jsdaniell/create-json@1.1.2
with:
    name: "credentials.json"
    json: "${{ secrets.GOOGLE_APPLICATION_CREDENTIALS }}"

```

With this step we are taking the secret value and creating a json file credentials.json

```
- name: Set environment variable with path to credentials.json
    run: echo "GOOGLE_APPLICATION_CREDENTIALS=$(readlink -f credentials.json)" >> $GITHUB_ENV
```

Now, we are exporting the file path with the GOOGLE_APPLICATION_CREDENTIALS variable name. We need this step so that firebase can pick up the file to get access to running the deployment.

```
 - name: Install dependencies
        run: yarn
```

We install the package.json dependencies here.

```
- name: Install firebase tools
    run: yarn global add firebase firebase-tools
```

Here, we are globally installing the firebase and firebase-tools in order to run the firebase command in the next step

```
- name: Deploy functions
    run: cd functions && yarn run deploy
```

This is the final step where we are running the deploy script under the functions folder. If you want to debug it, then you can add the `--debug` parameter at the end of the `yarn run deploy` command i.e. `yarn run deploy --debug`, to view the logs when the command is being run. This will help you to see any errors if any type of failure occurs.
