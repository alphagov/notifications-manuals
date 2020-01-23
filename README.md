# Team manual

Our team manual lives in the Github wiki for this repository - https://github.com/alphagov/notifications-manuals/wiki

This repository was previously used to host documentation on Github Pages but grew out of date. We should do a purge of all the material, taking anything worth while into the new wiki based manual and then deleting everything else here.


## Old documentation

This documentation is generated using mkdocs. The output website files are stored in /docs directory and viewed hosted by Github Pages here: https://alphagov.github.io/notifications-manuals/

### mkdocs official documentations
http://www.mkdocs.org/

#### Installing mkdocs on your local machine for compiling the website
```pip install mkdocs```

#### Modify the documents
cd into src/docs, modify exising files using markdown (.md) or create a new .md file

#### Configuration
You can change the theme, structure of table of content in src/mkdocs.yml

#### Build the webpage

cd into the src directory, run ```mkdocs build``` to build the website

You can run ```mkdocs serve``` to test the built webpage on your local machine

#### Refreshing
github.io webpage may take a little while to be refreshed. However, make sure you have used `mkdocs build` to build the webpage before pushing to github

