#In App Purchase

Store Kit Framework

![architecture](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/StoreKitGuide/Art/intro_2x.png)

## Retrieving Product Information
### Getting a List of Product Identifiers
https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/StoreKitGuide/Chapters/ShowUI.html#//apple_ref/doc/uid/TP40008267-CH3-SW5

获取商品信息列表，展示商品


```objc

- (void)fetchProductIdentifiersFromURL:(NSURL *)url delegate:(id)delegate
{
    dispatch_queue_t global_queue =
           dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_async(global_queue, ^{
        NSError *err;
        NSData *jsonData = [NSData dataWithContentsOfURL:url
                                                  options:NULL
                                                    error:&err];
        if (!jsonData) { /* Handle the error */ }

        NSArray *productIdentifiers = [NSJSONSerialization
            JSONObjectWithData:jsonData options:NULL error:&err];
        if (!productIdentifiers) { /* Handle the error */ }

        dispatch_queue_t main_queue = dispatch_get_main_queue();
        dispatch_async(main_queue, ^{
           [delegate displayProducts:productIdentifiers]; // Custom method
        });
    });
}

```

### Retrieving Product Information
```objc
// Custom method
- (void)validateProductIdentifiers:(NSArray *)productIdentifiers
{
    SKProductsRequest *productsRequest = [[SKProductsRequest alloc]
        initWithProductIdentifiers:[NSSet setWithArray:productIdentifiers]];
    productsRequest.delegate = self;
    [productsRequest start];
}

// SKProductsRequestDelegate protocol method
- (void)productsRequest:(SKProductsRequest *)request
     didReceiveResponse:(SKProductsResponse *)response
{
    self.products = response.products;

    for (NSString *invalidIdentifier in response.invalidProductIdentifiers) {
        // Handle any invalid product identifiers.
    }

    [self displayStoreUI]; // Custom method
}
```


### Suggested Testing Steps

* Sign In to the App Store with Your Test Account
* Test Fetching the List of Product Identifiers
* Test Handling of Invalid Product Identifiers
* Test a Products Request


## Requesting Payment

### Creating a Payment Request

```objc
SKProduct *product = <# Product returned by a products request #>;
SKMutablePayment *payment = [SKMutablePayment paymentWithProduct:product];
payment.quantity = 2;
```
## Detecting Irregular Activity
检查不合法交易

## Submitting a Payment Request
```objc
[[SKPaymentQueue defaultQueue] addPayment:payment];
```

## Delivering Products
## Waiting for the App Store to Process Transactions

Registering the transaction queue observer
```objc
- (BOOL)application:(UIApplication *)application
 didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    /* ... */

    [[SKPaymentQueue defaultQueue] addTransactionObserver:observer];
}

```

Responding to transaction statuses
```objc
- (void)paymentQueue:(SKPaymentQueue *)queue
 updatedTransactions:(NSArray *)transactions
{
    for (SKPaymentTransaction *transaction in transactions) {
        switch (transaction.transactionState) {
            // Call the appropriate custom method for the transaction state.
            case SKPaymentTransactionStatePurchasing:
                [self showTransactionAsInProgress:transaction deferred:NO];
                break;
            case SKPaymentTransactionStateDeferred:
                [self showTransactionAsInProgress:transaction deferred:YES];
                break;
            case SKPaymentTransactionStateFailed:
                [self failedTransaction:transaction];
                break;
            case SKPaymentTransactionStatePurchased:
                [self completeTransaction:transaction];
                break;
            case SKPaymentTransactionStateRestored:
                [self restoreTransaction:transaction];
                break;
            default:
                // For debugging
                NSLog(@"Unexpected transaction state %@", @(transaction.transactionState));
                break;
        }
    }
}

```


## Persisting the Purchase
保存 transaction receipt

### Persisting Using the App Receipt
保存收据

### Persisting a Value in User Defaults or iCloud
```objc

#if USE_ICLOUD_STORAGE
NSUbiquitousKeyValueStore *storage = [NSUbiquitousKeyValueStore defaultStore];
#else
NSUserDefaults *storage = [NSUserDefaults standardUserDefaults];
#endif

[storage setBool:YES forKey:@"enable_rocket_car"];
[storage setObject:@15 forKey:@"highest_unlocked_level"];

[storage synchronize];

```

### Persisting a Receipt in User Defaults or iCloud
```objc
#if USE_ICLOUD_STORAGE
NSUbiquitousKeyValueStore *storage = [NSUbiquitousKeyValueStore defaultStore];
#else
NSUserDefaults *storage = [NSUserDefaults standardUserDefaults];
#endif

NSData *newReceipt = transaction.transactionReceipt;
NSArray *savedReceipts = [storage arrayForKey:@"receipts"];
if (!receipts) {
    // Storing the first receipt
    [storage setObject:@[newReceipt] forKey:@"receipts"];
} else {
    // Adding another receipt
    NSArray *updatedReceipts = [savedReceipts arrayByAddingObject:newReceipt];
    [storage setObject:updatedReceipts forKey:@"receipts"];
}

[storage synchronize];

```

### Persisting Using Your Own Server



## Unlocking App Functionality

## Delivering Associated Content

## Finishing the Transaction
```objc
SKPaymentTransaction *transaction = <# The current payment #>;
[[SKPaymentQueue defaultQueue] finishTransaction:transaction];
```

## Suggested Testing Steps
* Test a Payment Request
* Verify Your Observer Code
* Test a Successful Transaction
* Test an Interrupted Transaction
* Verify That Transactions Are Finished
