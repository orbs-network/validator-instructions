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

Your infura account is by default open to anyone with access to your Infura Project ID. This ID is not readily known but is also not kept secret. Since your free account has
daily API request limits, you may choose to add a layer of protection to your API endpoint by requiring a secret to be included with each connection request.

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

### Testing your Ethereum Endpoint URL:

To test your Ethereum Endpoint is active and functioning execute this command in terminal (replacing [ETHEREUM_ENDPOINT_URL] with your url):
```bash
 curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["latest", false],"id":1}' [ETHEREUM_ENDPOINT_URL]
```

The resulting output should be a JSON object. Inspect the attribute `"number"`. This value represents the top and verify it is close in value to the current top block in Ethereum
