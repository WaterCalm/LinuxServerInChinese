# gitbook-plugin-page-footer

## gitbook plugin url

[page-copyright](https://plugins.gitbook.com/plugin/page-copyright)


## Install
Add the below to your `book.json` file, then run `gitbook install` in book folder:
```json
{
    "plugins": ["page-copyright"]
}
```
## config

### book.json
```json
{
    "plugins" : ["page-copyright"],
    "pluginsConfig" : {
        "page-copyright": {
          "description": "modified at",
          "signature": "Skylor.min",
          "wisdom": "Designer, Frontend Developer & overall web enthusiast",
          "format": "YYYY-MM-dd hh:mm:ss",
          "copyright": "Copyright &#169; skylor",
          "timeColor": "#666",
          "copyrightColor": "#666",
          "utcOffset": "8",
          "style": "normal",
          "noPowered": false,
        }
    }
}
```
