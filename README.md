# CIRKLEPAY PAYMENT GATEWAY Config

The given package is not storing any data into db, so please make sure the requested data and response data is stored 
in your database.

### Prerequisites
To Integrate CIRKLEPAY Bolt Checkout, you must :
* [Signup](https://cirklepay.com) with Cirklepay as the merchant
* Get the Key and Salt, which are available on your merchant [dashboard](https://www.cirklepay.com/merchant/dashboard)

## Config Payu with Django project


### projects/settings.py

```python
# project/project/settings.py

    INSTALLED_APPS = [
        'django.contrib.admin',
        # Register Cirklepay Here
        'cirklepay.apps.CirklePayConfig'
    ]
    # Manadatory Config details
    CIRKLEPAY_CONFIG = {
        "merchant_key": "*******",
        "merchant_salt": "******",
        "mode": "TEST/LIVE",
        "success_url": "http://127.0.0.1:8000/payment/success",
        "failure_url": "http://127.0.0.1:8000/payment/failure"
    }
```

### Config in Checkout view
 - Create a dict with data with your cart data and user data
 - Make sure the following data are included in the dict.
 - Please consider below checkout snippetts
 
    ``` python
    # project/checkout/views.py

        from cirklepay.cirklepay import CIRKLEPAY
        cirklepay = CIRKLEPAY()

        def checkout(request):
            if request.method == 'POST':
                data = { 'amount': '10', 
                        'firstname': 'renjith', 
                        'email': 'sraj@gmail.com',
                        'phone': '9746272610', 'productinfo': 'test', 
                        'lastname': 'test', 'address1': 'test', 
                        'address2': 'test', 'city': 'test', 
                        'state': 'test', 'country': 'test', 
                        'zipcode': 'tes', 'udf1': '', 
                        'udf2': '', 'udf3': '', 'udf4': '', 'udf5': ''
                    }
                # You can generate the transaction id, save to db
                # Here cirklepay.cirklepay providing dynamic transaction id's 
                # if  you this method please ensure that, the ID is not existed in the
                # db
                data['txnid'] = cirklepay.generate_txnid()
                # Please dont forget to include this part, The cirklepay.cirklepay included the hidden
                # Payu form, will post the data to cirklepay based on your mode selection, if you
                # required more detils please go through : 
                # https://github.com/cirklepay/Cirklepay-Python-SDK/blob/master/templates/cirklepay_checkout.html
                cirklepay_data = cirklepay.initiate_transaction(data)
                return render(request, 'cirklepay_checkout.html', {"posted": cirklepay_data})
            else:
                return render(request, 'checkout.html', {"posted": cirklepay_data})
    ```

### Verify Given Payment response[ verify hash value]
``` python
    from django.views.decorators.csrf import csrf_exempt
    # Payu success return page
    @csrf_exempt
    def cirklepay_success(request):
        data = dict(zip(request.POST.keys(), request.POST.values()))
        response = cirklepay.check_hash(data)
        # Store response to the db
        return JsonResponse(response)

```

## Make sure
* Included the success / failure urls in the settings.py
* It should be included your checkout views
    ``` python
    # checkout views
    cirklepay_data = cirklepay.initate_transaction(data)
    return render(request, 'cirklepay_checkout.html', {"posted": cirklepay_data})
   ```
* Make sure you have added `csrf_exampt` in the success & failure views