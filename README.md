## INTRODUCTION ##

**maxiPago!** offers it's merchants and developers an innovative way to integrate your checkout page with our payment platform: a **Lightbox** lib.

When called, the _**maxiPago.js**_ lib opens a modal window displaying the payment form ready to be filled by the buyer. It sends the payment directly to **maxiPago!**'s servers, with no credit card information being handled by the merchant.

It allows a PCI compliant integration that does not require your buyer to leave your website.

![maxiPago! Lightbox](https://raw.github.com/maxipago/Lightbox-Integration-lib/master/test/lightbox.png)

## INSTALLATION ##

The Lightbox lib contains the following files:

```
  /lib/  
  |-- maxiPago-1.2.js  
  |-- jquery-1.7.2.min.js   
  /maxipago/  
    |-- maxipago.css  
    |-- config.xml  
    |-- dictionary.xml
    |-- Return.htm  
```

Insert the **maxiPago!** and **jQuery 1.7.2+** files in your page, right before the end of the **&lt;/body&gt;** tag to allow for faster page loading:

```html
...
<script src="/path/to/directory/lib/jquery-1.7.2.min.js"></script>
<script src="/path/to/directory/lib/maxiPago-1.2.js"></script>
</body>
```

Please note that the directory structure of the lib must remain intact. The **maxiPago.js** will always look for it's config files inside /maxipago/ in the same directory it is installed at.

The **Return.htm** file also needs to be placed in your server, though anywhere you like. It is the Postback page the Lightbox uses to receive the transaction response.

Since this is a browser-based solution all files must be in a public directory.

## SETUP ##

The Lightbox settings are set in the **config.xml** file and are loaded along with your checkout page. This means you can change settings without restarting services or servers. 

The **config.xml** file contains the following settings:

* **merchantId:** Merchant ID assigned by maxiPago!
* **server_url:** Target URL assigned by maxiPago!
* **server\_url\_response:** Postback URL. This must be the address were the **Return.htm** file is being hosted
* **form_width:** Modal width in pixels
* **form_height:** Modal height in pixels
* **showRefNum:** Boolean the shows or hides the Reference Number ( _**hp\_refnum**_ ) from the Lightbox
* **showAmount:** Boolean the shows or hides the Order Amount ( _**hp\_amount**_ ) from the Lightbox


## CSS AND LABELS ##

The **maxiPago!** Lightbox allows full customization of it's look-and-feel and the text shown to the buyer.

* **maxipago.css**: Contains all the CSS classes used by the Lightbox to change how elements are displayed
* **dictionary.xml**: Contains all the text labels shown in the Lightbox

The details of how to change the default settings in the comments of each config file.


## REQUEST ##

```
Please notify our Support team if you intend to use the Lightbox integration, as  
there are settings we need to change before allowing you to make successful calls
```

To call the Lightbox you must pass a _**struct**_ in the **$MP.Buy()** method, as such:

```js
	var transaction = {
		hp_processor_id: "1",
		hp_txntype: "sale",
		hp_method: "ccard",
		hp_amount: "10.00",
		hp_refnum: "ORD1028307193",
		hp_sig_timestamp: "1312201855",
		hp_sig_id: "1234",
		hp_sig_itemid: "1028307193",
		hp_signature: "0c067be27be8e6cc092b7fe23bdb9b77"
	};
	$MP.Buy(transaction);
```

### Required fields ###

* **hp\_processor\_id**: Processor or acquirer code. Use **1** for testing
* **hp_method**: Always **ccard**
* **hp_txntype**: Authorization ( _**auth**_ ) or Sale ( _**sale**_ )
* **hp_amount**: Transaction amount with decimals (10.00)
* **hp_refnum**: Merchant reference number for this transaction
* **hp\_sig\_timestamp**: Unix time of the transaction, used for hash signature validation
* **hp\_sig\_id**: Random **4 digit** number user for hash signature validation
* **hp\_sig\_itemid**: Order code used for hash signature validation.
Must be **different** than _**hp_refnum**_
* **hp_signature**: HMAC-MD5 hash key (detailed below)

### Hash signature ###

To ensure the integrity of the transaction data we use a signature, a unique key that validates if the data received has not been altered or tampered.

This signature is an HMAC-MD5 hash based on the transaction information and a secret key that only **maxiPago!** and Merchant knows.

The transaction information must be concatenated in a specific order and the values must be separated by an **asterisk**. For more information and examples see [Hmac-Implementation](http://github.com/maxipago/Hmac-Implementation).


### Optional fields ###

* **hp_currency**: ISO 4217 currency code
* **hp\_number\_of\_installments**: Number of installments to divide the transaction. If regular/one-off transaction, **do not send**
* **hp_baddr**: Billing address 1
* **hp_baddr2**: Billing address 2
* **hp_bcity**: Billing city
* **hp_bstate**: Billing state, 2 letter format
* **hp_bzip**: Billing ZIP code
* **hp_bcountry**: Billing country, 2 letter format (ISO 3166-2)
* **hp_phone**: Customer phone number
* **hp_email**: Customer email address

### Saving a credit card number ###

**maxiPago!** gives you the alternative to save the customer's credit card number and return you a token, to be used for future purchases with the same card. In other to do so, the following fields are required:

* **hp\_save\_customer**: Boolean to create new profile (1 or 0)
* **hp_savepayment**: Boolean to save card under new profile (1 or 0)
* **hp\_c\_customer_id**: Customer profile ID (must be unique)
* **hp\_c\_addr**: Credit card billing address 1
* **hp\_c\_addr2**: Credit card billing address 2
* **hp\_c\_city**: Credit card billing city
* **hp\_c\_state**: Credit card billing state, 2 letters
* **hp\_c\_zip**: Credit card zip code
* **hp\_c\_country**: Credit card billing country, 2 letters (ISO 3166-2)
* **hp\_c\_phone**: Credit card billing phone
* **hp\_c\_email**: Credit card billing email address


## RESPONSE ##

When the Lightbox receives a response from **maxiPago!** it displays a Success or Fail message. Once the buyer closes the modal window the lib calls the **respMPLib()** method.

The **respMPLib()** method should be in your checkout page and you must use it to handle the payment response according to your payment flow. The **$MP.resp** object contains the transaction response fields and calling **$MP.resp.fieldname** will get it's value:

```js
function respMPLib() {
	if ($MP.resp.hp_responsecode == "0") { alert("Your payment was Approved"); }
	else { alert("Sorry, your payment has been declined"); }
}
```

### Response fields ###

* **hp_responsecode**: Transaction status code. **Success = 0**
* **hp_responsemsg**: Error message, if any
* **hp_time**: Date and time of the transaction
* **hp_refnum**: Confirmation of the value sent in the request
* **hp_transid**: Transaction ID created by **maxiPago!**. _**This must be stored for future use**_
* **hp_orderid**: Order ID created by **maxiPago!**. _**This must be stored for future use**_
* **hp_authcode**: Acquirer authorization code
* **hp_amount**: Confirmation of the amount sent in the request
* **hp_processortxnid**: Acquirer's transaction ID
* **hp_processorrefno**: Acquirer's reference number
* **hp\_fraud\_score**: Score value returned by the antifraud system, if available
* **hp\_customer\_token**: Customer ID created by **maxiPago!** if saving a card automatically
* **hp\_payment\_token**: Credit card token created by **maxiPago!** if saving a card automatically
* **hp\_signature\_response**: Hash signature  of the reply.


## SECURITY AND COMPLIANCE ##

This Lightbox was designed to be PCI compliant, so it does not store or log credit card information in any way. It also prevents you, the merchant, from having access to the card data.

While the **maxiPago.js** encrypts the transaction data before posting them to **maxiPago!**, in order to be PCI compliant, the Lightbox **must be place in a secure environment (HTTPS)**.

## SUPPORT ##

Feel free to contact our Support team if you have any questions about the **maxiPago!** Lightbox integration or about our payment platform in general. They can be reached at support@maxipago.com.

## LICENSE ##
Copyright 2013 maxiPago!

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at  
  
http://www.apache.org/licenses/LICENSE-2.0  
  
Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.