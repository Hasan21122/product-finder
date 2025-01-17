
import requests
from bs4 import BeautifulSoup
import re

# Function to search for a product on Amazon
def search_amazon(product_name):
    amazon_url = f"https://www.amazon.com/s?k={product_name.replace(' ', '+')}"
    headers = {'User-Agent': 'Mozilla/5.0'}
    response = requests.get(amazon_url, headers=headers)
    soup = BeautifulSoup(response.content, 'html.parser')
    products = []
    for item in soup.find_all('div', {'data-component-type': 's-search-result'}):
        title = item.h2.text.strip()
        price = item.find('span', 'a-offscreen')
        link = item.h2.a['href']
        if price:
            price_value = float(re.sub(r'[^\d.]', '', price.text))  # Extract numerical value from price
            products.append({
                'name': title,
                'price': price_value,
                'link': f"https://www.amazon.com{link}"
            })
    return products

# Function to search for a product on Walmart
def search_walmart(product_name):
    walmart_url = f"https://www.walmart.com/search/?query={product_name.replace(' ', '%20')}"
    headers = {'User-Agent': 'Mozilla/5.0'}
    response = requests.get(walmart_url, headers=headers)
    soup = BeautifulSoup(response.content, 'html.parser')
    products = []
    for item in soup.find_all('div', 'search-result-gridview-item-wrapper'):
        title = item.find('a', 'product-title-link').text.strip()
        price = item.find('span', 'price-main')
        link = item.find('a', 'product-title-link')['href']
        if price:
            price_value = float(re.sub(r'[^\d.]', '', price.text))  # Extract numerical value from price
            products.append({
                'name': title,
                'price': price_value,
                'link': f"https://www.walmart.com{link}"
            })
    return products

# Function to match products from Amazon and Walmart with price condition
def match_products(amazon_products, walmart_products):
    matches = []
    for a_product in amazon_products:
        for w_product in walmart_products:
            if (a_product['name'].lower() in w_product['name'].lower() or 
                w_product['name'].lower() in a_product['name'].lower()) and \
                w_product['price'] >= 2 * a_product['price']:
                matches.append({
                    'amazon_name': a_product['name'],
                    'amazon_price': a_product['price'],
                    'amazon_link': a_product['link'],
                    'walmart_name': w_product['name'],
                    'walmart_price': w_product['price'],
                    'walmart_link': w_product['link']
                })
    return matches

# Example usage
amazon_products = search_amazon("laptop")
walmart_products = search_walmart("laptop")
matched_products = match_products(amazon_products, walmart_products)

# Print matched products
for match in matched_products:
    print(match)
