# Pizza Cravings? I got your back!

The objective of this project is to perform web scraping on yellowpages.com to extract information on the top 30 "Pizzeria" shops in San Francisco. The project is divided into several steps:

*Step 1*: Use Python or Java to perform a GET request to yellowpages.com and search for "Pizzeria" in San Francisco. Save the search result page to disk as "sf_pizzeria_search_page.htm".

*Step 2*: Use Python or Java to parse the search result page saved in step 1 and extract information on each shop's search rank, name, linked URL, star rating, number of reviews, TripAdvisor rating, number of TA reviews, "$" signs, years in business, review, and amenities. Skip all "Ad" results.

*Step 3*: Modify the code from step 2 to create a MongoDB collection called "sf_pizzerias" and store all the extracted shop information in separate documents.

*Step 4*: Read all the URLs stored in "sf_pizzerias" and download each shop page. Save each page to disk as "sf_pizzerias_[SR].htm" (replace [SR] with the search rank).

*Step 5*: Read the 30 shop pages saved in step 4 and parse each shop's address, phone number, and website.

*Step 6*: Sign up for a free account with https://positionstack.com/ and modify the code from step 5 to query each shop's address for its geolocation (longitude and latitude). Update each shop document on the MongoDB collection "sf_pizzerias" to contain the shop's address, phone number, website, and geolocation.

Overall, this project involves using web scraping techniques to extract information from yellowpages.com and storing the extracted information in a MongoDB collection. The project also involves parsing each shop's website to extract its address, phone number, and website, and using an API from https://positionstack.com/ to obtain the shop's geolocation.
