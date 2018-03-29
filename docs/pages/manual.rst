.. _the-manual:

=========================
GPlay Integration Manual
=========================



GPlay IAP SDK	
--------------

This is an Android library for inapp purchases in GPlay based on
https://github.com/anjlab/android-inapp-billing-v3. Make sure you have
the latest GPlay application installed on your phone (if not do it on
https://gplay.io/. Open and press Download Android App button).


Getting Started
---------------

Download the latest **gc-android-sdk.aar** and import it to your Android application

::

    implementation(name:'gc-android-sdk', ext:'aar')

	
Firstly implement **IGPlayPurchaseCallback** interface to receive purchases
callbacks (you will find detailed information on the interface methods below):

::

	public class MainActivity extends Activity implements IGPlayPurchaseCallback{
	
For all interactions with our store you will use **GPlayPurchasing** class:

::

    private GPlayPurchasing gStorePurchasing;
	
	
Inside your main Activity onCreate use the following code to initialize SDK:

::

	gStorePurchasing.init(this, base64EncodedPublicKey, this);
	
or
	
::
	
	gStorePurchasing.init(this, base64EncodedPublicKey, new IGPlayPurchaseCallback() {
		...
	});


Use **queryInventory** method to fetch inapp data, sku ids should be
provided:

::

	@Override
	public void onBillingSupported() {
		List<String> skuList = new ArrayList<>();
		skuList.add(SKU_GAS);
		skuList.add(SKU_INFINITE_GAS);
		skuList.add(SKU_PREMIUM);
		try {
			gStorePurchasing.queryInventory(skuList);
		} catch (IabHelper.IabAsyncInProgressException e) {
			e.printStackTrace();
		}
	}
	
	String SKU_PREMIUM = "com.company.mygame.premium";
    String SKU_GAS = "com.company.mygame.gas";
    String SKU_INFINITE_GAS = "com.company.mygame.gas_infinite";

	

To start a purchase use **launchPurchaseFlow** maethod:

::

    public void onBuyButtonClicked(View view) {
        try {
            gStorePurchasing.launchPurchaseFlow(this, SKU_GAS, 10001, "payload", "externalOrderId");
        } catch (IabHelper.IabAsyncInProgressException e) {
            e.printStackTrace();
        }
    }



::

	launchPurchaseFlow(Activity act, String sku, int requestCode, String developerPayload, String externalOrderId)

Parameters for **launchPurchaseFlow** call are	

	* act - Activity context
	* sku - product id you are to buy
	* requestCode - code your Activity will be passed in onActivityResult callback
	* developerPayload - optional payload data you will get in a success purchase callback
	* externalOrderId - optional id of a payment transaction for external needs (analytics etc.)
    

When the purchase is processed you can consume it by calling:


::

	@Override
	public void onPurchaseSucceeded(String json, IabResult iabResult, Purchase purchase) {

		if (purchase.getSku().equals(SKU_GAS)) {
			gStorePurchasing.consumeProduct(purchase);
			...

::

	@Override
	public void onQueryInventorySucceeded(String json, IabResult iabResult, Inventory inventory) {
		List<Purchase> purchases = new ArrayList<>();
		Purchase gasPurchase = inventory.getPurchase(SKU_GAS);
		if (gasPurchase != null && verifyDeveloperPayload(gasPurchase)) {
			purchases.add(gasPurchase);
		}
		
		...

		gStorePurchasing.consumeProducts( purchases);			

Please use **consumeProducts** for bundle consumption instead of calling **consumeProduct** several times.
	


Within your Activity override **onActivityResult** method and forward
receiving callbacks to the GPlayPurchasing method.

::

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (!gStorePurchasing.onActivityResult(requestCode, resultCode, data)) {
            super.onActivityResult(requestCode, resultCode, data);
        } else {
            // Not a purchasing case. Your code goes here.
        }
    }


To enable/disable sdk logging use

::

    enableDebugLogging(boolean enabled)

    enableDebugLogging(boolean enabled, String tag)

To get the current status of logging call

::

    boolean isDebugLog()

To start Rate dialog for the current application use

::

    openRateDialog (String packageName)


	
Purchasing callbacks
---------------------

IGPlayPurchaseCallback class methods

**********************
Activity getActivity()
**********************

Utilitary method, just return your foreground activity like in code

::    

	@Override
	public Activity getActivity() {
		return MainActivity.this;
	}
	
	
*************************
void onBillingSupported()
*************************

Indicates successfull SDK initialization. The best place to **queryInventory** of your application.

::

	@Override
	public void onBillingSupported() {
		...
		gStorePurchasing.queryInventory(skuList);
	}
	
******************************************
void onBillingNotSupported(String message)
******************************************

Handle this callback to prevent further initialization of components depending on purchases data. Use the **message** parameter to find out the reason of failed initializing


***********************************************************************************
void onQueryInventorySucceeded(String json, IabResult result, Inventory inventory)
***********************************************************************************

Go throw **inventory** to find out SKUs available for purchase, **SkuDetails** data, purchased unconsumed SKUs. Handle unconsumed products. Use SKU details for your needs.

::

	@Override
	public void onQueryInventorySucceeded(String json, IabResult iabResult, Inventory inventory) {

		Purchase premiumPurchase = inventory.getPurchase(SKU_PREMIUM);
		mIsPremium = (premiumPurchase != null && verifyDeveloperPayload(premiumPurchase));
	
		List<Purchase> purchases = new ArrayList<>();
		Purchase gasPurchase = inventory.getPurchase(SKU_GAS);
		if (gasPurchase != null && verifyDeveloperPayload(gasPurchase)) {
			purchases.add(gasPurchase);
		}
		
		...

		gStorePurchasing.consumeProducts(purchases);
	}


	
*******************************************
void onQueryInventoryFailed(String message)
*******************************************

Use the **message** parameter to find out the reason of fail.


***********************************************************************************
void onPurchaseSucceeded(String message, IabResult result, Purchase purchase)
***********************************************************************************

Indicates succesful finish of the purchase flow.
Use **consumeProduct** for *consumable* products, handle *unconsumable* ones

::

	@Override
	public void onPurchaseSucceeded(String json, IabResult iabResult, Purchase purchase) {

		if (purchase.getSku().equals(SKU_GAS)) {
			gStorePurchasing.consumeProduct(purchase);
		} else if (purchase.getSku().equals(SKU_PREMIUM)) {
			mIsPremium = true;
			
		...

::

    void onPurchaseSucceeded(String message, IabResult result, Purchase purchase)

::

    void onPurchaseFailed(String message)

::

    void onConsumePurchaseSucceeded(String message, IabResult result, Purchase purchase)

::

    void onConsumePurchaseFailed(String message)

	

	
GPlay GameConsole
---------------------

bla bla bla



Publicating your game to GPlay
--------------------------------

bla bla bla


Configuration and set up
--------------------------

bla bla bla


F.A.Q.
------

bla bla bla

