import http.client, urllib.parse
from bs4 import BeautifulSoup
import requests
import re
import time
import pandas as pd
import numpy as np
import json
import os
from urllib.request import Request, urlopen
from tqdm import tqdm
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.chrome.webdriver import WebDriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.action_chains import ActionChains
import pymongo
from fastapi import FastAPI
from datetime import datetime
from pymongo import MongoClient
import random
from webdriver_manager.chrome import ChromeDriverManager
import os
import codecs


# In[2]:


class Best_buy_scrap:
    def __init__(self , product_name):
        self.search_term = product_name
        
    ### request search page information and return bs4 object
    def get_search_pages(self , search_term , p_number):
        ### URL
        search_page_url = "https://www.bestbuy.com/site/searchpage.jsp?id=pcat17071&st="
        ### search product
        search_product = search_term
        
        ### page_number
        page_number = 0
        if p_number == 0:
            page_number = 1
        else:
            page_number = p_number

        ### page_number search term
        search_page_encoding = f"&cp={page_number}"

        ### User agent
        User_Agent = 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.96 Safari/537.36 Edg/88.0.705.56'

        ### request access
        r = requests.get(search_page_url + search_product + search_page_encoding , headers = {'User-Agent' : User_Agent})
        
        ### print url's
        print(search_page_url + search_product + search_page_encoding)

        ### Assign bs4 object to soup
        soup = BeautifulSoup(r.content, 'lxml')
        
        #creating a file "ebay_amazon gift cards_webpages" for storing all the pages of seacrh results that we download. But before that we are checking if the we already have one file with the same name. If so they code piece will not run
        try:
            if not os.path.exists("best_buy_folder"):
                os.makedirs("best_buy_folder")
        except:
            pass
        
        #downloading the first pages of seacrh results 

        response_page1 = requests.get(search_page_url + search_product + search_page_encoding , headers = {'User-Agent' : User_Agent})
        soup_page1 = BeautifulSoup(response_page1.content, 'html.parser')
        with open(f'best_buy_folder/best_buy_page_{page_number}.html', 'w', encoding='utf-8') as file:
            file.write(str(soup_page1))

        return soup
    
    ### handle price text
    def handle_price(self , price):
        try:
            price = price.replace("$" , "")
            price = float(price)
        except:
            pass
        return price

    ### exrtract brand name from header
    def get_brand(self , header):
        try:
            brand = header.split("-")[0].strip(" ")
        except:
            brand = ""
        return brand
    
    ### get product info from bs4 object and return dict of info
    def get_product_info(self , product):
        return_dict = {}

        ### get header
        header = product.find('h4').text
        return_dict.update({'header' : header})

        ### get brand
        brand = self.get_brand(header)
        return_dict.update({'brand' : brand})

        ### get features
        try:
            features = product.find('div' , attrs = {'class' : 'lv-stacked-carousel'}).find_all('button')
            features = [lab.text for lab in features]
        except:
            features = []
        return_dict.update({'features' : features})


        ### get model
        model = product.find_all('span' , attrs = {'class' : 'sku-value'})[0].text
        return_dict.update({'model' : model})

        ### get sku
        sku = product.find_all('span' , attrs = {'class' : 'sku-value'})[1].text
        return_dict.update({'sku' : sku})
        
        ### get model link
        link = product.find("div" , attrs = {"class" : "list-item lv"}).find("a", attrs = {"class" : "image-link"})["href"]
        return_dict.update({'product link' : "https://www.bestbuy.com"+link})
                
                            
        ### get stars
        try:
            stars = product.find('p').text.split(' ')[1]
            return_dict.update({'stars' : float(stars)})
        except:
            pass 

        ### get number of reviews
        number_reviews = product.find('p').text.split(' ')[-2]
        return_dict.update({'number_of_reivews' : number_reviews})

        ### get price
        try:
            price = product.find("div" , attrs = {"class" : "priceView-hero-price priceView-customer-price"}).find('span').text
            return_dict.update({"price" : self.handle_price(price)})
        except:
            pass

        ### get old price
        try:
            old_price = product.find("div" , attrs = {"class" : "pricing-price__regular-price"}).text.split(' ')[1]
            return_dict.update({'original_price' : self.handle_price(old_price)})
        except:
            pass

        ### price diff
        try:
            reduction = self.handle_price(old_price) - self.handle_price(price)
            return_dict.update({'price_reduction' : reduction})
        except:
            pass

        return return_dict
        
        
    def save_product(self, product , directory):       
        product_dict = {}
        ### get product webpage
        try:
            url = product['product link']
            web_page = requests.get(url , headers = {'User-Agent' : 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/88.0.4324.96 Safari/537.36 Edg/88.0.705.56'})
            product_lxml = BeautifulSoup(web_page.content , "lxml")
        except:
            return 0
            
        ### Title
        try:
            Title = product_lxml.find("div" , attrs = {"class" : "shop-product-title"})
            product_name = Title.find("div" , attrs = {"class" : "sku-title"}).text
            product_dict.update({"product name" : product_name})
        except:
            pass
            
            
        ### SKUID
        try:
            skuid = product_lxml.select_one("div.sku.product-data").text.split(":")[1].strip(" ")
            product_dict.update({"skuid" : skuid})
        except:
            pass
        
        ### Installment
        try:
            install_fh = product_lxml.find("div" , attrs = {'class' : "total-cost-clarity-content__monthly-payment"}).text.split(".")[0]
            install_sh = product_lxml.find("div" , attrs = {'class' : "total-cost-clarity-content__finance-message"}).text
            install_combine = install_fh + " + " + install_sh
            product_dict.update({"installment" : install_combine})
        except:
            pass
        
        ### Return period
        try:
            return_period = product_lxml.find("div" , attrs = {'class' : "product-return-message__label product-return-message__label--pdp product-return-message__label--pdp-large"}).text
            product_dict.update({"return period" : return_period})
        except:
            pass

        ### Warrant
        try:
            warrant = product_lxml.find("div" , attrs = {'class' : "warranty-list"}).find_all("label")
            warrant = [deal.text for deal in warrant]
            product_dict.update({"warrant" : warrant})
        except:
            pass
        
        ### create directory
        try:
            os.mkdir(directory)
        except:
            pass
            
        ### save file
        with open(f"{directory}/{skuid}.html" , "w") as f:
            f.write(str(web_page))
            
        return product_dict
            
    ### execute whole process
    def execute_scrapping(self , directory):
        re_list = []
        advanced_info = []
        stopper = False
        counter = 1

        while stopper != True:
            page_n = self.get_search_pages(self.search_term , counter)
            page_source = page_n.find_all('div' , attrs = {'class' : 'shop-sku-list-item'})
            
            if len(page_source) == 0:
                stopper = True
            else:
                for product in page_source:
                    ### get individual product dictionary
                    product_info = self.get_product_info(product)
                    re_list.append(product_info)
                    
                    ### save individual product advanced info + webpage
                    advanced = self.save_product(product_info , directory)
                    advanced_info.append(advanced)
                    
                    time.sleep(0)
                        
                print(f"page {counter} done!")
                counter = counter + 1
            
            ### pause after request
            time.sleep(0)
        ### return dict of products
        return re_list , advanced_info



if __name__ == '__main__': 
    
    ### initiate object
    bb_init = Best_buy_scrap("TV")
    
    ### start scrapping
    dict_info , advanced_info = bb_init.execute_scrapping("Advanced_product_info")


    # Connect to MongoDB database
    client = MongoClient("mongodb://localhost:27017/")
    
    # Get the "bayc" collection
    db = client["best_buy_db"]

    # Drop the "bayc" collection if it exists
    if "best_buy_collection" in db.list_collection_names():
        db["best_buy_collection"].drop()

    # create a db 
    collection = db["best_buy_collection"]


    #pushing into collection
    collection.insert_many(dict_info)


    # Drop the "bayc" collection if it exists
    if "best_buy_product_level_collection" in db.list_collection_names():
        db["best_buy_product_level_collection"].drop()
        
    # create a db 
    collection = db["best_buy_product_level_collection"]


    #pushing into collection
    collection.insert_many(advanced_info)

