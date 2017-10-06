# Personal Composer repository

Sometimes a simple static Composer repository is useful, unlinked from the official Packagist. An example: you could host on it all your company's packages, which are not worth or allowed to be made available on platforms such as GitHub.

1. create your package, whose ***composer.json*** could be as follows:

    ```json
    {
      "name": "my/package",
      "description": "My package",
      "version": "1.0.0",
      "type": "library",
      "authors": [
        {
          "name": "Great Developer",
          "email": "great.developer@example.com"
        }
      ]
    }
    ```

1. put you package into a directory named ***package-1.0.0***

1. compress the directory into file ***package-1.0.0.zip***

1. upload package-1.0.0.zip file to a web server, e.g. at URL http://www.example.com/package-1.0.0.zip

1. create the repository main file packages.json, as follows:

    ```json
    {
      "packages": {
        "my/package": {
          "1.0.0": {
            "name": "my/package",
            "version": "1.0.0",
            "dist": {
              "url": "http://www.example.com/package-1.0.0.zip",
              "type": "zip"
            }
          }
        }
      }
    }
    ```

To use the repository in your projects, include it into ***composer.json*** as follows:

```json
{
  "repositories": [
    {
      "url": "http://www.example.com",
      "type": "composer"
    }
  ],
  "require": {
    "my/package": "1.0.0"
  },
  "config": {
    "secure-http": false
  }
}
```

***tutorial-scoped assumption*** *: if you understand the risk, by adding ***secure-http: false*** you can use plain HTTP *