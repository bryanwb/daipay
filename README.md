**DaiPay** is an open-source payment processor which allows you to receive Maker DAI tokens for payments.

### Motivation
I've been involved with payment processing in Bitcoin in some way for a long time. And, while i love the idea of Bitcoin in general, i think Bitcoin is too impractical for use by merchants because of its price volatility and that stablecoins will dominate payment processing eventually. To me Maker DAI seems to be an ideal electronic cash that has all chances to be widely adopted by merchants worldwide. The project is in its very infant stage, so do not expect much from it yet. Spreading the word, suggestions and contributing are welcome.

### Features
- Direct P2P payments
- No volatility risk as DAI stablecoin is tied to US dollar 1:1
- Complete control over private keys
- No middleman
- REST API for managing invoices
- Web interface for accessing invoices

### Requirements
- Node v10 or newer
- MongoDB server
- Access to the Ethereum network

## Installation

Run the below commands:

        git clone https://github.com/codevet/daipay.git
        cd daipay
        npm install



## Configuration

DaiPay needs to connect to the Ethereum network to process transactions. You have two options:

- Connect to a locally running Ethereum node (geth or parity)
- Use a third-party node service provider such as [VNode.app](https://vnode.app)

Open config/default.cson in a text editor and make necessary changes:

  1. If you are using an external node provider, update the provider section

    provider:
      type: "rpc"
        uri: "http://localhost:8545" # replace with an endpoint URL from an external provider


  2. Change the apiKey to some unique string. The API key is used to post invoices to DaiPay.


## Running DaiPay

To run the server in the development mode, use the following command:

        npm run dev 

For the production mode:

        npm run build
        npm start

By default, DaiPay connects to 127.0.0.1:27017/daipay. You can override the default connection settings by using MONGO_URL environment variable

        MONGO_URL=mongodb://user:password@dbhost:dbport/daipay npm start






## API

Invoice object has the following properties:

  - apiKey - The API key to verify that the invoice creator is trusted (it not stored in the database)
  - currency - Currency of the invoice. Only DAI is supported at the moment. (optional)
  - merchant - The information about the merchant (optional)
       - name - The merchant’s name
       - address - The address to be displayed in the invoice
  - items - an array containing the invoice items. DaiPay calculates totalAmount for all items and stores it with the invoice.
       -  description - item description
       -  quantity - number of units to be purchased (optional, default is 1)
       -  amount - the unit cost  
  - metadata - contains arbitrary data which is passed back to the app via the callback (see below). (optional)
  - expires - a timestamp until the invoice is valid (optional, default is 24 hours after the invoice was created)
  - callbacks - specifies where DaiPay server should send a POST request containing  token and metadata for notifying the app that the payment occurs.
       - token - a secret token to verify that the callback is originated from DaiPay server
       - paid  
            - url  - where POST request should be sent to

Invoices can be created by sending a POST request to the /api/v1/invoice endpoint

An example with curl (replace API_KEY with the apiKey from the config file):

        curl http://localhost:8000/api/v1/invoice --header "Content-Type: application/json" -X POST -d '{"apiKey":"API_KEY","items":[{"description":"My item","amount":1}]}'

The backend will create a new invoice record and return its id

        {"invoiceId":"bfuWuYgRs"}


The invoice can be viewed in a browser at http://localhost:8000/invoice/bfuWuYgRs

![DaiPay invoice](https://i.imgur.com/EGsJTPe.png)

Also, the invoice record can be accessed via REST API:

        curl http://localhost:8000/api/v1/invoice/bfuWuYgRs


        {"deposit":{"_id":"xm-8-8lThd","address":"0xE6F2392Fe8ED75f684cb93fA0e278f0E404a8522"},"_id":"bfuWuYgRs","items":[{"description":"item #1","amount":1}],"totalAmount":1,"paidAmount":0,"created":1548735086893,"state":"pending","__v":0}

Upon creation, an invoice gets linked with a deposit address generated by DaiPay server. Private keys are stored in the depositaddresses MongoDB collection in plain text format.

**IMPORTANT**: make sure your MongoDB instance is properly secured and regularly backed up as private keys in the only way to access collected payments.

Newly created invoices have *pending* state.  After an invoice is paid its state will be changed to *confirming*. Then the server waits for the number of blocks specified in the configuration as minConfirmations (default is 1) and if thereafter the invoice balance is not less than totalAmount, the server changes the invoice state to *paid* and notifies the app via callback that the payment was done. The app should return HTTP code 200 to acknowledge that the invoice was processed. Then the invoice status is changed to *closed*.

If the invoice was not paid on time, as configured by the *expires* property, its status will be set to *expired* and no callbacks will be called thereafter.


## TODO
- Adding a project logo
- Script for collecting and forwarding payments from deposit addresses


## CREDITS
- Inspired by [Baron](https://github.com/baronpay/baron) project
