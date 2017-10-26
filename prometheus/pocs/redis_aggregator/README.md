# Implementation of a collector that read redis aggregator values and exposes then to prometheus

Python has a nice prometheus client : https://github.com/prometheus/client_python.
I think it would be possible to implement a [custom collector](https://github.com/prometheus/client_python#custom-collectors) on top of our redis aggregator. But, currently because of the way we aggregate the data in redis and have no way to know what type of data we get out of redis. It's impossible to choose the correct metric type to use in prometheus.
Some changes in the current aggregator are required to be able to continue on this POC.
