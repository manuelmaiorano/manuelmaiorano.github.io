---
title: "IMDB Scraper with Python"
date: 2022-7-10
categories: Python Tools
---

This project is a python implementation of a web scraper. The main reason why i did this stems from the fact that i had a list of movie titles in a text file that i added at different times; looking back at the list, it wasn't very clear why those titles were there or why i inserted them on the first place. The hope was that downloading other relevant information associated with the movie from IMDB i would have better information when picking a movie to watch. There are probably much better ways to do this but working on a this project was an interesting exercise that had its very own challenges. 

The python module of choice is beautifulsoup that allows you to parse an html file and recurse through its hierarchy to find the information you need. The downloading part is handled by the request module; an interesting improvement was the addition of a cache that helped greatly during developement, avoiding downloading every time the same page during testing.
To present the gathered information i came up with a very primitive way of generating automatically generating a html page with all the images and the information. Finally i wrapped everything up with a shell script that reads the file, downloads the new movies added, generates the html and opens the page on a browser.

This is the main python script:
`
    from imdb_scraper import IMDB_scraper, NoSearchResultException
    from html_generator import HTML_generator
    from item_cache import Item_cache
    import os

    def read_queries():
        queries = []
        with open('queries.txt') as file:
            for line in file:
                line = line.replace('\n', '').replace(':', '')
                queries.append(line)
        return queries

    def store_website(index, other_pages):
        try:
            os.makedirs(f'htmldoc/{HTML_generator.PAGES_DIR_NAME}')
        except FileExistsError:
            pass
        
        with open('htmldoc/index.html', 'w') as file:
            file.write(index)

        for page in other_pages:
            with open(f'htmldoc/{page.path}', 'w') as file:
                file.write(page.html_doc)

    if __name__ == '__main__':
        scraper = IMDB_scraper()
        html_generator = HTML_generator()
        queries = read_queries()

        cache = Item_cache('htmldoc')

        new_queries, old_queries = cache.get_new_queries(queries, delete_old=True)
        print(f'new: {len(new_queries)}')
        print(f'old: {len(old_queries)}')

        for query in new_queries:
            try:
                item, photos = scraper.scrape_from_query(query)
            except NoSearchResultException as exception:
                print(exception.args)
                continue
            cache.add(query, item, photos)
        
        for cached_item in cache.retrieved_items.values():
            html_generator.add_item(cached_item.item, cached_item.photo_paths)

        store_website(html_generator.get_main_page(), html_generator.pages)

        cache.save()
`