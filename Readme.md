# lob.li API - Link Objectively

A few years ago I created [lob.li](https://lob.li). It was designed in PHP 5 and was switched from a MySQL database to Redis realitively easily. The service is still functional today, running on PHP 7, but it's significantly outdated, and API features that were promised to come never did. This is an attempt to create a new version of the API, coded in Python 3 as that's the language I'm most comfortable with currently. 

## Requirements
* [Bottle](https://bottlepy.org/docs/dev/) - This handles the Routes and Jinja2 templates
* [CherryPy](https://cherrypy.org/) - This is responsible for handling the WGSI processes
* [Beaker](https://beaker.readthedocs.io/en/latest/configuration.html#) - Responsible for handling user sessions
