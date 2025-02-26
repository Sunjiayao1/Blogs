# Load Balancer

| compare  | Application Load Balancer(ALB)                                                                                                                                                     | Network Load Balancer (NLB)                                                                                                                                                                                                           | Classic Load Balancer (CLB) | Gateway Load Balancer (GLB) |
|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------|-----------------------------|
| layer    | layer 7                                                                                                                                                                            | layer 4                                                                                                                                                                                                                               | layer                       | layer 3                     |
| features | content-based routing (based on the request content)<br/>Advanced targeting: path-based and host-based routing<br/>Websokets and HTTP/2<br/>SSL Termination<br/>Health checks<br/> | High Performance and Low latency<br/> static IP<br/>preserve client IP<br/>TLS termination<br/>Zone Failover                                                                                                                          |                             |                             |
| use case | often used with ECS EKS<br/>Ideal for HTTP and HTTPS traffic                                                                                                                       | applications requiring fast transport of Layer 4 TCP/UDP traffic<br/>Appropriate for non-HTTP traffic or legacy platforms<br/>Commonly used for applications that need to handle volatile workloads or require consistent performance |                             |                             |

SSL termination, also known as SSL offloading, is the process of decrypting SSL/TLS (Secure Sockets Layer/Transport
Layer Security) encrypted traffic at a specific point in a network, such as at a load balancer or proxy server, rather
than at the destination server. This process offloads the CPU-intensive task of encrypting and decrypting traffic from
the application servers, potentially improving performance and simplifying certificate management.

～
[setup-vm-2024-Dec.md](..%2F..%2F..%2FDownloads%2Fsetup-vm-2024-Dec.md)
