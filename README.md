# Performance Comparison Report: Gunicorn (WSGI), Uvicorn(ASGI), and Bjoern (WSGI)F

## Introduction

This report provides a detailed comparison of the performance of a simple Flask web API implemented using different server frameworks: Gunicorn (WSGI), Uvicorn (ASGI), and Bjoern (WSGI). The comparison is based on benchmarking results under moderate and high load conditions.

## Test Setup

### Initial Flask Application

    from flask import Flask, jsonify
    
    app = Flask(__name__)
    
    @app.route('/api/v2/<msisdn>', methods=['GET'])
    def get_status(msisdn):
        response = {
            "msisdn": msisdn,
            "status": "ok"
        }
        return jsonify(response)
    
    if __name__ == '__main__':
        app.run()
    

### Benchmarking Tool

* wrk: A modern HTTP benchmarking tool capable of generating significant load when testing web servers.

### Benchmark Configurations

* **Moderate Load**: 12 threads, 1000 connections, 60 seconds.
* **High Load**: 24 threads, 8000 connections, 120 seconds.

### Servers Tested

1. **Gunicorn (WSGI)**: A pre-fork worker model server for WSGI applications.
2. **Uvicorn (ASGI)**: An ASGI server for asynchronous web frameworks.
3. **Bjoern (WSGI)**: A high-performance, low-latency WSGI server written in C.

## Results

### Gunicorn (WSGI)

#### Modderate Load

    Running 1m test @ http://127.0.0.1:8000/api/v2/1234567890
      12 threads and 1000 connections
      Thread Stats   Avg      Stdev     Max   +/- Stdev
        Latency   136.66ms    8.41ms 195.02ms   87.20%
        Req/Sec   609.60    182.53   838.00     56.29%
      437200 requests in 1.00m, 76.30MB read
    Requests/sec:   7275.16
    Transfer/sec:      1.27MB

    Requests/sec:   7275.16
    Latency:        136.66ms (Avg)
    Transfer/sec:   1.27MB
    Errors:         None

#### High Load

    Running 2m test @ http://127.0.0.1:8000/api/v2/1234567890
      24 threads and 8000 connections
      Thread Stats   Avg      Stdev     Max   +/- Stdev
        Latency   606.14ms  252.39ms   1.99s    84.67%
        Req/Sec   153.12    197.78     1.60k    84.79%
      290107 requests in 2.00m, 50.63MB read
      Socket errors: connect 0, read 4530, write 0, timeout 15271
    Requests/sec:   2415.65
    Transfer/sec:    431.70KB

    Requests/sec:   2415.65
    Latency:        606.14ms (Avg)
    Transfer/sec:   431.70KB
    Errors:         4530 read errors, 15271 timeout errors

### Uvicorn (ASGI)

#### Moderate Load

    Running 1m test @ http://127.0.0.1:8000/api/v2/1234567890
      12 threads and 1000 connections
      Thread Stats   Avg      Stdev     Max   +/- Stdev
        Latency   242.11ms   59.68ms   1.01s    55.02%
        Req/Sec   341.83     82.57   808.00     72.96%
      244658 requests in 1.00m, 38.99MB read
    Requests/sec:   4074.00
    Transfer/sec:    664.86KB

    Requests/sec:   4074.00
    Latency:        242.11ms (Avg)
    Transfer/sec:   664.86KB
    Errors:         None

#### High Load

    Running 2m test @ http://127.0.0.1:8000/api/v2/1234567890
      24 threads and 8000 connections
      Thread Stats   Avg      Stdev     Max   +/- Stdev
        Latency     1.78s   214.92ms   2.00s    95.06%
        Req/Sec   165.78    145.62     1.75k    72.29%
      425175 requests in 2.00m, 67.74MB read
      Socket errors: connect 0, read 469, write 0, timeout 181492
    Requests/sec:   3541.29
    Transfer/sec:    577.73KB

    Requests/sec:   3541.29
    Latency:        1.78s (Avg)
    Transfer/sec:   577.73KB
    Errors:         469 read errors, 181492 timeout errors

### Bjoern (WSGI)

#### Moderate Load

    Running 1m test @ http://127.0.0.1:8000/api/v2/1234567890
      12 threads and 1000 connections
      Thread Stats   Avg      Stdev     Max   +/- Stdev
        Latency   110.29ms   43.09ms   2.00s    86.32%
        Req/Sec   691.96    308.91     3.32k    83.72%
      472744 requests in 1.00m, 59.96MB read
      Socket errors: connect 0, read 0, write 0, timeout 616
    Requests/sec:   7866.16
    Transfer/sec:      1.00MB

    Requests/sec:   7866.16
    Latency:        110.29ms (Avg)
    Transfer/sec:   1.00MB
    Errors:         616 timeout errors

#### High Load

    Running 2m test @ http://127.0.0.1:8000/api/v2/1234567890
      24 threads and 8000 connections
      Thread Stats   Avg      Stdev     Max   +/- Stdev
        Latency   107.00ms    8.59ms   1.98s    93.29%
        Req/Sec     1.15k   692.01     2.54k    52.34%
      1101724 requests in 2.00m, 139.74MB read
      Socket errors: connect 0, read 24, write 0, timeout 15
    Requests/sec:   9174.24
    Transfer/sec:      1.16MB

    Requests/sec:   9174.24
    Latency:        107.00ms (Avg)
    Transfer/sec:   1.16MB
    Errors:         24 read errors, 15 timeout errors

## Analysis

### Gunicorn (WSGI)

#### Stengths

* **Mature and Well-Supported**: Gunicorn is a well-established WSGI server with extensive documentation and a large community, making it a reliable choice for many applications.
* **Effective Worker Model**: Gunicorn uses a pre-fork worker model, where the master process forks multiple worker processes to handle incoming requests. Each worker handles one request at a time, which isolates requests and prevents one request from blocking others. This model leverages multiple CPU cores effectively, allowing for concurrent processing.
* **Performance Under Moderate Load**: Gunicorn performs well under moderate load with low latency and no errors, making it suitable for applications with predictable traffic patterns.

#### Weaknesses

* **Increased Latency and Errors Under High Load**: As the load increases, Gunicorn experiences significant latency increases and a higher rate of errors. This is due to its synchronous nature and the limits of the pre-fork worker model.
* **Synchronous Nature**: Each worker handles one request at a time, which can cause bottlenecks under high concurrency. This leads to increased latency and potential timeouts when the number of concurrent requests exceeds the number of available workers.

#### **Reasons for Behavior**

* **Pre-Fork Worker Model**: Each worker is a separate process that handles one request at a time. While this is effective for isolating requests, it can lead to high memory usage and CPU limits under heavy load.
* **Synchronous Processing**: Gunicorn's synchronous request handling means that each worker can only process one request at a time, causing increased latency and errors when overwhelmed by high concurrency.

### Uvicorn (ASGI)

#### Srengths

* **Asynchronous Programming**: Uvicorn is designed for asynchronous frameworks, allowing it to handle many concurrent connections efficiently by using non-blocking I/O operations.
* **Lightweight**: Uvicorn has low overhead, making it suitable for applications that need to handle a large number of simultaneous connections.

#### Weaknesses

* **High Latency and Errors Under Load**: The benchmark results showed high latency and significant errors under heavy load, which is unexpected for an ASGI server.
* **Inefficiencies with Synchronous Applications**: Running a synchronous Flask application on an asynchronous server like Uvicorn can introduce overhead and inefficiencies, as the server cannot fully utilize its asynchronous capabilities.

#### Reasons for Behavior

* **I/O-Bound vs. CPU-Bound Tasks**: Asynchronous servers excel at I/O-bound tasks, but Flask's synchronous nature means it can't take full advantage of Uvicorn's non-blocking I/O operations.
* **Suboptimal Use of ASGI**: Flask is not designed for asynchronous execution, leading to suboptimal performance when run on an ASGI server like Uvicorn.

### Bjoern(WSGI)

#### Srengths

* **High Performance and Low Latency**: Bjoern is optimized for performance, handling more requests per second and maintaining low latency.
* **Stability**: Bjoern maintains stability and low error rates even under high load, indicating efficient request handling.

#### Weaknesses

* **Limited Features**: Bjoern lacks some of the advanced features and flexibility found in more comprehensive servers like Gunicorn.
* **Single-Threaded Nature**: Bjoern runs as a single-threaded server, which might limit scalability without external process management.

#### Reasons for Behavior

* **Optimized C Implementation**: Bjoern is written in C, allowing it to handle HTTP requests with minimal overhead.
* **Focus on Performance**: Bjoern is designed to be fast and low-overhead, making it suitable for high-throughput scenarios.

## Further Optimizations

### Gunicorn(WSGI)

* **Optimizing with Nginx**: we can use Nginx To handle heavier loads more effectively,Nginx will act as a reverse proxy in front of Gunicorn. Nginx can handle a large number of concurrent connections and distribute the load among multiple Gunicorn worker processes.

### Uvicorn(ASGI)

* **Optimizing with an Asynchronous Framework**: If we want to fully leverage the benefits of asynchronous servers, we can consider switching to an asynchronous framework such as FastAPI. This change can significantly improve performance metrics under high concurrency.

### Bjoern(WSGI)

* **Optimizing with Nginx**: we can use Nginx with Bjoern to handle heavy loads more effectively. Nginx can act as a reverse proxy, distributing incoming requests to multiple instances of Bjoern, thereby enhancing the overall performance and scalability of our application.
* **Run Multiple Instances of Bjoern**: Running multiple instances of Bjoern on different ports ensures that our application can utilize multiple CPU cores effectively, enhancing performance and reliability, we can use a process manager like `supervisor` to do this.

## Conclusion

* For a synchronous Flask application, Bjoern provides the best performance and stability under both moderate and high load conditions, Gunicorn is a solid alternative, though it may require tuning for high load scenarios. Uvicorn, while excellent for asynchronous applications, is not ideal for synchronous frameworks like Flask.
  
* Using Nginx as a reverse proxy for Bjoern can significantly improve the scalability and performance of our Flask application, particularly under heavy load. Thus we can ensure efficient load balancing and better utilization of system resources, providing a robust solution for handling high concurrency.
