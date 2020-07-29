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


### Testing your Ethereum Endpoint URL:

To test your Ethereum Endpoint is active and functioning execute this command in terminal (replacing [ETHEREUM_ENDPOINT_URL] with your url):
```bash
 curl -X POST --data '{"jsonrpc":"2.0","method":"eth_getBlockByNumber","params":["latest", false],"id":1}' [ETHEREUM_ENDPOINT_URL]
```
The resulting output should be a JSON object. Verify there is no attirbute named `"Error"`. Additionally, you may inspect value of the attribute `"number"`. This value is a hexadecimal representation of the top block number in Ethereum blockchain. Verify your Ethereum node is in sync with the network by comparing this value to the current top block in Ethereum.


