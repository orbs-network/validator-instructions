# Obtaining an Infura free account Ethereum Endpoint URL

### Creating a project and extracting an Ehtereum Endpoint URL 
To create an Infura project for your Orbs Node Ethereum access, follow these instructions:

1. Navigate to `http://infura.io`

1. If you don't already have an account, press _SIGN UP_ at the top righthand side of the page, and follow the registration process up to and including email confirmation. 
If you already have an account, sign in.

1. Following sign-in or registration you should be redirected to Infura Dashboard page

1. Click the _ETHEREUM_ logo and link, second from the top on the lefthand side navigation bar.

1. Click _CREATE A PROJECT_, and give your beta project a name (e.g. orbs-node-beta). Then click _CREATE_

You have successfully created an Infura free account and project. You should now be looking at the _SETTINGS_ page of your new project.

Your Ethereum Endpoint URL is the first address listed under _KEYS_ -> _ENDPOINTS_ section, starting in `https://mainnet.infura.io/...`.
If you are not interested in enabling secret protection for your Infura project you may copy and use this address as your Ethereum Endpoint URL in Orbs Node deployment.

(Notice, in the drop down list near the _KEYS_ -> _ENDPOINTS_ section title `MAINNET` should be selected)

### Enabling Secret protection (Optional)

Infura places a limit on the number of daily API requests your free account provides. This quota is enough for supporting the API calls your Orbs Node generates.
However, your Ethereum Endpoint URL may be used by anyone with access to your project ID. In order to block unauthorized use of this project, you may require attaching a secret key to all API requests served by this Infura project.

To enable secret protection for your Ethereum Endpoint URL do the following:

1. In your Project's _SETTINGS_ page, in the _SECURITY_ -> _PRIVATE SECRET REQUIRED_ section, tick the _REQUIRE PROJECT SECRET FOR ALL REQUESTS_ checkbox.

1. Adjust your Endpoint URL to include the secret:
  1. copy the string labeled _PROJECT SECRET_ in the _KEYS_ section.
  1. embedd your project secret into the endpoint URL obtained in the [previous section](#creating-a-project-and-extracting-an-ehtereum-endpoint-url): 
  ```
  original enpoint URL: https://mainnet.infura.io/...  
  Secret protected URL: https://:PROJECT_SECRET@mainnet.infura.io/...
  ```
_* note the `:` and `@` around your project secret should not be replaced_

Use the secret protected version of the Ethereum Endpoint URL as your Ethereum Endpoint URL in Orbs Node deployment.

Keep the secret-enriched URL and secure to protect your secret.

### Testing your Ethereum Endpoint URL:

To test your Ethereum Endpoint is active and functioning execute this command in terminal (replacing [ETHEREUM_ENDPOINT_URL] with your url):
```bash
 curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["latest", false],"id":1}' [ETHEREUM_ENDPOINT_URL]
```
The resulting output should be a JSON object. Verify there is no attirbute named `"Error"`. Additionally, you may inspect value of the attribute `"number"`. This value is a hexadecimal representation of the top block number in Ethereum blockchain. Verify your Ethereum node is in sync with the network by comparing this value to the current top block in Ethereum.

#### Checking Secret protection 

To verify you have correctly enabled secret protection repeat the test using the Endpoint URL obtained in the [first section](#creating-a-project-and-extracting-an-ehtereum-endpoint-url) and see that your get this error message:

```
{"jsonrpc":"2.0","error":{"code":-32002,"message":"rejected due to project ID settings"}}
```

