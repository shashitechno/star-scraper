# Star Scraper

A PHP library by [Die Schittigs](https://www.dieschittigs.de).

## What's this?

Star Scraper is built to aggregate ratings from different websites (e.g. Google Places, Facebook) into one average rating intended to be visibe on your website. This is especially useful for (local) businesses, that have reviews on Facebook *and* other platforms and want these ratings to be displayed on their homepage in an unified way. It may also be used to display those little stars besides Google search results.

## Installation

Install via [Composer](https://getcomposer.org).

    composer require dieschittigs/star-scraper

## Usage

First, do some imports and initialize StarRating

    <?php

    use DieSchittigs\StarScraper\StarRating;
    use DieSchittigs\StarScraper\GooglePlaceProvider;
    use DieSchittigs\StarScraper\FacebookPageProvider;
    use DieSchittigs\StarScraper\FakeRatingsProvider;

    $starRating = new StarRating();

Now you may add RatingProviders (that's where your ratings are coming from). For now we only have support for Google Places (also called Google My Business) and Facebook Pages.

### Custom best rating value

Don't want 5 stars, but 100% as your best rating? No problem.

    <?php

    $starRating = new StarRating(100);

Star Scraper will normalize all individual results from your providers, so that the end-result is independent from their best rating value;

### Google My Business / Google Maps Places

You'll need an [Google API-key](https://console.developers.google.com) (activate *Google Places API Web Service*) and your [PlaceID](https://developers.google.com/maps/documentation/javascript/examples/places-placeid-finder) (your business on Google Maps).

    <?php

    $starRating->addProvider(
        new GooglePlaceProvider(
            '{{GoogleApiKey}}',
            '{{GoogleMapsPlaceID}}'
        )
    );

### Facebook Pages

You'll need an [Facebook App ID and App Secret](https://developers.facebook.com) and your [PageID](https://findmyfbid.com/).

    <?php

    $starRating->addProvider(
        new FacebookPageProvider(
            '{{FacebookAppID}}',
            '{{FacebookAppSecret}}',
            '{{FacebookPageID}}'
        )
    );

### Fake Ratings

If you just want to try things out, use some fake ratings.

    <?php

    $starRating->addProvider(
        new FakeRatingsProvider([5,4,3,5,4])
    );

    $starRating->addProvider(
        new FakeRatingsProvider([1,2,1,3,1,3])
    );

### Get the average rating

To get the *median* ratings from all of your providers, simply call

    <?php

    $rating = $starRating->getRating();

This will give you the median - if you prefer the less accurate *mean* average, call

    <?php

    $rating = $starRating->getRating('mean');

The result will look like this

    <?php

    DieSchittigs\StarScraper\Rating Object
    (
        [bestRating] => 5
        [ratingCount] => 18
        [ratingValue] => 4.5
    )

## Extend

If you want to add you own RatingProviders, it's pretty straighforward.

    <?php

    use DieSchittigs\StarScraper\RatingProvider;
    use DieSchittigs\StarScraper\Rating;

    class CustomRatingsProvider extends RatingProvider{
        private $reviews;
        public function __construct($apiKeysOrWhatever){
            $this->bestRating = 100;
            // call an API or fetch your results from a DB
            $this->reviews = $reviews;
        }
        public function getRating($method = 'median'){
            // Return null if e.g. your api call died
            if(!$this->reviews) return null;
            $rating = new Rating($this->bestRating, count($this->reviews));
            $scores = [];
            foreach($this->reviews as $review){
                $scores[] = $review->rating;
            }
            $rating->avgRatingValue($scores, $method);
            return $rating;
        }
    }

### Calling an external API in your Provider

When making API calls, consider using `BrowserTrait`.

    <?php

    use DieSchittigs\StarScraper\RatingProvider;
    use DieSchittigs\StarScraper\Rating;
    use DieSchittigs\StarScraper\BrowserTrait;

    class CustomRatingsProvider extends RatingProvider{
        use BrowserTrait;
        public function __construct($apiKey){
            $result = $this->getBrowser()->get("http://api.example.org?key=$apiKey");
            $data = $result->getContent();
        }
        ...
    }

`$this->getBrowser()` will give you an instance of [Buzz/Browser](https://github.com/kriswallsmith/Buzz).
The browser instance is shared with other Providers for performance reasons.

## Contribute

We need more RatingProviders! Sources for Ratings are:

- Yelp
- LinkedIN
- Xing
- ...

Your help is very welcome :)
