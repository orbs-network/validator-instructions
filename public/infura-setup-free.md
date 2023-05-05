# Obtaining an Infura free account Ethereum Endpoint URL

### Creating a project and extracting an Ehtereum Endpoint URL

To create an Infura project for your Orbs Node Ethereum access, follow these instructions:

1. Navigate to `http://infura.io`

1. If you don't already have an account, click _GET STARTED_ at the top righthand side of the page, and follow the registration process up to and including email confirmation.
   If you already have an account, sign in.

1. Following sign-in or registration you should be redirected to Infura Dashboard page

1. Click on _API Keys_ in the top navigation panel (if you aren't there already) and then click the "CREATE NEW API KEY" button.

1. Choose "**Web3 API**" for the _NETWORK_, and give your project a name (e.g. orbs-node). Then click _CREATE_.

You have successfully created an Infura free account and project. You should now be looking at the _SETTINGS_ page of your new project.

Your Ethereum Endpoint URL is the first address listed in the _ENDPOINTS_ section, starting with `https://mainnet.infura.io/...`.

(Ensure that the URL contains `MAINNET`).

### Testing your Ethereum Endpoint URL:

To test your Ethereum Endpoint is active and functioning execute this command in terminal (replacing [ETHEREUM_ENDPOINT_URL] with your url):

```bash
curl --url ETHEREUM_ENDPOINT_URL \
-X POST \
-H "Content-Type: application/json" \
-d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'
```

The resulting output should be a JSON object. Verify there is no attirbute named `"Error"`. Additionally, you may inspect value of the attribute `"number"`. This value is a hexadecimal representation of the top block number in Ethereum blockchain. Verify your Ethereum node is in sync with the network by comparing this value to the current top block on Ethereum (you can convert the hexadecimal to a decimal number using an online converter such as https://www.binaryhexconverter.com/hex-to-decimal-converter)
