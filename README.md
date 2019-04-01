# Python - Lenses for Apache Kafka

Python library for managing [Lenses](http://www.landoop.com/kafka-lenses) REST and WS APIs.

[![build status](https://travis-ci.org/Landoop/lenses-python.svg?branch=v2.3)](https://travis-ci.org/Landoop/lenses-python)

# Documentation

See [Lenses Python documentation](https://docs.lenses.io/dev/python-lib/).

# Installation

You can install by using pipp

    pip3 install lenses_python


# Use Cases and Examples

* CI/CD and Automation
* Jupyter Notebooks
* Machine Learning

## Jupyter Example

<p align="center">
  <img src="https://pbs.twimg.com/media/DbeXsAZXcAAw8uy.jpg" width="400"/>
</p>

## Integration Tests

#### Requirements

| Automatically installed |
| ------ |
| tox |
| pytest |
| flake8 |

| Must be Present |
| ------ |
| Docker |
| Lenses Box |



**Integration tests will run Lenses-Box, which requires ~4G of memory.**

Run tests:

```
make LICENSE_KEY=<YOUR-LICENSE-KEY> docker
make test
```

This command will start a Lenses docker box and when it's ready the integration tests will run.

*Note*: You can find a dev license key [here](https://www.landoop.com/downloads/)

## License

The project is licensed under the Apache 2 license.
