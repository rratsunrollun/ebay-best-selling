Для получения наиболее просматриваемых карточок по категории нужно отправить запрос на https://developer.ebay.com/devzone/merchandising/docs/CallRef/getMostWatchedItems.html

Лучше всего использовать SDK https://github.com/davidtsadler/ebay-sdk-php

Пример:
```php
use DTS\eBaySDK\Credentials\CredentialsProvider;
use DTS\eBaySDK\Merchandising\Services\MerchandisingService;
use DTS\eBaySDK\Merchandising\Types\GetMostWatchedItemsRequest;

$service = new MerchandisingService([
    'siteId'     => '0',
    'credentials' => CredentialsProvider::env()
]);

$request = new GetMostWatchedItemsRequest();
$request->categoryId = '10063'; // Motorcycle Parts
$request->maxResults = 50;

$response = $service->getMostWatchedItems($request);

if ($response->ack === 'Success') {
    echo '<pre>';
    print_r($response->itemRecommendations->toArray());
    die();
}
``` 
ответ
```php
Array
(
    [item] => Array
        (
            [0] => Array
                (
                    [itemId] => 182917157822
                    [title] => Honda Motorcycle Keys Cut to Code Replacement Spare New Ignition precut Key 
                    [viewItemURL] => https://www.ebay.com/itm/Honda-Motorcycle-Keys-Cut-to-Code-Replacement-Spare-New-Ignition-precut-Key/182917157822?_trkparms=mehot%3Dpp%26itm%3D182917157822%26pmt%3D1%26noa%3D1&_trksid=p0
                    [globalId] => EBAY-AUTOS
                    [timeLeft] => P19DT16H10M45S
                    [primaryCategoryId] => 6028
                    [primaryCategoryName] => Parts & Accessories
                    [subtitle] => 
                    [buyItNowPrice] => Array
                        (
                            [value] => 10.99
                            [currencyId] => USD
                        )

                    [country] => OS
                    [imageURL] => https://i.ebayimg.com/images/g/nNQAAOSw7nZasqPq/s-l225.jpg
                    [shippingCost] => Array
                        (
                            [value] => 2.99
                            [currencyId] => USD
                        )

                    [shippingType] => NotSpecified
                    [watchCount] => 2112
                )
...
``` 

Для получения ePID можно воспользоваться https://developer.ebay.com/devzone/product/CallRef/findProducts.html#findProducts

Но есть проблема. ebay не отдает информацию по нужной нам категории, по этому поиск лучше всего производить по имени <keywords>, но здесь также не все просто, так как по id листинга поиск сделать невозможно, но возможно по title.

пример:
```php
use DTS\eBaySDK\Credentials\CredentialsProvider;

$service = new \DTS\eBaySDK\Product\Services\ProductService([
    'siteId'     => '0',
    'credentials' => CredentialsProvider::env()
]);

$productRequest = new \DTS\eBaySDK\Product\Types\ProductRequest();
$productRequest->dataset = ['DisplayableProductDetails'];
//$productRequest->categoryId = '10063'; // Error: Product data is not enabled for category ID 10063.
$productRequest->invocationId = '1';
$productRequest->keywords = 'Honda Motorcycle Keys'; // сюда пишем title товара

$request = new \DTS\eBaySDK\Product\Types\FindProductsRequest();
$request->productSearch = [$productRequest];

$response = $service->findProducts($request);

// Output the response from the API.
if ($response->ack !== 'Success') {
    foreach ($response->errorMessage->error as $error) {
        printf("Error: %s\n", $error->message);
    }
    die();
}

echo '<pre>';
print_r($response->toArray());
die();
```
ответ
```php
Array
(
    [ack] => Success
    [version] => 1.3.1
    [timestamp] => 2020-04-02T11:43:14.000Z
    [productSearchResult] => Array
        (
            [0] => Array
                (
                    [products] => Array
                        (
                            [0] => Array
                                (
                                    [productIdentifier] => Array
                                        (
                                            [ePID] => 1458235383
                                        )

                                    [stockPhotoURL] => Array
                                        (
                                            [thumbnail] => Array
                                                (
                                                )

                                            [standard] => Array
                                                (
                                                )

                                        )

                                    [productDetails] => Array
                                        (
                                            [0] => Array
                                                (
                                                    [propertyName] => Ebay Title
                                                    [value] => Array
                                                        (
                                                            [0] => Array
                                                                (
                                                                    [text] => Array
                                                                        (
                                                                            [value] => Motorcycle Blank Key Uncut Blade For most Honda motorcycles Hon31rap.. Two keys
                                                                        )

                                                                )

                                                            [1] => Array
                                                                (
                                                                    [text] => Array
                                                                        (
                                                                            [value] => motorcycle blank key uncut blade for most honda motorcycles hon31rap two keys
                                                                        )

                                                                )

                                                        )

                                                )

                                        )

                                    [productStatus] => Array
                                        (
                                            [excludeForeBaySelling] => 
                                            [excludeForeBayReviews] => 
                                            [excludeForHalfSelling] => 
                                        )

                                    [type] => Head
                                )
...
``` 
На этом этапе у нас есть epid товара(ов). Далее нужно получить товар из каталога для того чтобы иметь больше информации по товару.
Чем больше я изучал старое АПИ тем больше убеждался что лучше изучать REST API.
Судя с документации информации можно получить больше в менльне итераций. Вот как можна получить товары которые лучше всего продаются https://developer.ebay.com/api-docs/buy/marketing/resources/merchandised_product/methods/getMerchandisedProducts

