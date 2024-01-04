**Section 34** Package Payment Gateways: PayPal Integration
================================================

#### Gateways we will be Learning

- Paypal Integration
- Strip Integration
- Razorpay Integration
- Sslcommerz Integration
- Instamojo integration
- Mollie integration
- 2checkout integration
- Paystack integration

### **1-** Installing the PayPal SDK for PHP

* [paypal Dashboarf](https://developer.paypal.com/dashboard/)<br>
* [Sandbox test accounts](https://developer.paypal.com/dashboard/accounts)<br>
* [Laravel Paypal GitHub](https://github.com/srmklive/laravel-paypal)<br>
* [Docs Laravel Paypal](https://srmklive.github.io/laravel-paypal/docs.html)
  
<br>

**Getting Started**

    composer require srmklive/paypal:~3.0

>After installation, you can use the following commands to publish the assets:

    php artisan vendor:publish --provider "Srmklive\PayPal\Providers\PayPalServiceProvider"

>After publishing the assets, add the following to your .env files .

    #PayPal API Mode
    # Values: sandbox or live (Default: live)
    #sandbox in local server
    #libe in online server
    PAYPAL_MODE=sandbox

    #PayPal Setting & API Credentials - sandbox
    PAYPAL_SANDBOX_CLIENT_ID=AaeelkhhJXCHlVABE1vEWh03BgQtF0cBGlUHp6HHcne9rt7-iXJIZl1Dn2a1DHd8dkS0gPvSDNzto06M
    PAYPAL_SANDBOX_CLIENT_SECRET=ENvONOd_LfaCekiNRkieGKgRVvwaINArAYw5v4cHc9EDib_lDp6pVv3EvUPNEOvI1kwffV5UZq7UgI9i

    #PayPal Setting & API Credentials - live
    # PAYPAL_LIVE_CLIENT_ID=
    # PAYPAL_LIVE_CLIENT_SECRET=

>The configuration file paypal.php is located in the config folder. Following are its contents when published:

    return [
    'mode'    => env('PAYPAL_MODE', 'sandbox'), // Can only be 'sandbox' Or 'live'. If empty or invalid, 'live' will be used.
    'sandbox' => [
        'client_id'         => env('PAYPAL_SANDBOX_CLIENT_ID', ''),
        'client_secret'     => env('PAYPAL_SANDBOX_CLIENT_SECRET', ''),
        'app_id'            => 'APP-80W284485P519543T',
    ],
    'live' => [
        'client_id'         => env('PAYPAL_LIVE_CLIENT_ID', ''),
        'client_secret'     => env('PAYPAL_LIVE_CLIENT_SECRET', ''),
        'app_id'            => '',
    ],

    'payment_action' => env('PAYPAL_PAYMENT_ACTION', 'Sale'), // Can only be 'Sale', 'Authorization' or 'Order'
    'currency'       => env('PAYPAL_CURRENCY', 'USD'),
    'notify_url'     => env('PAYPAL_NOTIFY_URL', ''), // Change this accordingly for your application.
    'locale'         => env('PAYPAL_LOCALE', 'en_US'), // force gateway language  i.e. it_IT, es_ES, en_US ... (for express checkout only)
    'validate_ssl'   => env('PAYPAL_VALIDATE_SSL', true), // Validate SSL when creating api client.
    ];

>after configuration file paypal so next step is create paypal controller and create functions (pyment, success, cancel)  
![Pyment](<markdown/function payment.png>)
![Success and cancel](<markdown/function success and cancel.png>)

- Function Pyment 

        public function payment(Request $request)
        {
        $provider = new PayPalClient;

        $provider->setApiCredentials(config('paypal'));

        $paypalToken = $provider->getAccessToken();

        $response = $provider->createOrder([
            "intent" => "CAPTURE",
            "application_context" => [
                "return_url" => route("paypal.success"),
                "cancel_url" => route("paypal.cancel"),
            ],
            "purchase_units" => [
                [
                    "amount" => [
                        "value" => $request->input('price'),//$request->input('amount'),
                        "currency_code" => "USD"
                        ]
                ]
            ]
        ]);
        if (isset($response['id']) && $response['id'] != null){
            foreach ($response['links'] as $link){
                if ($link["rel"] === 'approve') {
                    return redirect()->away($link['href']);
                }
            }
        }
        else {
            return redirect()->route('paypal.cancel');
        }
    }

- function success 

        public function success(Request $request)
        {
        $provider = new PayPalClient;

        $provider->setApiCredentials(config('paypal'));

        $paypalToken = $provider->getAccessToken();

        $response = $provider->capturePaymentOrder($request->token);

        if (isset($response['status']) && $response   ['status'] == 'COMPLETED') {
        return 'Paid Successfully!';
        }

        return redirect()->route('paypal.cancel');
        }

- function cancel 

        public function cancel(Request $request)
        {
            return 'Payment faild';
        }
