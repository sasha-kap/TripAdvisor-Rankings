# How Can Hotels Predict and Improve Their Popularity Rankings on TripAdvisor?

## Introduction

For my Capstone Project I chose to examine hotel popularity rankings on the travel site TripAdvisor and to build some data-driven recommendations on how hotels can improve their rankings.

What are the popularity rankings I am talking about?  On the partial screenshot of the New York City hotel listings page below, you can see two hotels with their Traveler Popularity Rankings.  There is the top hotel, named 414 Hotel, and a different hotel that is shown here because TripAdvisor sells sponsored listings that allow hotels to be shown at the top of the list.

![Listings Screenshot](./images/ListingsScreenshot.png)

There are other ways that travelers can sort hotels, but the Traveler Ranked sort order is the default sort order.  As you can imagine, hotels are interested in being ranked high, so that there is a higher likelihood that travelers will see them when they look for hotels and choose to book a stay at their property.

There are in fact specific factors that go into TripAdvisor’s formula for computing the rankings: number, quality and age of reviews. But even though hotels can ask their guests to leave reviews upon check-out, this applies to all hotels, and since hotel rankings are relative, there is still a lot of uncertainty left as to how high a certain hotel will be ranked, and it is hard for hotels to compete just based on the number of reviews they may or may not get.  Going back to the screenshot example, you can see that the sponsored hotel has a lot more reviews and an identical quality of reviews, but that hotel is only ranked 57th and had to resort to buying this sponsored listing.

So the intended value of this project is to provide hotels with insight into more tangible, discriminating, and modifiable predictors of their ranks.

## Data Collection

All the data that I used for this project came from the TripAdvisor website.  I chose to work with NYC hotels and wrote web scraping (Scrapy) code to collect data on hotel characteristics and the reviews they have received and to place the data in a PostgreSQL database.  Collected hotel features included: name, address, amenities, star rating, number of rooms, and price range.   Review data included review text, review date, month of stay, type of travel, and bubble rating.  There were 467 hotels identified in NYC under and I scraped a total of 100,000 reviews from their pages.

## Preprocessing

As part of data cleansing, I implemented the following steps:
- Removed 17 hotels without a rank and 4 hotels outside of Manhattan
- Cleaned hotel addresses to allow all of them to be geocoded and plotted on a map
- Imputed missing values of number of rooms in hotel (6), hotel's star rating (28), and price range (22) using k-Nearest Neighbors (after imputation I rounded star rating values to the nearest 0.5 to allow for categorical treatment of that feature)
- Removed reviews without text content or without hotel stay summary (< 2% of all reviews)
- Removed 6 hotels without any reviews
- Cleaned review text & aggregated reviews to hotel level

The final hotel count ended up being 440.  I shifted hotel ranks to fill any gaps created by removing the 27 hotels above.

I geocoded all hotel addresses using Google Maps API (with the help of the Geocoder package).

## Exploratory Data Analysis

Before building my predictive models, I undertook exploratory data analysis to see if I could uncover any evidence of a relationship between the rank of a hotel and its other attributes.  As you can see in the box plot below, the main relationship that I uncovered is that hotels with higher star ratings are more likely to have better popularity rankings.  This surprised me somewhat, as this suggests that in rating hotels, travelers at least in the NYC market prioritize amenities, luxury and comfort over value.

![Star Rating Box Plot](./images/StarRatingBoxPlot.jpg)

But what also stood out here is the presence of outliers (particularly at the top of the rankings) and I decided to study these outliers individually to see if I could formulate any additional hypotheses.  After browsing hotel reviews, the top hotels that stood out among their class-rating peers appeared to be small boutique hotels that offer unique and welcoming experience.  It is possible then that those are important features that can drive a hotel to the top of the rankings and should be incorporated in the model.

But then I also examined the distribution of room price ranges within each star rating class.  As shown in the plot below, the hotels that out-performed their class-rating peers with their rank are the same hotels that charge more for the rooms relative to the other hotels in their class.  This suggests that those hotels perform so well not necessarily because of offering a warm and welcoming experience to their guests, but perhaps because they offer more amenities or services that put them on par with higher-class hotels.

![Plot of Price Ranges by Class](./images/PriceRangesByClass.jpg)

Another hypothesis that I explored is the association between hotel's location and its ranking.  It is likely that a hotel will be rated highly by its guests not just because of what the hotel itself offers, but because of where it is situated.  But after plotting all hotels on the map of Manhattan (using QGIS open-source application), I did not notice this type of relationship.

![Manhattan hotel map](./images/map_001_last.png)

The highest-ranked hotel (1-100), marked in red on this map, exhibit a very similar spatial distribution to hotels 101-200 (yellow dots), 201-300 (purple), and 301-440 (green).  Given these findings, and given that Manhattan is so compact and it is easy to get around from place to place, I did not investigate the geospatial question any further.

Lastly, I explored the temporal dimension of hotel rankings by examining if the ratings left by guests vary over time, which would suggest that hotels undertake efforts to improve their service/customer experience.  For this, I plotted average monthly ratings of all hotels that had at least three years of review data.  The story I uncovered was that hotels' ratings do not increase or decrease dramatically over time.  The three examples shown below indicate that hotel's ranking is largely a function of time-invariant hotel attributes.

![Quality of Reviews Over Time](./images/RatingsOverTime.jpg)

## Modeling
