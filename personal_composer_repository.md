# Personal Composer repository

Sometimes a simple static Composer repository is useful, unlinked from the official [Packagist](https://packagist.org). An example: you could host on it all your company's packages, which are not worth or allowed to be made available on platforms such as [GitHub](https://github.com).

### Requirements

* a web server (reachable at URL http://www.example.com, in the following example).

### 0. Package creation and location
1. if not already available, create your package, whose **composer.json** (shortest version) could be as follows:

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

1. put your package into a directory named **package-1.0.0** and compress to file **package-1.0.0.zip**;

1. upload file package-1.0.0.zip to a web server, e.g. at URL **http://www.example.com/package-1.0.0.zip**;

### 1. Repository creation
1. create the repository main file **packages.json**, as follows:

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
    “url” item must of course match the package file location.

### Usage
To use the repository, together with the hosted package, include it into your projects’ **composer.json** (shortest version) as follows:

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

***tutorial-scoped assumption*** *: if you understand the risk, ""secure-http": false" allows you to use plain HTTP*

### References

* https://getcomposer.org/doc/05-repositories.md
* https://getcomposer.org/doc/06-config.md#secure-http