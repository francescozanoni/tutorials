# Personal Composer repository

1. create your package, whose composer.json could be as follows:

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

2. put you package into a directory named package-1.0.0
3. compress the directory into file package-1.0.0.zip
4. upload package-1.0.0.zip file to a web server, e.g. at URL http://www.example.com/package-1.0.0.zip
5. create the repository main file packages.json, as follows:

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

To use it in your projects, include it into your projects' composer.json as follows:

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
