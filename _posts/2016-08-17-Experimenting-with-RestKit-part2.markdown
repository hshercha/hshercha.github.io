---
title:  Experimenting with RestKit (Part 2)
date:   2016-08-21 21:00:00
description: Going down the rabbit hole.
---

In my [last post]({% post_url 2016-08-17-Experimenting-with-RestKit-part1 %}), I discussed REST and RestKit. Today, I will discuss how to request data using RestKit. Using the [Unofficial AirBnb API](http://airbnbapi.org/), I wanted to see if I could display the listings on a table view.

**Note:** Airbnb API is not officially open to third-party developers as of yet. Therefore, please be mindful towards the API usage. With great APIs comes great responsiblity! 

Here is my [project repository](https://github.com/hshercha/Objective-C/tree/master/Listings).

Here is the endpoint URL to retrieve the listings: https://api.airbnb.com/v2/search_results

The required parameter for this endpoint is **client_id**. I figured out a work around for retrieving the client_id since there is no official way to do so.

The key can be retrieved via the Network Diagnostics window in the Developers tool of the browser. Somehow Chrome's developer tools didn't show me any json related information. No worries though! Mozilla's developer tools worked out for me. 

[![Network Diagnostics Screenshot]({{ site.baseurl }}assets/images/RestKit/client_id_screenshot.png)]({{ site.baseurl }}assets/images/RestKit/client_id_screenshot.png)

Here is a [sample API request](https://api.airbnb.com/v2/search_results?client_id=3092nxybyb0otqw18e8nh5nty).

Before creating an object mapping, one needs to create an object model. Due to the nature of the nested relationship that exists in the response, the listing object has to be encapsulated inside the the SearchResult class.

{% highlight objc %}
@interface Listings : NSObject

@property (strong, nonatomic) NSString *name;
@property (strong, nonatomic) NSString *roomType;
@property (strong, nonatomic) NSString *neighborhood;

@end

@interface SearchResult : NSObject

@property (strong, nonatomic) Listings *listing;

@end
{% endhighlight %}

Here the **AFHTTPClient** object (part of AFNetworking library) is instantiated by providing a base url which is then used to intialize the **RKObjectManager* object. The object manager abstracts the HTTP interaction done via the AFNetworking library.

{% highlight objc %}
// Create a baseURL
NSURL *baseURL = [NSURL URLWithString:@"https://api.airbnb.com"];
AFHTTPClient *client = [[AFHTTPClient alloc] initWithBaseURL:baseURL];

// Initialize RestKit
RKObjectManager *objectManager = [[RKObjectManager alloc] initWithHTTPClient:client];

{% endhighlight %}

The object mapping mechanism follows the Key-Value Coding protocol of mapping property to variable names. Here it maps the JSON attributes to the object model attributes. The **searchResultMapping** object insures that "listing" property inside the "search_results" is mapped to the object named "listing" inside the SearchResult class. This is necessary since the relationship between the "search_results" and "listings" happens to be one-to-many. 

{% highlight objc %}
// Create object mapping for listing
RKObjectMapping *listingMapping = [RKObjectMapping mappingForClass:[Listings class]];
[listingMapping addAttributeMappingsFromDictionary:@{@"name": @"name", @"room_type": @"roomType", @"neighborhood": @"neighborhood"}];

// Create object mapping for search results
RKObjectMapping *searchResultsMapping = [RKObjectMapping mappingForClass:[SearchResult class]];

//Add nested properties 
[searchResultsMapping addPropertyMapping:[RKRelationshipMapping relationshipMappingFromKeyPath:@"listing" toKeyPath:@"listing" withMapping:listingMapping]];
{% endhighlight %}

Once the object mappings are established, the response descriptor is created. The response descriptor searches for the specific url path by using the provided path pattern. On the returned JSON response, the provided key path searches for values upon which the object mapping can be applied. 

{% highlight objc %}
// Register object mapping to the response descriptor
RKResponseDescriptor *responseDescriptor = [RKResponseDescriptor responseDescriptorWithMapping:searchResultsMapping method:RKRequestMethodGET pathPattern:@"/v2/search_results" keyPath:@"search_results" statusCodes:RKStatusCodeIndexSetForClass(RKStatusCodeClassSuccessful)];

[objectManager addResponseDescriptor:responseDescriptor];
{% endhighlight %}

Finally, after setting up the response descriptors and the object mapping, object manager sends a request with the provided parameters. Upon getting the response, the response descriptor parses the data into objects and passes it along to the completion block.  

{% highlight objc %}
- (void)getListing:(void (^)(NSArray *listings, NSError *error))finishBlock {
    
    //Describe the query parameters
    NSDictionary *queryParams = @{@"client_id" : CLIENTKEY,
                                  @"location"  : CITY,
                                  @"user_lat"  : USER_LAT,
                                  @"user_lng"  : USER_LONG};
    
    //Call the object manager with the appropriate parameters
    [[RKObjectManager sharedManager] getObjectsAtPath:@"/v2/search_results" 
    parameters:queryParams
    success:^(RKObjectRequestOperation *operation, RKMappingResult *mappingResult) 
    { 
        if (finishBlock) 
        { 
        finishBlock(mappingResult.array, nil); 
        } 
    }
    failure:^(RKObjectRequestOperation *operation, NSError *error)
    { 
        if (finishBlock) 
        { 
        finishBlock(nil, error); 
        }
    }];
}
{% endhighlight %}

---

AirBnb listings in San Francisco.

[![Listings Screenshot]({{ site.baseurl }}assets/images/RestKit/listings.png)]({{ site.baseurl }}assets/images/RestKit/listings.png)

---

**MISTAKE #1**
{% highlight objc %}
// Register object mapping to the response descriptor
RKResponseDescriptor *responseDescriptor = [RKResponseDescriptor responseDescriptorWithMapping:listingMapping method:RKRequestMethodGET pathPattern:@"/v2/search_results" keyPath:@"search_results.listing" statusCodes:RKStatusCodeIndexSetForClass(RKStatusCodeClassSuccessful)];

[objectManager addResponseDescriptor:responseDescriptor];
{% endhighlight %}

I thought that the **listingMapping** object would map the values found in the "search_results.listing" keypath. As a result, I wasn't receiving any objects in response. After understanding the relationship between the "search_results" and the "listing" values in the JSON response, I was able to establish the proper object mapping heirarchy.

---

## Future Work
As of yet, there isn't any iOS SDK/wrapper for Airbnb. I believe that this might be a good ground work for a project like that.